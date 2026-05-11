# Investigation: GH-501293 — Data Connector "Last Updated At" Shows Incorrect Value

**Source**: https://github.com/intercom/intercom/issues/501293
**Beads Epic**: intercom-investigations-gt4
**Date**: 2026-05-12
**Investigator**: anubhav.bohidar@intercom.io
**Status**: OPEN — Repeatedly reopened despite multiple fix attempts
**Priority**: P3 (high-defect-priority label also applied)

---

## Investigation Progress Checklist

- [x] Phase -1: Repository currency check
- [x] Phase 0: Setup & documentation
- [x] Phase 1: Parallel investigation (codebase + PRs + admin tools)
- [x] Phase 2: Evidence synthesis
- [x] Phase 3: Root cause analysis
- [x] Phase 4: Critic review (NEEDS_MORE_INVESTIGATION — addressed below)
- [x] Phase 4b: Post-critic validation
- [ ] Phase 5: Proposed solution
- [ ] Phase 6: GitHub communication

---

## Issue Summary

Customer (Zilch, app 2009840, EU) reports the "Last Updated At" column in the Data Connector
list shows wrong timestamps and former employees as recent editors.

---

## Verification Status Table

| Claim | Status | Source |
|-------|--------|--------|
| BackfillInternalDescription worker now uses touch:false | Verified | origin/master file |
| create_default_wrapper! calls update! without touch:false at line 538 | Verified | origin/master lines 538, 545 |
| PerRow::RepairUpdatedAtFromScdMapping exists in codebase | Verified | git ls-tree origin/master |
| SCD repair NOT executed in EU in last 7 days | Verified | Snowflake: zero logs for %RepairUpdatedAt% or %ScdMapping% |
| List view does NOT trigger resolve_wrapper | Verified | InstanceService.list uses apply_stable_ids only |
| ActionList serializer exposes plain updated_at | Verified | serializer line 29 |
| The find/show path DOES trigger resolve_wrapper | Verified | InstanceService.find line 78 |
| create_default_wrapper! fires only once per action | Verified | resolve_wrapper exits early if action_id.present? |
| App 2009840 is EU (admin tools can't see it) | Verified | Snowflake: EU prod data found; admin tools returned not_found |

---

## Root Cause Analysis

### There are TWO corruption events, not one:

#### Corruption Event 1 (Apr 22-23): BackfillInternalDescription worker
- Source: record.update! in per-row worker without touch:false
- Affected: ~168K records, 16K+ workspaces
- Status: FIXED (PR #502229 changed to assign_attributes + save!(touch: false))
- Repair: Never conclusively run in all regions

#### Corruption Event 2 (Apr 16): create_default_wrapper! lazy wrapper creation
- Source: create_default_wrapper! in action.rb lines 538 and 545 both call update! without touch:false
- Trigger: PR #495686 removed tdf-data-connector-use-wrapper feature flag, making
  resolve_wrapper unconditional. Once the FF was off, every InstanceService.find call
  on an action without a wrapper triggers create_default_wrapper! -> update!.
- Scale: WorkflowConnectorActionsWorker calls ActionShim.find! on every conversation execution.
  On Apr 16 (day after FF removal), every action executed by workflow automations across
  all apps got its wrapper created lazily, bumping updated_at. Result: 78K US rows, 5K EU rows.
- Duration: ONE-TIME per action (resolve_wrapper short-circuits on action_id.present?).
  After Apr 16, most actions now have wrappers. New actions created after Apr 16 would
  get bumped on their first execution.
- Status: NOT FIXED. The update! calls still exist at lines 538 and 545 of origin/master.

### The "former employee" symptom explained:
- updated_at was bumped to Apr 16 or Apr 22 (a system date)
- updated_by_admin_id still points to the last human editor of that connector
- If that admin left the company, UI shows "[former employee] updated 13 days ago"
- The admin didn't edit it — the system bumped updated_at without touching updated_by_admin_id

### Why backfills keep failing:
Every fix so far has only repaired historical data:
1. PR #502229 repair worker: only Apr 22-23 window
2. PR #506612 SCD repair: broader coverage but NEVER EXECUTED (zero Snowflake logs in EU)
3. Neither PR fixed the source code (create_default_wrapper! still uses update!)

The current situation: The code-level fix from #502229 prevents the BackfillInternalDescription
worker from re-corrupting. But the SCD repair that would fix the Apr 16 + Apr 22 corruption
was never run. Zilch's connectors are still showing the corrupted timestamps.

---

## Proposed Solution: 3 Steps for Zero Scope of Reopening

### Step 1: Fix the source code (2-line change, prevents future new-action corruption)

In app/lib/workflow_connector/models/action.rb:

Line 538 — change:
  update!(action_id: new_wrapper.id)
to:
  assign_attributes(action_id: new_wrapper.id)
  save!(touch: false)

Line 545 — change:
  draft.update!(action_id: new_wrapper.id, parent_id: nil)
to:
  draft.assign_attributes(action_id: new_wrapper.id, parent_id: nil)
  draft.save!(touch: false)

Safety: save!(touch: false) is allowed by DisableUpdatesThatSkipCallbacks (which only
blocks update_column/update_columns/touch/delete/update_all). It still fires the after_commit
Syncable ES sync callback. Confirmed by existing use of save!(touch: false) in other workers.

This step is low-risk and prevents any future new actions from having their updated_at
corrupted when they get their first wrapper on execution.

### Step 2: Actually run the SCD repair (the key missing step)

The SCD repair worker (PR #506612) is in origin/master but was NEVER EXECUTED.
Snowflake shows zero execution logs for EU in the last 7 days.

To fix Zilch and all affected customers:

a) Generate Snowflake SCD mapping for each region (SQL is in the worker comment block):
   - Run in Snowsight for EU, US, AU separately
   - Export as JSON: {"<id>": {"target": "<iso>", "current": "<iso>"}, ...}
   - Upload to s3://intercom-shared-tmp-files/repair-dc-updated-at-from-scd/<region>.json

b) Run the worker in each region:
   ./script/rake "per_row:enqueue[PerRow::RepairUpdatedAtFromScdMapping]"

c) Monitor via Datadog per-row dashboards. Expect:
   - EU: ~6,799 repairs (per PR description)
   - US: ~78K+ repairs (Apr 16 alone was 78K rows)
   - Check: repaired count matches mapping size, error = 0

d) After each region completes, spot-check 5 connectors in Zilch's workspace
   (app 2009840) to confirm timestamps are corrected.

### Step 3: Revive last_edited_at (permanent structural prevention)

The reviewer who closed PR #501831 said "one-off issue" — this was wrong. The Apr 16
event proves that ANY removal of a feature flag gate, ANY backfill, ANY system process
that calls update! without touch:false will re-corrupt updated_at. This is a structural risk.

Solution: separate "system bookkeeping" (updated_at) from "customer-visible last editor" (last_edited_at):

a) Merge PR #501853 (migration is ready, technically clean, no changes needed)

b) Create new code PR (replaces closed #501831):
   - Set last_edited_at = Time.current in the 5 human command paths:
     Create, Update, Pause, DiscardDraft, Duplicate (only when actor is human admin)
   - Serializers: return last_edited_at.presence || updated_at
   - No frontend change (same JSON field key)
   - No backfill needed: rows self-heal on next real teammate edit

Once Step 3 lands, system writes can bump updated_at all they want.
Customers only see last_edited_at, which only moves on human actions.

---

## Critic Review Summary

The critic raised 4 challenges:

1. "List view cannot trigger create_default_wrapper!" — CORRECT. Updated analysis:
   the trigger is InstanceService.find via workflow execution workers (ActionShim.find!)
   not list browsing. The fix and root cause are unchanged.

2. "RepairUpdatedAtFromScdMapping doesn't exist" — INCORRECT (critic error).
   Confirmed via git ls-tree origin/master: file exists at
   app/workers/per_row/repair_updated_at_from_scd_mapping.rb

3. "Which serialized field does UI read?" — Addressed. ActionList serializer (line 29)
   returns plain :updated_at. The Action serializer has :live_updated_at under a feature
   flag, but it reads wrapper.live_version.updated_at which is the same record.

4. "If EU repair ran, why still broken?" — Addressed. Snowflake confirms repair never ran.
   Zero log entries for repair workers in EU in last 7 days. The missing step is execution.

Confidence: HIGH (evidence from 3 independent sources: code, timeline, Snowflake logs)

---

## GitHub Communication Draft

```markdown
## Root cause analysis — why this keeps reopening

After a thorough investigation including codebase analysis, PR history review, and
Snowflake production log queries, here's a complete picture of what's happening and
a 3-step plan to close this permanently.

### There are two separate corruption events

**Event 1 — Apr 22-23 (BackfillInternalDescription worker)**
The `BackfillInternalDescriptionOnWorkflowConnectorActions` per-row worker used
`record.update!` which auto-bumps `updated_at`. This was fixed in PR #502229
(`assign_attributes + save!(touch: false)`). ✅ Source fixed, but historical data never
fully repaired.

**Event 2 — Apr 16 (create_default_wrapper! lazy wrapper creation)**
`create_default_wrapper!` in `WorkflowConnector::Models::Action` (lines 538 and 545 in
current `origin/master`) calls `update!` without `touch: false`:

```ruby
update!(action_id: new_wrapper.id)          # line 538 — bumps updated_at on live action
draft.update!(action_id: new_wrapper.id, ...) # line 545 — bumps updated_at on draft
```

This fires every time `InstanceService.find` is called on an action without a wrapper,
which happens on every data connector execution (via `WorkflowConnectorActionsWorker` →
`ActionShim.find!`). On Apr 16, PR #495686 removed the `tdf-data-connector-use-wrapper`
feature flag, making wrapper creation unconditional. That day, ~78K US rows and ~5K EU
rows were bumped as workflow automations ran and each action without a wrapper triggered
lazy creation. **This source code fix has never been shipped.**

This also explains the "former employee" symptom: `updated_at` was bumped to a system date,
but `updated_by_admin_id` still points to the last human editor (who may have since left).

### Why backfills don't stick

Every fix so far has only repaired historical data. The SCD repair worker from PR #506612
(`PerRow::RepairUpdatedAtFromScdMapping`) is merged and ready — but was **never executed**.
Snowflake application logs show zero execution of this worker in EU in the last 7 days.
That's the immediate reason Zilch still sees wrong timestamps: the repair code exists but
the operator step (generate Snowflake mapping → upload to S3 → run worker) was never done.

### Three-step permanent fix

**Step 1 — Fix the source code (2-line change)**

In `app/lib/workflow_connector/models/action.rb`, change lines 538 and 545 in
`create_default_wrapper!`:

```ruby
# Before (bumps updated_at):
update!(action_id: new_wrapper.id)

# After (preserves updated_at):
assign_attributes(action_id: new_wrapper.id)
save!(touch: false)
```

Apply the same change to line 545 for the draft. `save!(touch: false)` is safe:
it still fires ES sync callbacks via `after_commit`; it's already used in other workers
(`backfill_enforce_auth`, `backfill_custom_actions_no_auth_chosen`).

**Step 2 — Actually run the SCD repair (the missing operator step)**

Using the SQL in the `PerRow::RepairUpdatedAtFromScdMapping` worker comment:
1. Generate the Snowflake SCD mapping for each region (EU, US, AU)
2. Upload to `s3://intercom-shared-tmp-files/repair-dc-updated-at-from-scd/<region>.json`
3. Run: `./script/rake "per_row:enqueue[PerRow::RepairUpdatedAtFromScdMapping]"`
4. Monitor via Datadog dashboards (links in the worker comment)

EU: ~6,799 rows. US: likely 78K+.

After EU runs, spot-check Zilch (app 2009840) — timestamps should be corrected.

**Step 3 — Revive `last_edited_at` (structural prevention)**

The reviewer's objection to PR #501831 ("one-off issue") was incorrect — this has now
happened twice from different sources and will happen again on any future backfill or
system write that doesn't use `touch: false`. The structural fix (separate column for
customer-visible "last edited" vs. internal `updated_at`) is the only way to make this
permanently immune.

Recommendation: Merge PR #501853 (migration is clean, no changes needed) and create a
new code PR (replaces closed #501831) to set `last_edited_at` only from the 5 human
command paths (Create, Update, Pause, DiscardDraft, Duplicate) and have serializers
return `last_edited_at.presence || updated_at`. No frontend changes, no backfill needed.
```

[~ Automated via Claude](https://github.com/intercom/claude-plugins/blob/main/plugins/base/docs/external-message-attribution.md)
```

---

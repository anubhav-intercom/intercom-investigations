# Ticket #215473973691221 — CDA List Validation Regression breaking POST /companies

**Beads epic**: `intercom-investigations-4tt`
**Investigation date**: 2026-04-20
**Investigator**: Anubhav Bohidar (9751654, anubhav.bohidar@intercom.io)
**Status**: In progress (Phase 2 — Documentation Setup complete)

---

## Metadata

| Field | Value |
|---|---|
| Ticket ID | 215473973691221 |
| Intercomrades URL | https://app.intercom.com/a/apps/tx2p130c/conversations/215473973691221 |
| Intercomrades app | `tx2p130c` (app_id 6) |
| Ticket type | BO: (CS + Sales) Product Questions (ID 46) |
| Ticket category | **Back-Office ticket** (response target: support agent, not end customer) |
| Ticket state | open / "Passed to R&D" |
| Created | 2026-04-18 17:03 UTC |
| Reporter | Radoslav A (radoslav.andonov@intercom.io, admin 7040065, CS: API & Integrations Tech [SBR]) |
| Triaged by (On-Call CS) | Ebenezer O.L (ebenezer.olalaleye@intercom.io, admin 6611242) |
| Current team assignee | **[R&D] Team Integration Capabilities** (misrouted — see Phase 1c) |
| Urgency (per reporter) | Critical (Customer Extremely Upset/Frustrated) |
| Region | EMEA |
| Linked parent conversation | `215473953740748` (Email channel, same contact "Pink Anchor") |

---

## Customer Issue (verbatim from ticket description)

> **What is the question you're trying to solve?**
>
> RECORDING: https://d.pr/v/jiULw2
>
> **What we know** — Zero occurrences of this error prior to 15:00 on the 16th. Couldn't find anything in Snowflake for 9th-15th April. Then at 15:52 — first error hits in deploy `a9d0694ab1` in EU followed by around 5k/hour in EU and around 40k/hour in companies controller for over 60 workspaces...!
>
> **Description of the issue**
>
> A deployment on 2026-04-16 around 15:52 UTC introduced strict server-side validation of list-type Custom Data Attribute (CDA) values against the configured allowed values list. Prior to this change, the Intercom API accepted any value for list-type CDAs, even values not explicitly configured in the allowed options. The official Intercom documentation previously stated: "It's possible to set the value of a list attribute to a value that you haven't manually configured for the list by using our REST API."
>
> Fairly certain this is related — https://github.com/intercom/intercom/commit/a9d0694ab1be2bfcdc47a576ffdaa18aa4200e4d
>
> Can you confirm if this is expected or if we need to declare an incident/P1? Kibana US: [query link for `message:"not in the list of allowed values" and controller:"Api::V3::CompaniesController"`]

**Affected customer identified in CS note**: ClickSend, workspace `nqkkwhx0`. Sample failing payload (POST /companies):

```json
{
  "name": "Test Company",
  "company_id": "3333",
  "remote_created_at": "2026-04-17T05:20:20.000000Z",
  "monthly_spend": 0,
  "custom_attributes": {
    "spend_over_12_months_c": null,
    "billing_account_type_c": "prepaid",
    "account_balance_c": 3,
    "last_sent_message_at_c": "",
    "admin_dashboard_url_c": "https://test.url.com/#/users?user_id=123",
    "clicksend_user_id_c": 123,
    "account_status_c": "ACTIVE",
    "country_c": "PH",
    "paid_status_c": 0,
    "email_c": "test@gmail.com",
    "phone_c": "+63123456789",
    "contact_name_c": "Test contact"
  }
}
```

---

## Initial Hypothesis (to be verified)

A server-side validator was added to list-type Custom Data Attributes on POST /companies (and possibly /contacts). The validator rejects any value not present in the configured `allowed_values`. This changes a long-standing public API contract (previously documented as accepting arbitrary values).

**Preliminary findings (from two pre-restart subagents, to be re-verified in Phase 3 + critic review):**

1. **The commit SHA cited by Rad (`a9d0694ab1`) appears to be the wrong one.** That commit is a QA-reviews feature change. The actual regression commit appears to be **`ee1ae1ca85a8`** — "Validate list CDA values on contact write (#497043)" by Marc Meillac, deployed ~13 min before first customer errors at 15:52 UTC. **Confidence: Medium (pending Phase 3 codebase re-verification + Buildkite deploy timeline correlation).**
2. **No feature flag** was observed guarding the new validator — runs for 100% of apps with any list-type CDA. **Confidence: Medium (pending re-grep verification).**
3. **Team misrouting confirmed.** Validator file has `RESPONSIBLE_TEAM = "team-data-foundations"` inline; CompaniesController has `RESPONSIBLE_TEAM = "team-conversations"` and is owned by `@intercom/team-conversations` per CODEOWNERS line 84. Ticket assigned to Team Integration Capabilities — neither owner. **Confidence: High (verified in code during Phase 1c).**
4. **No incident declared** as of 2026-04-20 morning per Glean/Slack search. Intercom's own billing-service is hitting the error internally. **Confidence: Medium (pending Phase 3 Datadog/incident.io re-check).**

---

## Phase 1c — Team Routing Verification

| File | Team per code |
|---|---|
| `app/lib/api/v3/validators/custom_data_attributes_validator.rb` | `team-data-foundations` (inline `RESPONSIBLE_TEAM` constant, line 8) |
| `app/controllers/api/v3/companies_controller.rb` | `team-conversations` (inline constant line 5 + CODEOWNERS line 84) |

**Current ticket assignee**: [R&D] Team Integration Capabilities — **not a code owner of either file**.

**Recommendation**: Reassign to **Team Conversations** (owns the controller and appears to have inherited ownership of the validator after a team rename). Team Integration Capabilities has no clear ownership of the regression surface.

---

## Phase 3 — Parallel Investigation Findings

### A. Intercom API / Conversation Context
*Status: pending*

### B. Codebase Analysis
*Status: pending re-verification*

### C. Glean Knowledge Base
*Status: pending re-verification*

### D. Observability (Honeycomb + Snowflake + ES)
*Status: pending*

### E. Sentry
*Status: pending*

### F. Buildkite / Deploy Timeline
*Status: pending*

### G. Datadog
*Status: pending*

---

## Phase 4 — Root Cause Analysis
*Status: pending Phase 3 completion*

---

## Phase 4b — Critic Review (MANDATORY)
*Status: pending Phase 4 completion*

---

## Phase 5 — Customer Response Draft (for Radoslav A, TSS)
*Status: pending critic approval*

---

## Phase 7 — Resolution
*Status: pending*

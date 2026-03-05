# Investigation Repository

This repository stores **investigation artifacts only** for Intercom GitHub issues and support tickets.

**Skills Source**: All investigation skills are provided by the `intercom-investigations-claude-plugin` (marketplace plugin)
**Artifacts Storage**: This repository (investigation results, screenshots, evidence)

## Structure

```
├── .beads/              # Beads task tracking database
├── issues/              # GitHub issue investigations
│   ├── intercom/       # intercom/intercom repo issues
│   └── embercom/       # intercom/embercom repo issues
├── tickets/             # Intercom support ticket investigations
├── plans/               # Backlog analysis reports
└── screenshots/         # Evidence and visual artifacts
```

## Skills Available

**Investigation Skills** (from intercom-investigations-claude-plugin):
- `/gh-triage` - GitHub issue investigation (~25-30 min)
- `/ticket-triage` - Support ticket investigation (~20-25 min)

**Backlog Management Skills**:
- `/stale-issue-purge` - Identify stale issues (~15-20 min)
- `/issue-closer` - Bulk close issues (~10-15 min)
- `/low-hanging-fruit` - Find quick wins (~20-30 min)

## Usage

```bash
# Investigate GitHub issue
/gh-triage 470332

# Investigate support ticket
/ticket-triage 215473124483717

# Backlog management
/stale-issue-purge
/low-hanging-fruit
/issue-closer
```

## Documentation

See: [Intercom Investigations Plugin](https://coda.io/d/Engineering-Wiki_duvlW1VbQXK/Intercom-Investigations-Plugin_suZBJfJf)

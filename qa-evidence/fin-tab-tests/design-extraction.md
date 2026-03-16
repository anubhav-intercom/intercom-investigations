# Figma Design Extraction: Fin Tab

**Source**: https://www.figma.com/design/8Ve4zPovP5DaB5dvb7PWOH/Data-Connectors-Improvements?node-id=2001-21910
**Extracted**: 2026-03-15
**Section**: Fin (node 2001:21910)

## Design States

| # | State Name | Figma Frame | Description |
|---|-----------|-------------|-------------|
| 1 | Default (Toggle ON) | Data connectors 115 (2400:56208) | Full form: Directly toggle + Audience + Prompt + Optimize button |
| 2 | Toggle OFF | Data connectors 117 (2400:64940) | Directly toggle + Contextual-alert disabled banner |
| 3 | Optimize Loading | Data connectors 121 (2400:68214) | Prompt area with "Thinking..." + Stop button |
| 4 | Optimize Result | Data connectors 120 (2400:67789) | AI suggestion with reasoning, Apply + Dismiss buttons |
| 5 | Legacy Audience Dropdown | Data connectors 118 (2400:66199) | Dropdown open with 3 row items |
| 6 | Tooltip | Instance 2392:55373 | Directly toggle info tooltip (300x404) |

## Component Extraction Table

| # | Figma Element | Surge Component | Props | Tokens | Overrides | Notes |
|---|--------------|-----------------|-------|--------|-----------|-------|
| 1 | Customer auth frame | Card | size="large" | — | NO | Container for entire Fin tab content, 800px max-width |
| 2 | "How should Fin use this connector?" | h3 | className="text-h3" | — | NO | Card heading |
| 3 | "Directly" label | span | className="text-base" | — | NO | 96px width label column |
| 4 | Info icon button | Tooltip.Trigger > button > InfoIcon | — | — | NO | Adjacent to "Directly" label |
| 5 | Tooltip content | Tooltip.Content | — | — | NO | Rich: enabled/disabled descriptions with bullet lists |
| 6 | Switch toggle | Switch | checked={boolean} | — | NO | data-testid="fin-directly-toggle" |
| 7 | Toggle description | p | className="text-base" | — | NO | "Fin will directly trigger this connector based on your prompt" |
| 8 | "Audience" label | label | className="text-base" | — | NO | 96px width label column, conditional on toggle ON |
| 9 | Audience button | Button (DropdownMenu.Trigger) | variant="secondary" | — | NO | data-testid="audience-trigger", MultiplePeopleIcon + "Everyone" + Chevron |
| 10 | Audience dropdown | DropdownMenu.Content | — | — | NO | Single CheckboxItem "Everyone" (checked) |
| 11 | "Prompt" label | label | className="text-base" | — | NO | 96px width label column, conditional on toggle ON |
| 12 | Textarea | Textarea | maxLength=30000 | — | NO | data-testid="fin-prompt-textarea", auto-expand, resize-none |
| 13 | Help text | p | className="text-support text-muted" | — | NO | "Describe when Fin should trigger..." + "See best practices" link |
| 14 | Best practices link | a | className="text-link", target="_blank" | — | NO | External: intercom.com/help/en/articles/9916183 |
| 15 | Optimize button (idle) | Button | variant="secondary" | — | NO | data-testid="ai-optimize-button", AiStarsIcon + "Optimize", disabled when empty |
| 16 | Loading indicator | span + Button | — | — | NO | "Thinking..." text + "Stop" button (variant="secondary", size="small") |
| 17 | AI suggestion panel | div | border, rounded-medium | neutral-border | YES | Contains reasoning + suggestion text + action buttons |
| 18 | AiStarsIcon + "AI suggestion" | span | className="text-caption-bold" | — | NO | Header of suggestion panel |
| 19 | Reasoning text | p | className="text-caption text-muted" | — | NO | AI-generated reasoning |
| 20 | Suggestion text box | div > p | bg-base-module, rounded-medium | base-module | YES | Contains the suggested prompt text |
| 21 | Apply button | Button | variant="primary", size="small" | — | NO | data-testid="ai-optimize-apply" |
| 22 | Dismiss button | Button | variant="secondary", size="small" | — | NO | data-testid="ai-optimize-dismiss" |
| 23 | Disabled banner container | div | border, rounded-medium, bg-base-module | neutral-border, base-module | YES | Shown when toggle OFF |
| 24 | Alert icon square | ContentSquare | size="small", variant="notice" | — | NO | Contains TinyAlertIcon |
| 25 | Banner heading | span | className="text-support-bold" | — | NO | "Fin will not run this connector directly." |
| 26 | Banner description | span | className="text-support text-muted" | — | NO | "It must be added to a Workflow..." |
| 27 | "Using Data connectors" link | Button (as a) | variant="tertiary", size="small" | — | NO | KnowledgeIcon + external help link |

## Conditional Rendering Logic

| Condition | Visible Elements | Hidden Elements |
|-----------|-----------------|-----------------|
| Toggle ON (usage = 'fin' or 'fin_voice') | Rows 8-22 (Audience + Prompt + AI Optimize) | Row 23-27 (Disabled banner) |
| Toggle OFF (usage = 'workflow_and_inbox') | Rows 23-27 (Disabled banner) | Rows 8-22 (Audience + Prompt) |
| AI Optimize idle | Row 15 (Optimize button) | Rows 16-22 |
| AI Optimize loading | Row 16 (Thinking + Stop) | Rows 15, 17-22 |
| AI Optimize result | Rows 17-22 (Suggestion panel) | Rows 15-16 |
| Prompt empty | Row 15 disabled | — |

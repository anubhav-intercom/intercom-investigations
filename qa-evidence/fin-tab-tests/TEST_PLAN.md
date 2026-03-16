# Test Plan: Fin Tab QA

**Area**: fin-tab
**Date**: 2026-03-15
**Components**: 7 (fin-tab.tsx, directly-toggle-row.tsx, audience-row.tsx, prompt-section.tsx, ai-optimize-button.tsx, disabled-banner.tsx, config-missing-state.tsx)
**Figma**: https://www.figma.com/design/8Ve4zPovP5DaB5dvb7PWOH/Data-Connectors-Improvements?node-id=2001-21910

---

## Summary

| Group | Code | T1 | T2 | T3 | Total |
|-------|------|----|----|----| ------|
| Toggle | TG | 5 | 4 | 0 | 9 |
| Prompt | PR | 5 | 3 | 1 | 9 |
| AI Optimize | AI | 3 | 2 | 5 | 10 |
| Audience | AU | 3 | 2 | 0 | 5 |
| Disabled Banner | DB | 4 | 2 | 0 | 6 |
| Cross-Section | XS | 2 | 5 | 0 | 7 |
| Design Alignment | DA | 8 | 0 | 0 | 8 |
| **Total** | | **30** | **18** | **6** | **54** |

Estimated execution time: ~25-35 minutes (T1+T2 tests only)
T3 tests will be marked BLOCKED with reasons.

---

## Toggle (TG)

| ID | Test Case | Expected | Priority | Tier | Setup |
|----|-----------|----------|----------|------|-------|
| FIN-TG-001 | Directly toggle row renders with label and switch | "Directly" label visible, Switch component present, description text visible | High | T1 | None |
| FIN-TG-002 | Switch is in ON state when usage is 'fin' | Switch shows checked state (aria-checked=true) | High | T1 | None (assume default state is ON) |
| FIN-TG-003 | Info icon is visible next to "Directly" label | InfoIcon button present adjacent to label | Med | T1 | None |
| FIN-TG-004 | Tooltip appears on hover over info icon | Tooltip content appears with "When enabled" and "When disabled" sections, bullet lists | Med | T1 | None |
| FIN-TG-005 | Tooltip content includes both enabled and disabled descriptions | "When enabled" section with read-only connectors bullet, "When disabled" section with sensitive actions bullet | Med | T1 | None |
| FIN-TG-006 | Clicking switch toggles from ON to OFF | Switch becomes unchecked, Audience and Prompt sections disappear, Disabled banner appears | High | T2 | Start with toggle ON |
| FIN-TG-007 | Clicking switch toggles from OFF to ON | Switch becomes checked, Disabled banner disappears, Audience and Prompt sections appear | High | T2 | Toggle OFF first |
| FIN-TG-008 | Toggle state persists after switching tabs | Switch state should remain after clicking another tab and returning to Fin tab | Med | T2 | Toggle, switch tab, return |
| FIN-TG-009 | Toggle description text reads correctly | "Fin will directly trigger this connector based on your prompt" text is visible | Low | T1 | None |

---

## Prompt (PR)

| ID | Test Case | Expected | Priority | Tier | Setup |
|----|-----------|----------|----------|------|-------|
| FIN-PR-001 | Prompt section renders with label and textarea | "Prompt" label visible, Textarea present with placeholder text | High | T1 | None (toggle ON) |
| FIN-PR-002 | Textarea has correct placeholder | Placeholder reads "Describe when Fin should trigger this connector..." | Med | T1 | None |
| FIN-PR-003 | Help text renders below textarea | "Describe when Fin should trigger this data connector, what it does and which customer questions it should handle." text visible | Med | T1 | None |
| FIN-PR-004 | "See best practices" link is present and opens in new tab | Link text "See best practices" visible, has target="_blank", href points to intercom.com help article | Med | T1 | None |
| FIN-PR-005 | Textarea accepts text input | Typing in textarea updates the value, text appears in the field | High | T1 | None |
| FIN-PR-006 | Textarea auto-expands with content | After entering multi-line text, textarea height increases to fit content without scroll | Med | T2 | Type several lines of text |
| FIN-PR-007 | Textarea has maxLength of 30,000 characters | Textarea has maxLength=30000 attribute | Low | T2 | Inspect element attributes |
| FIN-PR-008 | Optimize button enables when text is entered | After typing text, the Optimize button becomes enabled (not disabled) | High | T2 | Clear textarea, then type text |
| FIN-PR-009 | Textarea value persists on save | After entering a prompt and saving, value persists on reload | Med | T3 | Requires save API to work |

---

## AI Optimize (AI)

| ID | Test Case | Expected | Priority | Tier | Setup |
|----|-----------|----------|----------|------|-------|
| FIN-AI-001 | Optimize button renders in idle state | Button with AiStarsIcon and "Optimize" text visible, variant="secondary" | High | T1 | None (toggle ON, text in prompt) |
| FIN-AI-002 | Optimize button is disabled when prompt is empty | Button has disabled attribute when textarea is empty | High | T1 | Clear textarea |
| FIN-AI-003 | Optimize button has correct testid | data-testid="ai-optimize-button" present on button | Med | T1 | None |
| FIN-AI-004 | Clicking Optimize triggers loading state | "Thinking..." text appears, Stop button visible, Optimize button disappears | High | T2 | Enter text in prompt, click Optimize |
| FIN-AI-005 | Stop button has correct testid | data-testid="ai-optimize-stop" present on Stop button during loading | Med | T2 | Trigger loading state |
| FIN-AI-006 | Loading state shows "Thinking..." text | Text "Thinking..." visible with muted styling during API call | High | T3 | BLOCKED: Requires backend API `generate_code_block_suggestion` running |
| FIN-AI-007 | Successful API response shows suggestion panel | AI suggestion panel appears with reasoning text, suggested text box, Apply and Dismiss buttons | High | T3 | BLOCKED: Requires backend API response |
| FIN-AI-008 | Apply button replaces prompt with suggestion | Clicking Apply updates textarea with suggestion text, suggestion panel disappears | High | T3 | BLOCKED: Requires suggestion state |
| FIN-AI-009 | Dismiss button removes suggestion panel | Clicking Dismiss hides suggestion panel, returns to idle state with Optimize button | High | T3 | BLOCKED: Requires suggestion state |
| FIN-AI-010 | Error shows notification toast | When API fails, error notification "Failed to generate suggestion" appears | Med | T3 | BLOCKED: Requires backend API error |

---

## Audience (AU)

| ID | Test Case | Expected | Priority | Tier | Setup |
|----|-----------|----------|----------|------|-------|
| FIN-AU-001 | Audience row renders with label and dropdown trigger | "Audience" label visible, Button with MultiplePeopleIcon + "Everyone" + Chevron visible | High | T1 | None (toggle ON) |
| FIN-AU-002 | Audience trigger has correct testid | data-testid="audience-trigger" present on the button | Med | T1 | None |
| FIN-AU-003 | Dropdown trigger shows "Everyone" text with icon | MultiplePeopleIcon visible, "Everyone" text, DropdownMenu.Chevron present | Med | T1 | None |
| FIN-AU-004 | Clicking trigger opens dropdown with "Everyone" option | DropdownMenu.Content appears with CheckboxItem "Everyone" (checked) | High | T2 | Click audience trigger |
| FIN-AU-005 | Dropdown "Everyone" item has correct testid and is checked | data-testid="audience-everyone" present, checkbox shows checked state | Med | T2 | Open dropdown |

---

## Disabled Banner (DB)

| ID | Test Case | Expected | Priority | Tier | Setup |
|----|-----------|----------|----------|------|-------|
| FIN-DB-001 | Disabled banner renders when toggle is OFF | Banner container visible with alert icon, heading, and description | High | T1 | Toggle OFF (if not already) |
| FIN-DB-002 | Banner heading text is correct | "Fin will not run this connector directly." text visible in bold | High | T1 | Toggle OFF |
| FIN-DB-003 | Banner description text is correct | "It must be added to a Workflow, Task, Procedure, Macro, or triggered by a teammate in the Inbox." text visible | Med | T1 | Toggle OFF |
| FIN-DB-004 | "Using Data connectors" link button is present | Button with KnowledgeIcon and "Using Data connectors" text visible, links to help article | Med | T1 | Toggle OFF |
| FIN-DB-005 | ContentSquare alert icon renders | TinyAlertIcon inside ContentSquare with variant="notice" visible | Med | T2 | Toggle OFF, inspect banner |
| FIN-DB-006 | Help link opens in new tab | Link has target="_blank" and rel="noopener noreferrer" | Low | T2 | Toggle OFF, inspect link |

---

## Cross-Section (XS)

| ID | Test Case | Expected | Priority | Tier | Setup |
|----|-----------|----------|----------|------|-------|
| FIN-XS-001 | Card renders with correct heading | Card with "How should Fin use this connector?" heading (h3) visible | High | T1 | None |
| FIN-XS-002 | Card has max-width 800px centered layout | Card container has max-w-[800px] and is centered with px-6 pt-5 | Med | T1 | None |
| FIN-XS-003 | Toggle ON shows Audience + Prompt, hides banner | When switch is ON: AudienceRow and PromptSection visible, DisabledBanner hidden | High | T2 | Ensure toggle ON |
| FIN-XS-004 | Toggle OFF shows banner, hides Audience + Prompt | When switch is OFF: DisabledBanner visible, AudienceRow and PromptSection hidden | High | T2 | Toggle OFF |
| FIN-XS-005 | Toggling ON→OFF→ON preserves prompt text | Enter text, toggle OFF, toggle ON — prompt text should be preserved | High | T2 | Enter text, toggle twice |
| FIN-XS-006 | Tab navigation to Fin tab loads correct content | Clicking Fin tab shows the connector config card | Med | T2 | Navigate from another tab |
| FIN-XS-007 | Sections are vertically stacked with gap-6 spacing | Toggle row, Audience row, and Prompt section have consistent vertical spacing | Low | T2 | Visual inspection of spacing |

---

## Design Alignment (DA)

| ID | Test Case | Expected | Priority | Tier | Setup |
|----|-----------|----------|----------|------|-------|
| FIN-DA-001 | Card component present (extraction row 1) | a11y tree: Card container renders as region or generic container | High | T1 | None |
| FIN-DA-002 | Switch component has role=switch (extraction row 6) | a11y tree: element with role="switch" and aria-checked present | High | T1 | None |
| FIN-DA-003 | Tooltip trigger has role=button (extraction row 4) | a11y tree: button element adjacent to "Directly" text | High | T1 | None |
| FIN-DA-004 | Audience trigger Button variant=secondary (extraction row 9) | a11y tree: role=button with text containing "Everyone" | High | T1 | None |
| FIN-DA-005 | Textarea has role=textbox (extraction row 12) | a11y tree: role=textbox with placeholder text | High | T1 | None |
| FIN-DA-006 | Optimize button has role=button (extraction row 15) | a11y tree: role=button with text "Optimize" | High | T1 | None |
| FIN-DA-007 | Banner alert styling matches design (extraction row 23) | Screenshot: border, rounded corners, bg-base-module background — REVIEW for human | Med | T1 | Toggle OFF |
| FIN-DA-008 | Suggestion panel styling matches design (extraction row 17) | Screenshot: border border-neutral-border, rounded-medium — REVIEW for human | Med | T1 | Requires suggestion state (capture from code inspection) |

---

## Test Execution Order

1. **TG + AU** (quick T1 tests, establish toggle state)
2. **PR** (form interactions with textarea)
3. **AI** (most complex, T3 blockers expected)
4. **DB** (requires toggle OFF)
5. **XS** (cross-section interactions, tab switching)
6. **DA** (a11y snapshot verification)

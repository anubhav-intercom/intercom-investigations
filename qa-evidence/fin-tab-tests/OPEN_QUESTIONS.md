# Open Questions: Fin Tab QA

**Generated**: 2026-03-16
**Context**: QA round 1 for fin-tab
**Pass Rate**: 49/49 (100% excluding BLOCKED)

## For Designer

- [ ] **FIN-DA-007**: Banner alert styling — please review [DA-07-banner-alert-disabled.png](screenshots/DA-07-banner-alert-disabled.png) to verify:
  - Border color matches `neutral-border` token
  - Background color matches `bg-base-module` token
  - Rounded corners match `rounded-medium`
  - Yellow alert icon square matches `bg-notice-container` (ContentSquare variant="notice")
  - Overall spacing and layout match Figma frame "Data connectors 117" (Contextual-alert instance)

## For Engineer

- No behavioral issues found. All T1+T2 tests pass.

## For Self-Investigation

- [ ] **FIN-AI-007/008/009**: AI Optimize suggestion panel — these tests require the backend API `generate_code_block_suggestion` to be running locally. Options to unblock:
  1. Start the backend service that handles prompt optimization
  2. Mock the API response at the network level (MSW or similar)
  3. Use Playwright's route interception to inject a mock response

- [ ] **FIN-PR-009**: Save persistence — requires the save API to work end-to-end. Test by clicking "Save draft" after modifying the prompt text and verifying on reload.

- [ ] **FIN-DA-008**: Suggestion panel design alignment — blocked by same dependency as AI-007. Once the panel renders, verify it matches Figma frame "Data connectors 120" (AI suggestion with reasoning, apply/dismiss).

## Blocked Tests — Infrastructure Needed

| Test ID | What's Needed | Who Can Help |
|---------|---------------|--------------|
| FIN-AI-007 | Backend API `generate_code_block_suggestion` running locally | team-data-foundations |
| FIN-AI-008 | Depends on AI-007 (suggestion panel must appear) | team-data-foundations |
| FIN-AI-009 | Depends on AI-007 (suggestion panel must appear) | team-data-foundations |
| FIN-AI-010 | Already tested (error toast observed naturally) | — |
| FIN-PR-009 | Save API end-to-end flow working in dev | team-data-foundations |
| FIN-DA-008 | Depends on AI-007 (suggestion panel must render) | team-data-foundations |

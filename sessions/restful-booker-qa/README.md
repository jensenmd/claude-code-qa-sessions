# Session 1 — restful-booker-qa

**Date:** April 28, 2026
**Claude Code Version:** 2.1.122
**Subject Repo:** [restful-booker-qa](https://github.com/jensenmd/restful-booker-qa)
**Session Type:** Coverage gap analysis + targeted improvement

---

## Subject Project

restful-booker-qa is a full-stack QA portfolio project targeting the Restful-Booker demo application — a hotel booking API and web UI built for QA practice. The project demonstrates a layered test strategy across API and UI layers:

- **Postman/Newman** — REST API test collection with JavaScript assertions
- **Playwright** — UI automation with Page Object Model
- **GitHub Actions CI** — both suites running in parallel on every push

---

## The Claude Code Session

Claude Code v2.1.122 was launched against the restful-booker-qa codebase with a single prompt:

> *"Looking at the booking tests, what additional test cases would improve coverage? Suggest 3 specific tests that are missing."*

Claude Code autonomously:
- Searched the project structure
- Read 16 files in parallel
- Analyzed test architecture, page objects, and existing coverage
- Returned 3 specific, actionable coverage gap recommendations

Total time: 35 seconds.

---

## Coverage Gaps Identified

**Gap 1 — Weak negative assertion in empty form validation test**

The existing test submitted an empty form and only asserted that the confirmation heading was NOT visible. Claude Code identified this as a weak negative assertion — it would pass even if the page crashed entirely. No test verified that validation error messages actually appeared.

**Gap 2 — Missing confirmation data verification**

The happy path booking test only checked that the confirmation heading was visible — not that it contained the correct guest data. A regression where booking succeeded but showed wrong name or dates would pass undetected.

**Gap 3 — Single room type hardcoded across all tests**

All three booking tests hardcoded `.nth(2)` — the Double room. No test exercised Single or Suite room types. The `bookRoomByType()` method in HomePage.js existed but was never called in any test.

---

## Human Judgment Applied

**On Gap 1:**

Claude Code initially proposed asserting specific error message strings:
- `'Firstname should not be blank'`
- `'Lastname should not be blank'`

**Human judgment:** Rejected. Hardcoding specific error strings creates brittle tests — if the demo site changes those strings, the test breaks even though validation is still working correctly.

**Decision:** Assert that the error container (`.alert.alert-danger`) is visible — meaningful coverage without brittleness.

This is the human-in-the-loop moment. Claude Code proposed. Human evaluated the tradeoff. Human directed a better implementation.

**On Gaps 2 and 3:**

Deferred to future session. The demo site (automationintesting.online) was experiencing environmental instability at the time — a known, documented characteristic of this free shared Heroku app. Proceeding with additional test changes against an unstable target would introduce noise into the validation process.

---

## The Implementation

Claude Code was directed:

> *"Good point on fragility. Let's go with a middle approach — assert that the validation error container (.alert.alert-danger) is visible after empty form submit, but don't assert on specific message strings. Apply that change."*

Claude Code read the existing test file and made a surgical two-file change:

**playwright/tests/booking.spec.js — before:**
```javascript
// Confirmation should NOT appear
await expect(booking.confirmationHeading).not.toBeVisible({ timeout: 3000 });
```

**playwright/tests/booking.spec.js — after:**
```javascript
// Validation error container should appear
await expect(page.locator('.alert.alert-danger')).toBeVisible({ timeout: 5000 });

// Confirmation should NOT appear
await expect(booking.confirmationHeading).not.toBeVisible();
```

Implementation time: 9 seconds.

---

## Commit Proof

commit ce241a6
Improve empty form validation test: assert error container visible
1 file changed, 4 insertions(+), 1 deletion(-)

[View commit on GitHub](https://github.com/jensenmd/restful-booker-qa/commit/ce241a6)

---

## What This Demonstrates

This session illustrates the Level 3 AI involvement pattern:

- **Claude Code operated autonomously** — it read the codebase, reasoned about coverage, and proposed specific changes without being told what to look for
- **Human judgment was essential** — the rejection of brittle string assertions was a QA decision that required understanding the tradeoffs
- **The outcome was better than either alone** — Claude Code found the gap, human shaped the solution, automation executed it

This is not AI replacing QA judgment. It is AI amplifying QA capacity.

---

## Try This Yourself

1. Install Claude Code: `npm install -g @anthropic-ai/claude-code`
2. Authenticate with your Anthropic account: `claude`
3. Navigate to any existing QA project: `cd your-qa-project`
4. Ask: *"Looking at the existing tests, what additional test cases would improve coverage? Suggest 3 specific tests that are missing."*
5. Evaluate each suggestion with your QA judgment
6. Direct the implementation: *"Implement suggestion #1 — but use [your approach] instead of [their approach]"*
7. Review the diff before accepting
8. Commit and push

The workflow scales to any QA codebase. The human judgment is always yours.

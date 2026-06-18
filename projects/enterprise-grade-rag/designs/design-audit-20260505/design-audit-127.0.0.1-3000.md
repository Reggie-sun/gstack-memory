# Design Audit: http://127.0.0.1:3000/login

Date: 2026-05-05 08:30:18 HKT
Scope: Login page only. The app redirected `/` to `/login`, and the review stayed on the unauthenticated surface by user choice.
Status: DONE

## Headline

- Design score: `C` -> `B`
- AI slop score: `C` -> `B`
- Total findings: `1` high-impact cluster
- Fixes applied: `1 verified`

## First Impression

The site communicated "there is a form here" before it communicated "this is the entry to an enterprise knowledge product."

I noticed a small centered card floating in a very large white field. The first three things my eye went to were the logo, the orange button, and the boxed support note that looked like a disabled input. If I had to describe the original page in one word: `empty`.

Page area test:

- Left and top whitespace: no clear job
- Main card: login form, but the page name was not obvious
- Support box: visually read like a read-only field, not help content

## Inferred Design System

- Fonts: `ui-sans-serif/system-ui` and `Inter/system stack`
- Colors observed: slate neutrals, white, warm copper accent, muted blue-gray helper text
- Heading scale before fix: no visible semantic headings on the page
- Touch targets under 44px: none found
- Console errors: none
- Perf snapshot: `total 488ms`, `domReady 478ms`

## Findings

### FINDING-001

- Impact: `high`
- Category: `visual hierarchy`, `spacing/layout`, `content`, `AI slop`
- Problem:
  - The page had no visible page title, so the trunk test was only partial.
  - Desktop composition felt like a generic form card dropped onto a blank canvas.
  - The support note used the same bordered-box language as form controls, so it read like a disabled textarea instead of help.
- User impact:
  - New or expired-session users had weak orientation.
  - The strongest emotional signal was "empty screen" rather than "secure enterprise entry point."
  - Help information looked secondary and slightly confusing.

## Fix Applied

- Status: `verified`
- Commit: `d56319fe`
- Files changed:
  - `frontend/src/pages/LoginPage.tsx`
  - `frontend/src/index.css`
  - `frontend/src/pages/workspacePages.test.tsx`
- What changed:
  - Added a real left-side intro block on larger viewports with a visible `h1`, concise product context, and a plain-language pre-login checklist.
  - Added a tighter card header so mobile still has a clear local hierarchy.
  - Restyled the support area into a support panel instead of a faux input box.
  - Strengthened the page background and card depth using existing copper/slate tokens instead of introducing a new visual language.
  - Added a focused test assertion for the new accessible heading.

## Evidence

- Before:
  - `screenshots/first-impression-chrome.png`
  - `screenshots/login-mobile.png`
  - `screenshots/login-tablet.png`
  - `screenshots/login-desktop.png`
- After:
  - `screenshots/finding-001-after-desktop.png`
  - `screenshots/finding-001-after-mobile.png`
  - `screenshots/finding-001-after-tablet.png`
  - `screenshots/finding-001-after-responsive-desktop.png`

## Validation

- `npm exec vitest run src/pages/workspacePages.test.tsx` -> passed (`7/7`)
- `git diff --check -- frontend/src/pages/LoginPage.tsx frontend/src/index.css frontend/src/pages/workspacePages.test.tsx` -> passed
- Real-page recheck on desktop/tablet/mobile screenshots -> passed
- Console errors on `/login` -> none

## Remaining Concerns

- The global product type stack is still `Inter/system`, which keeps the experience on the safe/generic side. I did not broaden this review into a product-wide typography change.
- Authenticated routes were not reviewed because this session stayed on the login surface only.

## PR Summary

Design review found 1 high-impact login-page issue cluster and fixed it. Design score `C -> B`, AI slop score `C -> B`.

---

## Authenticated Addendum

Date: 2026-05-05 08:36 HKT
Status: DONE_WITH_CONCERNS
Scope: authenticated smoke on `/portal`, `/portal/library`, `/portal/sop`, `/admin-console`

### Login verification

- Demo account `sys.admin.demo` successfully authenticated through the real local stack.
- Browser landed on `http://127.0.0.1:3000/portal`.
- API login endpoint returned `200 OK`.

### Additional findings

#### FINDING-002

- Impact: `high`
- Category: `empty state`, `content`, `visual hierarchy`
- Surface: `/portal/library`
- Problem:
  - After choosing a department, the page shows an empty green notification bar near the top with no text.
  - The detail pane also keeps a blank outlined strip plus a lone document icon, instead of a real empty-state message.
  - The user can tell "something changed" but cannot tell what to do next.
- Evidence:
  - `screenshots/portal-library.png`
  - `screenshots/library-selected.png`
- User impact:
  - This fails the "mindless click" rule. After selecting a department, the next step is not self-evident.
  - The empty bars look like broken status components, which hurts trust more than a plain empty-state card would.

#### FINDING-003

- Impact: `medium`
- Category: `content density`, `navigation hygiene`
- Surface: `/portal`
- Problem:
  - The recent-session rail is packed with many repetitive or probe-like conversation titles, so the left column dominates attention before the main chat entry surface does.
  - The first screen reads as "history archive" more than "start a focused query."
- Evidence:
  - `screenshots/portal-home.png`
- User impact:
  - The conversation-start surface loses its focal priority.
  - Scan cost rises because repeated session titles create visual noise.

### Surfaces checked

- `/portal`
- `/portal/library`
- `/portal/sop`
- `/admin-console`

### Validation

- Real browser login on `http://127.0.0.1:3000/login` -> success
- API smoke on `POST /api/v1/auth/login` -> `200 OK`
- Browser navigation confirmed:
  - `/portal`
  - `/portal/library`
  - `/portal/sop`
  - `/admin-console`

### Deferred

- No code changes were made in this pass because the repo worktree still contains an unrelated uncommitted `AGENTS.md` change, and we explicitly chose report-only continuation.

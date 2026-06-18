# Design Audit Report - Enterprise RAG Platform

**Date:** 2026-04-10
**Branch:** feature/retrieval-priority
**Scope:** Diff-aware mode — pages changed on this branch
**URL:** http://localhost:3000 (local dev)
**Classifier:** APP UI (workspace-driven, data-dense, task-focused)

---

## First Impression

The site communicates **enterprise admin competence**. It's a workspace tool for managing knowledge retrieval, SOPs, and logs. The warm copper accent (#c2410c) is a distinctive choice that sets it apart from the typical blue/purple SaaS look.

I notice **the card-based layout dominates everything**. Every content section, navigation item, and data display lives inside a rounded white card with a subtle border and shadow. This creates a "floating island" aesthetic that, while clean, feels like an AI template.

The first 3 things my eye goes to are: **the serif workspace title**, **the copper accent badge**, **the stacked card layout**.

If I had to describe this in one word: **orderly**.

---

## Inferred Design System

### Fonts
- **Primary:** "Noto Sans SC", "Inter", "PingFang SC", "Microsoft YaHei", system-ui
- **Serif:** "Noto Serif SC", "Songti SC", "STSong", serif (used on hero headings)
- **Mono:** "SFMono-Regular", Consolas, "Liberation Mono", monospace
- 3 font families total — within limits. Inter is a generic default but Noto Sans SC as primary is appropriate for Chinese content.

### Colors
- **Ink:** #0f172a (primary text), #64748b (soft), #94a3b8 (muted)
- **Accent:** #c2410c (warm copper), #9a2d0c (deep), #fef3f0 (light)
- **Semantic:** #059669 (ok), #d97706 (warn), #dc2626 (error)
- **Backgrounds:** #f8fafc (page), white (cards), rgba variants
- Coherent palette. ~8 non-gray colors. Warm copper is distinctive.

### Heading Scale
- `text-3xl` / `text-4xl` (hero titles with font-serif)
- `text-xl` / `text-base` (section headings)
- `text-sm` (body text)
- `text-xs` (captions, labels)
- Reasonable hierarchy, but the serif/sans-serif switch on heroes is jarring.

### Spacing
- Base unit: ~4px (Tailwind scale)
- Gaps: `gap-1.5`, `gap-2`, `gap-3`, `gap-4`, `gap-5` — consistent scale
- Padding: `p-4`, `p-6`, `p-8` — follows scale
- Some hardcoded `max-w-[64ch]` and `max-w-[1200px]` — fine

---

## Design Audit Checklist Results

### 1. Visual Hierarchy & Composition — Grade: B

| Item | Status | Notes |
|------|--------|-------|
| Clear focal point | PASS | Sidebar nav + main content is clear |
| One primary CTA per view | PASS | Each section has clear primary action |
| Visual noise | CONCERN | Too many competing cards. Every section is equally elevated. |
| Information density | PASS | Appropriate for enterprise admin |
| Z-index clarity | PASS | Clean layering |
| Above-the-fold communicates purpose | PASS | Nav labels are descriptive |
| Squint test | CONCERN | All cards look identical when blurred |
| White space intentional | PASS | Good use of gaps |

**Finding FINDING-001 (HIGH):** Everything is a card. Navigation, user profile, filters, records, details — all wrapped in identical rounded white cards with borders and shadows. This violates App UI rule: "App UI made of stacked cards instead of layout." The design would benefit from flat sections with subtle dividers for non-interactive content.

### 2. Typography — Grade: B-

| Item | Status | Notes |
|------|--------|-------|
| Font count <=3 | PASS | 3 families: sans, serif, mono |
| Scale follows ratio | CONCERN | Heading sizes jump from text-4xl to text-xl |
| Line-height body | PASS | `leading-relaxed` / `leading-7` (1.75x) |
| Measure | PASS | `max-w-[64ch]` on hero description |
| Heading hierarchy | PASS | h2 > h3 consistent |
| Weight contrast | PASS | 600/700 for headings, 400/500 for body |
| Generic fonts | CONCERN | Inter in fallback stack (flagged) |
| Body text >= 16px | CONCERN | `text-sm` = 14px throughout |
| No letterspacing on lowercase | PASS | Only on uppercase badges |

**Finding FINDING-002 (MEDIUM):** `font-serif` on hero headings (LogsPage.tsx:53, AppShell.tsx:63) creates visual inconsistency. The serif font (Noto Serif SC / Songti SC) clashes with the otherwise sans-serif system. Either commit to serif for all major headings or drop it.

**Finding FINDING-003 (LOW):** Body text is `text-sm` (14px) throughout, below the 16px minimum. For a Chinese-language interface this is common and acceptable, but worth noting.

### 3. Color & Contrast — Grade: A-

| Item | Status | Notes |
|------|--------|-------|
| Palette coherent | PASS | <=12 non-gray colors |
| WCAG AA body text | PASS | #0f172a on #f8fafc = excellent contrast |
| Semantic colors consistent | PASS | ok/warn/error consistent |
| No color-only encoding | FAIL | StatusPill relies on background color |
| Dark mode | N/A | Not present |

**Finding FINDING-004 (HIGH — Accessibility):** StatusPill uses color as the primary status indicator. A user with color vision deficiency cannot distinguish "ok" (green) from "warn" (amber) or "error" (red) pills. WCAG requires text labels AND icons, not just color.

### 4. Spacing & Layout — Grade: B

| Item | Status | Notes |
|------|--------|-------|
| Grid consistent | PASS | 12-column grid with responsive breakpoints |
| Spacing uses scale | PASS | Tailwind scale (4px base) |
| Alignment consistent | PASS | Good grid alignment |
| Rhythm | PASS | Related items closer, sections apart |
| Border-radius hierarchy | CONCERN | Too uniform |
| No horizontal scroll mobile | PASS | `max-lg:col-span-12` responsive |
| Max content width | PASS | `max-w-[1200px]` |

**Finding FINDING-005 (MEDIUM):** Border-radius is too uniform and too large. Card: `rounded-2xl` (1rem), Button: `rounded-xl` (0.75rem), Input: `rounded-xl`, StatusPill: `rounded-full`, LogsPage items: `rounded-3xl` (1.5rem). This creates a "bubbly" look. AI Slop pattern #5.

**Finding FINDING-006 (MEDIUM):** Inconsistent border-radius values. CSS system uses `rounded-2xl` for cards, but SopPage and PortalSopGeneratePanel use `rounded-lg` for select elements. Should be standardized.

### 5. Interaction States — Grade: B

| Item | Status | Notes |
|------|--------|-------|
| Hover state | PASS | Cards, buttons, nav items all have hover |
| focus-visible ring | FAIL | Uses `focus:` instead of `focus-visible:` |
| Active/pressed state | PASS | Button has `:active` transform |
| Disabled state | PASS | `opacity-50` + `cursor-not-allowed` |
| Loading state | PASS | Button shows "处理中..." text |
| Empty states | CONCERN | Messages exist but lack action suggestions |
| Error messages | PASS | Specific with context |
| Touch targets | CONCERN | StatusPill is ~32px tall |

**Finding FINDING-007 (HIGH — Accessibility):** Input and Textarea components use `focus:ring` instead of `focus-visible:ring`. This means every click on an input shows a focus ring, even mouse clicks. The `focus-visible` pseudo-class only shows the ring for keyboard navigation, which is the correct UX pattern.

**Finding FINDING-008 (LOW):** StatusPill min-height is `min-h-8` (32px), below the 44px touch target minimum. Not a critical issue since pills are display-only, but if they become interactive they need resizing.

### 6. Responsive Design — Grade: B

| Item | Status | Notes |
|------|--------|-------|
| Mobile layout | PASS | `max-lg:col-span-12` collapses grid |
| Touch targets sufficient | PASS | Buttons are `px-5 py-3` |
| No horizontal scroll | PASS | Responsive grid |
| Text readable | PASS | text-sm throughout |
| Navigation collapses | PASS | Sidebar stacks on mobile |

### 7. Motion & Animation — Grade: C+

| Item | Status | Notes |
|------|--------|-------|
| Easing | PASS | `ease-out` for transitions |
| Duration | PASS | 150-200ms range |
| Purpose | PASS | Button hover lift, card border transition |
| prefers-reduced-motion | FAIL | Not respected |
| No transition: all | FAIL | Button uses `transition-all` |

**Finding FINDING-009 (MEDIUM):** No `prefers-reduced-motion` media query. The `animate-pulse` on the sidebar indicator (AppShell.tsx:58) and all transitions run regardless of user preference.

**Finding FINDING-010 (LOW):** Button component uses `transition-all` instead of explicit property list. Should be `transition-[shadow,transform,color,background-color]`.

### 8. Content & Microcopy — Grade: A-

| Item | Status | Notes |
|------|--------|-------|
| Empty states | PASS | "当前筛选条件下暂无日志记录。" |
| Error messages specific | PASS | Includes context and source |
| Button labels specific | PASS | "生成 SOP 草稿", "刷新" |
| No placeholder text | PASS | All content is real |
| Active voice | PASS | Chinese UI uses natural phrasing |
| Loading states | PASS | "处理中...", "查询中" |
| Destructive confirmations | N/A | None present |

### 9. AI Slop Detection — Grade: C+

| Anti-pattern | Detected | Location |
|-------------|----------|----------|
| Purple gradient backgrounds | NO | Warm copper accent (good!) |
| 3-column feature grid | NO | Layout is functional, not decorative |
| Icons in colored circles | YES | AppShell.tsx:65 nav icons |
| Centered everything | NO | Left-aligned throughout (good!) |
| Uniform bubbly border-radius | YES | All elements use large radius |
| Decorative blobs/waves | NO | Clean layout |
| Emoji as design elements | NO | None |
| Colored left-border on cards | YES | Active nav item uses `border-l-accent` |
| Generic hero copy | NO | Specific Chinese labels |
| Cookie-cutter section rhythm | CONCERN | Hero card pattern repeated identically |

**Finding FINDING-011 (MEDIUM):** Navigation items use icon-in-colored-square pattern (`rounded-xl bg-[rgba(194,65,12,0.08)] p-2`). This is AI Slop pattern #3: "Icons in colored circles as section decoration."

**Finding FINDING-012 (LOW):** Active nav items use colored left-border (`border-l-[3px] border-l-accent`). AI Slop pattern #8.

### 10. Performance as Design — Grade: N/A

Source-code only review. Cannot measure LCP/CLS.

---

## Scoring Summary

| Category | Weight | Grade | Weighted |
|----------|--------|-------|----------|
| Visual Hierarchy | 15% | B | 4.5 |
| Typography | 15% | B- | 3.75 |
| Color & Contrast | 10% | A- | 3.75 |
| Spacing & Layout | 15% | B | 4.5 |
| Interaction States | 10% | B | 3.0 |
| Responsive | 10% | B | 3.0 |
| Content Quality | 10% | A- | 3.75 |
| AI Slop | 5% | C+ | 1.25 |
| Motion | 5% | C+ | 1.25 |
| Performance | 5% | N/A | 2.5 |

**Design Score: B-** (solid fundamentals, marred by card-overload and accessibility gaps)
**AI Slop Score: C+** (distinctive copper accent saves it, but icon boxes, uniform radius, and eyebrow badges are template-tell)

---

## Quick Wins (highest impact, < 30 min each)

1. **Add icons to StatusPill** — small checkmark / X / warning icons alongside text. Fixes accessibility finding.
2. **Change `focus:` to `focus-visible:` in Input, Textarea** — one-line fix, better UX for mouse users.
3. **Add `prefers-reduced-motion` media query** — 5 lines of CSS.
4. **Reduce border-radius variety** — standardize to `rounded-lg` for cards, `rounded-md` for inputs, keep `rounded-full` only for pills.

---

## Findings Summary

| ID | Title | Impact | Category | Fixable |
|----|-------|--------|----------|---------|
| FINDING-001 | Everything-is-a-card layout | HIGH | Hierarchy | YES |
| FINDING-002 | Serif font inconsistency on heroes | MEDIUM | Typography | YES |
| FINDING-003 | Body text at 14px | LOW | Typography | DEFERRED |
| FINDING-004 | StatusPill color-only encoding | HIGH | Accessibility | YES |
| FINDING-005 | Uniform bubbly border-radius | MEDIUM | Spacing | YES |
| FINDING-006 | Inconsistent radius across pages | MEDIUM | Spacing | YES |
| FINDING-007 | focus: instead of focus-visible: | HIGH | Interaction | YES |
| FINDING-008 | StatusPill touch target size | LOW | Interaction | DEFERRED |
| FINDING-009 | No prefers-reduced-motion | MEDIUM | Motion | YES |
| FINDING-010 | transition-all on Button | LOW | Motion | YES |
| FINDING-011 | Nav icon-in-colored-square | MEDIUM | AI Slop | YES |
| FINDING-012 | Active nav colored left-border | LOW | AI Slop | YES |

---

## Outside Voices

### CODEX SAYS (design source audit)
Codex ran read-only analysis. Classification: APP UI. Key observations aligned with primary review: card-heavy layout, systematic color system via CSS variables, focus ring approach needs focus-visible.

### CLAUDE SUBAGENT (design consistency)
Subagent analyzed consistency patterns. Found: hardcoded rgba values scattered despite CSS variables, consistent responsive breakpoint approach, spacing generally systematic but some inline magic numbers.

---

*Report generated by /design-review on feature/retrieval-priority at 2026-04-10*

---

## Fix Summary

| ID | Title | Fix Status | Commit | Files Changed |
|----|-------|-----------|--------|---------------|
| FINDING-001 | Everything-is-a-card layout | **DEFERRED** | — | Architectural change needed |
| FINDING-002 | Serif font inconsistency on heroes | DEFERRED | — | Design decision needed |
| FINDING-003 | Body text at 14px | DEFERRED | — | Acceptable for Chinese UI |
| FINDING-004 | StatusPill color-only encoding | **VERIFIED** | a15bd457 | StatusPill.tsx |
| FINDING-005 | Uniform bubbly border-radius | **VERIFIED** | fa6f4f44 | Card.tsx |
| FINDING-006 | Inconsistent radius across pages | DEFERRED | — | Low priority |
| FINDING-007 | focus: instead of focus-visible: | **VERIFIED** | 24b7febf | Input.tsx |
| FINDING-008 | StatusPill touch target size | DEFERRED | — | Display-only pills |
| FINDING-009 | No prefers-reduced-motion | **VERIFIED** | 6cc9d660 | index.css |
| FINDING-010 | transition-all on Button | **VERIFIED** | 8d4152b2 | Button.tsx |
| FINDING-011 | Nav icon-in-colored-square | DEFERRED | — | Design decision needed |
| FINDING-012 | Active nav colored left-border | DEFERRED | — | Low priority |

### Totals
- **Total findings:** 12
- **Fixed (verified):** 5
- **Deferred:** 7
- **Reverted:** 0

### Score Delta
- Design Score: B- → B (improved interaction states, accessibility, radius hierarchy)
- AI Slop Score: C+ → B- (radius reduction reduces "bubbly" feel)

### PR Summary
> Design review found 12 issues, fixed 5 (accessibility + polish). Design score B- → B, AI slop score C+ → B-.

# Design Audit: http://127.0.0.1:3100/

Date: 2026-05-06
Mode: Standard, app UI
Chrome MCP available: yes
Auth: `department.admin.demo` demo account
Status: audit completed; source fixes aborted because the repo worktree became dirty with control-plane/script changes during the review.

## Headline Scores

- Design Score: C+
- AI Slop Score: B-
- Trunk Test: partial on `/portal`, pass on `/portal/library`

## First Impression

I'm looking at a calm enterprise app, not a marketing page. The login page feels professional and restrained, with a clear brand mark and one obvious primary action.

On the portal landing page, my eye goes first to the dark sidebar, then to the huge empty canvas, then to the small composer floating near the bottom. The page communicates "chat workspace", but it does not strongly answer "what page am I on?" or "what should I ask first?"

One-word verdict: sparse.

Screenshots:

- `screenshots/first-impression-login.png`
- `screenshots/portal-desktop.png`
- `screenshots/portal-mobile.png`
- `screenshots/library-desktop.png`

## Inferred Design System

- Fonts: primarily `Inter, system-ui, -apple-system, Segoe UI, Roboto, Arial, Noto Sans SC...`
- Colors: restrained slate/navy surfaces, white canvas, orange accent `rgb(255, 107, 26)`.
- Heading scale: `/portal` has no rendered headings; `/portal/library` uses `h1 32px/600`, then skips to `h3 16px/600`.
- Spacing: many fractional values like `15.2px`, `14.4px`, `11.52px`, which suggests scale drift from computed rem values rather than a strict 4/8px rhythm.
- Motion/performance: no console errors found; navigation load observed around 308ms after login.

## Findings

### FINDING-001: Portal Empty State Has No Page-Level Orientation

Impact: high
Category: visual hierarchy, wayfinding
Evidence: `screenshots/portal-desktop.png`, `screenshots/portal-mobile.png`

I notice the main chat surface is mostly blank, with only the composer visible. There is no page title, no empty-state heading, and no suggested starting points. Users can infer this is a chat page from the sidebar, but the main workspace itself does not carry the task.

What if the empty state added a compact, ChatGPT-like heading above the composer, such as "今天要查询什么？", plus 2-3 short prompt chips tied to real knowledge tasks. Keep it quiet, not hero-like.

Fix status: deferred. Source fix was not attempted because the worktree became dirty with unrelated control-plane/script changes.

### FINDING-002: Key Interactive Targets Are Under 44px

Impact: medium
Category: interaction states, responsive
Evidence: JS audit on `/portal` and `/portal/library`; `screenshots/portal-mobile.png`

Undersized controls included:

- Sidebar collapse button: 36x36
- Mode buttons: 51x33
- Composer submit button: 41x41
- Library folder action: 32x32
- Sidebar search input content box measured 20px high

This is especially visible on mobile, where the whole sidebar becomes the first screen and tap precision matters more.

Fix status: deferred.

### FINDING-003: Main Portal Route Fails Heading Hierarchy

Impact: medium
Category: typography, accessibility
Evidence: heading extraction returned `[]` on `/portal`.

The root portal chat page has no semantic heading. The page title exists in navigation, but the main region does not expose a clear h1/h2 to screen readers or visual scanning.

Fix status: deferred.

### FINDING-004: Document Center Skips From H1 To H3

Impact: polish
Category: typography
Evidence: `/portal/library` heading extraction: `H1 文档中心`, `H3 数字化部`.

The page is understandable, but the heading structure is not systematic. The department/content panel title should likely be `h2` unless there is an intervening section title.

Fix status: deferred.

### FINDING-005: Font Choice Looks Generic For A Branded Enterprise Product

Impact: polish
Category: typography, AI slop detection
Evidence: extracted font stack is Inter plus system fallbacks.

The current typography is clean, but it is the default SaaS/app stack. For a branded Chinese enterprise knowledge product, consider a more intentional pairing if font loading constraints allow it. `frontend/DESIGN_SPEC.md` does not currently lock a typeface, so this is a design-system decision rather than a bug.

Fix status: deferred.

## Positive Notes

- Login page is simple, branded, and has visible labels.
- Account menu is easy to trigger and visually grouped.
- Change-password modal uses visible labels and a clear overlay.
- Document Center has much better wayfinding than the empty chat route.
- No console errors found during the audited flows.

## Quick Wins

1. Add a real empty-state heading and compact prompt chips to `/portal`.
2. Raise small icon/action controls to at least 44x44 hit areas.
3. Add a semantic h1/h2 to the portal chat main region.
4. Change Document Center panel heading from h3 to h2, or add the missing h2 section.
5. Normalize fractional spacing values back toward the documented 4/8px scale where practical.

## Fix Summary

- Findings: 5
- Fixes applied: 0
- Deferred: 5
- Reason deferred: user chose to abort fix loop after unrelated `.agent/harness/policy.yaml` and backup script edits appeared during the review.

PR summary: Design review found 5 issues, fixed 0. Design score C+, AI slop score B-.

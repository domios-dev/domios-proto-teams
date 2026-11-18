# Design review

How the design lead directs and reviews every surface (the app, dashboard, covers and clips).

## The loop

1. **Read first.** Read the owner's own words, the product's design system and its essence document, and the real shared components. Make the new thing look like it belongs to the product.
2. **Look live.** Open the real build in a real browser at 1440 desktop and 390 phone (DPR 2), with real data. Never judge from an implementer's description.
3. **Write a numbered spec.** Screenshots get numbered pins. Each item gives exact values (px, weight, token, source file) and is marked must-fix (consistency, correctness) or later polish. State plainly what supersedes your own earlier advice.
4. **One writer builds it.** The `implementer` builds it. Visible increments ship as they land, because one big final drop makes the owner think nothing is happening.
5. **Independent review.** The `reviewer` checks the real app with pointer, keyboard, focus return, phone widths of 390 and 320 and real labels, then gives numbered expected / actual / fix findings. Each bounded correction is rechecked.
6. **Verdict.** One PASS or BOUNCE line per pass, with the annotated PNG paths.

## Rules

- **Reuse shared components and tokens.** A parallel local style is a defect.
- **Calm and quiet.** No banners, badges or decoration, and no new labels; nothing flickers. Remove one accessory before shipping.
- **No walls of text** on product surfaces. Explanations go in hover or an info icon.
- **Real data only.** Sample numbers on a product page destroy trust; fixtures belong in tests.
- **Never change frozen math or contracts** to make a chart prettier. Restyle around them.
- **Navigation lives in a slim left sidebar** when there are several screens. Each page has one job: what is published, what is planned, the dashboard.
- **Lists show the newest first, unambiguously sorted.** Upcoming items get their own page.
- **Errors** are one short human sentence, one action and a collapsed Details. No raw codes.
- **The owner's explicit constraints beat taste.** Record them, adapt the layout and don't argue twice. For example: dark only, no refresh button, one screen per job.
- **Tooltips sit next to the pointer,** in one clean card. On phone they open on tap.

## Asking for design

A brief file contains: the goal, the audience and channel, the owner's words verbatim, constraints, the live URL or build, and what is frozen. The answer contains: the spec, the annotated screenshots and the verdict.

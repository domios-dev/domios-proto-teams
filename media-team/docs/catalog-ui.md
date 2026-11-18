# Catalog UI: the owner's publishing desk

The catalog is a small local web page where the owner sees every post, picks what goes out next and watches how it did. These are the rules for its interface. How a change gets reviewed is in `design-review.md`; the data it reads is in `catalog-app.md`.

## Look

- **Use the product's own design system if it has one.** Lift its tokens, cards, menus, segmented controls and back button by value, and never import a parallel local theme. If the product has none, use a calm dark default:
  - near-black page;
  - cards a few percent lighter, radius 16, no borders;
  - one accent colour;
  - a sans-serif UI face at 13 px body and 11 px labels;
  - a monospace face for every figure;
  - a 4/8/12/16 spacing rhythm.
- **Dark only by default.** Add a light theme only when the owner asks for one.
- **Quiet.** No decoration, banners, badges or explanatory paragraphs that the owner didn't ask for. One screen per job. No "Refresh" or "Update metrics" button: data arrives from scheduled collectors, and the page re-reads local files when they change.

## Three views (one sidebar)

1. **Posts.** Published posts, newest first, grouped by day. Each card shows the channel mark, time, title, an opening excerpt, media thumbnails and the latest counters. A compact strip of follower counts sits at the top, for posting channels only.
2. **Planned.** Planned and draft posts dated today or later, plus undated ones last, in ascending order. These are where variants are picked.
3. **Dashboard.**
   - Follower KPIs with small sparks.
   - A "views gained" chart across channels.
   - "Top posts by views gained" for Today / 7 days / 30 days: what is moving.
   - An "All posts" table of cumulative counters: the inventory.
   - The top list and the table answer different questions. Keep both, and never make them look like the same ranking.

**Shared rules:**
- Each view keeps its own filters, page and scroll.
- Every list or table pages at 20 and sorts and filters globally BEFORE slicing. Ties are stable and unknown values sort last in both directions.
- Clicking a post opens a detail view that replaces the main column (about 640 px wide). One Back button returns to the exact page, filter, sort, scroll and opener.

## Numbers

- **No bare numbers.** Every figure carries its unit or label ("226 views", "3 replies", "+23", "7d"). Every chart and KPI carries a small info control that states its scope: the period, the baseline and what is counted.
- **Unknown hides, zero is real.** A value nobody observed is not shown at all. A known 0 is shown as 0.
  - A zero delta is silent.
  - A value with no earlier reading gets a small "first reading" badge instead of an invented change.
- **Deltas:** the arrow alone carries the colour and the text stays muted. Show whole percent at 10 % and above, one decimal below.
- Never round, clamp or smooth the arithmetic to make a chart prettier. The displayed number is the computed number.

## Variant board (Planned → a post with variants)

A planned post can carry up to 26 text variants, keyed A–Z (a finite capacity; don't call it unlimited). The owner compares them and chooses one.

- **Layout.**
  - The detail view widens to the dashboard width for the board.
  - Cards are 320 px wide in a horizontal row that scrolls and snaps, with the next card peeking.
  - Above the row is a strip of letters, one per variant that exists. The strip is its own scroller: it never pushes the page sideways, shows a soft fade on any side with hidden letters, and keeps the active letter in view.
  - The board header (title, strip, undo) stays pinned on one opaque band while the owner reads long variants.
- **Card.**
  - Letter badge and angle (up to 2 lines, no ellipsis).
  - Directly under them, the action row: Copy · Choose · Reject. Actions sit at the top because the row is as tall as the longest variant.
  - Then the text, split into thread parts with "1/3" numbering. Each part has a copy icon when there are two or more parts. A separate reply section has its own copy.
- **Counts per channel.**
  - Each part and reply shows `count / limit`, from the channel registry's compose entry. Exact counters show `257 / 280`. Approximate ones show `~392 / 500`, with ONE info note on the board, not per part.
  - Over the limit shows `+12 over` in words plus a warning edge, never colour alone. Over-limit never blocks Choose.
  - A channel without a limit shows the count only.
- **Choose.**
  - One click, no confirm, reversible until publication.
  - The chosen card gets an accent ring and its button becomes a focusable "✓ Chosen", with Clear choice beside it.
  - Until the canonical post text is updated, the page says "chosen · awaiting post text sync", and Copy post copies the chosen variant.
- **Reject.**
  - A quiet action, not available on the chosen card.
  - The card leaves the row and its letter leaves the strip. Remaining letters never renumber, and focus moves to the next card.
  - "N rejected · Show" in the header lists them with Restore, which is the undo.
  - A rejected variant that is later rewritten returns, marked "Revised".
- **List card.**
  - Shows the letter pills when 8 or fewer variants are visible; above that, only "Pick a variant · N".
  - After a choice it shows the chosen letter plus its sync state.
  - When everything is rejected: "All variants rejected. New ones are coming."
- **States.**
  - Every refusal (catalog changed, busy, variant changed since you read it, published and locked) has plain copy. It keeps the board's scroll and never selects anything automatically.
  - Unreadable variants show the reason and a Copy path, never a Choose.
  - Published posts fold the board, read-only, labelled with what was chosen.

## Phone (below 600 px)

- The sidebar becomes a drawer. Navigation rows and close are 44 px, and focus returns to the opener.
- Board cards are almost full width, with the next one peeking. The letter strip stays pinned with 44 px targets.
- Inventory rows become two blocks: the title (up to 2 lines), then compact facts (date, views, likes...). Each fact stays whole; separators trail the previous fact.
- Nothing scrolls the page sideways at 320 px; only boards, strips and wide tables scroll inside themselves.
- Dates and figures never break a title or footer.

## Accessibility

Accessibility follows the quality bar in `design-review.md`:
- contrast: 4.5:1 for text, 3:1 for control edges and focus;
- visible keyboard focus that lands on the right element;
- clean accessible names (for example "Choose variant B: <angle>");
- colour never the only signal;
- reduced motion respected.

Review the real rendered page, not the code.

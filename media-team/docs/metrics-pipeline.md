# Metrics pipeline

How the team knows what each post did, honestly.

## Building blocks

- **Channel registry** (`channels.json` or equivalent) is the single source of truth per channel:
  - collection method: a script, a browser job or an API adapter, with its guide;
  - identity rules that turn URLs into post keys;
  - declared optional counters, colour and analytics source names.
  The UI and the collectors iterate the registry. **No channel ids hardcoded in code.**
- **Content catalog:** one line per post or asset.
  - Every post gets a **permanent code** (`p1`, `p2`, …) from a ledger with a high-water mark. Codes are never reused.
  - New posts are registered through one writer command under a lock, never appended by hand.
- **Snapshots:** append-only `{at, channel, source, followers, posts:{key:{views, likes, replies, reposts, [declared extras], first_line}}}`, written only through a validator.
  - The validator refuses unknown channels, negative numbers and undeclared counters.
  - History is never rewritten. A misread is flagged, not deleted.
- **Site analytics:** aggregates only (daily series, true unique visitors per range, events) plus a per-code breakdown by `utm_content`, within a fixed call budget per refresh.

## Collection: official API first, then a script, then an agent

An agent that browses every post costs about 15–20 minutes per channel per run and often times out. A script costs seconds. An official API costs one or two seconds and needs no browser. Move every channel down this ladder as soon as you can.

1. **Official API where one exists** (for example the YouTube Data API or the Threads API). The owner creates the OAuth client and gives consent once; the team never sees the secret (see engineering-contracts.md).
2. **Record routes.** For each remaining channel, an agent does one normal collection while writing down every URL, locator, label and gotcha (`routes/<channel>.md` and `.json`).
3. **Script it.** Engineering turns the route into a deterministic script on the owner's signed-in browser profile, with no LLM.
   - **One aggregate page per channel** (a stats list, an activity feed), not one page per post. Prove the signed-in identity once per run, not on every page.
   - Use per-post pages only where no aggregate shows exact counts.
4. **Launch.** One hourly launcher per channel, offset by a minute (:30, :31, :32 …). If the platform does not let a graph step drive the browser, the launcher is a cron agent on a cheap model with a thin body:
   - run the script exactly once;
   - if it committed, report one line and stop;
   - only when the script reports a typed parse-change failure with a verified, still-reserved slot does the same agent finish the collection by the channel guide, with the SAME reservation.
5. **Health.** An hourly read-only health check compares each channel's last receipt-verified commit with its schedule: ok, late, down or paused. It shows on the dashboard. A cron's own "Success" only means it delivered a report, never that it saved data.
6. **Cadence:** hourly for social, with extra early points at about 15, 30, 60 and 180 minutes after a post goes out; daily for search data, which lags 2–3 days.

### Collector rules (scripts and agents)
- **Read-only.** Never like, reply, follow, export or download.
- **One tab per job, closed at the end,** even after a failure. Check the browser's status afterwards; a close that was sent is not a close that happened.
- **Exact counts only.** Read numbers from the post's own page or analytics. If a header abbreviates ("3.2K", "1,7 тыс."), find the exact value elsewhere (an insights or analytics view) or leave the counter out. Never expand an abbreviation.
- **Omit, never wrong.** When a count cannot be tied to its own control (an icon-only button, a value that could belong to the next post or a reply), leave it out. A missing value is honest; a value from the wrong post poisons every chart.
- **New posts first:** take every post from the last 14 days from the activity feed, then fill up from the top lists. Top lists miss fresh posts. **Exception: long-tail channels** (video) take the newest N owned uploads with no age cutoff.
- **Never write a number you did not read.** Leave out optional counters you could not read.
- **One reservation per slot.** Begin once, save its stdout to `reservation.json` the moment it returns, and commit with that file. Never retype a key or an attempt id: a model swapping two characters loses the slot. A `--help` call is not a reservation.
- **Pass timestamps through unchanged.** The builder's `at` goes to the commit byte for byte. In PowerShell, `ConvertFrom-Json` turns it into a local DateTime unless you add `-DateKind String`.
- **Close stdin** for every CLI the collector runs (`< /dev/null`). A child that inherits an open, never-closing input can hang forever.
- **Write a progress trace** (stage, elapsed time, post keys) before each navigation, build and commit, so a hard timeout still shows where it stopped.
- **Absolute paths for scratch files.** Never rely on `$TMP`-style variables.
- **Report one line per channel.** The validated file is the truth, not the agent's own arithmetic.
- **History backfill** is a one-off per channel: it reads up to N old posts and registers them in the catalog.
- **Recipes** that an agent writes for itself between runs hold steps only, never numbers. Quarantine a recipe that contradicts the guide; the guide wins.

## Publish instant from post ids

Where the platform encodes time in the id, derive it instead of trusting a displayed relative date:
- **X:** `(BigInt(id) >> 22n) + 1288834974657` ms.
- **LinkedIn activity:** `BigInt(id) >> 22n` ms.
- **Threads and Instagram shortcode:** decode the base64url alphabet `A–Z a–z 0–9 - _` to a BigInt, then `(id >> 23n) + 1314220021721` ms.
- **Others:** use the platform's shown date and mark it as shown.

## Metric definitions

- **Per post key**, keep the post's own observations over time. A snapshot that lacks a post says nothing about it; carry its last value.
- **Views gained in a period** = the sum over posts of (latest in the period − first in the period), counting only posts with two or more observations. A post's first observation contributes 0, so backfills never create jumps.
- **Views over the last 7 days** = the sum of the latest views of posts whose publish instant falls within 7 days.
- **Thread parts and self-replies** are children of their root post. They are the same readers, so channel reach counts roots, and the parts give read-through (the last part's views ÷ the root's views).
- **Followers** are per snapshot.
- **Never sum unique visitors** across days, ranges, codes or channels.
- **Channel attribution from a code is catalog-derived.** Label it as such.
- **Baselines:** each channel's median views per post and its hit line, from the history backfill. Judge new posts against their own channel's baseline.
- **Never mix sources in one series.** An API count and a browser count for the same video are different measurements (the public API often lags the studio by a view or two). Key every series by source, definition and precision; a new source starts a new series. Otherwise 24 then 23 shows as a fake drop.
- **Rounded values stay rounded.** For example, an API that rounds subscribers above 1,000 to three significant figures: show them as approximate and never compute an exact-looking delta.

## Dashboard rules

- **No bare numbers.** Every number has a compact delta against a named baseline: `158 ▲ 4 · 2.6%`. The baseline wording appears on hover only.
- **Without a baseline, hide the delta.** Unknown values stay hidden. No walls of provenance text.
- **One home per metric.** No duplicate boxes. One y-axis per chart. Use a real chart library, not hand-rolled SVG.
- **Every table is paginated** (about 20 rows), keeping sort and filters across pages.
- **Before claiming a fix, recompute the shown numbers from the raw file** with an independent script.

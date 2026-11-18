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

## Collection: scripted first, agent fallback

1. **Record routes.** For each channel, an agent does one normal collection while writing down every URL, locator, label and gotcha (`routes/<channel>.md` and `.json`).
2. **Script it.** Engineering turns the route into a deterministic script on a dedicated browser profile, with no LLM.
3. **Schedule.** Run one scheduler or graph that fans out per channel in parallel. If a script fails, returns incomplete data or hits a login wall, start **one cheap-model agent for that channel only**. The agent collects through the owner's signed-in browser with the channel guide and reports what changed in the route.
4. **Refresh on demand.** A "refresh everything now" command runs the API sources immediately (guarded against floods, for example 10 minutes) and triggers every channel job.
5. **Cadence:** about 4 times a day for social, matching the posting rhythm, and daily for search data, which lags 2–3 days.

### Collector rules (scripts and agents)
- **Read-only.** Never like, reply, follow, export or download.
- **One tab per job, closed at the end,** even after a failure.
- **Read numbers from the post's own page or analytics.** List and overview cards often count a different window.
- **New posts first:** take every post from the last 14 days from the activity feed, then fill up from the top lists. Top lists miss fresh posts.
- **Never write a number you did not read.** Leave out optional counters you could not read.
- **Absolute paths for scratch files.** Never rely on `$TMP`-style variables.
- **Report one line per channel.** The validated file is the truth, not the agent's own arithmetic.
- **History backfill** is a one-off per channel: it reads up to N old posts and registers them in the catalog.

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

## Dashboard rules

- **No bare numbers.** Every number has a compact delta against a named baseline: `158 ▲ 4 · 2.6%`. The baseline wording appears on hover only.
- **Without a baseline, hide the delta.** Unknown values stay hidden. No walls of provenance text.
- **One home per metric.** No duplicate boxes. One y-axis per chart. Use a real chart library, not hand-rolled SVG.
- **Every table is paginated** (about 20 rows), keeping sort and filters across pages.
- **Before claiming a fix, recompute the shown numbers from the raw file** with an independent script.

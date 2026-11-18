# Local catalog and dashboard: engineering blueprint

Build a local app for the current product from the available project material. A new team must work with an empty folder: accept “start” without requiring a setup form or extra team-start fields. Discover existing files and configured sources first. Build the planning workspace even when no account or analytics provider is connected. Add each available source independently; a missing network must not block the others. Ask product questions in the conversation only when their answers are needed.

This blueprint describes what to build; it ships no precollected history. Reuse the product's stack and design system where practical, otherwise the calm dark default. Serve locally with one development server.

## Files, identities and writers

Use project-relative references and a registry-driven model. Keep credentials in the runtime's private credential store, outside exported files and evidence.

| Data | Contents | Authoritative writer |
|---|---|---|
| `catalog.jsonl` | Post, asset and campaign records; post status, file reference, permanent code, optional `chosen` and `rejected` | Explicit registration allocator for new coded posts; guarded updater for edits |
| `post-codes.json` | Code assignments and a monotonically increasing high-water mark | Registration allocator only, with locking and a receipt |
| `channels.json` | Available channels, collection descriptors, identity rules, metrics, compose rules and schedule policy | Producer-controlled registry migration |
| `metrics.jsonl` | Append-only timestamped social observations | Validated social append boundary; maintenance uses the same archive lock |
| `site-aggregates.jsonl` | Bounded site aggregates and per-code outcomes, with windows, timezone and coverage | Dedicated maintenance collector |
| Markdown and media | Canonical post bodies, variant files and registered assets | Assigned content writer; explicit chosen-text synchronization |
| Private operational sidecars | Locks, attempt ledger, receipts and derived media index | Their owning storage commands; never a UI read |

Allocate permanent codes once; never reuse them or infer an assignment from a counter alone. Registration compares the expected catalog revision, validates allocator state, and preserves a durable association. Editing publication metadata does not allocate another code. Imports and reads never register posts.

Snapshots retain observation time, append time, source, channel and counters actually read. Validate nonnegative safe integers and declared fields. Unknown is absent, not zero. Label scopes, baselines and catalog-derived attribution; never sum unique visitors. Preserve published text and established links. See `metrics-pipeline.md` for metric definitions.

## Collection and durable appends

Record one successful read-only route for each available source, then implement a deterministic script or API adapter. Run the script first. Use one bounded agent fallback only for the failing channel when access permits it. At a login wall, stop and request the owner's action; do not invent observations or loop through authentication. Keep session material out of logs; bound requests, pages and duration per adapter.

The registry supplies schedule timezone, logical slots and a start window. Use the user's timezone when known; otherwise record a neutral default explicitly. Stagger job starts to reduce contention, but rely on locking for correctness. Keep one scheduler; never replay a missed slot implicitly.

Implement this two-stage social boundary before scheduling appends:

1. **Begin before collecting.** Supply the registered job and scheduled, manual or backfill mode. Scheduled identity is channel plus logical slot; admit it only inside the start window. Manual/backfill requires an explicit request identity and purpose. Capture the registry's exact-byte revision. Only `reserved` permits collection; retain its key and attempt.
2. **Commit once.** Supply key, attempt and original UTC observation time; pass the frozen snapshot bytes through standard input. Only `committed` is success. Equal values at different valid observation times remain separate observations.

Both operations acquire the attempt-state lock, then the shared metrics archive lock. Use one monotonic **10-second acquisition budget across both locks**, with asynchronous backoff. Retry only contention before that lock's callback starts. Never repeat an entered callback, append, initialization failure or cleanup failure. Never take over or delete another writer's lock. Long maintenance holds return typed `busy` after the bounded wait.

After acquiring both locks, keep the separate **15-second cooperative critical deadline**. No collection runs under these locks. Validate the bounded input and history, recheck the registry, capture the archive prefix, durably prepare a receipt, append with a complete write loop, fsync, verify prefix and appended region, then atomically publish the committed receipt. Receipts include input digest, offset, length, region/prefix hashes, attempt and both timestamps. Filesystem operations can overrun checkpoints; these limits are not hard wall-clock guarantees.

A repeated committed identity with identical payload returns its receipt without appending. Conflicting payload refuses. Reconciliation recognizes only an exact proved region; uncertain or torn outcomes require attention, never truncation or automatic reappend. `busy`, `refused`, `attention` and capacity exhaustion stop the collector. Retain dedup identities: a bounded ledger of 4,096 identities fails closed when full; plan its migration before exhaustion. Do not enable competing legacy refresh writers.

## Variant files and guarded decisions

Canonical text lives in Markdown. Thread parts use standalone `---` separators; `## Reply` starts the first reply. Variants live beside it as `<stem>.variants/<A-Z>.md`: up to **26**, with a first line `angle: <label>` of at most 60 code points, a blank line, then the body. Variants have no post codes.

Discover existing keys in letter order. Enforce containment, regular-file checks, no symlink/hardlink traversal, strict UTF-8, **64 KiB per file**, **2 MiB shared raw-read budget per complete assembly**, and **256 directory entries**. Exhaustion produces honest unavailable items; it does not raise the cap or fail the whole GET. Ignore/count non-allowlisted names. An unknown ignored count remains `null`.

Use one locked, same-origin guarded updater with expected catalog revision:

- Choose: `{id, version, patch:{chosen:"C"}, variantRevision}`.
- Clear: `{id, version, patch:{chosen:null}}`.
- Reject: `{id, version, patch:{reject:"C"}, variantRevision}`.
- Restore: `{id, version, patch:{restore:"C"}}`.

Each action is isolated from publication edits and other actions. The server validates the current readable preview and rechecks before rename. Persist `chosen` or a server-derived `rejected` map of key to SHA-256 revision; never persist command fields or accept a client-authored map. Preserve unrelated catalog bytes and every text file. Rejecting the chosen key requires clearing it first. Choosing a matching rejected revision refuses. Published decisions are frozen.

Keep all items in the read model, with revision and readable/error state. Add `rejected` and `revised` booleans: an exact stored revision match hides an unpublished candidate; a changed readable revision returns with Revised. Unreadable items never disappear. Restore removes the stored entry without deleting files. Malformed maps hide nothing and block variant mutations. Return typed conflict, changed-preview, frozen, busy and attention results; never replay uncertain writes. Choosing changes only metadata: explicit writer synchronization updates canonical text later. Until then, copy the selected candidate and show “awaiting sync”.

## Read API, watching and the first useful day

`GET /api/data` assembles bounded catalog, registry, media, variants and observations. It makes no network calls, allocations of permanent codes, appends or persisted cache writes. A cold media read may build an index in memory; explicit scan persists it. Warm reads honor the saved index. Watch admitted files and shallow variant directories, including parents of missing folders; debounce notifications and reread. Reject late GET results that predate any successful mutation, preserve drafts and focus, and never automatically replay a write.

Build the first day in this order:

1. Initialize empty stores explicitly, then deliver usable Posts, Planned and Dashboard empty states. Show existing real material immediately; leave missing metrics absent. If there are no facts or channels yet, keep planning usable and gather the next useful fact in conversation.
2. Register one real draft once a channel is known. Add a few factual variants, exact copying, guarded choice and explicit canonical synchronization. No automatic publishing.
3. Prove reject/restore and storage guards on disposable fixtures. Add one available collector through begin/commit, then compare its receipt and raw observation with the displayed reading. Schedule only after that pilot works.
4. Add remaining sources, histories and richer comparisons incrementally. Verify bounds, race/crash outcomes and desktop/phone behavior before each landing. Report fixture proof, live collection proof and remaining unknowns separately.

# Engineering contracts

How engineering work runs in the media team so that data stays trustworthy.

## Ownership and slices

- Turn a request into a **bounded slice**: a goal, the declared files, a concrete observable oracle and the exact focused test command.
- **Prefer a small runnable pilot over a grand plan.** Use a Planner only for coupled multi-surface waves.
- **One writer per surface.** One implementer owns the UI, one data engineer owns the collector and storage slice, and their paths never overlap.
- A worker's ACK starts exclusive ownership. **Never read or steer a healthy worker's partial work;** judge the pushed report.
- **Close workers as soon as their slice is accepted,** after copying their evidence into the project.

## Evidence

- Completion means the requested behaviour **plus runtime evidence**. A fixture pass, a graph check or a "submitted" line are different, weaker claims. Say which one you have.
- **Report what passed,** at which source revision or hash, and what remains unproved.
- **Use an independent read-only reviewer** on risky changes. Reviewers inspect proof; they don't rerun everything.
- **Run the finished wave's tests and build once,** then check bounded fixes with focused tests.

## Data integrity

- **Unknown is not zero.** Unread fields stay absent.
- **Append-only history.** Never rewrite observations, reassign retired codes or replay an uncertain append.
- **Keep original timestamps.** Backfill time is not publication time.
- **Never sum unique visitors** or incompatible observation scopes.
- **Atomic writes** go to a temp file and then rename. A collector removes leftovers from a dead run on its next start; it never removes a live lock.
- **Changing storage** (for example JSONL to SQLite) needs parity checks, a backup, a rollback and one authoritative writer path. Post texts can stay as Markdown files.

## Schedules and graphs

- Keep **schedulers inside the team's orchestrator** (crons, graphs) rather than OS tasks, so the team can see them.
- **A failed command step can end a scheduled graph.** Make the step exit 0 and carry `ok` and `state` in its JSON result, so one network error doesn't kill the schedule.
- **Interval schedulers don't land on exact minutes.** Give slot-based jobs a window (for example 15 minutes) and dedupe by slot id.
- **Don't flood the owner's or the lead's inbox with routine results.** Report real runs and failures only.
- **Never turn a platform's authority rejection into an obfuscation workaround.** Report it to the platform's owner with a reproducible note.
- **Never pass "download and run" command text on a shell command line.** Antivirus flags it. Use script files.

## Secrets

- Credentials live in a gitignored, permission-restricted `.env` or the platform's secret store. They are never printed, logged or put in URLs.
- Tokens get the narrowest scope. A token that is no longer needed is deleted.

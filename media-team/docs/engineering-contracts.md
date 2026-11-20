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
- **Two reviewers on data-path changes:** the Engineering lead gives a GO or NO-GO with the exact pilot command and a FRESH request id, and an independent reviewer gives PASS or FAIL. Both review the delta since their last verdict, not the whole history.
- **A manual pilot is not a cutover.** Switch a channel to a new collector only after two consecutive natural scheduled slots committed by the new path, each independently checked (receipt, archive prefix and region hashes, cleanup closed). Fallback commits don't count.
- **Never retry a failed pilot under the same request id.** Read its evidence, fix the cause, and use a new id.

## Testing that catches real failures

Fourteen review rounds on one collector taught these:
- **Test against the real boundary, not a mock of it.** A test that fabricates the writer's response shape passed while the live run misread a valid reservation as a refusal. Add an integration test that calls the real begin and commit on a temp copy.
- **Run the live dry-run yourself before reporting.** A scraper passes every fixture and finds zero posts on the real page. A read-only dry-run (no begin, no commit) on the real signed-in page is allowed and required. Fixtures come from real captures, never from imagined page shapes.
- **Browser automation facts:**
  - accessibility snapshots often have no `href`; prove links with an exact CSS selector read;
  - snapshot scopes may accept only `role:name`, not CSS;
  - nested `:has()` and `:nth-match` are invalid CSS;
  - innerText drops icon-only controls, so positions shift;
  - the UI may be in the owner's language; never wait on an English label.
- **Inject clocks; never assert on wall time.** A test that passes on an idle machine and fails under load is a broken test. Never "fix" a flake by raising a tiny timeout until it passes; give the real operation a real budget and inject a small one in the test.
- **Tests must not depend on the live registry or data.** Strip live entries from the fixture before adding test ones. Run the full suite after every registry change.
- **Keep the failing run.** The suite writes its full TAP to a file; copy it before rerunning, and report the `not ok` block even when the rerun passes. A flake that nobody saved can't be fixed.
- **A fresh worktree has no dependencies.** Run `npm ci` first; a pile of `ERR_MODULE_NOT_FOUND` is not a test failure.

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
- **Interval schedulers don't land on exact minutes.** Give slot-based jobs a window (for example 30 minutes) and dedupe by slot id.
- **Check the next run after every failed run.** Some schedulers push the next start by a random backoff after a failure, which silently skips slots. Re-apply the schedule if it drifted, and report it to the platform owner.
- **Edit the registry between slots,** never while a collection is in flight: the writer refuses commits from reservations made before the registry changed.
- **Diagnostics on every exit path.** A timeout must say which phase took the time and how much budget was left. A bare "total-timeout" sent the team down the wrong path twice.
- **Don't flood the owner's or the lead's inbox with routine results.** Report real runs and failures only.
- **Never turn a platform's authority rejection into an obfuscation workaround.** Report it to the platform's owner with a reproducible note.
- **Never pass "download and run" command text on a shell command line.** Antivirus flags it. Use script files.

## Secrets

- Credentials live in a gitignored, permission-restricted `.env` or the platform's secret store. They are never printed, logged or put in URLs.
- Tokens get the narrowest scope. A token that is no longer needed is deleted.
- **Agents never see secret values.** The owner downloads the OAuth client file; one command pipes its fields into the secret store through stdin, without echoing them, and deletes the file. Agents may run that pipe; they never read, print or copy a value.
- **A vault can be slow.** A remote password-manager backend took about 4 s per read and about 14 s per save. Read each credential once per run under one lock; give the single save its own bounded window plus one exact readback; reserve cleanup time inside the hard kill.
- **An unconfirmed save is attention, not failure.** If the save's outcome is unknown, keep a durable "write pending" marker, make no further writes, and never let that state trigger a fallback that would exchange the token again.

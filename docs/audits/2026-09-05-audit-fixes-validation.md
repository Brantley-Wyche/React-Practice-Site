# Season 1 audit fixes — validation

All seven findings from the Season 1 code audit have been addressed in the main checkout. File restructuring remains deferred. No commit or publication was performed.

## Changes

1. Installed React 19 TypeScript declarations as development dependencies and added `typecheck:lessons`. The starting cartridge now produces a clean compiler run; its intentional runtime bugs remain.
2. Check runs now cancel on unmount and reset. An abandoned run cannot record telemetry or completion; unexpected failures release the busy state.
3. Progress uses additive completion records and reset generations. Tabs synchronize completion without overwriting one another, while stale writes cannot resurrect reset progress. Existing legacy completion remains readable.
4. Persistence failures return explicit results. State updaters stay pure. Unsaved completion is labelled as belonging to this visit, with an actionable retry. Reset failures retry reset, including after a cross-tab update. A committed reset remains successful even if subsequent reads become unavailable.
5. Previews and individual checks now own disposable same-origin frames. Frame removal disposes timers and module state; checks have a 15-second asynchronous timeout. Restart preview reloads the exercise. Vite builds both application and exercise entry points.
6. Learning telemetry validates record shapes, counters, timestamps, and hint tiers before updating. Invalid telemetry remains nonfatal.
7. Corrected the inaccurate starting descriptions in lessons 14 and 15 and the inaccurate conceptual explanation in lesson 14. Exercise implementations and executable checks were preserved.

Also promoted curriculum validation to a supported command, corrected README setup/reset/testing claims, removed its obsolete screenshot embed, and updated design documentation. Compatible transitive development-tool updates cleared the vulnerabilities reported during dependency installation; npm reported zero vulnerabilities afterward. No production dependency was added.

## Verification

| Command | Exit | Result |
|---|---|---|
| `npm.cmd test` | 0 | 14/14 shell tests passed |
| `npm.cmd run typecheck:lessons` | 0 | No TypeScript diagnostics |
| `npm.cmd run validate:levels` | 0 | 15 lessons, 43 checks, 45 encoded hints validated without decoding |
| `npm.cmd run build` | 0 | Both production entry points built |
| `git diff --check` | 0 | No whitespace errors; Windows line-ending notices only |
| `node C:/Users/d69ha/.agents/skills/impeccable/scripts/detect.mjs --json src` | 0 | Empty findings list; existing narrow exercise-style exceptions retained |
| `node docs/superpowers/verify-learning-boundary.mjs` | 0 | Exercise code, checks, encoded hints, harness, and exercise CSS match HEAD after line-ending normalization; approved narrative differences only; design sidecar parses |

Regression tests reproduced stale completion writes, storage exceptions, malformed telemetry, and the committed-reset/read-failure defect before their fixes. Frame unit tests cover successful disposal, cancellation, and timeout. They use a DOM stub; the browser checks below supply integration evidence.

Vite's config loader was initially blocked by filesystem sandbox access, and npm's cache was blocked during installation/update. Authorized retries succeeded. These were environment failures, not application defects. The final validation table records completed commands, not attempted starts. No lint script is currently declared.

## Browser evidence

Used the actual app plus an ignored, disposable fixture in `docs/superpowers/reliability-browser.*`. The fixture overrides storage only in its own document with an in-memory implementation. It never reads, clears, or writes the learner's real progress.

- Actual Level 1 still shows its expected planted crash inside a titled preview frame.
- The Level 5 preview's working control changed its counter from 0 to 1; Restart preview returned it to 0.
- A real baseline check ran successfully through the frame entry point and left zero check frames behind.
- An observer in the fixture removed ChecksRunner while its check frame was active. Afterward, completion callbacks and remaining check frames were both zero. An initial manual two-click probe completed before removal; it was replaced with this deterministic active-run probe.
- A blocked reset retained completion and showed Retry reset. A same-generation storage event preserved that action. Restoring test storage and retrying cleared completion and the error.
- A separate save-failure fixture temporarily limited its parent registry to one existing passing baseline check, without changing source files or the child runtime. Completion showed Completed this visit and explicitly said it was unsaved. Restoring storage and retrying changed it to Completion saved and removed the error.
- Desktop at 1440×1000: preview controls/error display fit; the desktop setup notice remained hidden; scroll width was 1425px.
- Mobile at 390×844: the setup notice was prominent, the preview resized to its content, and register/lesson scroll width was 375px with no horizontal overflow.
- Tablet at 820×1180: setup notice displayed; scroll width was 805px with no horizontal overflow. Tablet verification used DOM measurements; desktop/mobile included screenshots.
- Section navigation retained focus behavior. Temporary viewport overrides were reset.

These checks did not solve the curriculum or prove every planted check against corrected implementations. No physical-device or full screen-reader test was performed.

## Final reviews and limits

An independent source reviewer found the two reset-recovery issues and a synchronization issue involving the retry action. All were fixed; the final narrow re-review found no actionable blocker.

The final focused React Best Practices review covered state-updater purity, listener cleanup, cancellation, storage schema, sequential check execution, and bundle implications. Sequential checks intentionally preserve deterministic lesson behavior. No Next.js APIs, state framework, blanket memoization, or additional production libraries were introduced.

The final [Web Interface Guidelines review](https://raw.githubusercontent.com/vercel-labs/web-interface-guidelines/main/command.md) covered the changed shell UI and frame boundary: titled frames, hidden check frames, native retry controls, status announcements, focus styling, text wrapping, and responsive behavior. It found no additional applicable blocker. This was targeted implementation QA, not another formal Impeccable critique.

Frames provide lifecycle isolation for trusted local exercise code. They are not a security sandbox and cannot reliably interrupt synchronous infinite JavaScript on the same browser thread; closing or reloading a frozen tab may still be necessary. The README now documents that limit.

Progress-selector extraction, CSS separation, and test-folder moves remain for the user-requested file-structure stage.

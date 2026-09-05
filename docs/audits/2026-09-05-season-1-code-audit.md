# Season 1 code audit and final critique fixes

Scope: the current main checkout, after the notebook redesign. This audit separates application infrastructure from deliberate learning bugs. New findings below are recommendations, not changes silently applied to the curriculum or application.

## Approved fixes completed

- Completed incidents no longer promise restored check history. An empty completed-visit log explicitly says no checks have run during this visit.
- The four lesson section jumps and the register's Browse the incidents action use buttons. They preserve the route and move focus; incident navigation remains links.
- Reset confirmation accurately says later levels lock while Level 1 stays available.
- DESIGN.md and its sidecar reflect the changed controls and copy.
- Nine value-specific, file-scoped detector exceptions document the preserved exercise radius and colors. These account for the 13 previously reported exercise-style advisories. No rule or whole file was disabled. The exception reasons and values are reviewable in .impeccable/config.json.

## Coverage and method

Reviewed all 50 JS/JSX/TS/TSX source files: the shell, entry point, level registry, all 15 manifests, exercise components and supporting modules. Also reviewed CSS, progress tests, package/lockfile configuration, Vite and TypeScript configuration, README, and relevant project/design instructions. Encoded hint coverage was checked without decoding content. Encoded solutions were not opened or decoded. Season 2 and Season 3 were not audited.

The exercise source was inspected to distinguish curriculum behavior from shell defects. This was not a playthrough proving every lesson's private solution. No claim is made that every check was executed against both buggy and corrected implementations.

Used source review, fresh browser inspection, isolated runtime fixtures, in-memory storage probes, structural manifest checks, repository tests/build, and a TypeScript diagnostic run. The temporary probes are ignored local files in docs/superpowers/. They introduce no production dependency. The browser fixture records only a synthetic check-activity ID; it does not mark an actual lesson complete or reset user progress.

## Prioritized findings

### 1. P1 — React TypeScript declarations are missing

**Location:** package.json:15; tsconfig.json; TypeScript diagnostic output.

The compiler produced 93 diagnostics: 84 missing JSX intrinsic-element diagnostics, five missing React declaration diagnostics, and four implicit-any diagnostics. The first actionable error is TS7026, followed by TS7016 for react/jsx-runtime. Missing framework declarations obscure the feedback that the TypeScript lessons are meant to teach. Successful Vite transpilation does not establish working type checking.

**Recommendation:** add React-19-compatible @types/react and @types/react-dom as development dependencies, then reassess the remaining diagnostics. Preserve intentional lesson errors. Keep the runtime build separate from lesson type checking and offer a clearly named compiler command; do not make clearing all curriculum errors a prerequisite for starting the app. These packages were not installed during the audit.

**Evidence:** node node_modules/typescript/bin/tsc --noEmit --pretty false exited 2. [React's TypeScript setup](https://react.dev/learn/typescript) explicitly requires these declarations.

### 2. P2 — A check run outlives the lesson that started it

**Location:** src/shell/ChecksRunner.jsx:10–26; src/shell/App.jsx:45–50.

The asynchronous loop has no invalidation on unmount or reset. It can still call onAllPass after the learner leaves the lesson. In the application, this callback can save completion after a reset that the learner expected to clear it.

**Evidence:** an isolated browser fixture started the real ChecksRunner with a delayed synthetic success, immediately unmounted it and reset its completion counter, then observed Completion callbacks: 1. No actual lesson completion was changed.

**Recommendation:** give a run an identity/cancellation signal, invalidate it on unmount and reset, and check it before starting another check or committing results, telemetry, and completion. Include finally-based busy-state cleanup for unexpected harness failures. Do not substitute parallel checks for lifecycle management.

### 3. P2 — A stale browser tab can overwrite newer progress

**Location:** src/shell/App.jsx:30–40; src/shell/progress.js:20–21.

Each tab reads completion once. Saving replaces the full stored set from that tab's state. A slower tab can erase completions saved by another tab.

**Evidence:** in-memory probe: two tabs read an empty set; A saved two completions; B saved its stale one-completion set; reloading storage returned only one. The probe executed the real persistence helpers.

**Recommendation:** define synchronization and reset semantics, listen for storage changes, and merge newly earned completion with current stored progress at the save boundary. Test stale updates and reset separately; a naive always-union operation would resurrect intentionally reset progress. [MDN documents cross-document storage events](https://developer.mozilla.org/en-US/docs/Web/API/Window/storage_event).

### 4. P2 — Progress writes and reset errors are uncaught

**Location:** src/shell/progress.js:20–25; src/shell/App.jsx:35–41, 45–50.

Reads recover from unavailable storage, while writes and removal throw. Saving also happens inside a React state updater. Restricted storage or a write failure can interrupt the completion path without an actionable message. The only application error boundary surrounds the exercise preview, not App's persistence work.

**Evidence:** injected in-memory storage throwing storage blocked was handled by loadCompleted, but escaped both saveCompleted and clearProgress. Full-app failure rendering was not simulated.

**Recommendation:** keep state updaters pure, handle storage results at an explicit persistence boundary, and distinguish in-memory completion from successfully saved completion. Report a recoverable save/reset failure instead of implying success. [React requires updater functions to be pure](https://react.dev/reference/react/useState).

### 5. P2 architectural limit — DOM teardown does not isolate faulty exercise resources

**Location:** src/shell/harness.jsx:99–120; preview boundary in src/shell/LevelPage.jsx.

Unmounting a React root and removing its host does not stop a timer that the component failed to clean up. That is especially relevant to a product whose components are intentionally faulty. Such work can survive a check or preview lifetime; module-level state also shares the page's JavaScript realm.

**Evidence:** a synthetic non-curriculum component created a timer without cleanup. After runCheck returned and unmounted it, the counter advanced six more times. The audit fixture explicitly cleared its timer afterward.

**Recommendation:** decide the isolation boundary before implementing a larger change. A disposable frame can isolate exercise execution, but it adds coordination work. For Season 1, document the limitation and full-refresh recovery while prioritizing cancellation and bounded asynchronous checks. Do not fix the planted cleanup bugs, globally monkey-patch timers, or claim a timeout can interrupt synchronous infinite JavaScript on the same thread.

### 6. P3 — Learning telemetry accepts malformed versioned stores

**Location:** src/shell/learning.js:7–10, 16–30, 36–49.

The loader checks version and truthiness of levels, but not the container or record shapes. An array passes that check; assigning string-keyed activity to it disappears on JSON serialization. Incorrect counter and hint types can also corrupt updates or trigger the catch that silently drops them.

**Evidence:** the real recordCheckRun accepted a version-1 store whose levels was an array and saved it as the same empty array. This used only an in-memory Node storage stub.

**Recommendation:** normalize the levels container, finite nonnegative counters, and hint-tier arrays on load. Keep telemetry failure nonfatal. Do this before making the currently unreferenced createLearningProfile export part of a visible feature.

### 7. P2 curriculum documentation — Two starting descriptions do not match their implementations

**Locations:** src/levels/14-hook-line-sinker/manifest.js:12 onward; src/levels/15-final-boss/manifest.js:11 onward.

Source review found a mismatch between the described starting behavior and the actual starting implementation in each of these lessons. One conceptual explanation also describes a JavaScript failure mode incorrectly. These are content-accuracy findings, not requests to remove deliberate bugs.

**Recommendation:** review those descriptions separately and adjust only the inaccurate narrative unless a curriculum change is explicitly approved. Mechanism-level details and solutions are intentionally omitted from this report. These observations are source-backed; a fresh runtime playthrough of those locked lessons was not performed.

## Cleanup patterns

- Extract pure progression selectors into a small progression.js module. LevelMap currently imports isUnlocked from App, while App imports LevelMap. Breaking that cycle improves ownership and makes progression testable without loading the application tree.
- Keep one clear owner for progress persistence and error reporting. Do not grow a generic state framework for this small application.
- Extend Node tests around progress read/write/error/reset behavior and telemetry normalization. Add a small shell runtime regression set for cancellation and completion states. The existing two tests cover sanitization only.
- Promote structural curriculum validation into a supported command after choosing a maintainable loader. The audit probe validated 15 unique, ordered lessons, 43 named executable checks, existing file references, and 45 encoded hint entries.
- Distinguish compile/transpile, shell validation, and intentional lesson failures in scripts and documentation. A shell-scoped lint pass can be added later; applying blanket automatic fixes to src/levels would damage the product.
- README's claim that the shell is bug-free is too absolute. Describe it as infrastructure outside the learner's editing scope. The README screenshot also predates the notebook redesign.

## Optimization recommendations

The production JavaScript bundle is 267.12 kB / 86.57 kB gzip, and CSS is 16.12 kB / 4.16 kB gzip. The current registry eagerly includes all 15 lessons, but this is a small local desktop learning application. There is no measured reason to add lazy-loading infrastructure, a state library, list virtualization, or widespread memoization now.

Prioritize leaked work and stale asynchronous completion over micro-optimizing repeated searches through 15 entries. Preserve the intentionally expensive teaching examples. Revisit code splitting only if the lesson set grows or startup measurements justify it; React/Vite APIs would apply, not Next.js APIs.

## Suggested file-structure pass

Keep the existing one-folder-per-lesson structure and the small shell mostly flat. A restrained third-stage change would:

1. Extract progression.js from App.
2. Split the shell styles and preserved .lv-* exercise styles into separately named CSS files, keeping import order and exercise rules unchanged.
3. Move test files from scripts/ to tests/ when expanding them; reserve scripts/ for runnable maintenance commands.
4. Keep audit reports under docs/audits/ and design experiments separate from production assets.

These are recommendations for the requested structure stage, not moves performed during this audit.

## Validation and limitations

| Command / check | Exit | Outcome |
| --- | --- | --- |
| npm.cmd test | 0 | 2/2 tests passed |
| npm.cmd run build | 0 | Production build completed |
| git diff --check | 0 | No whitespace errors; standard LF/CRLF notices |
| node node_modules/typescript/bin/tsc --noEmit --pretty false | 2 | Tooling issue confirmed: 93 diagnostics, including missing React declarations |
| node docs/superpowers/code-audit-probes.mjs | 0 after permitted retry | Manifest contracts passed; storage findings reproduced |
| Impeccable detect.mjs --json src | 0 | No unignored findings; nine explicit exercise-style value exceptions apply |
| Learning-boundary comparison against HEAD | 0 | Exercise source/CSS, harness, persistence and dependency files unchanged |
| Design sidecar parse | 0 | JSON valid |

The first isolated probe attempt exited 1 because esbuild could not read a parent directory under the sandbox. The permitted retry passed. Initial source reads also encountered Python's Windows output encoding limit; retrying in UTF-8 succeeded. These were environment failures, not application failures.

Fresh browser checks verified the corrected section buttons, route preservation, target focus, concept opening, register jump, and completed-visit empty copy. Desktop and 390px layouts were inspected; mobile scroll width was 375px within 390px. The two runtime probes used synthetic components. No full screen-reader or physical-device test was performed. Build and tests were not presented as proof that planted lesson bugs pass.

The focused final React/Web Interface Guidelines reviews found no new blocker introduced by the three approved fixes. [Guidelines reviewed](https://raw.githubusercontent.com/vercel-labs/web-interface-guidelines/main/command.md).

No commit, dependency installation, publication, file-structure migration, or new audit-finding fix was performed. The preview remains available for review.

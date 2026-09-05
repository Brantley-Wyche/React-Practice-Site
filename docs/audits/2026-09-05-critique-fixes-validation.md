# Season 1 critique fixes — validation

Implemented the three approved findings from the formal critique and the requested desktop setup notice. This is implementation validation, not another formal critique.

## Changes

- Saved completion uses historical language; verification results explicitly describe the last run, including failures after an earlier success.
- Earned continuation is available beside the verification log as well as near the page heading.
- Restart preview explains that it resets component state while preserving source files and saved progress. Crash recovery says Retry after editing.
- A prominent amber notice above the page explains the local editor, Vite recompilation, and desktop browser requirement. It appears at widths up to 1100px and on devices matching both coarse primary pointer and no hover. Lessons remain browsable.
- PRODUCT.md records the desktop-first requirement. DESIGN.md and its sidecar describe the implemented shell, typography, notice, and verification states.

## Verification

| Check | Exit | Outcome |
| --- | --- | --- |
| `npm.cmd test` | 0 | 2 tests passed |
| `npm.cmd run build` | 0 | Production build completed |
| `git diff --check` | 0 | No whitespace errors |
| Learning-boundary comparison against HEAD | 0 | Exercise source, shared exercise CSS, harness, persistence, and dependency manifests unchanged |
| Design documentation validation | 0 | Frontmatter, sidecar, shell font-size coverage, snippets, and narrative checked |
| Impeccable detector: `detect.mjs --json src` | 2 | 13 advisory findings, all in preserved shared exercise styles; no shell findings |

The remaining detector advisories are five 5px-radius declarations and eight color declarations in `.lv-*` teaching examples. These are outside the approved shell changes. No detector suppressions were added and the examples were preserved. The detector reports these as advisory severity but returns exit 2; this is not a clean detector exit.

## Browser evidence

Reviewed the live application at 320px, 390px, 768px, 1024px, and desktop width. The setup notice remains readable without horizontal overflow on narrow layouts and is hidden on the normal 1280px desktop viewport. The actual first exercise still contains its intentional crash.

An isolated, ignored QA fixture exercised the real shell components without completing actual lessons. A passing run displayed saved completion; a subsequent failing run displayed the failing last-run summary without claiming current success. Continuation remained beside the log. Restart preview reset an incremented component counter from 1 to 0. Running checks disabled the run control and exposed the busy state.

Focused React review found the changes use derived values, existing state, native links, and CSS media queries without new effects or dependencies. The focused Web Interface Guidelines review checked descriptive controls, the restart description association, result announcements, and responsive layout. No new applicable blocker was found in these changes.

Limits: responsive browser viewports are not physical-device testing; the wider touch-device media condition was inspected in source. No full screen-reader session was performed. Previously deferred minor critique observations remain outside this fix batch. No commit or publication was performed.

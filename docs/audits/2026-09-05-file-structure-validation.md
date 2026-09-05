# Season 1 file restructuring

Baseline: `768e2ab0340e76428247c72b488a6fd387c61ced`, committed and pushed to `origin/main` before restructuring. GitHub's branch SHA was checked against the local commit. This report records the restructuring verified against that baseline.

## Changes

- Extracted `isUnlocked(level, completed, levels)` into `src/shell/progression.js`. App and LevelMap pass their existing registry, removing the App/LevelMap import cycle without changing prerequisite behavior.
- Moved the three existing Node test files from `scripts/` to `tests/` unchanged. Added four focused progression tests and updated the npm test command. `scripts/` now holds the curriculum validator.
- Split the stylesheet into `src/styles/shell.css` and `src/styles/exercises.css`. `global.css` remains the common entry point, importing shell defaults before exercise styles. Both application and frame runtime imports remain valid.
- Moved the nine existing value-specific detector exception scopes to `exercises.css`. No rule, value, or justification was broadened.
- Added the README file guide and documented stylesheet ownership in DESIGN.md. Existing audit reports remain historical records. Lesson directories, local settings, critique working notes, and design experiment images were not reorganized.

## Verification

| Command | Exit | Result |
|---|---|---|
| `npm.cmd test` | 0 | 18/18 tests passed from `tests/` |
| `npm.cmd run typecheck:lessons` | 0 | No TypeScript diagnostics |
| `npm.cmd run validate:levels` | 0 | 15 lessons, 43 checks, 45 encoded hints |
| `npm.cmd run build` | 0 | Both entry points built; CSS remains 16.34 kB / 4.21 kB gzip |
| `node docs/superpowers/verify-restructure.mjs` | 0 | Built CSS byte-identical to baseline; split source CSS reassembles exactly after line-ending normalization; lesson/harness files and moved tests unchanged; exception scopes and JSON valid |
| `node C:/Users/d69ha/.agents/skills/impeccable/scripts/detect.mjs --json src` | 0 | No unignored findings |
| `git diff --check` | 0 | No whitespace errors |

The new progression test file initially failed with `ERR_MODULE_NOT_FOUND` before extraction, then passed with the helper available. This verifies the new import boundary and the four behavioral cases; it is not evidence of a newly repaired curriculum defect. Existing reliability tests were retained unchanged.

Desktop browser QA at 1440×1000 showed the notebook register with one open lesson and fourteen locked lessons for the existing empty progress state. Starting Level 1 and using Try the preview preserved route, title, focus, and the titled exercise frame. Mobile at 390×844 showed the desktop setup notice, wrapping lesson content, and a correctly sized frame; page scroll width was 375px. Desktop scroll width was 1425px. Screenshots and DOM measurements showed no horizontal overflow. Temporary viewport overrides were reset and the test tab closed. User progress was not changed.

The focused React Best Practices review found no added rendering work or dependency. The [Web Interface Guidelines review](https://raw.githubusercontent.com/vercel-labs/web-interface-guidelines/main/command.md) found no new applicable issue: the markup and CSS declarations are preserved. This behavior-preserving restructuring did not initiate another formal design critique.

An independent source reviewer checked the helper's three call sites, both CSS entry imports, moved test imports, npm script, detector scopes, and curriculum boundary. No actionable findings were reported. This review did not rerun the parent's verification commands.

No dependency was installed and no exercise implementation, hint, check, or harness behavior was changed during restructuring. No full curriculum solution playthrough, physical-device test, or full screen-reader test was performed.

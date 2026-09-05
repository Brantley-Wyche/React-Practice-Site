# Season 1 frontend audit

Date: September 4, 2026. Checkout: `C:/Users/d69ha/Desktop/react-practice-site`, branch `main`, HEAD `7aa07bf` (one commit ahead of `origin/main` at inspection).

Method: Impeccable technical audit, applicable React Best Practices review, current Web Interface Guidelines, and local browser inspection. This is a single-agent audit with visual-design observations, not a formal dual-agent Impeccable critique. No application code, planted bugs, lesson checks, or existing design concepts were changed.

## Verdict

**The interface has a coherent foundation, but its interaction details need work before visual polish.** The incident metaphor fits a debugging game. The largest problems are faint instructional text, invisible keyboard focus in hints, and a layout that separates the preview from the brief at the widths learners use beside an editor.

The visual identity is recognizable as an operational dashboard, but not yet strongly specific to Bugbound. That judgment comes from the composition and repeated UI treatments, not an assertion that particular code or visuals were AI-generated.

### Audit health

These are qualitative review scores, not certification or measured performance scores.

| Dimension | Score / 4 | Main evidence |
|---|---:|---|
| Accessibility | 2 | Semantic progress and live results exist; contrast and hint focus fail |
| Performance | 3 | Small dependency surface; all lessons eagerly included |
| Responsive design | 2 | Reflows without measured overflow, but the workspace becomes remote from the brief |
| Theming | 3 | Good core tokens; faint token is unsuitable for text and some content colors are literal values |
| Implementation integrity | 3 | Coherent incident language; workflow highlighting claims a state the app does not track |
| **Total** | **13 / 20** | **Significant improvements needed, without a wholesale rebuild** |

Nine prioritized findings: **0 P0, 2 P1, 5 P2, 2 P3**. P1 means major/accessibility issue; P2 means meaningful friction with a workaround; P3 means lower-priority refinement or opportunity.

## Scope and evidence limits

- Read all ten modules in `src/shell/`, shared CSS, the entry point, level registry, package/configuration files, README, and existing test script. Inspected metadata for all fifteen level manifests to understand titles, progression, and display contracts.
- Browser-tested the fresh map and Level 1 shell, its expected preview crash, check failures, map return, and keyboard hint focus. Hints stayed collapsed; no encoded solutions were opened.
- Sampled widths of 1440, 1024, 768, 390, and 320 CSS pixels. Screenshots informed desktop, editor-sized, mobile, and focus inspection. No elements extended beyond the content viewport in the measured 768/390/320 Level 1 samples or the 320 map sample.
- The completion banner, post-incident review, subsequent-level navigation, and later lesson variations were source-reviewed, not exercised through a completed playthrough. No claim is made that all fifteen live lessons or success paths have passed browser QA.
- Did not change saved learner completion, unlock levels, solve exercises, or inspect Season 2/Season 3. Running the expected Level 1 failures recorded an attempt only on the separate audit origin, `http://127.0.0.1:5187`.
- No NVDA/VoiceOver session, true browser zoom/text-only zoom, reduced-motion emulation, slow-device profiling, or cross-browser suite was run. Viewport reflow is not a substitute for those checks.
- The detector returned `[]`, exit 0, for `src/shell`. This is a narrow mechanical signal; it did not catch the verified contrast, focus, navigation, or workflow issues. No live detector overlay was injected.

## Prioritized findings

### F01 — P1: Faint instructional text fails normal-text contrast

**Locations:** [global.css:17](C:/Users/d69ha/Desktop/react-practice-site/src/styles/global.css:17), [global.css:625](C:/Users/d69ha/Desktop/react-practice-site/src/styles/global.css:625), [global.css:840](C:/Users/d69ha/Desktop/react-practice-site/src/styles/global.css:840).

Browser-computed colors give these ratios:

| Text | Foreground / background | Contrast |
|---|---|---:|
| Where to look | `#55657a` / `#0f141b` | 3.11:1 |
| Hint tier descriptions | `#55657a` / `#0d1117` | 3.18:1 |
| Learn / Reproduce / Repair labels | `#55657a` / `#0d1117` | 3.18:1 |
| Reset progress | `#55657a` / `#0a0d12` | 3.27:1 |

These are 11–11.5px text, below the 4.5:1 normal-text minimum. They communicate instructions and available actions, so treating them as decoration is inappropriate. [WCAG contrast guidance](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html).

**Recommendation:** Use an accessible secondary-text token for functional copy; reserve the faint value for non-text decoration. Validate actual surface combinations. Suggested workflow: `$impeccable colorize`, then `$impeccable harden`.

### F02 — P1: Hint buttons lose their visible keyboard focus

**Locations:** [global.css:76](C:/Users/d69ha/Desktop/react-practice-site/src/styles/global.css:76), [global.css:816](C:/Users/d69ha/Desktop/react-practice-site/src/styles/global.css:816).

The global focus rule draws a 2px outline with a 3px offset. `.hint-item` has `overflow: hidden`, and its button fills the inner area. Keyboard testing focused Hint 2, but its outline was outside the clipping boundary and invisible in the screenshot. Computed styles confirmed the outline and clipping combination.

**Impact:** Keyboard users cannot reliably identify which hint will open. This undermines deliberate spoiler control as well as navigation. [WCAG focus-visible guidance](https://www.w3.org/WAI/WCAG22/Understanding/focus-visible.html).

**Recommendation:** Draw the ring inside the button, style the container with `:focus-within`, or remove the clipping without losing corner treatment. Verify all three collapsed and expanded hints with Tab/Shift+Tab. Suggested workflow: `$impeccable harden`.

### F03 — P2: Page navigation lacks normal link and orientation behavior

**Locations:** [App.jsx:19](C:/Users/d69ha/Desktop/react-practice-site/src/shell/App.jsx:19), [App.jsx:60](C:/Users/d69ha/Desktop/react-practice-site/src/shell/App.jsx:60), [LevelMap.jsx:9](C:/Users/d69ha/Desktop/react-practice-site/src/shell/LevelMap.jsx:9), [LevelPage.jsx:17](C:/Users/d69ha/Desktop/react-practice-site/src/shell/LevelPage.jsx:17), [LevelPage.jsx:75](C:/Users/d69ha/Desktop/react-practice-site/src/shell/LevelPage.jsx:75).

Navigation uses buttons and `window.location.hash`. Entering a level by keyboard left focus on `BODY`; the document title remained the generic app title. There is no skip link. The fresh level's heading outline is H1 followed by H3 sections, with no H2.

**Impact:** Learners lose their keyboard position on navigation, cannot use normal link actions, and get weak orientation through page titles/headings.

**Recommendation:** Use hash anchors for navigable destinations, retain buttons for actions, focus the new main heading on route changes, update the title, add a skip link, and use H2 for peer level sections. No routing dependency is needed. These align with the current [Web Interface Guidelines](https://raw.githubusercontent.com/vercel-labs/web-interface-guidelines/main/command.md). Suggested workflow: `$impeccable harden`.

### F04 — P2: The workflow strip presents a false current step

**Location:** [LevelPage.jsx:28](C:/Users/d69ha/Desktop/react-practice-site/src/shell/LevelPage.jsx:28).

On first entry, step 4, Verify, is highlighted while Learn, Reproduce, and Repair are dim. The code assigns `current` whenever the level is incomplete; no learning-step state exists. Browser inspection confirmed this before checks ran.

**Impact:** A first-time learner may read the strip as an instruction to verify immediately, or assume the first three steps are already behind them.

**Recommendation:** Present a neutral process guide, optionally linking its items to page sections. Only use a current-step treatment if a meaningful state is actually tracked. Do not infer that the learner has understood or repaired something from scrolling. Suggested workflow: `$impeccable clarify`.

### F05 — P2: The stacked layout weakens the core debugging loop

**Locations:** [LevelPage.jsx:72](C:/Users/d69ha/Desktop/react-practice-site/src/shell/LevelPage.jsx:72), [global.css:522](C:/Users/d69ha/Desktop/react-practice-site/src/styles/global.css:522), [global.css:537](C:/Users/d69ha/Desktop/react-practice-site/src/styles/global.css:537).

At widths of 900px or less, the entire brief column—including hints—comes before the preview/checks column. Level 1's preview begins at approximately 1,089px down at 768px wide and 1,627px at 390px. Checks begin at approximately 1,429px and 2,064px respectively. Measurements used the untouched, collapsed-hint lesson with its expected preview error.

**Impact:** The supported editor-and-browser arrangement becomes a repeated scroll between symptom, file, preview, and evidence. This is an everyday desktop working width, not just a phone edge case.

**Recommendation:** Design explicitly for a 700–900px browser beside an editor. Keep the ticket near the preview and checks, make the concept easy to revisit, and put optional hints after the main loop or in a deliberate disclosure. Validate long lessons and long check output before choosing sticky behavior. Suggested workflow: `$impeccable layout`, then `$impeccable adapt`.

### F06 — P2: Locked curriculum is visually suppressed too strongly

**Locations:** [LevelMap.jsx:9](C:/Users/d69ha/Desktop/react-practice-site/src/shell/LevelMap.jsx:9), [global.css:390](C:/Users/d69ha/Desktop/react-practice-site/src/styles/global.css:390).

Fourteen of fifteen cards begin locked. Applying `opacity: 0.45` to the entire card dims the lesson title, concept, and status together. The desktop map becomes mostly faint, repeated boxes.

**Impact:** The curriculum is difficult to scan and feels unavailable rather than inviting. Learners should be able to understand the route ahead even while progression is gated.

**Recommendation:** Preserve readable titles and concepts; express locking with the explicit status and restrained control treatment. Consider a compact incident list with the next available item emphasized. Preserve the sequential unlock contract. This is a product/readability recommendation, not a blanket WCAG failure: inactive controls are exempt from the text contrast criterion.

Suggested workflow: `$impeccable layout`, then `$impeccable colorize`.

### F07 — P2: Small action targets make pointer and touch use unnecessarily precise

**Locations:** [global.css:204](C:/Users/d69ha/Desktop/react-practice-site/src/styles/global.css:204), [global.css:456](C:/Users/d69ha/Desktop/react-practice-site/src/styles/global.css:456), [global.css:655](C:/Users/d69ha/Desktop/react-practice-site/src/styles/global.css:655).

At 390px, measured heights were approximately 14px for Back to the map, 13px for Reset progress, 26px for Remount, 34px for hint toggles, and 35px for the primary check button.

**Recommendation:** Enlarge the hit area of small navigation/actions and use roughly 44px targets where space permits on touch layouts. Keep the visual density through internal padding and typography rather than tiny targets. Do not classify every sub-44px control as a WCAG AA failure: WCAG 2.2's minimum criterion uses 24px and allows spacing/equivalent-control exceptions. [Target-size guidance](https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html).

Suggested workflow: `$impeccable adapt`.

### F08 — P3: Hint guidance foregrounds implementation and can discourage help-seeking

**Location:** [HintBox.jsx:25](C:/Users/d69ha/Desktop/react-practice-site/src/shell/HintBox.jsx:25).

The explanation about encoding does not help the learner decide which hint to use. The claim that an attempt is worth more than all three hints combined can sound judgmental to someone returning to React.

**Recommendation:** Keep the deliberate tiers and collapsed defaults. Explain what each tier reveals and reassure the learner that help is optional. Example direction: "Start with a nudge. Reveal more only when you need it." Keep technical encoding details in contributor documentation. This is an editorial judgment, not a correctness defect. Suggested workflow: `$impeccable clarify`.

### F09 — P3: Every lesson ships in the initial JavaScript bundle

**Locations:** [levels/index.js:1](C:/Users/d69ha/Desktop/react-practice-site/src/levels/index.js:1), [HintBox.jsx:2](C:/Users/d69ha/Desktop/react-practice-site/src/shell/HintBox.jsx:2).

The registry statically imports all fifteen manifests, which bring their components/checks with them; the shared hint component imports the complete hint dataset. The production build emits one app JavaScript asset: 264.43kB, 85.95kB gzip. CSS is 18.19kB, 4.27kB gzip.

**Recommendation:** Treat per-level dynamic loading as a future growth optimization. Keep lightweight map metadata separate from executable lessons if bundle growth justifies it, and preserve HMR/check behavior with explicit loading/error states. This is an application of conditional loading to React/Vite, not a recommendation to add Next.js APIs. No slow runtime or render bottleneck was measured, and this size does not justify an urgent architecture rewrite. Suggested workflow: `$impeccable optimize`.

## Visual identity recommendation

Keep the incident premise and evolve it toward an **on-call field notebook**: precise operational details combined with readable learning material. That gives Season 1 a distinct, simple starting point which can mature across seasons without borrowing Season 3's agent features.

- Give the active incident the strongest space and typographic emphasis. Present future incidents as a legible docket rather than fifteen equally framed cards.
- Keep monospace for identifiers, paths, statuses, and check output; let lesson prose use the existing readable sans serif. The current mixed-font approach is worth retaining.
- Reduce repeated micro-labels and framing. The current map repeats uppercase metadata, blue concept chips, thin borders, dark cards, and status accents so uniformly that little feels specifically authored.
- Make distinction come from the interaction: ticket, reproduction, evidence, resolution. A consistent issue-number treatment and understated resolved mark would contribute more than adding glows, decorative animation, or more dashboard ornament.
- Keep error red meaningful and completion rewarding, while framing an intentionally failing exercise as expected practice. The app should help the learner feel curious rather than perpetually behind an emergency.

This is a direction for the next design conversation, not an approved redesign or a recommendation to make Season 1 visually elaborate.

## Good foundations to retain

- Clear primary start action and an understandable learn/reproduce/repair/verify loop.
- Native buttons for actual actions, semantic season progress with numeric values, text-based OPEN/LOCKED/RESOLVED and PASS/FAIL states, `aria-busy` and live check output.
- Encoded, opt-in hints and meaningful preview crash containment. Expected exercise errors do not blank the shell in the tested first-level flow.
- A restrained dark palette, useful shared tokens, system fonts, no remote font requirement, and only React/React DOM as runtime dependencies.
- Responsive breakpoints, wrapping file paths, no measured overflow in the sampled states, and reduced-motion CSS already exist.

## Applicability decisions and follow-ups

- Do not "fix" planted React bugs, enable StrictMode, change check ordering indiscriminately, or add memoization inside lessons as a performance cleanup. Those could alter the learning contract.
- Do not apply Next.js server components, server caches, `next/dynamic`, or hydration-specific advice to this client-rendered Vite app.
- A 15-item curriculum does not need virtualization. The evidence does not justify blanket `memo`/`useCallback`, a UI library, a font dependency, or a framework migration.
- Missing PRODUCT.md/DESIGN.md is a documentation gap, not proof that the UI lacks a design system. Capture the agreed design direction before a broad redesign; do not create fictional design requirements during this audit.
- Reduced motion currently uses a global duration override. Prefer targeted static variants when revisiting motion, but no loss of meaningful feedback was established here; this is not counted as a defect.
- Later source-only concerns for phase two: progress writes/removal lack the recovery handling used by learning telemetry; storage side effects appear in state updaters; the offscreen harness root is visually moved away without assistive-technology exclusion. These deserve targeted behavior tests before assigning final severity.
- Phase three should assess module boundaries and CSS organization after phase-two findings, including navigation helpers exported from App and the separation between shell styles and lesson styles. No file moves are proposed as established fixes yet.

## Validation and installation record

Commands ran from `C:/Users/d69ha/Desktop/react-practice-site`. The installer and detector paths below are under the already-installed local skills.

| Command | Exit | Result |
|---|---:|---|
| `python C:/Users/d69ha/.codex/skills/installing-frontend-workflow/scripts/install_frontend_workflow.py --project C:/Users/d69ha/Desktop/react-practice-site` | 0 | Preview inspected; new root managed block |
| Same installer with `--write` | 0 | Created AGENTS.md, untracked |
| Same installer with `--write`, repeated | 0 | Unchanged; no duplicate managed block |
| `npm.cmd test` | 0 | 2/2 existing progress-sanitization tests pass |
| `npm.cmd run build` | 0 | Production build succeeds; sizes recorded above |
| `node C:/Users/d69ha/.agents/skills/impeccable/scripts/detect.mjs --json src/shell` | 0 | Empty findings array |
| `npm.cmd run dev -- --host 127.0.0.1 --port 5187 --strictPort` | 1 initially | Environment restriction: `Cannot read directory "../..": Access is denied.` |
| Same dev command with approved escalation | 1 on intentional stop | Vite ready; actual application loaded and inspected in browser; Ctrl+C stopped the audit server |
| `git diff --check` | 0 | No tracked whitespace errors |
| Lint / typecheck | Not run | No repository scripts declared for either; build is not type-check evidence |

The Level 1 crash and three failed checks are expected exercise behavior, not validation regressions. The two passing Node tests do not establish broad shell or curriculum coverage.

Only `AGENTS.md` and this report were added. AGENTS.md includes the managed frontend workflow and a separate learning-project boundary preserving planted bugs, check contracts, and deliberate StrictMode behavior. The installer leaves that boundary intact. Future tasks inherit the instructions at session start; this audit applied them explicitly in the current task. Neither file was staged or committed; existing untracked work was left alone.

## Suggested order for the next pass

1. `$impeccable harden` / `$impeccable colorize`: repair hint focus, functional text contrast, and navigation orientation.
2. `$impeccable layout` / `$impeccable adapt`: improve the editor-width workspace and readable locked curriculum; enlarge small hit areas.
3. `$impeccable clarify`: make the workflow guide truthful and hint language supportive.
4. Agree the visual direction and document it before broad visual changes. Keep bundle splitting optional until measurement or growth warrants it.
5. `$impeccable polish`: final consistency pass after approved changes, followed by one focused re-audit.

These can be addressed individually or grouped after review. The broader code audit and file-structure work remain separate subsequent stages.

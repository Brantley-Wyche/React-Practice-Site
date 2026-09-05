# Product

<!-- impeccable:product-schema 1 -->

## Platform

web

## Users

People learning or returning to React, working in their own editor beside a browser. The owner's original goal is to re-learn React through debugging.

## Product Purpose

Bugbound Season 1 is a sequence of fifteen intentionally broken React exercises. Read a concept and bug report, reproduce the behavior, edit the local source, then run the checks to unlock the next level.

## Operating Context

React 19, Vite, JavaScript and a final TypeScript arc. This is a desktop-first app: learners edit actual project files in a local code editor, Vite recompiles the app, and checks run in a desktop browser. There is no embedded editor. Progress and hint/check telemetry are stored locally. Narrow desktop browser windows are a primary working context. Mobile and tablet layouts must prominently explain the desktop setup requirement while still allowing lesson browsing.

## Capabilities and Constraints

- Preserve lesson implementations, manifests, test IDs, checks, encoded hints, and encoded solutions.
- Preserve sequential unlocks, local completion, remount, check output, and optional hints.
- No StrictMode: the exercise harness intentionally measures render/effect counts.
- Keep Season 1 code simple: plain CSS, no Tailwind/SCSS design system, minimal libraries. Base UI may be added only for a demonstrated interaction need.
- Season 2 and the Season 3 branch are outside this work. Agent-generated exercises belong to Season 3.

## Brand Commitments

The owner approved an on-call field notebook direction: a readable incident register and continuous lesson document, simple enough for Season 1. Remove the generic card grid and colored side-border cards. Preserve the Bugbound name and the debugging/incident premise.

## Evidence on Hand

The live exercises and existing lesson text are authoritative. The September 4 frontend audit records browser evidence. The subsequent conversation approved the redesign and researched Field Notes, Linear, and Carbon as references. No claims of deployments, user counts, or testimonials are authorized.

## Product Principles

- Make the reproduce/edit/verify loop easy to revisit.
- Keep future learning visible while preserving unlock rules.
- Offer hints without pressure or accidental spoilers.
- Keep the shell reliable while preserving intentional exercise failures.
- Ask before running the formal post-redesign critique.

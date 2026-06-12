---
name: supercat-skills
description: Build, debug, optimize, preview, and package Tauri/Vue desktop pet apps
  driven by user-provided visual assets. Use for desktop companions, virtual cats,
  mascot pets, transparent-window apps, cursor-follow interactions, asset QA,
  browser-first previews, Tauri/Electron packaging, and desktop pet workflows where
  Codex should give asset-generation prompts for external tools such as Doubao or
  Jimeng, inspect supplied assets, then implement only after assets pass review.
---

# Supercat

## Core Workflow

1. Start with the product shape and asset plan. For a brand-new app, read
   [from-zero-build.md](references/from-zero-build.md) first.
2. Do not call image or video generation models yourself. Give the user reusable prompts and
   ask them to generate assets in external tools such as 豆包 or 即梦, then provide the files.
3. Inspect supplied assets before changing rendering code. Read
   [asset-generation.md](references/asset-generation.md) for prompt templates, export specs,
   and QA checks.
4. Build a browser-previewable version before desktop packaging. Start the dev server and give
   the user the local URL so they can experience the interaction in the web page first.
5. Only after the user confirms the browser preview, continue to the desktop runtime and package.
   Browser preview catches visual fit, cropping, state flow, and basic interaction issues; Tauri
   or Electron catches transparent-window behavior, drag, monitor bounds, and installer output.

## Process Gates

- Asset gate: do not implement final rendering until the provided media is checked for framing,
  transparency or chroma-key suitability, consistency, loop quality, file names, and resolution.
- Preview gate: do not package before a browser preview has run and the user has had a chance to
  confirm the product direction.
- Desktop gate: after browser confirmation, verify desktop-only behavior before building
  installers or source packages.

## Non-Negotiable Rules

- Treat transparent PNG or WebP frame sequences as final alpha assets. Draw them directly; do not
  run a second green-screen pass, alpha normalization, edge cleanup, or opacity fade over the
  active frame.
- Treat green-screen videos as chroma-key sources. Remove only confident green background pixels;
  for fuzzy edges, dampen green color instead of reducing alpha.
- Make watermark masks action-aware. Do not apply a fixed rectangular mask to frames where the
  subject overlaps that area.
- Keep cursor-follow release quick. Fade only the previous visual layer and keep the current
  follow frame fully opaque; long release fades create duplicate-subject ghosting.
- Keep settings independent of pet position. For desktop runtimes, resize and clamp the settings
  window to the current monitor work area while remembering the previous pet window position.
- Persist user-facing settings immediately through the app's settings store or localStorage layer.
  Do not rely on component defaults after first launch.
- Preserve project conventions: Vue `script`/`template`/`style` order, i18n text keys, semicolons
  in TypeScript, no `any`, two-space indentation, and concise Chinese comments/logs when needed.

## Common Task Map

- Need asset prompts, naming, or acceptance criteria: read `references/asset-generation.md`.
- Need a complete project from nothing: read `references/from-zero-build.md`.
- Need cursor following, drag, click, runtime bridge, transparency, or panel positioning: read
  `references/implementation-patterns.md`.
- Need reusable Vue/Tauri code for renderer, asset loading, settings, i18n, or runtime bridge:
  read `references/code-recipes.md`.
- Need to adapt an existing app: inspect its package scripts, asset folders, renderer structure,
  and i18n/logger conventions before editing.
- Need to fix visual artifacts: inspect the asset category first. Transparent frame artifacts
  usually come from unnecessary post-processing; green-screen video artifacts usually come from
  chroma-key thresholds, watermark masks, or source cropping.
- Need packaging: build and verify the browser version first, ask for user confirmation, then run
  the desktop build. On memory-constrained Windows builds, lower Cargo parallelism before retrying.

## Verification Checklist

- Supplied assets match the agreed subject, style, framing, transparency/chroma-key mode, and
  export format.
- Browser preview is running and accessible through a local URL.
- Browser preview shows no rectangular transparent/opaque blocks near the subject.
- Core interactions work in the browser preview where possible.
- User has confirmed the browser preview before desktop packaging begins.
- Desktop runtime handles click, right click, drag, follow/release, and settings panel placement.
- Switching modes does not show duplicate subjects or stale faded frames.
- Reloading the app preserves scale, enabled actions, ratios, and interaction preferences.
- Packaged artifacts launch without losing transparent-window interactions.

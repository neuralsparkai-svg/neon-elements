# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

**Atomica** ("Blast the elements to life") — a single-file, dependency-free HTML5 canvas arcade game that teaches the periodic table. The player blasts glowing element orbs; the game speaks each element, shows facts, and plays cinematic per-element "spotlight" reveals. Live at https://neuralsparkai-svg.github.io/neon-elements/ (GitHub Pages).

There is **no build system, no package manager, no test suite, and no lint config**. Everything is vanilla HTML/CSS/JS in one file. Do not look for `package.json`, `npm`, webpack, etc. — they don't exist and aren't needed.

## Critical layout fact (read before editing/deploying)

- **`neon-blaster.html`** (project root, ~2000 lines) is the **single source of truth** for the entire game — HTML, CSS, and JS inline. Edit this file.
- The project root is **not a git repository**. The deployable git repo is the **`site/`** subfolder (remote `neuralsparkai-svg/neon-elements`).
- **`site/index.html` is a copy of `neon-blaster.html`.** Changes to the game only reach the live site after you copy the source into `site/index.html` and push from `site/`. Forgetting this step is the most common mistake.
- `Images/` holds the original full-res source PNGs (named `Element — Symbol.png`). `site/img/el-<symbol>.jpg` holds the web-optimized versions the game actually loads. `img/` at the root is a copy of `site/img/` so the game works when opened locally via `file://`.

## Common commands

Both a PowerShell tool and a Bash tool are available. The shell is PowerShell; `start` opens files in the default app.

```bash
# Run the game locally (just open it — no server needed for basic play):
start "" "neon-blaster.html"          # Windows
# For an exact production-like run (manifest/icons resolve), serve site/:
#   cd site && python -m http.server 8080   → http://localhost:8080

# Deploy: sync the source into the git repo and push (Pages auto-rebuilds ~30-60s)
cp neon-blaster.html site/index.html
git -C site add -A
git -C site commit -m "..."
git -C site push

# Verify a deploy went live
gh api repos/neuralsparkai-svg/neon-elements/pages/builds/latest --jq .status   # -> "built"
curl -s https://neuralsparkai-svg.github.io/neon-elements/index.html | grep -o '<title>[^<]*</title>'
```

When updating docs, `README.md` and `NOTES.md` exist in **both** the root and `site/` — keep them in sync (edit `site/` copy, then `cp` to root, or vice-versa). `NOTES.md` is the maintainer cheat-sheet (links, file map, redeploy steps, done/roadmap).

### Adding / optimizing element images

The game loads ~210 KB JPEGs, not the ~2 MB source PNGs. To add a spotlight image: drop a 16:9 themed PNG in `Images/`, then re-encode it with PowerShell + `System.Drawing` (load → resize to ≤1600px wide → save JPEG quality 82) into `site/img/el-<symbol>.jpg`, and `cp` it into the root `img/` too. See the existing optimizer pattern referenced in `NOTES.md`.

## Architecture (the big picture)

The whole game is one IIFE-style `<script>` organized into numbered sections (search the section banners). The parts that span multiple concerns and aren't obvious from one function:

**Rendering & coordinates.** All game logic is in **CSS pixels**. `resize()` sets the canvas backing store to `innerWidth/Height × DPR` (DPR capped at 2) and `ctx.setTransform(DPR,...)`, so drawing code never thinks about device pixels. A `unit` scalar (derived from screen size) multiplies object radii/speeds so the game feels consistent from phone to desktop. Orbs are pre-rendered once to cached offscreen sprites (`makeOrbSprite`, also DPR-aware) and blitted each frame — this is the main performance strategy; don't draw orb text/glow per-frame.

**The `paused` flag is the central state gate.** The main loop is `if (running && !paused) { update(); draw(); }`. Several *modal* systems each set `paused = true` and show a full-screen overlay: manual **Pause**, **Study Mode** (auto-pause on destroy), **between-wave Quiz**, **Element Spotlight**, and the collection **Celebration**. These are mutually guarded — e.g. `togglePause()` returns early if `quizActive || spotActive`; the keydown handler routes keys to whichever modal is active. **When adding any new modal/overlay, follow this same pattern** (set `paused`, add an `xActive` flag, guard the others, reset it in `startGame`) or you will create overlapping or soft-locked states. `update()` also early-returns right after the wave-increment if a quiz just opened.

**Audio is fully procedural (no audio files).** Web Audio only: a look-ahead scheduler (`setInterval` + `actx.currentTime`) drives the music; SFX and the per-element "stings" are synthesized from small reusable voices (`stBell`/`stNoise`/`stBlip` → `playSting`). Spoken element names use the Web Speech API (throttled, `cancel()` before each utterance). **Key subtlety:** most pauses call `actx.suspend()` to freeze music, **but spotlights call `duckMusic()` instead** (ramp the music gain to 0 while keeping the context running) so the element sting can still play. Never `suspend()` during something that needs sound. `muted` short-circuits all SFX/speech and zeroes the music gain.

**Game modes** (`gameMode`: `facts` / `find` / `category` / `discovery`) are variations on one engine: `applyMode()` toggles the relevant HUD panel and `destroyElement()` branches scoring/speech per mode. Element data is the `ELEMENTS` array (all 118) + `categoryOf()` (atomic-number → group) + `CATS` (group → label/color) + `FACTS`. Spawns are weighted toward `COMMON_ELEMENTS` for friendliness.

**Element Spotlights are data-driven.** `SPOTLIGHTS[symbol]` = `{ img, fx, sting, what, real }`. On the first destroy of a featured element per game, `triggerSpotlight()` runs the cinematic sequence: a DOM **orb-morph** (the blasted orb flies to center and grows into the badge) → a flash that masks the cut → the reveal, with a **themed particle layer** on a separate `#fxCanvas` + its own `requestAnimationFrame` loop (`fxStep`, independent of the paused game loop) and the synthesized sting. `fx` keys must exist in `FX_STYLES` (kind + color palette); `sting` kinds must be handled in `playSting`. Images **lazy-load** per element via `ensureSpotImage()` called in `spawnOrb` (no eager preloading). **To add a featured element:** add its optimized image, then one `SPOTLIGHTS` entry referencing an existing/added `fx` and `sting`.

**Persistence.** `localStorage`: per-mode high scores (`neonElementsHigh`) and a settings object (`neonElementsSettings`: difficulty, studyMode, muted, aimAssist, quizzes, spotlights). `refreshSettingsUI()` syncs the start-screen segmented controls to these.

**Mobile.** Touch uses a dual-thumb scheme reusing the same `mouse.x/y/down` the desktop path uses (left half = joystick → movement vector; right half = aim+fire, with optional aim-assist that snaps shots to the nearest orb). Layout is responsive via `clamp()`/media queries; `:hover` is gated behind `@media (hover:hover)`; overlays scroll and respect `env(safe-area-inset-*)`. Installable PWA via `manifest.webmanifest` + icons; the fullscreen button is intentionally a no-op on iPhone (iOS blocks the Fullscreen API).

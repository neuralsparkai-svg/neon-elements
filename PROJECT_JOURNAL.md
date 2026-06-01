# Atomica — Project Journal & Learnings

A narrative record of how this project started, evolved, and what we learned along the way.
Kept for brainstorming and continuity. (Companion docs: `README.md` = public pitch, `NOTES.md` = maintainer cheat-sheet, `CLAUDE.md` = guidance for AI coding sessions.)

---

## 1. What this is now (snapshot)

**Atomica — "Blast the elements to life."** A single self-contained `index.html` (sourced from `neon-blaster.html`), zero dependencies, ~2000 lines of inline HTML/CSS/JS. A neon arcade survival game that secretly teaches the **periodic table**.

- **Live:** https://neuralsparkai-svg.github.io/neon-elements/ (GitHub Pages, repo `neuralsparkai-svg/neon-elements`)
- **Core loop:** move (WASD / left thumb), aim+shoot (mouse / right thumb) glowing orbs, each orb carries a real element (number + symbol + name). Destroy before they touch you; 3 health bars; rising waves.
- **Learning layers:** spoken element names (Web Speech), fun-fact flash, Study Mode (auto-pause to read), between-wave quizzes, and cinematic per-element **Spotlights** (16 elements) with real-world themed art + themed particles + synthesized audio stings.
- **Modes:** Fun Facts · Find the Element · Element Groups (category colors) · Collection Quest (light up all 118).
- **Production polish:** procedural music+SFX (no audio files), per-mode high scores, settings (difficulty / study / aim-assist / quiz / spotlights / sound) persisted to localStorage, fully responsive + touch, installable PWA, QR code, animated logo.

---

## 2. Where we started

Original brief: *"a beautiful, exciting, kid-friendly neon arcade shooting game."* Player in a glowing arena with an energy blaster; enemies are **glowing circles/energy orbs** (explicitly non-violent — not creatures, no blood). WASD movement, left-click to shoot lasers, 3 health bars, score, waves, particle effects, Game Over + restart.

First deliverable: a single `neon-blaster.html` with canvas rendering, neon grid arena, cached-sprite glowing orbs, bullets, particles, HUD, start/gameover overlays. The non-violent "energy orb" framing was a hard constraint from day one and shaped every later decision (e.g., enemies became *educational objects*, not targets).

---

## 3. How it evolved (timeline of decisions)

1. **Base arcade game** — canvas, WASD, mouse-aim auto-fire, waves, particles, hearts, score.
2. **Educational pivot → periodic table.** Each orb became a real element (symbol + number + name baked into the cached sprite). Added procedural music + SFX (Web Audio) and **spoken element names** (Web Speech). Added a fun-fact flash on destroy. This was the turning point from "a game" to "a learning platform."
3. **Game modes as a config.** Four selectable modes built on one engine (`gameMode` switch): Fun Facts, Find the Element, Element Groups (category colors + legend), Collection Quest (118-cell board). Added a **pause** that shows the fun fact big and readable — directly solving the "facts go by too fast" complaint.
4. **Replayability + accessibility.** Per-mode high scores, collection-complete celebration, **touch controls** (dual-thumb), then **difficulty / Study Mode / mute / aim-assist** settings (localStorage).
5. **Mobile-native pass.** DPR-crisp rendering, `unit` device scaling, responsive layout (`clamp()`/media queries), safe-area insets, hover gating, fullscreen, dynamic-viewport handling. Decided to **support both orientations** and keep dual-thumb + aim-assist.
6. **Distribution.** Realized "single file shared as a file" ≠ installable; the right path is a hosted URL. Deployed to **GitHub Pages**, added a **PWA manifest + icons** (Add-to-Home-Screen, full-screen), generated a **QR code**.
7. **Rebrand → Atomica.** New name + tagline ("Blast the elements to life"), neon "A"-atom app icon, animated `logo.svg`, full README.
8. **Between-wave quizzes.** 4 question types generated from the data (symbol↔name, group membership, fact→element), bonus points, On/Off toggle.
9. **Element Spotlights (the wow feature).** Cinematic full-screen reveals for featured elements using real-world themed background art. Then pushed quality: **orb-morph transition** (orb flies to center → grows into badge → flash masks the cut), **themed particle FX** (per-element, separate canvas + rAF), and **per-element synthesized audio stings**. Started with 6 user-provided images; extended to **16** after the user generated 10 more.

---

## 4. Key technical learnings (what worked, and why)

- **One file, zero deps was the right call.** Trivial to share, host, reason about, and hand to an AI session. No build step = no toolchain rot. Trade-off: a ~2000-line file needs disciplined section banners and naming (we use numbered section comments) to stay navigable.
- **Cached offscreen sprites for orbs** = the core performance win. Drawing glow + text per-frame for many orbs tanks FPS; rendering each orb once to a small DPR-aware canvas and blitting is cheap. Same lesson applied again to keep things smooth on mobile.
- **Keep all game math in CSS pixels; handle DPR once in `resize()`.** This kept logic readable while fixing blur on high-DPI phones. The `unit` scalar (screen-size → object size/speed) made the feel consistent phone↔desktop.
- **A single `paused` flag with mutually-guarded modals.** Pause, Study Mode, Quiz, Spotlight, Celebration all set `paused=true` and guard each other. This pattern is powerful but fragile — every new modal must follow it (set an `xActive` flag, guard the rest, reset in `startGame`) or you get soft-locks. This is the most important invariant in the codebase.
- **Procedural audio (Web Audio) beat shipping audio files.** No assets, instant load, infinitely tweakable. Music uses a look-ahead scheduler; SFX/stings are built from tiny reusable voices (`stBell`/`stNoise`/`stBlip`). Subtlety learned the hard way: **suspending the AudioContext to pause music also silences stings** — so spotlights had to *duck* the music (gain→0, context still running) instead of suspending.
- **Speech needs throttling.** `speechSynthesis` queues and lags; `cancel()` before each utterance + a min-interval gate keeps it responsive.
- **Browser autoplay/gesture rules:** initialize/resume audio + voices on the PLAY tap. Clean single entry point.
- **Data-driven content scales.** `SPOTLIGHTS[sym] = {img, fx, sting, what, real}`, `FX_STYLES`, and the sting recipes mean adding an element is one data entry + one image. `ELEMENTS`/`categoryOf`/`CATS`/`FACTS` similarly keep the 118-element domain declarative.
- **Lazy-load heavy assets.** Eager-preloading 16 background images (~3.7 MB, and ~5.8 MB *each* decoded in RAM) is bad on mobile. Loading per-element on first spawn (`ensureSpotImage` in `spawnOrb`) gave us "adds no startup cost; only fetch what you encounter."
- **Image optimization matters.** 2 MB source PNGs → ~210 KB 1600×900 JPEGs (q82) via PowerShell `System.Drawing`. ~10× smaller, visually indistinguishable as backgrounds.
- **Touch reuse.** Mapping touch to the same `mouse.x/y/down` the desktop path already used meant the whole aim/fire engine worked on mobile unchanged — big simplicity win.
- **Cinematic "cut-masking."** The morph→reveal seam is hidden by a white flash (a classic film/game trick). Measuring the badge's real on-screen rect lets the flying orb land exactly on it.

---

## 5. Pros / cons & trade-offs we accepted

| Decision | Pro | Con / risk |
|---|---|---|
| Single HTML file | Portable, no build, AI-friendly | Large file; merge-unfriendly; no module boundaries |
| Source = `neon-blaster.html`, repo = `site/` (copy) | Keeps a clean local working file | **Easy to forget to copy into `site/index.html` before pushing** |
| Procedural audio/art for engine bits | No assets, instant | Can't match photoreal art (see §6) |
| Spotlight on first-hit-per-game | Feels special, not naggy | With 16 elements, many reveals per game; may want "first-ever" (localStorage) for return players |
| Public GitHub Pages repo | Free hosting, shareable link/QR | Repo is public (fine here); no offline mode without a service worker |
| `clamp()`/media-query responsive, no framework | Light, fast | Hand-tuned breakpoints; more manual than a UI kit |
| Quizzes every wave | Strong active recall | Can interrupt arcade flow (hence the toggle) |

---

## 6. The honesty checkpoint (capability boundary)

When asked to generate 20 more themed images, I assessed candidly: **I have no image-generation model in this environment** (no diffusion/DALL·E/Midjourney, no image API). I can produce **hand-coded SVG/canvas vector art** (the app icon, animated logo, and a sample procedural "Silver" scene demonstrate the ceiling) — good, on-palette, but a *flat vector* aesthetic that does **not** sit seamlessly beside the user's photoreal 3D renders. Mixing the two styles would *lower* perceived quality. Conclusion given: I cannot match/beat those renders natively; best paths are (A) user generates them in the same tool, or (D) provide an image-gen API key and I orchestrate generation at scale. The user chose to generate them — and did (10 images, wired in). **Lesson: be truthful about tool boundaries; offer concrete alternatives instead of overpromising.**

---

## 7. Gotchas / invariants to remember

- **Deploy = copy `neon-blaster.html` → `site/index.html`, then push from `site/`.** The root is not a git repo.
- **Spotlights duck music; other pauses suspend the context.** Don't suspend during anything needing sound.
- **Every modal must respect/guard `paused`** and reset in `startGame`.
- **`fx` keys must exist in `FX_STYLES`; `sting` kinds must be handled in `playSting`.** (Adding an element silently no-ops the sting/FX otherwise.)
- **iPhone:** Fullscreen API is blocked (button is a deliberate no-op); all browsers there are WebKit; the silent-switch can mute Web Audio.
- **README.md / NOTES.md exist in both root and `site/`** — keep in sync.

---

## 8. Open ideas / brainstorming backlog

- **"First-ever" spotlight memory** (localStorage) so seasoned players aren't re-shown reveals; keep first-per-game for newcomers.
- **Parallax depth** in spotlights (needs 2–3 layered image exports per element) — the last ~0.3 toward a perfect-10 reveal.
- **Compound crafting** (Na + Cl → salt) and **electron-shell** orbs — deeper chemistry.
- **Periodic-trend boss waves** (by period/group) to teach reactivity/electronegativity.
- **Teacher/classroom mode** — pick a focus set of elements + simple progress view.
- **Offline PWA** via a service worker (2nd file; breaks the strict single-file ideal).
- **Extend Spotlights toward all 118** — pipeline is ready (image → optimize → one `SPOTLIGHTS` entry); gated only on art.
- **More languages** for spoken names + facts.

---

## 9. Where to resume (for a fresh session)

Read `CLAUDE.md` (architecture + invariants + deploy workflow), then `NOTES.md` (done/roadmap + redeploy commands), then this journal. The game is feature-complete and live; natural next steps are the §8 ideas. State is fully externalized in git + these docs + the `.claude` memory file `atomica-project.md`.

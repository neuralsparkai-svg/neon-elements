# Atomica — Maintainer Notes

Quick reference for whoever maintains/deploys Atomica.

## 🔗 Links
- **Live game:** https://neuralsparkai-svg.github.io/neon-elements/
- **Repo:** https://github.com/neuralsparkai-svg/neon-elements
- **QR (hosted):** https://neuralsparkai-svg.github.io/neon-elements/qr.png

## 📁 What's in the repo
| File | Purpose |
|------|---------|
| `index.html` | **The entire game** — single self-contained file (HTML + CSS + JS, no dependencies) |
| `manifest.webmanifest` | PWA manifest (name, icons, standalone display) |
| `icon-192.png`, `icon-512.png` | App icons (Android / manifest) — neon "A" atom mark |
| `apple-touch-icon.png` | iOS home-screen icon (180×180) |
| `logo.svg` | Animated brand logo (atom + wordmark) |
| `qr.png` | QR code pointing at the live URL |
| `README.md` | Public README |
| `NOTES.md` | This file |

> Note: the local working copy of the game is `neon-blaster.html` (in the project folder). When changing the game, edit that, then copy it to `index.html` in this repo before pushing.

## 🚀 How to redeploy (after any change)
```bash
# from the repo folder (the "site" folder locally)
git add -A
git commit -m "Describe the change"
git push
```
GitHub Pages rebuilds automatically (~30–60s). Verify it's live:
```bash
gh api repos/neuralsparkai-svg/neon-elements/pages/builds/latest --jq .status   # -> "built"
curl -s https://neuralsparkai-svg.github.io/neon-elements/ | grep -o '<title>[^<]*</title>'
```

## ✅ Current status — done
- Core arcade game (move/aim/shoot, 3 health, waves, score, particles, neon visuals)
- Periodic-table elements inside every orb (number + symbol + name), all 118
- Procedural music + SFX; spoken element names (Web Speech)
- Fun-fact flash + **Study Mode** (auto-pause to read)
- **4 modes:** Fun Facts · Find the Element · Element Groups · Collection Quest
- **Between-wave quizzes** (4 question types: symbol↔name, group membership, fun-fact recall; +25 bonus; On/Off setting)
- Difficulty (Easy/Normal/Hard), **Aim Assist**, mute, per-mode high scores
- **Mobile build:** DPR-crisp rendering, responsive layout, dual-thumb touch + aim assist, safe-area, fullscreen button
- **Installable PWA:** manifest + icons (Add to Home Screen, full-screen)
- **Rebranded to Atomica** — tagline "Blast the elements to life", new icon, animated logo
- **Deployed** to GitHub Pages + QR code

## ⏳ Pending / known limitations
- **iPhone fullscreen button** does nothing (iOS WebKit blocks the Fullscreen API on iPhone). Workaround = Add to Home Screen. Not a bug.
- **No offline play** yet — needs a service worker (would be a 2nd file). iOS install full-screen works without it.
- README logo may render **static on GitHub** (GitHub sanitizes SVG animation); it animates on the live site and in browsers.

## 🛣️ Roadmap (learning features)
Compound crafting (Na+Cl→salt) · electron shells/config · states of matter · periodic-trend boss waves · classroom/teacher mode · more languages. See README for details.

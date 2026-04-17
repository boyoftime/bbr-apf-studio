# Changelog

All notable changes to **BBR APF Studio** are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com).

## [1.5.0] — 2026-04-17

### 🎉 First public release.

### Added
- Full 4-step workflow: **Import → Edit → Textures → Repack**.
- Smart form JSON editor with typed inputs (numbers amber, strings green, booleans cyan).
- Hint tooltips for 150+ of the most-modded fields (boost, AI, powerups, prices, explosions, handling, lap count, etc.).
- Raw JSON editor powered by CodeMirror — syntax highlighting, line numbers, bracket matching, code folding, active-line highlight.
- Native multi-error JSON linter — detects parse errors, unmatched braces, unclosed strings, trailing commas, each as separate red-underline annotations.
- Pulsing red error-count bubble with click-to-jump.
- Copy-to-clipboard button for the whole current JSON.
- Real-time progress via Tauri IPC channels (0 → 100% live during extract and repack).
- Hacker-style decode log for visual flair.
- Per-APF texture summary cards (Assets / Expansion) with counts and individual Explorer shortcuts.
- Force-include PNGs option for image editors that preserve file timestamps.
- Output folder picker for repack export.
- Clean workspace / start fresh option.
- Window auto-fits to screen (max 1200×760, min 92% of available resolution).
- Built-in tutorial with **8 numbered hacks** for BBR1 beginners (title text, 99-charge boost, rocket turbo, no sky ceiling, texture swap, giant trees, sky-rain powerups, powerup purge).
- Built-in About panel with YouTube / Donate / WhatsApp CTAs.
- 3-second splash screen with "by someless" branding.
- Subscribe YouTube button + 100% FREE badge in top bar.

### Under the hood
- Bundled Python 3.12 runtime (embedded, no external install required).
- Bundled Pillow, NumPy, texture2ddecoder.
- PyArmor obfuscation on the entire Python toolkit (8 files).
- Full APF + VUJB format support: dual fnv1a32 integrity hashes, VUJB types 0-8, trailing metadata preservation, body-offset-order layout, 64-byte header byte-exactness.
- Tauri 2 + Rust backend with streaming stdout parser.
- Custom native icon (bbr.png) baked into the Windows executable.

### Compatibility
- ✅ Beach Buggy Racing 1 (BBR1) — fully tested.
- ⚠️ Beach Buggy Racing 2 (BBR2) — untested.

### Distribution
- NSIS installer — **23 MB**.
- Installs to `%LOCALAPPDATA%\BBR APF Studio\`.
- Standard Windows install flow with Start-menu + desktop shortcuts.
- Self-contained — zero external dependencies required.

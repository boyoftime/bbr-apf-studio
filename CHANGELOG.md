# Changelog

All notable changes to **BBR APF Studio** are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com).

## [1.7.0] — 2026-04-23

### 🎁 Mod Config export (new)
- **New "Mod Config" tab (step 6)** — package just the files you changed into a small `.bbc` file instead of sharing the full 100+ MB modded APF. Perfect for publishing mods or pairing with BBR Manager V3.
- **Selective export** — the tab lists every changed file grouped by category (Databases, Textures, Music, SFX, Level Templates, Track Scripts, Localisation). Tick only what belongs to *this* mod; leave the rest of your in-progress edits in the workspace.
- **Metadata form** — mod name, author, version, game (BBR1 / BBR2), description, optional thumbnail PNG. Required fields guarded; auto-generated safe filename like `rocket-turbo-v1.0.bbc`.
- **Target auto-detection** — Studio infers whether the `.bbc` targets `Assets.apf`, `Expansion.apf`, or both, based on the selected files' paths.
- **Cross-category selection UI** — per-category "select all" checkbox with tri-state (all / some / none), live file/selected/size counters, search filter across names and paths.
- **No-baseline warning** — if the workspace has no baseline (`.bbrws` loaded without one, or extracted with "Skip baseline"), the tab shows a clear amber warning with a "Go to Import" button instead of dumping every file as "changed".

### File format
- `.bbc` is a zip containing `manifest.json` (mod metadata + file list with blake3 hashes) and `files/<rel>` payload mirroring the extracted workspace tree. PNGs, APFs, WAVs, etc. are stored uncompressed (already-compressed containers); text files are deflated.

### Under the hood
- New Tauri commands `list_changed_files` and `export_bbc` — reuse the existing baseline hashing pipeline for zero duplicate work. Baseline detection is parallel via rayon.
- Path-safety checks on export (`..` rejection) so a malformed workspace can't write outside the zip.

## [1.6.0] — 2026-04-19

### 🎧 Sound modding (new)
- **Sounds tab** — browse every music track and SFX sample extracted from the game's FMOD banks.
- **In-app preview player** — custom play/pause button, professional scrubber with buffered-range indicator, volume slider, live time read-out, per-row loading spinner.
- **Replace-sound** — pick any WAV from your disk; Studio automatically renames it to match the in-game sample name, re-encodes the parent FSB5 bank as PCM16, and patches every ancestor size field (WRAP / RIFF / SND / SNDH).
- In-game playback verified — edited SFX actually play after repack (required a custom SNDH size-field fix not documented anywhere; vgmstream only reads, doesn't rebuild).

### 💾 Workspace persistence
- **Auto-restore on launch** — if you extracted or loaded a `.bbrws` in a previous session, Studio reopens straight into the editor with every edit intact. No re-import needed.
- **Remembered `.bbrws` path** — after load or save, the filename pill below the topbar shows the current workspace. Sticky across app restarts via a tiny sidecar file.
- **Smart Save** — Ctrl+S or the Save button writes to the known `.bbrws` path instantly; Save As always prompts. Amber ● dot indicates unsaved edits.
- **Session Active banner** — clear source line ("Loaded from `X.bbrws`" vs. "Imported APFs"), with Start Fresh and Continue Editing buttons so manual import and `.bbrws` loading never get confused.

### ⚡ Performance
- **Save workspace 2-5× faster** — `.apf` / `.bin` / `.wav` / `.png` now stored uncompressed in the `.bbrws` zip (they're already compressed containers; deflating them was pure CPU cost). Text-like assets still deflated for size.
- **Parallel baseline hashing** — the post-extract baseline write and pre-pack change scan now use rayon to hash files across all CPU cores (3-6× faster on modern multicore).
- **Sounds search debounced** — the filter input rebuilds the list via `DocumentFragment` and debounces at 80 ms, keeping the UI smooth with 1000+ samples.

### Fixed
- FSB5 SNDH chunk size-field is now patched on bank rebuild — without this, FMOD silently stops reading the bank at the old length and replaced SFX never play.
- Audio cache-bust: replacing a sound no longer plays the previous version from cache; per-sound timestamp token forces a fresh fetch.
- Workspace filename pill layout moved into a dedicated bar below the topbar, eliminating collisions with the brand logo, 100% FREE badge, and SUBSCRIBE button on narrow windows.

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

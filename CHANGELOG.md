# Changelog

All notable changes to **BBR APF Studio** are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com).

## [1.9.0] — 2026-05-04

### Added
- **BBR2 APF support promoted into Studio.** The bundled toolkit now handles BBR2 JSON tables, PNG texture decode/repack, and APF compression variants used by desktop and Android builds.
- **Optional `HF.apf` import card.** Import now supports `Assets.apf`, optional `Expansion.apf`, and optional `HF.apf`.
- **Multi-APF repack loop.** Studio repacks every imported `.apf` archive instead of hardcoding only Assets/Expansion.
- **Embedded `etcpak` encoder.** Android ETC/ETC2 texture replacements can be re-encoded inside the self-contained installer.

### Changed
- Compatibility docs now list BBR1 PC/Android and BBR2 desktop/Android APF workflows as supported.
- Texture and sound panels now scan imported APF folders dynamically, so extra archives such as `HF.apf` appear naturally when present.

## [1.8.1] — 2026-04-24

### Mod Config UX polish
- **Mod Config tab is now always clickable** — no need to import an APF first. Author a `.bbc` from scratch, re-import an existing one to edit, or drop in texture replacements without any workspace extraction step.
- Tab rendered in **gold + bold** to visually separate the authoring surface from the numbered Import → Edit → … → Repack pipeline.

### Password field polish
- The **Edit password** field in the Mod Metadata panel now has a framed pill with a lock icon on the left and an eye toggle on the right. Click the eye to reveal / hide the password; focus lifts a subtle cyan ring around the input.
- The **Edit-protected mod** dialog (shown when re-importing a locked `.bbc`) gets the same treatment — lock icon, monospace input, eye toggle, placeholder hint. Always opens masked even after a previous reveal.

### Thumbnail preview
- Thumbnail in the Mod Metadata panel now renders in a fixed **16:9 frame** with `object-fit: cover` — no more oddly stretched or vertically-cropped previews. Subtle inner shadow gives it depth.

## [1.8.0] — 2026-04-24

### ⚡ New `.bbc` format — v2 (authored mods, not scanned)
- **Mod Config tab fully rewritten.** No more baseline-scan workflow. You now *author* a mod explicitly: add JSON patches (only the keys you want to inject) and drop file replacements into a dedicated folder.
- **JSON patches with two strategies.**
  - `merge` (RFC 7396 JSON Merge Patch) — deep-merges your payload into the base file. Nested objects merge recursively, `null` deletes a key, arrays replace wholesale. Multiple mods can now stack on the same file without clobbering each other, as long as they don't touch the same leaf key.
  - `replace` — clobbers the entire file (the old v1 behaviour).
- **Patch authoring dialog** — dedicated CodeMirror-powered editor with the same material-ocean theme as the main Edit tab: JSON syntax highlighting, line numbers, bracket matching, code folding, live lint underlines, and the existing multi-error linter. Wide by default (`min(1180px, 96vw)`) and grows vertically to fill the dialog height. Backdrop/Esc dismissal disabled so a stray click doesn't nuke an in-progress payload — only the ✕ button and Cancel close the dialog.
- **Target path autocomplete** — the Target input shows a live datalist of every `.bin.json` in the current workspace, so you rarely need to type the path by hand.
- **Per-patch preview + edit/remove** — each saved patch renders as a card in the authoring panel with badge, path, size, collapsed JSON preview, and Edit / Remove actions.

### 📦 File replacements — any binary, not just PNG
- The authoring folder now accepts **any file**, not only PNG. Audio (`wav`, `ogg`, `mp3`, `fsb`), level binaries (`bin`), images (`png`, `jpg`, `webp`), and anything else the `.bbc` replace path can overlay.
- Panel renamed to **File Replacements**. Each row shows a tinted extension badge (PNG blue, WAV/OGG/MP3/FSB purple, BIN green, other grey) so a mixed bag of replacements is scannable at a glance.
- Walkers now skip OS metadata and editor clutter — `Thumbs.db`, `desktop.ini`, dotfiles, `.tmp`, `.crdownload`, `.part` are all filtered out so they never sneak into a shipped mod.
- Manifest category for each replacement is computed from its APF path (`Textures`, `Music`, `SFX`, `Level Templates`, `Track Scripts`, …) instead of a hard-coded "Textures".

### 📥 Re-import a `.bbc` for editing
- New **📥 Import .bbc…** button lets you open any v2 mod you've already exported (or downloaded) and pull its patches, file replacements, metadata, and thumbnail straight back into the authoring surface. Edit anything, export a new version.
- Import wipes and repopulates the authoring state cleanly (with a confirm prompt if you have unsaved patches so you don't nuke your work).

### 🔐 Optional edit password (Studio-only gate)
- New **Edit password** field in the Mod Metadata panel. Leave it blank to keep the mod openly re-editable. Set it to stamp a blake3 hash of the password onto `manifest.editPasswordHash`.
- On re-import, Studio checks the hash and prompts for the password — wrong password rejects the load with a clear error.
- **BBR Manager ignores the hash entirely.** The payloads inside `files/` stay plaintext, so password-locked mods apply in-game without asking for anything. The lock is an honest obfuscation layer, not encryption (the `.bbc` is still just a zip).

### 🧰 Schema bump — manifest v2 only
- v2 mods carry two arrays (`patches[]`, `textures[]`) instead of the single v1 `files[]`. Each entry records `patchType` (`merge` | `replace`), hash, size, category, and target APF.
- v1 mods are refused on import with a clear message asking the user to re-author in the latest Studio. Keeps the apply pipeline honest.

### Breaking
- The old baseline-scan Mod Config workflow is gone. Workspaces extracted with the previous flow still work for editing; only the *export* step is new.
- v1 `.bbc` files cannot be imported into Studio or applied in BBR Manager v3.5+. Re-author them in the new Studio to regenerate as v2.

## [1.7.3] — 2026-04-23

### Security / hardening
- Swept the remaining plaintext toolkit scripts and obfuscated all of them: `toolkit_sound_extract.py`, `toolkit_sound_repack.py`, `tools/bank_encode.py`, `tools/fsb5_decode.py`, `tools/fsb5_encode.py`. The entire bundled Python toolkit now ships as encrypted bytecode — no readable source left in the install directory.

## [1.7.2] — 2026-04-23

### Security / hardening
- Re-obfuscated the newly-fixed packer scripts (`toolkit_3_pack.py`, `tools/apf_pack.py`) with PyArmor 9.2.4. Ships as encrypted bytecode so the toolkit source isn't readable from the installed folder.
- No functional changes — identical behaviour to 1.7.1, just with source concealed again.

## [1.7.1] — 2026-04-23

### Fixed
- **Music stream replacements now actually apply.** A stale bundled `apf_pack.py` was silently ignoring edited WAVs in `VuAudioStreamAsset/` at repack time, so your replaced music track stayed original in-game. Studio now ships the current packer that detects and re-encodes music WAVs correctly.
- **Repack script reworked to split concerns cleanly** — script dir holds the toolkit (`tools/apf_pack.py`), working directory holds the user's workspace. Progress emitted as `PROGRESS:N` for a smooth progress bar during pack.

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

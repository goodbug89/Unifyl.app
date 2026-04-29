<p align="center">
  <img src="docs/screenshots/icon.png" alt="Unifyl" width="128" height="128">
</p>

<h1 align="center">Unifyl</h1>

<p align="center">
  <strong>The AI-powered dual-pane file manager for macOS.</strong><br>
  Total Commander's depth. ForkLift's polish. Intelligence built in.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/macOS-14%2B-000?logo=apple&logoColor=white" alt="macOS 14+">
  <img src="https://img.shields.io/badge/Swift-6.0-F05138?logo=swift&logoColor=white" alt="Swift 6">
  <img src="https://img.shields.io/badge/Apple%20Silicon-Optimized-8E8E93" alt="Apple Silicon">
  <img src="https://img.shields.io/badge/License-Proprietary-blue" alt="License">
</p>

<p align="center">
  <a href="https://unifyl.app">Website</a> &middot;
  <a href="https://unifyl.app/docs">Documentation</a> &middot;
  <a href="https://unifyl.app/download">Download</a>
</p>

---

![Unifyl Screenshot](docs/screenshots/main.png)

## Features

### :file_folder: Dual-Pane File Management

- Single, dual, triple, and free-split pane layouts
- Tabbed browsing with tab groups and persistent workspaces
- Drag-and-drop with spring-loaded folders
- Queue manager with pause, resume, and speed throttling
- Inline terminal per pane

### :mag: Search & Filter

- Real-time filtering as you type
- Regex and wildcard pattern matching
- Spotlight integration for instant system-wide search
- Content search across file bodies
- Advanced filters by size, date, kind, and tags

### :robot: AI-Powered Tools

- **Semantic search** -- find files by meaning, not just name (local embeddings + vector DB)
- **AI rename** -- suggest filenames from image recognition or document content
- **Smart tagging** -- auto-classify files using on-device NLP
- **Duplicate intelligence** -- perceptual hashing to surface near-duplicates
- **Natural language commands** -- tell the command palette what you want in plain English
- All AI processing runs locally on Apple Neural Engine. Your files never leave your Mac.

### :globe_with_meridians: Remote & Cloud

- FTP, FTPS, SFTP (PPK, FIDO2, 2FA), WebDAV, SMB, AFP, NFS
- Amazon S3, Backblaze B2, Google Cloud Storage, Azure Blob
- Google Drive, Dropbox, OneDrive, iCloud Drive
- Docker container and Kubernetes pod browsing
- iOS/Android device connectivity

### :wrench: 75+ Professional Tools

- File → PDF conversion (images, text, Office docs via LibreOffice; HWP / HWPX with Hancom-app fallback when LibreOffice's filter rejects the file)
- Multi-rename with regex, numbering, EXIF data, AI suggestions, and undo
- Text diff, binary hex compare, image overlay, recursive directory diff
- 3-way file merge with per-conflict Accept Left/Right/Both
- Bidirectional directory sync (local and remote)
- Archives as virtual folders: browse, copy, move, delete inside ZIP, 7z, TAR, GZ, BZ2, XZ, ZSTD, RAR
- Encrypted archives: ZIP and 7z password support
- Smart filename decoding for legacy CJK ZIPs — Chinese (CP936/GBK), Japanese (Shift-JIS), Korean (CP949) detected by Hangul/Kana/Hanja distribution, not by guess
- Overwrite confirmation dialogue when extracting or copying out of archives (Skip / Rename / Overwrite, with "Apply to all")
- Entry mtimes preserved on extract — no more "extracted file is 9 hours off" on KST machines
- Inline hex editor with pattern search and edit/save
- EXIF/metadata bulk editor: batch edit date, GPS, author, copyright
- Incremental backup with delta analysis (new, modified, unchanged)
- Real-time log viewer with keyword highlighting and filter
- File operation undo/redo (copy, move, delete reversible)
- Macro recorder: record file operations, save, replay with variables
- Multi-panel workspace layouts: save and restore tab arrangements
- File operation scheduler with cron-based automation
- Checksums: CRC32, MD5, SHA-1/256/512, SHA-3, BLAKE3
- App uninstaller with leftover cleanup
- Automation engine: Shell, AppleScript, JavaScript (JXA)

### :computer: Developer Tools

- Git integration panel: status, stage/unstage, commit, push/pull, inline diff
- SSH tunnel manager: save port forwarding profiles, start/stop tunnels
- Docker container file editing with auto-sync
- Process map, port viewer, environment manager, REST API explorer

### :art: Customizable Themes

- Light and dark mode with system auto-switching
- 12 built-in presets: Classic Navy, Midnight, Nord, Solarized, and more
- Full theme editor with 20+ color tokens
- Export and share `.ultratheme` files
- Per-filetype color coding

### :keyboard: Keyboard-First Design

Every action is reachable from the keyboard. Command palette (`Cmd+Shift+P`), Vim-style navigation, and full shortcut customization. Remap any of 120+ commands across 12 categories in Settings > Keyboard Shortcuts, with conflict detection and one-click reassignment.

### :earth_americas: 15 Languages

English, Korean, Japanese, Chinese (Simplified & Traditional), German, French, Spanish, Portuguese, Italian, Russian, Turkish, Arabic, Thai, Vietnamese.

---

## Installation

### Download

Get the latest release from [unifyl.app/download](https://unifyl.app/download).

1. Open the `.dmg` file
2. Drag Unifyl to your Applications folder
3. Launch and grant file access when prompted

### Homebrew (planned)

```bash
brew install --cask unifyl
```

---

## Pricing

| | **Free** | **Pro -- $39.99** |
|---|---|---|
| Pane layouts | Single & Dual | Single, Dual, Triple, Free-split |
| Tabs & workspaces | 5 tabs | Unlimited |
| Search | Basic + Spotlight | Full regex + content search |
| AI tools | -- | All AI features |
| Remote connections | FTP, SFTP | All protocols + cloud storage |
| Archives | ZIP | ZIP, 7z, TAR, GZ, BZ2, XZ, ZSTD, RAR |
| Multi-rename | Basic | Full (regex, EXIF, AI, presets) |
| Compare & sync | -- | Text, binary, image, directory |
| Themes | Light & Dark | 12 presets + custom editor |
| Plugins | -- | All 6 plugin types |

One-time purchase. No subscription. Free updates for life via built-in Sparkle auto-updater.

---

## Keyboard Shortcuts

| Shortcut | Action |
|---|---|
| `Cmd + Shift + P` | Command palette |
| `Tab` | Switch active pane |
| `Return` | Open file / Enter folder (cursor item only) |
| `Shift + Return` | Rename selected file |
| `Space` | Quick Look preview |
| `Cmd + Delete` | Move to Trash |
| `Cmd + Z` | Undo last file operation |
| `Cmd + Shift + N` | New folder |
| `Cmd + F` | Find / filter |
| `Cmd + Shift + F` | Advanced search |
| `Cmd + Opt + F` | Content search |
| `Cmd + Opt + S` | AI Semantic search |
| `Ctrl + F7` | Advanced search panel |
| `Insert` | Toggle select + move down |
| `Shift + ↓/↑` | Toggle select range |
| `Ctrl + S` | Add to Smart Folder |
| `Cmd + T` | New tab |
| `Cmd + W` | Close tab |
| `Cmd + 1` | Single pane layout |
| `Cmd + 2` | Dual pane layout |
| `Cmd + K` | Open terminal |
| `Cmd + Opt + C` | Compare files |
| `Cmd + Opt + G` | Git panel |
| `Cmd + I` | Get Info |

All shortcuts are customizable in Settings > Keyboard Shortcuts.

---

## Building from Source

### Requirements

- macOS 14 Sonoma or later
- Xcode 16.0+
- Swift 6.0+

### Steps

```bash
git clone https://github.com/niceguy61/unifyl.git
cd unifyl

# Verify your toolchain (xcodegen / swift / dmgbuild / 7zz / notarytool …)
make doctor

# Generate Xcode project
make gen

# Build all packages — sequentially with clean log per package
make build

# Or in parallel (8.0s → 0.5s no-op rebuild, ~3s cold)
make build-fast

# Run tests
make test

# Lint + check that user docs / in-app help / KeyBindingManager don't drift
make lint
```

See the [Makefile](Makefile) for all available targets.

---

## Plugin Development

Extend Unifyl with `.unifylplugin` bundles. Six plugin types are supported: Viewer, Packer, Content, FileSystem, Action, and AI.

```swift
import UnifylPluginSDK

public class MyViewerPlugin: ViewerPlugin {
    public var manifest: PluginManifest { ... }
    public func canView(_ url: URL) -> Bool { ... }
    public func makeView(for url: URL) -> AnyView { ... }
}
```

Install plugins to `~/Library/Application Support/Unifyl/Plugins/`. SDK documentation coming soon.

---

## Contributing

Unifyl is proprietary software. We welcome:

- **Bug reports** -- open an issue with reproduction steps
- **Feature requests** -- describe your workflow and what you need
- **Translations** -- help us reach more languages
- **Plugins** -- build on the plugin SDK

---

## License

Proprietary. Copyright 2024-2026 Unifyl. All rights reserved. See [LICENSE](LICENSE) for details.

---

<p align="center">
  <a href="https://unifyl.app">Website</a> &middot;
  <a href="https://unifyl.app/docs">Docs</a> &middot;
  <a href="https://twitter.com/unifylapp">Twitter</a> &middot;
  <a href="https://discord.gg/unifyl">Discord</a>
</p>

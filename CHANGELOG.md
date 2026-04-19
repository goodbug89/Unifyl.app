# Changelog

All notable changes to Unifyl will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]


## [1.0.6] — 2026-04-19

### Added
- **Theme Editor opens as an independent, resizable NSPanel** via `ViewerWindowManager` (same pattern used by the Log Viewer / Git Panel / Hex Editor). Previously it was a modal sheet locked to the main window with no way to size it against a real file list.
- **Theme Editor `Apply as Active Theme` button + `Live Apply` toggle**. `Apply` persists the edited custom theme and makes it the active app theme in one step (previously `Save Theme` only wrote to disk — the theme never actually painted the chrome because `ThemeManager` didn't know how to look up custom-theme IDs). `Live Apply` streams every ColorPicker / hex-field edit into the running app for real-time tuning against actual panels.
- **Cursor-color theme token** (`cursorHex`), separate from `selectionHex`, shipped with every built-in preset. File-table rows now distinguish the keyboard-focus cursor (single row, bright, drawn with a 3-px left-edge stripe on top of the row fill) from mark-selection (multi-row, dimmer). Matches Total Commander's behaviour and the Theme Editor's preview.
- **Total Commander-style right-drag selection** in the file table. Hold the right mouse button and drag — the drag range is mark-selected (additive onto any existing marks, never replacing) and the cursor indicator tracks the pointer row so you can see where the drag will land. A right-drag that starts on an already-marked file subtracts its range from the selection instead.
- **Live keyboard-shortcut sync**. 42 menu items in `UnifylApp.swift` now carry a `.keyBinding("command.id")` modifier that reads the current mapping from `KeyBindingManager.shared` every render. Remapping a shortcut in Settings updates the indicator next to the menu item in the macOS menu bar immediately — no quit-and-relaunch. Clearing a binding removes the indicator. Infrastructure in `Unifyl/Services/KeyBindingShortcutParser.swift` parses the human-readable stored strings (`⌘T`, `⇧⌘N`, `⌘⌫`, `⌥←`, `F5`) into SwiftUI `KeyboardShortcut` values, including F1..F19 via the NSFunctionKey unicode range.

### Changed
- **Theme Editor UI restructured** into "General" / "Chrome Colors" (8 tokens) / "File Category Colors" sections. Chrome colours match the `AppTheme` schema exactly; file-category colours are pushed to `FileColorSettings` on Apply.
- **CustomTheme schema harmonised** with `AppTheme`'s 7-field chrome token set + cursor. Backward-compatible decoder keeps legacy `.ultratheme` exports readable: `textHex` / `selectionHighlightHex` are mapped to the new key names, and missing `surface` / `altRow` / `cursor` / `textSecondary` tokens are synthesised from accent / background so imported pre-1.0.6 themes don't come through half-black.
- **Theme menu in the toolbar** splits into "Built-in" and "Custom" sections; the currently active theme displays an `ACTIVE` badge in the Theme Editor's sidebar.
- **`edit.undoFileOp` default shortcut** corrected from `⌘Z` → `⌥⌘Z` in `KeyBindingManager`. `⌘Z` conflicts with macOS's system text-field undo, which is why the menu already used `⌥⌘Z` — the two were silently out of sync.

### Fixed
- **Theme Editor `Apply` silently did nothing** when the custom theme's ID wasn't one of the 13 built-in presets. `ThemeManager.currentTheme` now falls back to the custom-theme store for unknown IDs, and the `themeRevision` counter is bumped on every activation so `NSViewRepresentable` wrappers (file table / grid) don't short-circuit their update pass when colour tokens happen to end up byte-identical.
- **Theme Editor fatal error** on open inside the independent window: `ViewerWindowManager`'s fresh `NSHostingView` does not inherit the main window's SwiftUI environment, so `@Environment(ThemeManager.self)` tripped a "No Observable object" crash on first render. Environment values (`appVM`, `themeManager`) are now re-injected at the panel's entry point.
- **Right-drag selection wiped earlier marks**. Every new `dragSelectRows` call used to replace `selectedIds` with just the current range, so multi-gesture selection was impossible — each new drag silently lost everything the user had already marked. The gesture now snapshots the mark set at `rightMouseDown` and unions/subtracts subsequent ranges against that baseline, so sequential drags accumulate like Shift-clicking in Finder.
- **Right-drag cursor didn't track the pointer**. Previously `tableView.selectRowIndexes(range, byExtendingSelection: false)` selected the whole drag range in NSTableView's model, making the cursor span the range and losing the "where will this drag end" signal. Cursor is now updated to just the row under the pointer; the range is visible via the marked-item text colour + the cursor-row stripe.

## [1.0.5] — 2026-04-18

### Fixed
- **Live LemonSqueezy checkout**: builds up through 1.0.4 shipped with LemonSqueezy **Test-store** credentials (variant `1522057`, checkout UUID `6266de7a-...`, Test JWT). The checkout page showed an orange "Test mode is currently enabled" banner and real cards could not complete a purchase, so no existing user could actually buy Pro. Now wired to the Live store: variant `1544858`, checkout `40f09b3c-9638-4e7c-871e-51f0e391f67c`, Live JWT. Users who attempted to purchase on 1.0.4 saw no charge on their card — retrying after updating to 1.0.5 will go through normally.

### Changed
- **Tab bar / path bar chrome unified**: the top strip above each panel now shares a single background (`bgElevated`) and height (28 px per row) instead of stepping from 30 px elevated → 26 px surface. Removes the visible seam the eye kept catching between the two rows and keeps the archive path bar + status bar in the same palette.
- **Theme tokens applied to chrome**: panel background, folder-tree background, active-panel edge indicator, toolbar status badge, command-line output pane, and the onboarding CTA now follow the selected theme (accent / background / surface) instead of raw AppKit semantic colors. Non-"System Default" presets now visibly tint every piece of chrome, not just the accent.

### Added
- **`EmptyStateView` component** — single reusable empty-state presentation (icon + title + optional message + optional CTA) now used by empty folders, the Smart Folder picker before a preset is chosen, the advanced search with no matches, and three Git-panel states (not a repo, working tree clean, no diff selected). Replaces ~6 ad-hoc "Spacer / bare Text" placements that each had a different rhythm.

## [1.0.4] — 2026-04-18

### Fixed (respin build 8)
- **Crash hunt, round 2 — `file:///..` from `goUp` at root**: build 7's `isFileURL` + existence guards still let `Process.currentDirectoryURL` reject with `NSInvalidArgumentException` because `deletingLastPathComponent()` on `/` returns `/..` (not `/`), and `/..` passes both guards — Foundation then rejects the setter with "must provide a directory path". Fix is two-sided: (1) `GitStatusProvider` now calls `.standardizedFileURL` before assignment (collapses `..` segments to absolute path); (2) `PanelViewModel.goUp` bails when already at `/` so we never produce the bogus URL in the first place.

### Fixed (respin build 7)
- **Crash when navigating to the top-level folder**: `GitStatusProvider.runGitCommand` passed the panel's `currentPath` directly into `Process.currentDirectoryURL`, which raises `NSInvalidArgumentException` (→ `SIGABRT`) if the URL isn't a file URL. On certain navigation paths (top-level / volume root / Smart Folder edge cases) `currentPath` could land on a scheme-less URL that bypassed the upstream `!isRemote` guard, killing the app the moment the git status pass started. Now guards on both `isFileURL` AND directory existence before touching `Process`.

### Fixed (respin build 6)
- **"Check for Updates…" menu item stuck disabled**: the previous build wired the menu item's `.disabled` modifier through a KVO-mirrored `@Observable` view model, and the initial `false` value sometimes stayed stuck — users couldn't click "Check for Updates" at all and had to reinstall manually. The KVO/view-model layer is removed; the button is always clickable now (Sparkle's own session guard prevents double-check). Users on 1.0.4 build 5 MUST redownload build 6 manually — the broken button blocks Sparkle from reaching this fix.

### Fixed
- **Mirror Active Panel on Other Side** (Volume picker menu) and the "copy path to opposite panel" shortcut silently did nothing when the destination panel was inside a Smart Folder or inside an archive (zip/7z/…). `PanelViewModel.navigate` bails early in those modes, so the mirror looked broken. New `navigateForce(to:)` exits the special mode first, then navigates.
- **Back / Forward arrows** in Smart Folder and inside an archive used to do nothing silently. They now leave the special mode (Smart Folder → real directory listing, archive → parent folder) so the click is never ignored.
- **Cmd+R Refresh** in Smart Folder rebuilds the cart's listing; in an archive it re-enters the archive. Previously it was a silent no-op in both cases, which read as "the app is frozen".
- **F3 (Preview) and F4 (Edit)** now fall back to the cursor item when nothing is marked, matching the project-wide `selectedOrCursorItems` rule. Previously they were silent no-ops if the user hadn't toggled a selection.
- **AI Smart Tag / Classification apply**: tag assignment to a file's `NSURLTagNamesKey` used `try?`, so a sandbox / permission / disk error was invisible — the UI said "Applied" while the tag never actually hit disk. Failures are now logged via `UnifylLogger.ai` and surfaced on `AIEngineViewModel.aiError` so the AI view can alert the user.
- **AI summary report export**: `exportSummaryReport` wrote the file with `try?` and exited silently on failure (permissions, disk full, bad format). Now logs via `UnifylLogger.ai` and sets `aiError` so the user sees why the file isn't there.
- **Archive entry removal (tar family)**: `ArchiveEngine.removeFromArchive` ignored individual file removal failures inside the tar staging dir with `try?`, which could leave a partially-modified tar on disk and silently lose the "rollback" signal. Failures now accumulate and throw so the overall operation fails loudly.
- **VectorIndex schema migration**: `ALTER TABLE ADD COLUMN` was wrapped in `try?`, so a failed migration silently left the on-disk index incompatible with the running binary and AI features depending on the new column just disappeared. Failures now log at `Logger.fault` level.
- **LLM model update check**: manifest fetch + decode were both `try?`, making every failure mode indistinguishable from "no update available". Both now log specifically (fetch vs decode) at `.error` level.
- **Automation manager — load**: saved automation rules were lost silently via `try?` on JSON decode when the schema changed between versions. Load now distinguishes "no file yet" (first launch) from "decode failed" (corrupted / schema drift) and logs the latter.
- **Script library — built-in install**: first-launch copy of bundled automation scripts used `try?`, so a permission / disk error showed up as an empty "Automations" list with no diagnostic trail. Directory creation and per-file copy failures now log via `UnifylLogger.automation`.
- **Per-sheet state leakage** across reopens: Multi-Rename (rules, AI suggestions), Regex Rename (pattern, replacement, flags, preset), AI Rename (suggestions, chosen names), and Advanced Search (results, "Select All" toggle, status message) now reset their `@State` in `.onAppear` so a second open doesn't ghost values from the previous session.
- **Duplicate Detector — hardlink data-loss guard**: hardlinks to the same file (same `(st_dev, st_ino)`) were being grouped as "duplicates". When the user bulk-deleted "duplicates" it unlinked the inode's only remaining name, destroying the underlying file. Duplicate grouping now dedupes by inode first, so a hardlinked file appears as a single entry the user can't accidentally wipe.
- **AudioPlayer timer leak**: `Timer.scheduledTimer` kept firing forever if the view was dismissed without `.onDisappear` (e.g. rapid sheet open/close). Replaced with a `DispatchSourceTimer` that a `nonisolated deinit` can safely cancel.
- **Large File Finder / Advanced Search — scan-on-dismiss crash**: both views spawned untracked `Task.detached` scans that continued after the user dismissed the sheet and then wrote to released `@State`, tripping a "main actor-isolated state mutation" trap. Scan tasks are now tracked in `@State` and cancelled in `.onDisappear`.
- **Circular symlink traversal**: `FileOperationManager.enumerateAllFiles` now tracks `(st_dev, st_ino)` for every directory it descends into, so a cycle like `a/link_to_b → b`, `b/link_to_a → a` terminates instead of hanging the copy.
- **Zip-slip / Windows-style path traversal pre-check**: `ArchiveEngine.extract(…)` now lists archive entries and rejects the archive *before* running `ditto`/`tar`/`7z`. Previous code only verified after extraction, by which time the malicious file had already touched disk. `isSafeArchivePath` also now normalises `\` → `/`, rejects UNC prefixes, and rejects Windows drive specs (`C:\foo`).
- **Decompression bomb guard**: archives that claim `>50×` and `>2 GiB` of uncompressed content are rejected before extraction with an actionable error message (extract manually if source is trusted). Prevents a 10 MB zip from filling a 500 GB disk silently.
- **AppleScript injection via malicious filename**: `PDFConvertSheet` interpolated source/destination paths into AppleScript after only escaping `"`. A filename containing `\"` broke out of the string context and let an attacker execute arbitrary AppleScript on iWork / MS Office apps just by shipping the file. Full escape sequence (`\`, `"`, `\n`, `\r`) now applied, in that order.
- **Regex ReDoS in Advanced Search**: user-supplied regex against arbitrary filenames could hang the whole scan on patterns like `(a+)+b`. Matching now goes through `NSRegularExpression` (ICU-based, bounded backtracking) with a 4 KB filename cap. An invalid pattern returns "no match" instead of crashing the scan.
- **OneDrive `idCache` staleness**: renaming / moving / deleting a file / folder via Unifyl used to leave the cached `path → graphID` mapping pointing at the old location. A subsequent operation on the old path would then hit a wrong (or deleted) OneDrive item. Each mutating operation now invalidates the cache entry for the affected URL AND every descendant, so a stale entry can't silently resolve the wrong file.
- **RemotePollingWatcher — silent disconnection**: after 5 consecutive list failures the watcher used to just stop, leaving the panel displaying the last cached listing as if the server were live. It now emits a new `DirectoryChange.connectionLost(reason:)` and `PanelViewModel` stores the reason on `remoteConnectionError` so the panel header can show a "disconnected" banner.
- **Case-insensitive filesystem rename** (`Photo.jpg` → `photo.jpg`): the collision resolver saw the existing lowercase inode at the target path and appended `_1`, producing `photo_1.jpg` instead of the intended case change. An `(st_dev, st_ino)` self-collision check now lets the rename proceed through the 2-step staging that already handles case-only renames.
- **cloudDownloadTask panel swap**: the 60-second cloud-download watcher stored `self.activePanelVM` (computed property), so if the user switched panel focus during the download the wrong panel's cloud status got refreshed on completion. Task now captures the originating panel as a weak reference at start, so refresh targets the correct panel (or is silently skipped if that panel was torn down).
- `ViewerWindowManager`: panel `willClose` cleanup now also drops the NSHostingView to release SwiftUI observation captures immediately instead of via the autorelease pool.
- **Undo move / copy with rename-on-conflict**: when a conflict resolution renamed the destination to e.g. `file_1.txt`, undo used to look for the original name `file.txt` in the destination and silently do nothing. `FileOperationHistory.Record` now carries the actual per-source destination URLs; undo uses them so the rename-suffixed file is actually reversed.
- **Undo move no longer clobbers newly-created files** at the original source path. If a newer file lives there, the restored file is placed alongside with a `.unifyl-undo` suffix so the user can resolve the collision instead of silently losing data.
- **Undo delete searches the correct Trash** on each source's volume (via `FileManager.url(for:.trashDirectory, appropriateFor:)`) instead of only the user's home `~/.Trash`. External-drive deletions are now actually undoable.
- **Workspace JSON schema evolution safety**: every `SavedWorkspace` field is now optional with sensible defaults (`nil` → home directory, "left" active panel, etc.). A new required field in a future app version no longer throws at decode time and wipes the user's entire tab/panel layout. Added a `schemaVersion` field to make future migrations explicit. Tab restoration also skips paths that no longer exist so deleted folders don't ghost-restore as dead tabs.
- **KeyBindingManager — orphan custom shortcuts preserved**: when an app upgrade renames or removes a command ID, the user's saved custom binding used to vanish silently. The manager now keeps the mapping in `orphanedCustomizations` and logs the count so the Settings UI can offer "re-assign or discard" instead of dropping it.
- **MetadataEditor batch failure — per-file reasons**: the EXIF editor used to report "Applied to N, M error(s)" with no way to know WHICH files failed or why. Each failure now carries a name + reason (`metadataFailures`), surfaced in the status line.
- **PanelViewModel tab-close race**: `loadCurrentDirectory` used to capture `activeTabIndex` as an integer, so closing a tab during an async load could write the results into a *different* tab. Now captures the tab's `UUID` and re-resolves the target index after every await suspension, bailing cleanly if the tab was closed.
- **SSH tunnel / Docker container status** state indicators now layer a small SF Symbol (`checkmark` / `xmark` / `play.fill` / `pause.fill`) on top of the green/red/orange dot, with an accessibilityLabel ("Running" / "Stopped"). Colour-blind and low-vision users can now tell the states apart.
- **SSH Tunnel Manager close button** now has `.help("Close")` + `.accessibilityLabel("Close SSH Tunnel Manager")` so VoiceOver announces its purpose.
- **Archive extract confirmation OK** now has `.keyboardShortcut(.defaultAction)` so Return key confirms — previously keyboard-only users had to reach for the mouse.
- **Quake terminal decorative rule** marked `.accessibilityHidden(true)` so VoiceOver no longer announces the visual separator.
- **Hex editor / code viewer** NSViewRepresentable wrappers now set an accessibilityLabel + role so VoiceOver announces "Hex editor" / "Code viewer (\<ext\>)" instead of a bare "text area".
- **HTML / SVG preview** WKWebView wrappers set `accessibilityLabel` including the filename so VoiceOver has a container name for the web content.
- **SecurityBookmarkManager**: unresolvable bookmarks (volume unmounted, folder deleted) are now logged via `UnifylLogger.settings` instead of silently dropped, so "my bookmarks disappeared" is diagnosable from Console.app.

### Security
- **Markdown preview XSS**: `MarkdownPreviewView` loaded user markdown-as-HTML via `WKWebView.loadHTMLString` without sanitisation. A shared `.md` file containing `[click](javascript:fetch('…'+document.cookie))` or inline `<img src=x onerror="…">` could execute arbitrary JavaScript in the app's WKWebView context. The renderer now strips `javascript:`/`vbscript:`/`data:text/html`/`file:` link schemes, inline `on*=` event handlers, and `<script>` blocks. The rendered HTML is also wrapped in a restrictive Content-Security-Policy (`default-src 'none'; img-src 'self' data:; style-src 'self' 'unsafe-inline'; font-src 'self' data:`) and loaded with `baseURL = nil` so relative `file://` paths can't resolve against the markdown's own directory.
- **License trial clock-rollback**: `LicenseManager.isWithinGracePeriod` used `Date()` directly. A user could extend a trial by moving the system clock backward. Added `ratchetedNow()` that persists the highest wall-clock ever observed to Keychain (`unifyl.license.maxObservedDate`) and refuses to advance if the wall clock reports a time more than 5 minutes before the stored ratchet — effectively freezing "now" in the past for license checks, which causes the trial to expire promptly on rollback.

### i18n / locale
- **CJK dictionary-order sort** (Korean 한글, Japanese 仮名/漢字, Chinese 汉字): `ProcessFileMapView` and similar lists used `.sorted { $0.lowercased() < $1.lowercased() }` which sorts by Unicode codepoint rather than locale-aware dictionary order. Switched to `localizedStandardCompare` so CJK names sort as users expect and embedded numbers sort numerically (`file2` before `file10`).
- **Turkish / Greek case folding**: `KeyBindingSettingsView`, `SubtitlePreviewView`, and `LogViewerView` filters used `.lowercased().contains()`, which mis-folds Turkish "İ"/"i" and Greek σ/ς. Replaced with `localizedCaseInsensitiveContains` throughout.
- **CompanionSettings port input**: `TextField(..., formatter: NumberFormatter())` picked up the system locale, so Thai / Arabic / Bengali users saw port numbers rendered in native digits. Pinned to POSIX locale since port numbers are ASCII by network convention.
- **CommandPalette NFC / NFD mismatch**: search normalises both title and query with `precomposedStringWithCanonicalMapping` so a Korean query typed composed (`한`) matches titles stored decomposed (`ㅎㅏㄴ`).
- **File-copy speed readout locale**: `FileOperationProgress.speedString` used `String(format: "%.1f")` which hardcodes `.` as decimal separator. German/French users now correctly see "1,5 GB/s" via a shared `NumberFormatter` keyed to `.autoupdatingCurrent`.

### UX
- **Inline Terminal / CommandLineBar PATH**: `process.arguments = ["-c", command]` ran a non-login, non-interactive zsh. Homebrew / nvm / pyenv / rbenv paths set up in `~/.zprofile` were invisible, so commands like `brew`, `node`, `python3` produced "command not found". Now uses `-l` (login shell) so the profile is loaded.
- **Password-protected PDF preview**: `FilePreviewView.pdfPreview` used to render a blank pane for encrypted PDFs with no indication of why. Now detects `PDFDocument.isEncrypted && .isLocked` and shows an actionable "Password-Protected PDF" placeholder with an "Open in Preview" button.
- **Text preview size cap**: `String(contentsOf:)` used to load the entire file into memory, freezing the UI for seconds on a multi-GB log file and potentially OOMing. Now reads at most 5 MB via a bounded `FileHandle.read(upToCount:)` and shows a "Large file — showing first 5 MB of X MB" banner + "Open in TextEdit" button.
- **Text preview encoding detection**: previous `try? utf8 ?? try? ascii` missed UTF-16 LE/BE, UTF-32, and Windows-1252. Added BOM sniffing (UTF-8 / UTF-16 LE / UTF-16 BE / UTF-32 LE / UTF-32 BE) + fallback to `windowsCP1252` before ASCII, so Windows-generated text files decode correctly instead of showing "Cannot read file".
- **Git status parser robustness**: `git status --porcelain` output was parsed by newline-split + `dropFirst(3)`, which broke on filenames containing spaces (git C-quotes the whole path) or newlines (multi-entry split). Switched to `git status --porcelain -z` (NUL-separated, unquoted paths) and split on `\0`.
- **Drag-and-drop modifier race** (copy vs move): `ThumbnailGridView` and `FileTableView` independently read `NSApp.currentEvent?.modifierFlags` in both `validateDrop` and `acceptDrop`. A user releasing Option between the final validate and the drop produced inconsistent behaviour (validation said copy, acceptance said move). Both coordinators now snapshot the modifier in `validateDrop` and honour it in `acceptDrop`.
- **Search regex fallback**: `SearchEngine.filter` silently returned the unfiltered list when the user's regex pattern was malformed (e.g. unbalanced brackets). Now falls back to a literal substring search of the pattern so the filter still does *something intuitive* instead of looking broken.
- **File-size column right-aligned**: the Size column in the file table now right-aligns cell text AND header — numeric data reads better flush-right (Finder convention).

### Added
- **Per-row size gauge for files**: file rows now render the same proportional background bar that folders already had, so the user can scan a directory and spot the largest files visually. Files are normalised against the max file size (green tint, `.systemGreen 0.10`), folders stay on their own scale (blue tint, `.systemBlue 0.12`), so a multi-GB folder can't flatten every file bar to invisible. Computed eagerly at listing time — no async cost since regular-file sizes are already populated.
- **Native PRO badge in the main menu bar**: PRO-only menu items now show macOS's native `NSMenuItemBadge` pill (the small rounded-rect shipped in macOS 14's menu redesign) on the trailing edge of the item so Free users can see at a glance which actions require an upgrade. Uses AppKit's own pill rendering rather than a text suffix, so it reads as a system badge — not a hack — and stays visually light. The badge is set/cleared by a `ProBadgeMap` walker that runs on app launch (after SwiftUI finishes building the menus), after licence validation, and on `didBecomeActiveNotification` (to catch re-built menus after background). Pro / Trial / Team tiers see no badge — it's cleared the moment the tier elevates.
- **Theme presets now paint the full surface** (background, alt-row, selection, text), not just the accent colour. Previously switching between "Dracula" / "Nord" / "One Dark" only tinted a few SwiftUI controls (and the HSplitView divider) because `AppTheme` only carried `scheme` + `accentHex`. Each preset now declares six colour tokens (`background`, `surface`, `selection`, `altRow`, `textPrimary`, `textSecondary`) with canonical hex values from each palette's project page. `ThemeManager` exposes both SwiftUI `Color?` and AppKit `NSColor?` accessors (`nil` = fall back to system defaults so the `System Default` preset stays indistinguishable from vanilla AppKit). The main window paints `background`, and `FileTableView` uses a new `ThemedTableRowView` to paint `altRow` and `selection` via `drawBackground`/`drawSelection` overrides. Switching Dracula ↔ Nord ↔ Solarized Dark ↔ GitHub Light now produces a visibly distinct look, not just an accent swap.

### Performance
- **File copy progress**: `FileManager.default.attributesOfItem(atPath:)` was called on MainActor after each file copy — on iCloud / network volumes this blocked the UI for 100+ ms per file. Moved to a detached Task so the progress UI stays responsive.
- **PanelView total-size badge**: the Smart Folder path bar's total size used `items.compactMap(\.size).reduce(0, +)` without filtering the `..` parent entry and without `.lazy`. Now uses `items.lazy.filter { $0.name != ".." }.compactMap(\.size).reduce(0, +)` matching the neighbouring code and avoiding intermediate arrays on every body evaluation.
- **LargeFileFinder formatter allocation**: `ByteCountFormatter` was allocated per row; replaced with a static shared instance.
- **FolderReportView stats**: `totalFiles` / `totalDirs` / `totalSize` / `typeBreakdown` were computed properties that re-scanned `reportEntries` on every body render (O(n) × 3 + O(n log n) sort). Computed once off-main when the scan finishes, cached into a `ReportStats` struct.

### Changed
- **Keyboard — Copy Path to Opposite Panel**: default shortcut moved from **Ctrl+← / Ctrl+→** to **Opt+← / Opt+→**. Ctrl+←/→ is reserved by macOS Mission Control (*"move a Space"*), so the app never saw the event on machines with multiple Desktops. Opt+←/→ is safe outside text fields (text-responder guard is already in place).
- Toolbar status center: removed the outer `Capsule` background that overlapped macOS's native toolbar item container, producing a visible "two ovals" artefact around the `Unifyl v1.0.3 FREE` badge. The inner FREE/PRO capsule is unchanged.
- Function-key bar Terminal toggle: replaced the rounded teal pill (whose corners got clipped by the surrounding bar chrome) with a full-height rectangular cell that matches the F2–F8 cells. Solid teal when the bottom terminal is open, `teal.opacity(0.18)` when closed.

### Reliability
- `IconThemeManager.externalThemesDirectory` / `directory(for:)`: replaced the `urls(for:…).first!` force-unwrap with a safe `?? ~/Library/Application Support` fallback. Fixes a theoretical-but-real crash path on sandbox-container corner cases.
- `RemoteConnection.baseURL`: replaced the `URL(string: "<scheme>:///")!` fallbacks with a `schemeFallback` helper that never crashes, even when `URLComponents.url` is nil and the fallback itself fails to parse.

### Build
- Styled DMG generation moved off `create-dmg.sh`'s Finder AppleScript (which hangs in non-GUI shells) onto `dmgbuild` + a pre-made `.DS_Store` via `scripts/dmg_settings.py`. New `resources/dmg-background.png` (600×400 soft gradient with drag-hint arrow).
- `release.sh` now notarises + staples the `.app` bundle itself before DMG creation so offline first-launch Gatekeeper works without round-tripping to Apple.
- New `scripts/check-docs-sync.sh` (wired into `make lint`) scans user-facing docs and the in-app help HTML for references to the retired **Ctrl+←/→** shortcut so future contributors don't silently reintroduce the Mission Control conflict.


## [1.0.3] — 2026-04-17

Large-scale quality release. Focus areas: data integrity, Swift 6 concurrency safety, Korean/CJK filename correctness, Total Commander keyboard compatibility, and discoverable UI.

### Added
- **Total Commander key compatibility**: F2 = Rename, Shift+F5 = Duplicate in same folder, Alt+F5 = Pack, Shift+F9 = Pack (alt), Alt+F9 = Unpack, Shift+F4 = New text file + edit, Shift+F6 = Rename (alt), Ctrl+U = Swap left/right panels, Ctrl+← / Ctrl+→ = Copy path to opposite panel *(later changed to Opt+← / Opt+→ because of the macOS Mission Control reservation — see Unreleased)*, Alt+Shift+Enter = Compute all folder sizes, Ctrl+\ = Go to volume root, Ctrl+B = Branch view (flatten subdirectories), Ctrl+M = Multi-rename alias, Insert = Toggle select + advance cursor.
- **Volume / drive picker** in the toolbar — click the external drive icon to switch the active panel between any mounted volume (boot disk, USB, Thunderbolt, network). Auto-refreshes on mount/unmount/rename. Shows per-volume free space and removable/ejectable flags.
- **Bottom terminal toggle button** in the function-key bar (teal pill, right side) so users discover Cmd+Opt+T without memorising the shortcut.
- **Toolbar status center**: replaces the static "Unifyl" label with product name + version badge + Free/Pro tier capsule + active-panel selection/free-space summary. No more wasted toolbar space.
- HWP / HWPX (Korean Hangul Word Processor) to PDF conversion via LibreOffice's built-in HWP import filter.

### Changed
- **Pro price**: $49.99 → **$39.99**. Landing page, in-app upgrade sheet, and localized strings updated.
- Incremental backup: failed files are now tracked individually with reason, shown in a "View failed…" sheet with copy-to-clipboard and Retry Failed button. Previously only a count was surfaced.
- `SecurityBookmarkManager.saveBookmark` is now `@discardableResult Bool`; load/save path logs real failures via `OSLog` instead of silently returning `nil`.
- Git panel: shell errors capture stderr and log via `UnifylLogger.git` so an empty Git panel can be distinguished from a corrupt repo or missing `git` binary.
- OAuth token storage: decode failures (e.g. after a schema change) delete the corrupt entry so the user gets a clean re-sign-in prompt rather than an infinite retry loop.
- PDF conversion error message now names the likely cause for HWP failures (LibreOffice branch, missing filter) and points users at Console.app for the underlying LibreOffice stderr.
- Sparkle auto-update failures now show a targeted alert explaining how to enable "App Management" in System Settings, with a one-click "Open Settings" button.
- First launch: one-time prompt guides users to enable "App Management" for auto-updates. Dismissed permanently after first interaction.

### Fixed
- **Critical — data integrity**: Checksum verification treated compute errors as a literal `"Error"` hash string, so typing `"error"` into the verify field falsely matched. Errors now live in a separate dictionary and verify no longer compares against failure strings.
- **Critical — mass delete risk**: `DirectorySyncEngine` and `DiffEngine` silently swallowed `contentsOfDirectory` errors via `try? … ?? []`, meaning a permission-denied folder looked empty and generated a mass-delete plan under one-way sync. Both engines now throw on unreadable directories.
- **Critical — silent move failures**: AI Organization "Apply recommendation" used `try? fm.moveItem(…)`, so failed moves vanished while the user saw "organized". Failures are now collected per-recommendation, surfaced in the UI, and the recommendation is kept for retry.
- **Critical — duplicate false positives**: Vision FeaturePrint failure in the Duplicate Detector was being treated as `distance = 0`, causing unrelated images to be flagged identical. Errors now return `nil` and the pair is skipped.
- **Critical — FSEvents use-after-free**: `PluginFSEventsWatcher` passed `passUnretained(self)` to the C callback; a callback firing after `deinit` dereferenced freed memory. Switched to `passRetained` + explicit `release()` in `stop()`.
- **Critical — NSFileCoordinator race**: `FileOperationManager` called `coordinator.cancel()` concurrently from three tasks. Serialized through a new `CoordinatorBox` with NSLock; watchdog and timeout merged into one task.
- **Critical — Enterprise audit log compliance**: `AuditLogger` silently swallowed persistence failures (a SOX/GDPR incident). Now emits `Logger.fault` and the load path distinguishes "first run" from "decode failed".
- **Korean filename corruption on cloud**: APFS stores CJK filenames in NFD (decomposed) but S3, Dropbox, OneDrive, WebDAV, SMB, SFTP, FTP, and Google Drive all store NFC. Added `String.asRemoteFilename` normalization to every remote adapter — Korean/Japanese/Chinese filenames now round-trip correctly.
- **Legacy CJK text preview**: Text viewer and AI content extractor only tried UTF-8 and isoLatin1, turning EUC-KR/CP949/Shift-JIS/GB18030/Big5 files into mojibake. Added `TextEncodingDetection` with BOM sniffing + NSString statistical guess + CJK fallback chain.
- LogViewer DispatchSource race: event handler could read a just-closed FileHandle (previously band-aided with a 50 ms sleep). Replaced with `setCancelHandler` for deterministic close-after-cancel.
- Cross-volume move rollback: if removing the source AND cleaning up the partial copy both failed, the original error was rethrown and the user never heard about the duplicate file. Now throws `CrossVolumeMoveRollbackFailed` with both paths.
- S3 `deleteDirectory`: `try? deleteObject(markerKey)` silently hid real errors. Marker-not-found is now expected and benign; other errors propagate. Added `ObjectStorageError.invalidURL` case.
- `VectorIndex` batch upsert: silent `try? ROLLBACK` on failure left the DB with an open transaction that locked all subsequent writes. Now logs `Logger.fault` so the caller can reset the connection.
- `AppViewModel` auto-dismiss and cloud-download tasks are now stored as `Task` properties and cancelled before replacement, preventing stale timers from clearing newer notifications.
- `AudioMetadataView`: leftover `.bak.<ext>` file after successful metadata save is now logged (was silently ignored, leaving ghost files in the user's folder).
- `DateFormatter` instances in 9 UI call sites now set `.locale = .autoupdatingCurrent`; filename/CSV export formatters explicitly pin `en_US_POSIX` for stability.
- `NSAlert` hardcoded button titles ("OK", "Cancel", "Replace", "Go", "Open Download Page") now localized via `NSLocalizedString`.
- `URL(string: …)!` force-unwraps (11 sites) replaced with `StaticURL.make(…)` that crashes loudly at file/line on malformed input.
- `SyntaxHighlighter` `try!` regex compilation now goes through a `makeRegex` helper with precise error reporting.
- Workspace layouts, keybindings, macros, automations, and OAuth tokens: persistence failures are logged via appropriate `UnifylLogger` category (was previously silent — settings/macros would vanish between launches with no explanation).

### Developer
- DEBUG builds bypass the real login keychain and use `~/Library/Application Support/<bundle-id>/debug-keychain.json` instead. This eliminates the "Unifyl wants to access key 'Unifyl'" password prompt that appeared on every rebuild because ad-hoc code signatures change between builds. Release builds are unchanged.

### Added (respin)
- **Terminal pin / follow mode**: every terminal session defaults to 🔒 pinned (stays in its original directory — safer for in-flight commands). Click the lock icon in the prompt bar to switch to ⇆ follow mode, where the terminal's current directory tracks the active panel (subfolder navigation, "..", Back/Forward, Tab panel switch, direct panel click). Per-tab state so you can mix pinned and follow sessions.

### Fixed (respin)
- Bottom terminal output lag: `pwd`, `ls`, etc. used to appear only after the NEXT command was typed — `Process.terminationHandler` blocked on `readDataToEndOfFile()` until something else unblocked the OS pipe. Replaced with incremental `readabilityHandler` draining + non-blocking `availableData` on termination. Output now appears instantly.
- Panel click regression introduced by the initial terminal-sync observer: `.onChange(of: activePanel)` attached directly to `MainWindowView` made the whole view tree depend on both panels' paths, causing the bridged `NSTableView` inside `FileTableView` to be invalidated mid-click and dropping the click. Observers relocated to a zero-size helper view (`TerminalSyncObservers`) so they no longer pollute the main dependency graph.

### Fixed (respin build 4)
- **Critical — auto-update configuration lost**: the previous 1.0.3 build (CFBundleVersion 3) shipped without `SUFeedURL` and `SUPublicEDKey` in `Info.plist` because `project.yml` contained a duplicate orphaned `info:` block that clobbered the Sparkle settings during `xcodegen generate`. Users who installed the earlier 1.0.3 would not have received any future auto-updates. `project.yml` has been corrected (single `info:` block with feed URL restored and EdDSA public key uncommented). Users on the earlier 1.0.3 must reinstall from the replaced DMG to regain auto-update capability.
- Release pipeline now stapler-staples the `.app` bundle itself (in addition to the DMG) so first-launch Gatekeeper validation works offline.


## [1.0.2] — 2026-04-16

### Fixed
- **Critical**: Sparkle auto-update failed with "An error occurred while launching the installer" because the release re-signing step stripped Sparkle's internal entitlements (Autoupdate's `com.apple.application-identifier`, XPC-services' empty-dict entitlements). The release script now passes `--preserve-metadata=entitlements,flags` when signing every nested Sparkle binary. **Users on 1.0.1 must download 1.0.2 manually** — their existing Sparkle install can't apply updates.

## [1.0.1] — 2026-04-15

### Added
- Thumbnail grid (⌃G): full parity with list view — multi-select (Shift/Cmd), rubber-band drag-select, multi-file drag to Finder/other pane, drop target with Option-to-copy modifier.

### Changed
- Thumbnail grid rewritten on `NSCollectionView` for native multi-file drag (SwiftUI `.onDrag` single-provider limitation).
- Plugin folder watcher switched to `DispatchSourceTimer` so debounce cancellation is queue-agnostic and no longer races with deinit.
- Automation scheduler (`TriggerScheduler`) migrated to `DispatchSourceTimer` with a lock-protected storage; timers are now reliably canceled on scheduler deallocation.
- `ArchiveEngine` no longer shells out through `/bin/sh -c` — archive tools (unzip, tar, ditto, zip, 7z) are invoked directly with argv, removing shell reinterpretation surface (IFS, locale, NUL truncation).
- LibreOffice PDF conversion now captures stderr and logs the failure reason instead of silently returning `false`.
- Regex-rename match highlight uses the accent color instead of hardcoded blue so it stays legible in dark mode and honors the user's theme.

### Performance
- Spotlight search: `resourceValues` fetch moved off MainActor. Thousand-result searches no longer block the UI for hundreds of milliseconds while stating each hit.
- Enhanced duplicate scan: pair scoring now runs in a `TaskGroup` per size bucket instead of serial `await`, cutting wall-clock time on large same-size groups from O(n²) to roughly O(n²/cores).

### Fixed
- **Natural-language command execution**: `move`/`copy`/`delete` operations no longer swallow errors. Partial failures are collected and surfaced in a summary alert so the user knows which files didn't move.
- **"Keep Best" duplicate removal**: failed `trashItem` attempts no longer hide the cluster from the UI; users see an alert listing the files that couldn't be trashed and the cluster remains visible.
- **Hardlink replacement**: when `linkItem` fails AND the rollback also fails, the error now includes the orphan `.unifyl-backup` path so the original isn't silently lost.
- **PDF "Replace" flow**: if removing the existing file fails, the conversion aborts and the user sees a clear alert (previously succeeded silently, leaving a stale PDF).
- **MultiRename rule list**: replaced `ForEach($rules)` binding with index-based iteration to avoid transient binding corruption during reorder/delete.
- **Open-in-Finder failures**: `NSWorkspace.open` return value is checked and failures are logged (silent no-ops previously).
- Removed stale `nonisolated(unsafe)` on `AudioPlayerViewModel.player`/`displayLink`; both now rely on the class-level `@MainActor` isolation.
- File move/paste-cut now falls back to copy-then-remove when the source and destination are on different volumes (EXDEV). Previously move failed outright and the user had to redo the operation manually.
- File permissions dialog: error/success messages now carry a status icon (warning/checkmark), so the state is legible for users with color-vision deficiencies.
- Filter bar (⌘F): pressing Esc now also releases focus from the text field, so arrow-key navigation returns to the file list immediately instead of getting stuck on a hidden input.
- SFTP remote file operations refuse paths containing C0 control characters, blocking a class of remote-shell injection via attacker-crafted filenames on the SSH server.
- Archive extraction's zip-slip check now uses a `destPath + "/"` boundary (so siblings like `/tmp/extract-evil` don't match `/tmp/extract`) and enumerates hidden files too.
- Periodic scan scheduler replaced the NSString tilde-expansion bridge with native home-directory expansion.
- Batch Extension dialog strings are now localizable (previously hardcoded English).
- LicenseManager no longer silently ignores Keychain save failures — each failure is logged so "randomly downgraded to basic" reports can be correlated.
- Workspace decode failures are logged instead of being silently swallowed by `try?`; makes "lost tabs after update" reports actionable.
- `CheckForUpdatesViewModel` migrated from `ObservableObject`/`@Published` to `@Observable` for consistency with the rest of the app.

### Security
- `.netrc` credential file for FTP now rejects hosts/usernames/passwords containing newlines or control characters (previously a `\n` in the password could inject a second `machine` line that curl would honor for other hosts). File is also created with `O_CREAT|O_EXCL|O_NOFOLLOW` so a pre-existing symlink at the temp path can't redirect the write.
- SFTP askpass script now uses `O_CREAT|O_EXCL|O_NOFOLLOW` + additional random suffix so a predictable-UUID collision can't preempt the temp path with a symlink attack.
- FTP operations (LIST/STOR/RNFR/RNTO/DELE/RMD/MKD) now reject paths with C0 control characters, matching the SFTP guard. Previously a `\n` in a remote filename could smuggle a second FTP command.
- Companion "Open in Terminal" command refuses paths containing quotes, backslashes, or control characters before dispatching to AppleScript — a crafted filename can no longer break out of the AppleScript string literal.
- `LSApplicationCategoryType`, `NSAppleEventsUsageDescription`, and folder-access usage strings added to Info.plist so macOS prompts the user with intent when Unifyl triggers Terminal/Pages/Numbers automation or accesses Desktop/Documents/Downloads/removable volumes.

### Performance (continued)
- `GitStatusProvider` coalesces concurrent lookups for the same directory into a single in-flight task; rapid bursts (e.g. switching panes while git status is populating) no longer fan-out into redundant `git` subprocesses.

### Fixed (continued)
- LibreOffice PDF conversion and AboutView's git-info probe close the read handle of the stderr/stdout pipe after `readDataToEndOfFile`, eliminating a file-descriptor leak per conversion / app launch.
- `AppViewModel.loadBookmarks` now falls back to `URL(fileURLWithPath:)` when a stored bookmark string is a raw path — previously `URL(string:)` dropped bare paths silently and the user's bookmark list could appear to "shrink" after a version.
- WebDAV operations (COPY/MOVE/DELETE/MKCOL) now reject paths with C0 control characters. Previously a `\n` in a filename could smuggle additional HTTP headers via the `Destination:` line.
- Archive tool output (tar/7z/unzip listings, etc.) decodes with UTF-8 → EUC-KR → Shift-JIS → Latin-1 fallback. Archives with non-UTF-8 filenames from legacy Windows no longer return empty listings.
- `TerminalSession` migrated from `ObservableObject`/`@Published` to `@Observable`; its SwiftUI consumers use `@Bindable` to match the rest of the app.
- Thumbnail cache: `get` now moves the hit entry to the MRU tail (previously FIFO masquerading as LRU) and eviction uses `>` correctly so the cache can't hold `maxEntries + 1`. Added a `clear()` entry point for future memory-warning handling.
- Incremental backup: mtime comparison now tolerates 1 second of drift so cross-filesystem copies (APFS ↔ SMB/FAT) don't flag unchanged files as modified. Size change still forces a re-copy inside the tolerance.
- Media converter: ffmpeg subprocess is now terminated when the Task is canceled (closing the sheet, pressing cancel) and the partial output file is removed so users don't find half-written artifacts.
- File-table date column explicitly uses `Locale.autoupdatingCurrent`; the date format now follows the user's system locale preference instead of the app's default.
- Icon-theme color components are clamped to `[0, 1]`; an out-of-range value in a custom theme file no longer produces an undefined NSColor.
- `RemotePollingWatcher` stops polling after 5 consecutive errors instead of silently converting failures into "empty directory" notifications. A dropped SFTP/FTP session is now observable.
- `LemonSqueezyClient` uses a dedicated `URLSession` with 15 s request / 30 s resource timeouts. A hung license server can no longer freeze the license check indefinitely.
- WebDAV curl invocations add `--max-time 300` so a stalled transfer body (not just a bad connect) is bounded.
- `FileIndexer` logs vector-store upsert/remove failures instead of `try?`-swallowing them, so silent indexing gaps become diagnosable.
- AsyncStream lifecycle: `AIClassifier.classifyBatch`, `SmartTagger.suggestTagsBatch`, `DuplicateDetector.findDuplicates`, and `SummarizationEngine.summarizeBatch` now install `continuation.onTermination` handlers that cancel the internal Task. Previously a consumer dropping the stream left the expensive work running until completion.
- `GitStatusProvider`: detected HEAD (checked-out tag or commit) is now displayed as `(detached <short-sha>)` instead of the literal string "HEAD" that git returns for `--abbrev-ref`.
- `VectorIndex.allVectors`: replace `LIMIT \(batchSize) OFFSET \(offset)` string interpolation with bound SQLite parameters. No live injection path (values are internal Int), but avoids the footgun for future refactors.
- Selective extraction from ZIP/tar/7z archives with non-ASCII entry names (Korean CP949/EUC-KR, Japanese Shift-JIS, etc.) now falls back to full extraction via `ditto` / libarchive. Previously `unzip` would fail with "caution: filename not matched" for every entry because macOS InfoZip 6.0 lacks a `-O` encoding flag; users got the archive as a virtual folder but couldn't extract any file out of it.

## [1.0.0] — 2026-04-13

Initial public release.

### Highlights
- Dual-pane file management with tabs, workspaces, and keyboard-first navigation.
- Press **F3** — 10 built-in viewers (PDF, code, image, hex, video, audio, web, markdown, media info).
- **AI-powered** semantic search, smart tagging, and intelligent rename — all local on Apple Neural Engine.
- **30+ tools**: hex editor, git panel, media converter, EXIF editor, PDF tools, folder heatmap, app uninstaller.
- **Archive virtual folders** — browse ZIP, 7z, TAR, RAR as regular folders.
- **Advanced search** with multi-criteria, regex, tags + Smart Folders for batch operations.
- **Developer tools**: git integration, log viewer, port viewer, process inspector, Docker explorer.
- **Themes & customization**: 12 built-in themes, custom editor, SVG icon packs, 120+ keyboard shortcuts.
- **Cloud & remote**: FTP, SFTP, WebDAV, S3, Google Drive, Dropbox, OneDrive.

[Unreleased]: https://github.com/goodbug89/Unifyl.app/compare/v1.0.4...HEAD
[1.0.4]: https://github.com/goodbug89/Unifyl.app/releases/tag/v1.0.4
[1.0.3]: https://github.com/goodbug89/Unifyl.app/releases/tag/v1.0.3
[1.0.2]: https://github.com/goodbug89/Unifyl.app/releases/tag/v1.0.2
[1.0.1]: https://github.com/goodbug89/Unifyl.app/releases/tag/v1.0.1
[1.0.0]: https://github.com/goodbug89/Unifyl.app/releases/tag/v1.0.0

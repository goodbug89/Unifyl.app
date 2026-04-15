# Changelog

All notable changes to Unifyl will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

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

[Unreleased]: https://github.com/goodbug89/Unifyl.app/compare/v1.0.1...HEAD
[1.0.1]: https://github.com/goodbug89/Unifyl.app/releases/tag/v1.0.1
[1.0.0]: https://github.com/goodbug89/Unifyl.app/releases/tag/v1.0.0

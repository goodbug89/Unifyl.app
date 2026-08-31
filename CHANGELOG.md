# Changelog

All notable changes to Unifyl will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [1.7.3] — 2026-08-31

### Security

- **The keyboard monitor logged every character you typed.** An app-wide
  `NSEvent` keyDown handler wrote `chars=` to the unified log on every
  keystroke — including into password fields, since an in-process local
  monitor is not covered by macOS secure event input, and `.notice` persists
  to disk where any process running as you can read it. The key code and
  modifier flags it actually needs are still logged. If you have been running
  a build before this one, `sudo log erase --all` clears the history.
- **119 log sites wrote your paths, file names, search queries and error text
  with redaction disabled.** `privacy: .public` opts a value out of the
  unified log's redaction; Cocoa error descriptions are the worst of it,
  because file errors routinely embed the path that failed.
- **FTPS could fall back to plaintext while the UI still showed a lock.**
  curl's `--ftp-ssl` is opportunistic: a server that declines AUTH TLS was
  talked to in the clear. It now requires TLS.
- **WebDAV chose its scheme from the port number** — 443 meant https, and
  anything else meant plain HTTP with Basic auth. A TLS server on 8443 or
  Synology's 5006 had your password sent unencrypted with no way to say
  otherwise. TLS is now a checkbox that defaults to on; connections saved
  before this keep their existing behaviour rather than being switched behind
  you.
- **The connection test passed your password on the command line**, visible
  through `ps` to anything else running as you. It writes a 0600 `.netrc`,
  which the FTP provider itself has done since it was written.
- **Plugin signature checks did not descend into nested code**, so a bundle
  with a correctly signed executable could carry an unsigned dylib and pass.
  Nested-code, strict and all-architecture validation added, plus a
  re-validation after load to cover the swap window on a user-writable
  directory.
- **The companion server ignored its own "Allow local network only" setting**
  and accepted from every interface, and its 6-digit pairing code had no
  attempt limit. Both fixed.

### Fixed

- **Undo of a Multi-Rename across several folders half-worked.** The record
  rebuilt every path under the first file's directory, so a rename over a
  marked set spanning folders (Search results, a Smart Folder) put some files
  back, left the rest renamed, and reported success.
- **⌘Z could undo an operation older than the one you were looking at.** A
  refused or failed undo left its record popped off the stack, so the next
  ⌘Z reached past it.
- **Trashing from Duplicate Finder, Large File Finder and Reclaim Space left
  no undo entry at all** — Finder's own Put Back was the only way back.
- **Backup files outlived the undo entries that could use them.** A hex edit
  of a 1 GiB file left 1 GiB beside it forever once its history entry aged
  out. Evicted entries now take their backups with them.
- **Disconnecting a remote panel did not stop transfers already running.**
  They kept writing against a session the panel had discarded.
- **Directory Diff destroyed newer files.** The plan was built when you
  pressed Compare; a file created or edited on the other side since then was
  replaced outright. It goes to the Trash first.
- **Duplicate Finder trusted a scan that might be stale.** A file edited after
  the scan is no longer the duplicate the sheet claims it is; those are
  skipped with a rescan prompt.
- **Archive operations could act on the wrong panel.** Pressing Tab during a
  long archive job redirected the rest of it — a folder move out of an archive
  silently became a copy.
- **Up to three permission dialogs on a brand-new install.** The size column
  enumerated Desktop, Documents and Downloads unasked on a panel that opens in
  your home folder, stacking macOS prompts on top of the welcome screen.
- **The Tips sheet and the auto-update hint landed on top of onboarding.**
- **Two migration paths could destroy what they were migrating.** A refused
  keychain write was followed by deleting the original; the bookmark store
  cleared its old location before confirming the new one was written.
  Upgrading users could also have every restored tab judged unrestorable and
  the collapsed workspace saved over the real one.
- **Light themes made filenames nearly unreadable.** There was one hardcoded
  palette, tuned for dark backgrounds, with no awareness of the appearance —
  on GitHub Light or Solarized Light every file-type colour sat near 1.3:1
  against the row. There is now a light palette that clears 4.5:1 everywhere.
- **Marked filenames failed contrast in all twelve built-in themes** on the
  cursor row, and five themes drew a toolbar rule in the same colour as the
  surface beneath it. The mark colour is now lifted against whatever it is
  actually drawn on, and the Theme Editor checks contrast as you edit.
- **Splitting an empty file reported success** while writing nothing.
- Archive-mode rename and New Folder skipped name validation entirely; no path
  length was checked against PATH_MAX; Folder Sync showed "Click Compare"
  after comparing two identical folders; an empty archive offered a drop
  target and F7, which archives refuse; Duplicate Finder had no size floor, so
  every empty file on the disk formed one enormous group.
- The Mac App Store folder picker told users, in 15 languages, to grant more
  folders via "File → Open Folder…" — a menu item that has never existed.

### Accessibility

- **Marked, git status, cloud-only, download progress and tags were conveyed
  by colour alone.** Each is now stated in words, which also helps anyone who
  cannot separate the hues.
- **The thumbnail grid had no accessibility information whatsoever** —
  VoiceOver read a wall of unlabelled images.
- **Nothing was ever announced.** Every operation, count and refusal completed
  in silence; they are spoken now.
- The App Store description claimed "full VoiceOver support, WCAG 2.1 AA" in
  all 15 languages. That was not true, and the line now says what is:
  every action is reachable from the keyboard.

### Fixed (earlier in this cycle)

- **Copying a folder no longer empties its symbolic links.** A link inside a
  copied folder — the kind every framework bundle, Homebrew tree and Python
  venv is built from — arrived at the destination as an empty folder, and
  everything it pointed at was silently missing while the copy reported
  success. Drag-and-drop was never affected; F5 and ⌘D now behave the same way
  it always did. Pipes and sockets are skipped with a reason instead of
  aborting the batch, and they can no longer hang a speed-limited copy or a
  checksum forever.
- **Folder Sync with a server finally finishes.** Nothing preserved a file's
  modification time across a transfer, so the next comparison saw every file
  as changed and copied the whole set again — in alternating directions when
  syncing both ways, leaving a Trash copy behind each time. Dates read from a
  server are also no longer assumed to be from this year, which could date a
  file from last December into the future and overwrite a newer local copy
  with an older remote one.
- **Every menu item and shortcut in the App Store edition now does something.**
  Features macOS sandboxing doesn't allow (Git, Docker, SSH tunnels, scheduled
  tasks, automations, terminals, media conversion) were still listed in menus
  and the command palette, and clicking one offered to sell it. They're hidden
  in that edition, and nothing that edition can't do can be purchased in it.
  The cloud list shown there names only the services it can actually reach, and
  the PDF converter explains the limit instead of prescribing an install that
  can't work.
- Closing a second window no longer kills the keyboard in the one you keep —
  F-keys and custom shortcuts stopped working entirely after using ⌘N.
- A ZIP larger than 4 GB no longer browses as empty (it says what it is), and
  a folder deleted while the App Store edition was closed no longer takes a
  different tab's place — with its pin, title and sort order — on the next
  launch.
- Scheduled automations survive sleep: a daily rule armed before your Mac
  slept used to fire hours or days late. Directory Diff's "Sync All" copies
  the newer side, as its description always claimed. A wrong system clock can
  no longer permanently expire a trial or a license's offline grace period.
  Archive edits can't be discarded because the archive was made in another
  timezone.

- **Cancel now cancels.** The queue bar's ✕ never reached a running
  copy/move — no code path ever marked an operation as running, so the button
  only signalled ops it believed were active, which was none of them; the
  ⌘D/⇧⌘D copy-to-other-panel paths additionally polled neither Pause nor
  Cancel at all, and a cancelled operation left a frozen ghost row in the
  queue bar forever. Executors now advance the queue state, poll the queue
  between items, mark cancelled operations terminal, and clean up the
  half-written file a mid-file cancel leaves behind. Also newly cancellable:
  SFTP/FTP/SMB/WebDAV transfers (a stalled child process now dies with the
  task instead of waiting forever), remote batch download/upload (which had
  no progress overlay or cancel at all), content search (Cancel used to hide
  the results while the scan ran the whole tree), AI indexing (a Stop button
  exists now — accidentally indexing your home folder was previously
  unstoppable), Folder Sync's compare phase, copying a file out of a large
  archive, and the connection test probe.
- **Archive write-back can no longer double-fire.** Saving an edited archive
  entry is triggered by app activation, launch recovery and quit; two of
  those arriving while a slow archive rewrite was still in flight could run
  a second rewrite of the same archive concurrently, or crash on a stale
  index. Flushes are now serialized and sessions matched by identity.
- **Folder Sync stops re-transferring CJK files forever.** Local names come
  from the filesystem in decomposed Unicode, remote names in precomposed;
  the comparator matched them as different files, so every Korean-named file
  present on both sides was copied again on every single sync. Keys are now
  normalized on both sides.
- **Archive extraction rejects hostile entry paths again.** The pre-extract
  guard re-checked an already-filtered listing (so it could never fire) and
  the post-extract check couldn't see files that escaped the destination —
  protection against `../` entries had quietly devolved to whatever the
  external unarchiver does. The raw entry names are checked now.
- **Failures stopped being silent.** Shift+F5 duplicate reports which items
  failed instead of showing a partial success; an offline icon-pack download
  no longer reports "complete (0 icons)" and wedges itself as installed;
  OAuth token and connection-password Keychain failures are logged and
  surfaced instead of signing you out mysteriously on the next launch;
  Ctrl+P on a folder says why nothing happened; and when macros, layouts,
  shortcuts, Saved Searches, connections or the workspace can't be written
  to disk, a banner says so before quitting loses them.
- **Long Korean filenames are no longer over-rejected.** The rename length
  check counted every name in decomposed form (right for HFS+, verified by
  real creates), but APFS accepts 255 UTF-16 units as supplied — Korean
  names were refused from 86 characters. The check now knows which
  filesystem it's talking to.
- An instantly-failing ffmpeg conversion could hang the converter sheet
  forever (termination handler installed after launch); closed windows no
  longer keep background folder-size scans alive; failed SQLite opens leak
  no handle; 16 new user-facing strings ship in all 15 languages, 3 dead
  keys and 3 duplicate Korean entries removed.

- **The last English in a Korean UI.** Four shapes of text that no
  localisation check had ever looked at, found by reading three fresh
  screenshots: an enum's raw value used as a tab label ("Semantic / Index /
  Classify"), a message assigned to a status variable and shown later ("Scanned
  6 items"), a literal handed to a `title:`-style argument ("Advanced Search"),
  and a literal inside a ternary ("Free: …"). 463 such sites are localisable now
  and the 390 strings behind them are translated in all 15 languages; the audit
  catches all four shapes from here on. Also: a `%` that was meant literally in
  "content overlap" was being read as a format specifier.


## [1.7.2] — 2026-08-23

### Added

- **Every language is complete again.** The 620 strings that were in no
  catalogue at all are now translated into all 13 secondary locales — German,
  French, Spanish, Italian, Portuguese (Brazil), Russian, Turkish, Arabic, Thai,
  Vietnamese, Japanese, Simplified and Traditional Chinese — 8,060 entries,
  each checked to carry the same format specifiers as its English source. Every
  locale holds the same 2,072 keys.
- **596 more strings now speak Korean.** They were in no catalogue at all — not
  missing a translation, absent as keys — so every locale rendered them in
  English while a locale-parity check reported all 15 languages in step. Settings
  ▸ General was the clearest case: the row that chooses the app's language was
  itself labelled "App Language". Also the connection sheet, Automations,
  Multi-Rename, Directory Diff, Folder Sync, the theme editor, Advanced Search,
  backups, the converters, the viewers, tab and header menus, and the panel's own
  status line. The other 13 languages fall back to English until they are
  translated, as they already do for the rest of the catalogue.
- **The app answers in Korean where it used to answer in English.** Go to Path,
  the PDF overwrite and replace-failed dialogs, the "LibreOffice required"
  prompt, the duplicate-cleanup and natural-language partial-failure reports,
  File ▸ New Window, Unifyl ▸ Check for Updates…, and the overwrite dialog's
  message and "apply to all" checkbox. Thirteen of these were plain Swift
  strings that no translation could ever have reached, including the entire
  HWP/HWPX conversion failure path — a Korean-market feature that explained
  itself only in English.

- **Column widths are now per panel, with a choice.** They were a single shared
  set, which is right when you want both panes to match and wrong when you are
  comparing a wide listing against a narrow one. Settings ▸ General ▸ **Link
  column widths across panels** decides; it is on by default, so nothing changes
  unless you turn it off. A workspace saved by an earlier build gives both panes
  the widths you already had rather than resetting them.

### Fixed

- **Holding an arrow key stuttered.** AppKit asks every menu in the menu bar
  to update itself on every key press, while it searches for a key equivalent
  — not only when a menu opens. The PRO-badge delegate answered each of those
  requests by re-decorating the *entire* main menu, nine menus deep, with a
  localisation table rebuilt from scratch each time. Sampled at 16% of the main
  thread while an arrow key was held down, on top of the panel's own work.
  A menu now decorates only itself, the table is built once per language, and
  the delegate no longer hops through a task to do it.
- **Finder's "Open With", a folder dropped on the Dock icon, and `open -a Unifyl
  <folder>` all ignored the folder.** The app declared no document types, so it
  never appeared in Open With at all, and had no handler, so a path passed on
  the command line was discarded and a window opened wherever the last session
  left off. For a file manager that is the most ordinary way in. A folder is now
  entered; a file opens its folder with the file selected, the same as the
  long-standing "Send to Unifyl" Service — which now shares the code rather than
  keeping its own copy. Unifyl offers itself for folders but never takes the
  default away from Finder.
- **Right-clicking a column header did nothing.** The standard place to add or
  remove a column, or to put the widths back, was empty — the only route was
  Settings. The header now has that menu, including **Reset Column Widths** for
  undoing a drag that pushed the other columns off the edge.
- **Listing a large folder kept a core busy longer than it needed to.** The
  guard that keeps the unrequested size walk out of media libraries rebuilt its
  answer — three URLs and a set — for every single file it looked at. Sampling
  the app while it listed a home folder put that function at the top of the
  profile. It cannot change while the app runs, so it is computed once.
- **Opening your home folder asked for your Photos library.** The size column
  measures every folder row in the background, and measuring `~/Library` reached
  `~/Library/Photos` — a plain directory, not a `.photoslibrary` bundle and not
  under `~/Pictures`, so neither rule the guard had could see it, and macOS gates
  it behind the Photos service all the same (it holds the "Shared with You"
  library). Every earlier fix assumed the walk never got past `~/Pictures`, and
  every test agreed, because none of them knew this directory existed. Found by
  attaching a debugger while the dialog was up: four walker threads were all
  blocked in `open("~/Library/Photos")`. It is skipped now, like `~/Pictures`.
  Two smaller holes closed on the way: ⌥⇧Return sized every row with no filter
  at all (one filter now, shared with the automatic pass), and the walk no longer
  asks the enumerator to pre-read each entry's attributes, a read that happened
  before the guard could rule on it.

- **The pointer never showed a column could be resized.** The dividers dragged,
  but the cursor stayed an arrow the whole way across the header, so the only way
  to find out was to try. `NSTableHeaderView` installs those cursor rects itself
  normally; inside the SwiftUI hosting view it does not. The header now adds them
  and refreshes them whenever a column moves.
- **Only the file-name column could be resized.** Every other divider ignored
  the mouse and the cursor never changed over it. The table used
  `.firstColumnOnlyAutoresizingStyle`, which keeps the table's width tied to the
  pane by flexing the first column — so a drag on any other divider had nowhere
  to put the width and AppKit refused it. Column autoresizing is off now, so
  every divider works.
- **Column widths could not be set.** Dragging a column wider re-proportioned
  every column on the next layout pass, so a column pulled to 900pt came back at
  its 60pt minimum — "it returns to its optimal width". The app no longer adjusts
  column widths at all: a drag is left exactly as made, and the other columns are
  not touched. Pane resizing still flexes the first column only. Columns can now
  sum wider than the pane, which clips the trailing one; narrow a column or
  reorder to bring it back.
- **Widening a column made it narrower.** Column widths are shared by both panes,
  and the panel broadcast every width change it heard from AppKit — which posts
  the same notification whether the user dragged a divider, the app applied a
  saved width, or `sizeToFit()` squeezed the columns into a narrow pane. Only the
  first of those is a preference. So dragging Size wider in one pane pushed the
  new width to the other, whose narrower pane made `sizeToFit()` shrink it, and
  that shrunken value came back and overwrote the drag. Now only changes the app
  did not initiate are reported. Note the remaining ceiling is deliberate: there
  is no horizontal scroller, so a column can grow until the others reach their
  minimum widths — hide columns you don't need to give one more room.

- **Text painted in a colour that cannot be read.** `textMuted` tops out at
  2.36:1 against the lightest background token in the system and `textTertiary`
  at 3.97:1 — neither can reach the 4.5:1 a body label needs, on any background.
  They are decorative colours, and three pieces of real content were using one:
  the empty-state message under "nothing to show here", the palette's "no
  commands found", and the shortcut badges (⌘D and friends) that are the whole
  point of a palette row for a keyboard user. All three now use `textSecondary`
  — 4.74:1 to 5.72:1 where they sit. Audit F63 fails on either colour unless the
  call site marks itself `// decorative:`.

- **Command palette group headers were unreadable, and untranslated.** The
  headers used `textMuted`, which sits at 1.86:1 against the palette's surface —
  and lightening that surface in the fix above made it *worse*, not better
  (2.25:1 before). A 10pt label needs 4.5:1, so they now use `textSecondary` at
  4.74:1. They were also bare English literals, so a Korean or Japanese user read
  "FILE" over a list of translated commands; all nine category names are
  localised across the 15 shipping locales.

- **Arrow keys in the command palette jumped around instead of stepping.** ↓ walked
  `registry.search(query)` while the rows on screen were that list grouped by
  category. Those are two different orders, so the next item in search order sat
  somewhere else on screen — far below, then back above — and the auto-scroll
  chased it. Selection now indexes the same flattened order the rows are drawn in.
- **The palette's selected row was invisible.** It used `bgOverlay`, which had
  just become the palette's own surface — ΔL\* +0.0, the identical colour. It now
  uses `bgSelectedActive`, +6.9 above the surface, which is also the token that
  means "this has keyboard focus".

- **The command palette had no visible edge on dark themes.** It drew on
  `bgSurface` behind a 35% black scrim — but a black scrim cannot darken a
  backdrop that is already near-black, which every dark theme is. Measured in
  CIE L\*, the palette sat between ΔL\* +6.9 and **−6.3**: on Nord it was
  *darker* than the pane it covered, so it read as a dent rather than a panel,
  and its border managed 1.2:1 against its own surface. The palette now uses
  `bgOverlay` with a dedicated modal scrim and a brighter edge token, putting it
  at ΔL\* +7.5 to +15.2 across the built-in themes — lighter than its backdrop
  everywhere — with the border at 2.9:1.

- **The Edit menu showed the wrong command names, all badged PRO.** Undo read
  "Save Layout… PRO", Copy read "Text Tools… PRO", and basic operations —
  Cut, Copy, Paste, Select All — appeared locked. The shortcuts were right and in
  the right places because SwiftUI owns those; only the titles rotted.
  `ProBadgeMap` remembered each item's undecorated title in a static dictionary
  keyed by `ObjectIdentifier`, which is the object's address. SwiftUI destroys
  and rebuilds the `NSMenuItem`s behind `CommandGroup`/`CommandMenu`, a new item
  gets allocated on a dead one's address, and the stale entry then answered for
  it — so a recycled address made an ungated command render as whichever gated
  command used to live there. The badge is now stripped back off the item's own
  title instead of being remembered anywhere. Audit F60 rejects a long-lived
  static keyed by an object's address.

- **Holding an arrow key crawled instead of scrolling.** Moving the cursor made
  the app read the keychain, once per keystroke. `tableViewSelectionDidChange`
  assigned `appVM.activePanel` unconditionally, and `@Observable`'s setter
  notifies even when the value is unchanged — so an arrow key inside one pane
  invalidated every observer of `activePanel`, which `MainWindowView` reads in 42
  places. Its body reads `isTrialActive`, which read two uncached computed
  properties, each of which called `KeychainService.loadString`. A 15-second
  profile of a real held key had the main thread 68.1% busy, with
  `isTrialExpiredPermanently` alone accounting for about a second of it and the
  rest going to AttributeGraph updates and repeated full-window Auto Layout
  passes. Debug builds fared worse still: the file-backed dev keychain re-read
  and re-decoded its JSON store and called `createDirectory` once per service
  name on every get, putting `open()` at 4.4% of the profile and `mkdir` on the
  list at all. The panel now compares before assigning, the trial state is cached
  and invalidated where it is written, and the dev keychain resolves its path
  once and keeps the decoded store in memory. Re-measured on the same folder
  after the fix: the keychain is absent from the profile entirely. Audit F59
  rejects a keychain or filesystem read in an uncached getter that a view body
  can reach.


## [1.7.1] — 2026-08-19

### Security

- **The plugin loader accepted any self-signed or ad-hoc-signed native-code
  bundle.** Third-party plugins load from `~/Library/Application Support/Unifyl/
  Plugins` — a user-writable path — and are loaded as native code with the
  (non-sandboxed) app's full privileges, so the code-signature check is the only
  gate. It called `SecStaticCodeCheckValidity` with a nil requirement, which
  verifies only that the signature seal is unbroken and accepts ANY valid
  signature — including the ad-hoc (`codesign -s -`) and self-signed signatures
  an attacker produces in seconds. Verified: an ad-hoc-signed bundle passed. The
  check now pins `anchor apple generic`, requiring a real Apple-issued
  Developer ID / Mac App Store certificate (traceable and revocable), which
  rejects ad-hoc and self-signed bundles while still allowing legitimately
  distributed plugins. Fails closed if the requirement can't be built. Audit F50
  rejects any signature check that passes a nil requirement.
- **A filename could inject a shell command through the terminal's `$e` variable.**
  The inline and drop-down terminals expand `$F`/`$D`/`$S`/`$f`/`$n`/`$e` into a
  command and run it as `/bin/zsh -lc`. Every variable was shell-escaped except
  `$e` (file extension), interpolated raw on the theory that "extensions are
  safe". They are not: the extension comes from a filename the app didn't author
  (an archive entry, a remote share, a download), and `URL.pathExtension` rejects
  only whitespace and `/` — it preserves `` ` ``, `$( )`, `;`, `|`, `&`, `>`,
  none of which need a space to be dangerous. A file named
  `photo.jpg$(id>~/pwned)` made any command containing `.$e` execute the embedded
  command. Proven end-to-end against a real login zsh. `$e` is now escaped like
  every other variable; legitimate use (`convert $F out.$e`) is unchanged. Audit
  F49 rejects any terminal variable that reaches the shell unescaped, and a
  regression test runs the expanded strings through zsh to confirm no injection
  executes.

### Added

- **Four AI features left the command palette for the menus.** Natural Language
  Command, Summarize, Organize and Enhanced Duplicate Scan had a palette entry
  and nothing else — no menu item, and no way to assign them a shortcut.
- **Seven more commands can finally be given a shortcut** (Disconnect Remote,
  Undo History, the three AI tools, Statistics, Reclaim Space): they had no row
  in the shortcut catalogue at all.

- **Thumbnail Grid now behaves like the list.** Clicking a thumbnail moved the
  cursor *and wiped every mark you had made* — the grid's selection was bound
  straight to the mark set — so marking ten files and then clicking one left
  F5/F6/F8 acting on a single file. The grid also had no right-click menu at
  all, which put Copy Path, Permissions, Create Symbolic Link, X-Ray and the
  media tools out of reach in grid mode. Marks and the cursor are now separate
  and drawn differently, Insert toggles a mark, and both views build their
  context menu from the same code.

- **Colour tags and the 12 themes are in the command palette.** Tags could only
  be applied with ⌃0–⌃7, a binding documented nowhere a user reads; themes lived
  only in the toolbar's ⋯ menu, which the keyboard can't reach.

### Fixed

- **Two windows could be dragged narrower than their own contents** — the main
  window with the Connections sidebar open (the splitter stopped responding) and
  Directory Compare (the Actions column was clipped away).
- **Tabs jumped sideways as the pointer crossed the tab bar**, because each
  tab's ✕ was inserted on hover and changed the tab's width.
- **Three-Way Merge asked for three files through three separate dialogs**, for
  files the app was already showing. It pre-fills from the two panes now.

- **The F1–F8 bar, the folder-sync verdicts and the overwrite dialog's Replace
  button were English in every language.** `Text` given a plain string doesn't
  localize — meanwhile eight translated function-key strings had been sitting
  unused in all 15 locales, written for an older layout.
- **Three flows couldn't be finished without a mouse**: running a folder sync,
  deleting from Duplicate Finder, and using Clipboard History at all.

- **The panel filter bar rendered in the middle of the file list**, floating over
  the rows, because the stack it lives in was never given an alignment. It also
  outlived the folder it was typed for — entering a subfolder left the new
  listing mysteriously filtered — and when it emptied a pane, the "no matches"
  state read a *different* filter, so its Clear Filter button couldn't clear it.
- **⌘W on the last tab replaced it with your home folder** instead of closing
  the window, silently throwing away where you were.
- **Compare Files with an empty opposite panel diffed the file against itself**
  and reported "identical"; **Compare Directories on a remote or in-archive pane
  opened a window listing nothing**, which reads the same way. Both now say what
  they need instead.
- **Pinned tabs and renamed tabs didn't survive a relaunch** — there was nowhere
  in the saved workspace to put either.
- **22 Pro commands looked free in the command palette** until you picked one:
  the PRO pill came from the command's category rather than the gate it actually
  enforces.
- Escape now dismisses both update-permission alerts, one of which appears
  unannounced shortly after first launch.

- **Drag-and-drop told you the wrong thing twice.** Dwelling over a folder
  inside an open archive threw the pane *out* of the archive mid-drag, so the
  files landed on real disk; and ⌘-dragging into a remote pane or an archive
  showed the move badge and then copied — the remote case apologised afterwards,
  the archive case said nothing. Panes that can only copy now say so in the
  badge, before the drop.
- **Cancel now stops a folder sync.** It used to dismiss the sheet while the
  loop kept copying, so the window closed and files carried on landing.
- **AI Auto-Classify and AI Smart Tagging** silently swallowed tag-write
  failures and closed as if they had worked, the same way Smart Tags did.

- **Cancel didn't cancel an archive extract or compress.** Copy/move/delete poll
  for cancellation between files, but extract and compress hand the whole job to
  one child process — so pressing Cancel flipped the button to "Cancelling…"
  permanently while 7zz ran to completion. The engine already knew how to
  terminate its child; nothing was wired to ask it to. A cancelled compress also
  removes the partial archive, since a truncated .zip is not usable.
- **Two Pro gates fired only after the work was done.** Choosing S3 in New
  Connection looked free — the picker listed every protocol unmarked — so the
  user typed host, bucket, region and their AWS access key and secret before
  being refused at Connect. Reclaim Space ran a multi-minute home-folder scan and
  let the user tick candidates before revealing the lock at "Move to Trash", and
  its menu item carried no PRO badge at all. Both now say so up front.
- **Three more failures only the log ever heard about.** A custom theme that
  failed to save looked fine until it was gone on the next launch; a failed
  automation write silently lost every scheduled rule; and Smart Tags swallowed
  per-file tag failures with `try?` and dismissed the sheet, so on volumes
  without tag support (exFAT, many SMB shares) nothing was tagged and it looked
  like it had worked.

- **Ten operations changed your files without ever reaching Undo.** The app has
  an undo stack and an Undo History timeline, and 1.6.7 fixed the copy/move
  paths — but the natural-language bulk command, AI Organize, macro replay,
  ⇧F5 Duplicate, the Hex editor, the EXIF editor, audio tags, delete inside an
  archive, Text Tools and Git "Restore this version" all recorded nothing. ⌘Z
  after any of them silently reversed an *older, unrelated* operation while the
  thing you wanted back stayed gone. The bulk operations now record per-file
  landing paths (undo refuses a record without them rather than guess a path and
  delete the wrong file), and the in-place rewrites keep a `.unifyl-bak` sidecar
  and reuse the restore path the encoding converter already had. Backups are
  capped at 1 GiB — past that the edit proceeds exactly as before rather than
  quietly consuming gigabytes. Audit F52 covers the class and found the last two
  instances itself.

- **Dismissing the file-conflict sheet with Escape hung the copy or move
  forever.** Copy/move, drag-and-drop and archive extraction suspend the
  operation on a continuation and ask what to do about a name collision. The
  sheet's three buttons all answered it, but SwiftUI performs its own dismissals
  — Escape, click-outside, interactive dismiss, window close — and signals them
  by driving the presentation binding to `false`, not through any button. That
  setter only cleared the sheet's state, so the dialog closed and the
  continuation was never resumed: the operation waited forever and every job
  queued behind it stalled, unrecoverable without quitting the app. Eight call
  sites fed that one sheet. Dismissal now resumes with "skip" — never overwrite
  — and the cloud-only and remote delete confirmations got the same treatment so
  a future view binding to their flags can't reintroduce the hang. Audit F51
  covers the class, and regression tests dismiss each dialog and fail on a
  timeout rather than hanging the suite.
- **Remote and archive file dates were wrong by up to five centuries in some
  regions — and folder sync trusted them.** An `ls -l` line from an SFTP server
  and a `tar -tv` line from an archive both print Gregorian dates ("Aug 17"), but
  both parsers assembled them through `Calendar.current`, which inherits the
  system region's calendar. Measured under ar_SA's Islamic calendar: "Aug 17
  09:30" parsed as 2027-01-25, and "Aug 17 2025" as 2586-11-20. Folder sync
  decides which side is newer from exactly these timestamps, so a remote file
  dated centuries ahead always won and would overwrite the user's current local
  copy. Both parsers now pin a Gregorian calendar while keeping the user's
  timezone. Audit F48 covers the whole class, distinguishing relative date
  arithmetic (calendar-agnostic, fine) from constructing an absolute date out of
  month/day/year numbers (not fine) — it found the third instance itself.
- **Every scheduled automation fired at the wrong time, or never, for anyone whose
  Mac region uses a non-Gregorian calendar.** Cron fields are Gregorian by
  definition — month 1-12, day 1-31 — but the matcher pulled date components
  through `Calendar.current`, which inherits the system region's calendar. In
  Saudi Arabia and other ar_SA regions that is the Islamic calendar, so a
  `0 9 15 * *` trigger compared its "15" against an Islamic day number and never
  matched a Gregorian the 15th. The calendar is now pinned to Gregorian while
  still honouring the user's timezone. Found by running the whole suite under
  `-testRegion SA` for the first time: three cron tests failed there and passed
  in every other locale.
- **Pasting multiple files crashed the app in Korean — and would have in
  Japanese and Thai.** Seven translations had reordered `%d` and `%@` for
  natural word order without positional specifiers, so `String(format:)` handed
  the file count to `%@`, which dereferenced it as an object pointer. A compiled
  probe segfaults on the exact string. Reachable from the ⌘V paste toast and
  Select Same Extension. All seven now use `%1$d`/`%2$@` positions, keeping the
  natural word order. Found by a new cross-locale format-specifier check
  (audit F45), which now guards all fifteen catalogues; no existing gate could
  see it — key parity looked fine, the catalogue parsed, and the test suite runs
  in English.

## [1.7.0] — 2026-08-10

### Added
- **A short file list now says what the empty space below it is hiding.** A pane reading "4 items" could mean four files, or four out of two hundred with a filter on, and nothing distinguished them — which is how you conclude a folder is empty and re-download what you already have. Under a short list the pane now reports how many items the filter and the hidden-files setting are withholding, how many of these names are also in the other pane, and what changed here recently. It appears only when the list leaves room and only when one of the three has something true to say; a folder with nothing to report stays quiet.
- **Swapping panels (⌃U) shows the panes trading places** instead of the contents changing under you, so you can see which side went where. The folder tree now slides in like the terminal and the command palette already did, and switching theme cross-fades rather than snapping the window to a new colour in one frame.
- **A few quiet asides.** Open Unifyl between 1 and 5 in the morning and it says something short about the hour. Come back after a fortnight away and it tells you nothing moved while you were gone — which is the first thing you want to know about a file manager. Cross a thousand, ten thousand, fifty thousand or a hundred thousand files copied or moved and it says so once. Step into an archive and the ARCHIVE badge drops in rather than appearing. All of it is off with Settings ▸ Playful touches, and the animated part skips itself under Reduce Motion.
- **Undoing an operation rewinds the timeline instead of blinking.** In Undo History, the entry you are undoing lifts up out of the list and fades as the rows below slide into its place, so the step back is something you watch rather than a row that vanishes between frames. It is decorative: Settings ▸ Playful touches turns it off, Reduce Motion skips it, and the undo itself runs either way.


### Changed
- **Pro is now $24.99, down from $39.99, and Korea is priced at ₩39,900.** The
  in-app price, the bundled manual in all fifteen languages, and every landing
  page follow the new figure. Korea gets a custom App Store price rather than
  the automatic conversion of the base tier, so Korea lands on the advertised
  figure exactly. Direct is a dollar price — about ₩40,100 all-in — so in Korea
  the App Store is now the cheaper of the two.

  The reason is not a sale. Korean launch copy quoted ₩39,900 — a number lifted
  from a regional-pricing proposal that was never adopted — and it went out
  publicly while the real price was ₩66,000. Repricing only Korea would have
  made the cheaper channel the Mac App Store build, which is the one without
  Sparkle, 7-Zip, unalz, the Git panel or the terminal; and a Korea-only Direct
  discount cannot be enforced at all, since LemonSqueezy has no per-country
  pricing and a coupon compiled into the app is a coupon for everybody. So the
  base price came down far enough to make the advertised number true everywhere.
- The upgrade sheet's Mac App Store fallback label read from a second hardcoded
  price string, so a price change could leave the two disagreeing while StoreKit
  loaded. It now falls back to the same constant everything else reads.

### Removed
- A dead localization key carried the price inside the key itself
  (`"Buy Pro — $39.99"`, in all fifteen catalogues). Nothing referenced it — the
  live button has used a format string for some time — but a key like that puts
  the price where a missing translation displays it verbatim.

## [1.6.7] — 2026-08-09

### Changed
- **The website finally matches the app.** The landing page had been describing 1.2.0 — four releases back — and not one of the 15 localised variants had ever been deployed: every locale URL returned 404 while the app itself shipped all 15 languages. All 15 pages are live now, each carrying the 60-second promo video, and two claims that were wrong went with them: "120+ rebindable shortcuts" is exactly 133, and the "50+ professional tools" line is gone in favour of naming the work instead of counting features.
- **The other 13 languages caught up.** 128 strings had accumulated since 1.3.0 with a translation in Korean only — every Tools, Compare, Developer and Automation menu title, the panel and view labels, and the refusal messages added in 1.6.4–1.6.6. German, French, Spanish, Italian, Portuguese (Brazil), Japanese, Simplified and Traditional Chinese, Russian, Turkish, Arabic, Thai and Vietnamese now carry all 1,358 keys, matched to Apple's own macOS terminology per language rather than translated literally.

### Fixed
- **Animation now honours Reduce Motion everywhere.** Thirty-one of the app's animation sites called SwiftUI's own APIs, which ignore the accessibility setting — including the two largest movements, the Quake terminal dropping over half the window and the first-run onboarding slides. All of them now go through the app's Reduce Motion-aware helpers, and a release check fails the build if a new one does not.
- **Undoing a copy or move could delete or displace the wrong file.** Undo reconstructed what an operation did from the names it was *asked* for, not where files actually landed. So after answering a name-conflict dialog with "keep both", ⌘Z deleted the pre-existing file the copy had collided with — and after "skip", where nothing was copied at all, it still did. Undoing a folder copy that had merged into an existing same-named folder deleted that folder with everything already inside it. Every copy, move, drag and paste now records the real landing spot of each file — the renamed copy under its actual name, skipped and failed files not at all — and undo acts only on those. If a record somehow lacks them, undo now says it can't rather than guessing.
- **⌘D and the command palette's copy/move were not undoable at all.** Only the F5 confirmation sheet recorded anything for undo; Copy to Opposite Panel, Move to Opposite Panel, the palette commands, the skip-confirmation paths and the single-file rename-copy recorded nothing. All of them do now.
- **A file you deleted to the Trash could not be put back.** The delete itself was always correct, but Finder's Put Back refused with "…no longer exists", naming a folder that was still right there — so the one-click way back from an accidental delete was gone, for anything deleted out of Downloads, Desktop, Documents or another folder macOS protects.
- **⌘Z did not undo a file operation.** Rename, copy, move and delete were all undoable from Edit ▸ Undo File Operation, and that item is where the keyboard shortcut is documented, but pressing ⌘Z did nothing: macOS was handing the key to its own Undo, which is inactive unless you are typing. ⌘Z now reaches the file undo, and text undo still works while editing a name or filling in a field.


## [1.6.6] — 2026-08-03

### Fixed
- **Nothing you changed in Settings ▸ Shortcuts had any effect.** The pane listed 131 remappable commands and the app read none of them. Menu items declared their key twice — a hardcoded one and a catalogue-bound one layered on top — and the hardcoded one is the one macOS uses, so every catalogue binding was inert. The function-key handler never consulted the catalogue at all; F1–F8, ⌃0–⌃7 for tags, ⇧F4/⇧F5/⌥F5/⇧F6/⇧F9/⌥F9, ⌃T, ⌃Q, ⌃G, ⌃S, ⌃H, ⌃U, ⌃B, ⌃\, ⌥←, ⌥⇧↩, ⌘F and the file clipboard keys were all fixed in code. Every one of them now resolves through the catalogue, so remapping a command moves the key everywhere it is handled and clearing a row unbinds it. Refresh's hidden ⌘R handler and Reveal in Opposite Panel, which bypassed the pane entirely, are included.
- **Two commands were listed twice in Settings ▸ Shortcuts.** "Select All" and "Invert Selection" appeared under both Edit and Selection with the same default; only one row could own the key, so editing the other did nothing. The duplicates are gone, and any customisation stored against them is surfaced as a legacy shortcut to re-assign rather than dropped. "Function Key Bar" and "Reveal in Opposite Panel" gained the rows they were missing.
- **37 of the 133 rows in Settings ▸ Shortcuts still could not be given a shortcut** — every Tools, Compare, Developer, Automation and Analysis entry. Assigning one showed the new key in the row and in the menu, and pressing it did nothing, because the wiring pass had only reached menu items that already had a built-in shortcut to replace. All 133 rows are now connected. No new default shortcuts were added: these commands still ship with none, exactly as before — the difference is that a key you assign to them now works.
- **Four menu items had their shortcut typed into the title** — "Compress… (⌥F5)", "Extract to Folder… (⌥F9)", "Duplicate in Same Folder (⇧F5)", "New Text File + Edit (⇧F4)". Now that those commands can be remapped, a printed key becomes wrong the moment you change it. The text is gone; macOS shows the real key, which follows the binding.
- **Some Settings ▸ Shortcuts rows named a command differently from the menu item they control**, so searching Settings for what you had just clicked found nothing — "PDF Merge / Split" for the menu's "PDF Tools", "Folder Size Heatmap" for "Folder Heatmap", and six more. The rows now use the menu's wording.
- **A shortcut you recorded on Home, End, Page Up, Page Down, keypad Enter or F13–F19 was accepted, displayed, and never fired**, while the old key quietly kept working. The recorder wrote one spelling of those key names and the two things that read shortcuts understood a different one. A key the app genuinely cannot bind is now refused with a reason instead of saved. Recording a bare letter or digit is refused too — those are what jump to a file by name — and so is a chord the file panel already answers to itself (⌘1–⌘9, ⌃V, ⌃X, ⌃A, Return and the rest), which used to be accepted and then swallowed.
- **Dragging files between panels skipped four safeguards that every other way of copying had.** A drop into an open archive wrote the files next to the archive instead of into it; a drop onto a remote connection failed with a raw "URL type sftp isn't supported"; a drop onto a Smart Folder wrote into whatever directory the panel last showed; verify-after-copy never ran, even though this release made dragging copy by default and made verification free; and ⌘Z could not undo a drag, in the same release that made ⌘Z the main undo key. Dragging now uploads to a remote panel, lands inside the archive you are looking at, refuses a Smart Folder with an explanation, verifies when you have asked it to, and is undoable. Dragging a row *out* of an archive or a remote panel — which could only ever hand over a reference to nothing — no longer starts.
- **Acting on a file inside an archive, or on a remote connection, failed without saying so.** ⌘C reported "Copied" and put an unusable reference on the clipboard; ⌃1–⌃7 announced a tag and drew it on the row without writing anything; Create Symlink did nothing at all; the preview pane went blank with a file plainly under the cursor; Get Info sat on "Calculating…" forever; and double-clicking with "Open with default app" selected handed the file to macOS, which answered with a dialog about a URL type no application can open. Each of these now explains what it cannot do and what to do instead.
- **The right-click "Get Info" ran Reveal in Finder.** The menu-bar item was split into a real Get Info panel and a separate Reveal item; its twin in the right-click menu was missed, so the same words did two different things depending on where you clicked. Both menus now offer both, and each does what it says.
- **F5 or F6 to a remote panel was refused whenever exactly one file was selected** — the ordinary case — with an instruction to navigate the other panel somewhere it already was. It also only ever checked the remote destination when the confirmation sheet was shown, so ⌘D, ⇧⌘D, the command palette and the skip-confirmation path all wrote the files to a local folder instead of uploading them.
- **Get Info kept measuring a folder after you closed it**, and asked this Mac about a remote symlink's target, reporting either an unrelated local file or a broken link for a link that is fine where it lives.
- **The App Store build still offered the terminal** through the command palette, the backtick key and the Function Key Bar button, after the View-menu items were removed. That build cannot run a shell, so every command in it failed.
- **The manual described the old drag behaviour, a shortcut that belongs to another command, a Free edition that includes cloud storage, and a "Pro + AI" tier that does not exist.** Unifyl is Free and Pro, one-time purchase.

## [1.6.5] — 2026-08-02

### Fixed
- **Editing a file inside an archive lost the edit if you quit before switching back to Unifyl.** The write-back into the archive only ran when Unifyl regained focus. Save in your editor and then quit Unifyl from the Dock — or log out, or restart — and it never regained focus: the edit was never written back, and the staged copy holding it was deleted on the way out. The message shown when a write-back failed even pointed at that copy as somewhere safe, and the next launch erased it. Edits are now kept outside the temporary sweep, written back before the app quits, and picked up on the next launch if the app was force-quit or crashed.
- **`defaults write` instructions in the release notes named a preference domain the app does not use.** Following them enabled nothing and reported nothing. Three internal log subsystems named it too, so Reclaim Space, purchase and pricing activity was missing from Console for anyone filtering on the app's real subsystem.

## [1.6.4] — 2026-08-01

### Fixed
- **Folder sync moved the files it promised to copy.** A local-to-local sync ran `replaceItemAt`, which consumes its source — so every file the plan listed as "copy to right" was deleted from the left. Anyone syncing a folder as a backup lost the thing they were backing up. A name differing only in case from a file the plan said it would *skip* destroyed that file too. Cross-volume syncs copied nothing at all, and symlinks were planned as ordinary files.
- **Multi-Rename crashed the app.** The Auto Number Start and Step fields accept any number, and the preview recomputes on every keystroke — a large value overflowed and took the app down mid-typing.
- **Multi-Rename refused Korean, Japanese and Chinese names that macOS accepts.** The length check counted UTF-8 bytes against a limit the filesystem actually expresses in UTF-16 units, so a Korean name was capped at roughly a third of its real length and the error blamed a "/" that wasn't there. Folder names were also split at the last dot, so `v1.2.3` was renamed in the middle.
- **The hex editor truncated large files on save.** Files over 10 MB are shown 10 MB at a time; saving wrote only that much back, permanently discarding everything past it. It also overwrote changes another program had made since the file was opened, with no warning.
- **Content search behaved differently on the two editions.** The direct build treated your query as a regular expression and the App Store build treated it literally. Matches in files whose path contains a colon were silently dropped, an unbalanced bracket reported "No matches found" instead of an error, and only `*.ext` masks worked in the sandboxed build — `Makefile` or `report*.txt` quietly matched nothing.
- **Encoding conversion had no Korean encoding in its fallback chain.** The entry commented as CP949 resolves to Shift-JIS, so Korean files fell through to a Latin decode and were reported as "ISO-8859-1". A valid UTF-8 file containing a single damaged byte was rewritten as whole-file mojibake, and a file cut mid-character lost all of its text where the system tool recovers everything but the last character.
- **Reclaim Space offered to free space that does not exist.** Hardlinks and APFS clones — the copies Finder and `cp` make on modern Macs — were listed as deletable duplicates at full size. Deleting one frees nothing. Sparse files were counted by logical length, overstating by orders of magnitude.
- **Advanced Search could hang on a short filename.** Its protection against runaway regular expressions limited the length of the text being searched rather than the time spent searching it, which is not a limit at all; a 40-character name was enough. Closing the sheet also left the scan running.
- **Checksum verification failed for every binary-mode file.** Lines written as `hash *name` — what `shasum -b` and most Windows tools produce — parsed the mode flag as part of the filename, so every file was reported missing.
- **Three-way merge reported conflicts that were not there** on about one in sixteen cleanly mergeable merges, and silently reordered content on some merges it did complete.
- **Deleting a file inside an archive could destroy a different file outside it.** Pressing Delete on an archive entry ran the ordinary delete path, and inside an archive a row's underlying path is the archive's own folder plus the entry name — so if a real file of the same name sat beside the archive, that file was removed instead, and not to the Trash. The archive entry itself was left untouched. With the delete confirmation switched on the dialog routed correctly and hid this; with it off, nothing did.
- **Copying a folder into one of its own subfolders filled the disk instead of refusing.** The refusal existed for drag-and-drop and nowhere else, so F5 and F6 walked straight in: 385 nested copies before it failed on path length, all left behind. Ordinary to hit in a dual-pane window — the left panel on a folder, the right panel already inside it.
- **Copying a file onto itself deleted it.** After a failed copy Unifyl removes the half-written destination, and it could not tell that the destination *was* the source. On a case-insensitive disk — the macOS default — "DOC.txt" and "doc.txt" are one file, so the cleanup removed the original. It is now refused before anything is written.
- **Multi-Rename could rename the folder you were standing in.** With nothing selected the sheet opens on the whole listing, and the ".." row is a real entry pointing at the parent directory. Applying any rule renamed that folder out from under the panel.
- **Archives listed every file as 0 B, and mangled Korean, Japanese and Chinese filenames.** The tar listing asked for paths only, so no size or date ever came back, and without a UTF-8 locale the underlying tool escapes every non-ASCII byte — 한글.txt arrived as \341\204\222…. 7z had the same 0 B fault for a different reason. ZIP was never affected.
- **Editing a file inside an archive now writes it back.** F4 did nothing at all — no editor, no error — and opening a file with Return unpacked it to a temp folder that was deleted sixty seconds later, so an edit was lost and the archive never changed. Both now stage the entry, open it, and save it back into the archive when you save in the editor. F3 and Space had the same root cause and showed an empty window.
- **Any change inside a tar archive corrupted its structure.** Adding, removing, renaming or creating a folder inside a .tar/.tar.gz/.tar.bz2/.tar.xz/.tar.zst rewrote every entry with the full path of a temporary directory. Creating one had the same fault: an archive built from /tmp/x/a.txt contained "tmp/x/a.txt". Replacing a file that already existed failed outright.
- **Opening Statistics could quit the app.** The all-time totals are summed with plain addition, which crashes rather than wraps if the stored numbers are large enough — and the stored numbers are read straight off disk without a sanity check. Totals now saturate.
- **Corrupt settings were overwritten instead of backed up.** Workspace layouts, automations, custom keyboard shortcuts and — worst — your saved folder permissions each got emptied and then saved over the file that still held them. The folder permissions are the painful one: losing them means granting access to every folder again by hand. All four now set the damaged file aside first.
- **A re-download could replace a good file with a truncated one.** Five cloud providers wrote straight over the destination, so a crash or a full disk partway through left the fragment where a complete earlier download had been.
- **The App Store build pointed users at a web address that does not exist.** Every "not available in this version" message ended by naming a domain that was never registered.
- **Git History never worked on a folder — including the repository you were standing in.** It resolved git's working directory as the parent of whatever it was opened on, which is right for a file and wrong for a directory: asking for the history of a repository root ran git from *outside* the repository, so it reported "Not a git repository or git is not available." while the path bar was showing the branch name two inches away. A directory is now its own git context (a nonexistent path still falls back to its parent). The same formula was behind the diff, restore, and is-in-a-repo paths, so all four were affected. Locked by tests.
- **Content search couldn't find Korean, Japanese or Chinese text in legacy-encoded files** — the one thing the CJK pitch leads with. Searching is done with grep, which compares raw bytes, so a Korean query could never match a CP949 document however the file was written; the sandboxed build had a Swift fallback but it only tried UTF-8 and Latin-1, and Latin-1 accepts any byte, so those files decoded to gibberish and reported no matches. Non-ASCII queries now go through a decoder that recognises CP949, Shift-JIS, EUC-JP, GBK and Big5 — and picks between them by detection rather than by order, since those encodings share byte space and a fixed order turns Chinese into Korean nonsense. ASCII searches still use grep and are as fast as before.
- **File X-Ray called every arm64e binary "arm64".** The reader was already parsing the CPU subtype; the name just ignored it. Slices are now labelled the way `lipo` labels them, including x86_64h — a distinction that matters in a tool whose whole purpose is telling you precisely what a binary is.
- **Archives made by the App Store build dated every file 1980-01-01.** ZIP stores timestamps in an MS-DOS format, and the sandboxed build's own ZIP writer left those fields at the format's epoch instead of reading the source file's date — so unzipping a Unifyl-made archive anywhere else showed 1980 for everything. The direct build was unaffected (it shells out to a tool that gets this right), which is why archive timestamp preservation could ship in 1.1.0 and still be missing here. Dates outside what the format can express (anything before 1980) still fall back to the epoch rather than wrapping to a wrong year.
- **Uninstalling an app no longer stops at the first thing it can't remove.** The App Uninstaller bailed out on the first failure, so a locked or in-use support file left the app half-removed — usually the bundle gone but its leftovers still on disk — and reported only that one problem. It now moves everything it can, keeps the entry in the list when anything survives (removing it would claim an uninstall that didn't happen), and says how many items moved, how many didn't, and why the first one failed.
- **Four windows could trap the app with no way out.** About Unifyl, the Plugin Manager, the Macro Recorder and the Enhanced Duplicate scan are presented as sheets, which carry no titlebar — and none of them had a close button. Escape did nothing, clicking outside was swallowed, and even ⌘W was blocked, so the only escape was force-quitting. All four now have a Close button (with the standard Escape binding), and a release check fails the build if any sheet ships without one.
- **AI scans no longer ask for your Photos and Apple Music libraries.** Reclaim Space was taught to step around ~/Music, ~/Pictures and ~/Movies, but the AI engines weren't — so AI Organize, Enhanced Duplicate Scan, AI Summarize, smart tagging and folder profiling each still walked straight into them and raised a privacy prompt mid-scan. Seven walk sites now share one rule, and the rule lives in a common module rather than inside one scanner, which is why the two had drifted apart in the first place. Pointing a tool at one of those folders on purpose still scans it — only incidental descent during a broader sweep is skipped.

### Added
- **Reclaim Space (Tools ▸ Disk & System).** One sweep that answers "where did my disk go": byte-for-byte duplicates, regenerable caches and build artifacts, big files you haven't opened in months, and apps you've stopped launching — all totalled into a single recoverable number instead of four separate tools nobody found. The scan is free (the menu item carries no PRO badge — the number is exactly what a free user needs to see); clearing the whole selection in one go is Pro. Nothing is ever pre-selected and everything goes to the Trash, so a mis-click is recoverable. The sweep steps around Photos, Music and Movies entirely and says so on screen — merely listing those folders raises a privacy prompt nobody asked for, and a cleanup tool has no business offering to delete what's inside a photo library. Point Reclaim Space at one of them deliberately and it's scanned normally. (Measuring other apps' caches still asks for the standard "access data from other apps" permission on first run — every disk-cleanup tool needs it.) Duplicates always keep the most recently used copy of each set, and only well-known regenerable trees (Xcode DerivedData, ~/Library/Caches) are ever offered as "safe" — a folder merely *named* cache is not.
- **The Tips sheet now tells you the free trial is running.** New installs get 14 days of full Pro, but nothing said so on the one screen every new user actually sees — so the PRO badges read as "locked" when they were open, and the trial routinely expired unused. A strip above the card now counts down the remaining days while the trial is live, and switches to a one-line unlock note once it lapses.
- **Launch-sale pricing without a rebuild (direct edition).** The upgrade sheet's price copy — sale price, struck-through regular price, a localized promo note, and the local-currency estimates — can now be updated by publishing a small static file instead of shipping a notarized build. The payload must declare its own expiry date and the app falls back to the built-in price the moment it passes, so a sale ends on schedule even if nobody remembers to change anything. The Mac App Store build ignores all of it and keeps showing Apple's own localized price.

### Changed
- **The rating prompt is no longer limited to enormous transfers (Mac App Store build).** It only ever fired after a second 1 GiB+ copy, which most users never make. Ordinary successful copies, moves, and deletes now count toward it as well — big transfers simply count for more — and it still only appears after an error-free operation, at most once per version, on Apple's own 3-per-year leash.

### Fixed
- **329 strings showed up in English no matter which language you picked.** Menu titles, context-menu commands, half the feature names on the paywall, and most error messages were never in the string catalogs at all — they only existed in the source, so every language fell through to English. All 15 languages are now complete (1,122 strings each). A further 67 already *had* translations in every language that the app could never reach, because the code looked them up under one name while the catalogs stored them under another.
- **"残り 3 日s".** The free-trial countdown made plurals by gluing an "s" onto the day word, which is only correct in English and the Romance languages — Japanese, Chinese, Korean, Italian, Turkish, Russian, Arabic, Thai and Vietnamese all got a stray Latin "s". Both trial lines now use proper per-language plural rules.
- **VoiceOver read four labels in English in every language** — the path-bar breadcrumbs ("Navigate to Documents"), the function-key hints, and the Smart Folder summary. Each built its own lookup key at runtime, so no translation could ever match.
- **Purchase dates are recorded from now on.** Nothing displays them yet; they exist so a future paid upgrade can grandfather recent buyers, which is only possible if the date was captured at purchase time. Recorded from the App Store transaction on the Mac App Store build and from the license activation on the direct build, keeping the earliest date ever seen so a reinstall or a Family Sharing hand-off can never move it forward.

## [1.6.3] — 2026-07-22

### Added
- **Tips.** A card-per-feature tour of everything Unifyl can do: one tip on every launch (rotating through the whole catalog), Previous/Next to browse all of them, and a "Don't show tips at launch" checkbox for opting out — Help ▸ Tips… and the command palette reopen it any time. Cards are generated from the localized feature catalog (so all 15 languages come along), a curated set carries handwritten usage hints with real shortcuts, Pro features wear a PRO badge with an unlock button on the free tier, and the Mac App Store build automatically skips cards for features that edition doesn't include.

## [1.6.2] — 2026-07-20

### Added
- **A well-timed ask for a rating (Mac App Store build).** After your second 1 GiB+ transfer lands cleanly, Unifyl may show the standard macOS rating prompt — once per version at most, on Apple's own 3-per-year leash, and only ever right after something went well.
- **File > New Window (⌘N).** Closing the main window previously left no menu item to reopen it (App Review, Guideline 4) — the File menu's standard New Window now recreates the main window, and the shortcut matches the macOS convention.
- **Unifyl > Unifyl Pro… menu item.** A direct menu path to the upgrade/purchase sheet (one-time Pro purchase + Restore Purchases), instead of only reaching it through the upgrade banner or a PRO-gated feature (App Review, Guideline 2.1(b)).
- **Grant Access… actually grants access (Mac App Store build).** The permission-denied banner's button now opens a folder-picker (powerbox) grant pre-pointed at the blocked folder and persists it as a security-scoped bookmark for future launches — previously it pointed at System Settings, which cannot pierce the app sandbox. If persisting the bookmark fails, the banner now says so instead of silently forgetting the grant on relaunch.

### Changed
- **Mac App Store entitlements slimmed.** Removed the music/movies/pictures asset-folder entitlements from the MAS build — they had no matching functionality (App Review, Guideline 2.4.5(i)); media and iCloud browsing already flow through user-selected access and the Grant Access flow. Downloads access is kept: the sidebar's ~/Downloads is core file-manager functionality behind the standard sandbox consent prompt.
- **Descriptive consent prompts (Mac App Store build).** The Downloads/Documents/Desktop sandbox consent dialogs now explain what Unifyl actually does with each folder instead of showing a generic one-liner (App Review, Guideline 5.1.1(ii)).
- **The Mac App Store paywall no longer advertises features absent from that build.** Docker/Git-panel and other sandbox-dropped items were purged from the upgrade sheet's Pro feature list.
- **Paywall stops promising what Pro doesn't deliver.** The upgrade sheet and nudge banner advertised "60+ professional tools" (the audited count is 50+) and a "Team Management · SSO · Audit Log" row — a feature set that sits behind an off-by-default pilot flag no buyer can reach. The count is corrected everywhere, and the team row is replaced with shipped Pro value (Multi-Rename · Saved Searches · Verify-After-Copy). The Pro-unlock subhead is now actually localized too — its lookup key never existed in the string tables, so every locale silently fell back to English.

### Fixed
- **Right-click menu and Cmd+Delete now act on the marked files again.** The red-mark rendering fix below stopped mirroring marks into the table's native selection — but the context-menu actions (Copy/Move/Delete/Rename/Get Info/…) and the table-level Cmd+Delete/⌦ handlers still read the native selection, so with 5 files marked they silently operated on only the cursor row. They now use the marked set with the cursor row as fallback, matching every other operation path. Locked by tests.
- **Free tier no longer gets Pro remote protocols, clouds, or archive editing.** Connecting to WebDAV/SMB and to S3/B2/GCS/Azure/Google Drive/Dropbox/OneDrive, and editing archives in place (delete/rename/add/drag-drop into an existing archive) had no tier gate at any entry point — all Pro features per the published tier table. They now show the upgrade prompt on Free; FTP/FTPS/SFTP, archive browsing/extraction, and creating new ZIPs stay free.
- **Restore Purchases now tells you what happened.** It distinguished nothing: restored, "no purchase on this Apple Account", and "App Store sync failed" all looked like a dead button. It now shows a spinner while running and a per-outcome message. An Ask to Buy purchase (`.pending`) likewise gets an "awaiting approval — activates automatically" notice instead of silence, a failed product load surfaces the real StoreKit reason instead of a generic "Product unavailable", and a stale purchase error from a previous attempt is cleared when a new attempt starts.
- **Unifyl > Unifyl Pro… works with every window closed.** The upgrade sheet is presented from the main window; with none open the menu item silently did nothing. It now reopens a main window first — keeping the deterministic IAP path App Review asked for actually deterministic.
- **Advanced Search's Delete reports why files failed.** Bulk-deleting search results showed only "N trashed, M failed", discarding the per-file reason — including the new online-only-file guidance. The first failure's message is now appended to the status.
- **3-way merge no longer silently drops a side it couldn't read.** A file that isn't UTF-8 (e.g. CP949/Shift-JIS) or fails to read was treated as empty, so the merge looked clean while one side's entire content vanished from the result. The merge now aborts with a message naming the unreadable file and suggesting encoding conversion first.
- **Cancelling a delete warns when an in-flight item may still be deleted.** Cancel abandons a blocked cloud-provider call by design (so the UI never wedges), but the orphaned call can still complete — the file later disappears with no explanation and no Undo entry. A notice now says so.
- **Mac App Store build no longer fails to compile in Release/archive.** A strict-concurrency isolation error in the 3-way merge's MAS-only code path (`normalizedText`) broke `xcodebuild archive` for the MASLite target.
- **No more surprise "enter your login keychain password" dialogs.** Saved connection passwords, licenses, and trial state moved to the data-protection keychain, which is gated by app identity rather than per-item ACLs — so switching between the direct and App Store editions (or a future signing-certificate renewal) can never again trigger macOS's keychain-access prompt. Existing entries migrate automatically and losslessly on first read.
- **The cursor row no longer drowns its own filename.** The keyboard-focus row painted the theme's cursor colour as a solid band while cell text kept its file-type colour — on the default theme that meant dark text on a saturated indigo band. The band is now a 30% tint (text reads against effectively the normal background) and the solid 3-px left-edge stripe continues to mark the cursor position unambiguously.
- **Marked files are visible again on dark themes.** Marked filenames were painted with the theme's `selection` token — a row-highlight *band* colour that is deliberately dark and near-surface — so on the default Unifyl Dark Pro theme a marked file's text sat at ≈1.6:1 contrast against the background, effectively invisible. Marks now use a dedicated mark-text colour derived from each palette's bright red (TC's red-marking convention), with an adaptive system-red fallback for the automatic and custom themes.
- **Assorted main-thread stalls removed.** AI duplicate scan no longer enumerates the whole folder tree on the main thread; AI Keep-Best / organization recommendations and macro replay run their file operations off-main (multi-GB replays no longer beachball with a frozen progress bar); AI search results can no longer be overwritten by a slower, older query; hovering a cloud-only file's tooltip no longer triggers a blocking download; the copy-progress size probe on slow network volumes moved off-main; the companion server's search no longer wedges every other companion request behind a full-tree walk; and double-clicking an iCloud placeholder twice no longer opens it twice.
- **The upgrade sheet now recognizes an existing Pro license.** Opening Unifyl > Unifyl Pro… after purchasing showed the buy button again; it now shows a "Unifyl Pro is active" confirmation (verified end-to-end in the App Store sandbox: purchase → relaunch → owned state persists).
- **Cmd+A now marks files in red like every other marking path, instead of blue-washing the whole list.** Select All from the keyboard (and select-by-pattern / Invert Selection — anything that sets the marks from outside the table) mirrored the marked set into NSTableView's native row selection: every row got the cursor-colored background with emphasized white text, looking nothing like the same marks made with Insert, right-click, or Edit > Select All (red filename text). The native (blue) selection is reserved for the single cursor row again; externally-set marks repaint as red text, exactly like in-table marking. Drag payloads and batch operations always used the marked set, so only the rendering changes. Locked by tests asserting both select-all entry points produce identical selection state.
- **Deleting a folder of online-only cloud files no longer downloads everything first — and Cancel actually cancels.** The 1.6.1 dataless-placeholder guard checked only the deleted item itself; a *folder* of OneDrive/iCloud online-only files usually stats as materialized, so it slipped into the `trashItem` path, where the file provider hydrates (downloads) every child before letting the folder leave the sync root — the progress dialog sat at 0/N looking like a file transfer, and Cancel wedged at "Cancelling…" forever because the provider call is synchronous and uninterruptible. Delete now scans for dataless descendants (budget-capped, never descending into online-only directories) and routes such folders to the cloud-delete confirmation, and every blocking trash/remove call is awaited abandonably — Cancel closes the progress UI immediately even while a wedged provider call is still unwinding in the background. The nine remaining direct `trashItem` call sites (Duplicate Finder, App Uninstaller, Directory Diff, Large File Finder, macro playback, NL command preview, Advanced Search, AI Keep-Best, companion delete) now refuse online-only placeholders with a readable message instead of hanging or failing cryptically.

## [1.6.1] — 2026-07-10

### Fixed
- **THE chronic drag-breakage root cause: an exception inside AppKit's drag-completion procedure.** Caught live with a full backtrace: `UnifylTableView.draggingEnded(_:)` (added for spring-loaded-folder timer cleanup) called `super.draggingEnded(sender)` — but `draggingEnded:` is an *optional* `NSDraggingDestination` method that NSTableView/NSView do not implement, so the super call raised an unrecognized-selector exception **inside `NSCoreDragCompletionProc`, on every single drag's completion**. The aborted completion meant `draggingSession:endedAt:` was never delivered (confirming why the 10 s watchdog fired after every drag), the session stayed stranded in AppKit's registry, and once enough state accumulated the process wedged: every new drag was refused with AppKit's "Cannot start a new drag. A previous drag has not finished." and left-drags degraded to a confusing band-selection until relaunch. `super` is now called only when an ancestor actually implements the selector (same guard on `draggingExited:`), locked by tests that dispatch both selectors through ObjC. Additionally, the missing-`endedAt` watchdog (list + grid) is now button-aware: it only treats `endedAt` as swallowed after the primary button has been *up* for ~10 s, so a legitimately long held drag (spring-load browsing, hovering another app) can no longer be force-cleared and reloaded from underneath.
- **Drags no longer come loose mid-motion after the display-wake fix.** The 1.6.x recovery for post-display-sleep drag corruption recreated both panels' file tables on every `didChangeScreenParameters` notification — but macOS also posts that for Night Shift/gamma transitions, HDR (EDR) headroom changes, and Dock/menu-bar geometry churn, none of which need recovery. If one landed while a file drag was in flight, SwiftUI tore the drag's source view out of the hierarchy and the drag silently died mid-motion. Table recreation is now gated on a *material* display-layout change (screen count, frame, or scale — a wake always qualifies), and any recreation requested during a drag is deferred until the drag fully closes via a new app-wide drag-session census (list + grid, both panels, with a failsafe so recovery can never be blocked forever). The thumbnail grid also gains the 10 s missing-`endedAt` watchdog the list view already had. Locked by tests covering the census balance, mid-drag deferral, coalescing, and layout-signature stability.
- **Deleting an online-only cloud file no longer just fails.** Moving an OneDrive/iCloud "online-only" file (a dataless placeholder whose data lives in the cloud, 0 bytes on disk) to the Trash failed with a cryptic "the volume doesn't contain the item" — `trashItem` can't move a file with no local data into the local Trash. Unifyl now detects these, and after a single confirmation deletes them from the cloud instead (recoverable from your provider's own trash, e.g. OneDrive online). Fully-downloaded files in the same cloud folder were never affected.
- **One-click Restart (⌥⌘R) to recover drag after the display sleeps.** macOS can corrupt this process's drag promotion at the window-server level after a display sleep/wake or monitor change — file drags degrade to a rubber-band selection and ⌘-shortcuts stop landing, even though the window stays focused. Unifyl mitigates it (opts out of App Nap; rebuilds the file tables on display wake) which holds for days, but no in-app rebuild fully escapes the corruption once it sets in. So there's now **Restart Unifyl…** (Unifyl menu, ⌥⌘R, or the command palette): it saves your workspace and relaunches — the reliable fix when drag stops, and the menu still works even while drag/keys are stuck. Tabs and panels come right back.
- **"Drag stops working until restart" — root cause fixed.** After a long session, file drag in both the list and the thumbnail grid could stop working entirely until the app was relaunched. The cause: `updateSelection` and `ensureCursor` (list) and `applySelection` (grid) issued `selectRowIndexes`/`selectItems`/`scrollRowToVisible` straight at the table on *every* background-driven view update — folder-size recalcs, cloud-status polls, file-watcher refreshes — and the drag guard only deferred `reloadData`, not these selection mutations. One such background update landing during a drag (or its teardown tail) disturbs AppKit's drag session; over hours the coincidences accumulate until the session is corrupted for good. Selection and cursor updates are now deferred during the drag window and flushed when it closes, the same way reloads already were — closing the last unguarded path. Locked by tests asserting both defer mid-drag and flush afterward.

## [1.6.0] — 2026-06-13

### Added
- **Compact list density (View → Compact List Density).** TC-style tight rows for the file list: row height drops from 28 to 19 px at the default font size, icons shrink to 16 px, and the macOS edge inset goes away — roughly 50% more files visible per screen. On by default; the View menu (and command palette) toggles back to the original airy Comfortable layout, and the shortcut is customizable in Settings → Key Bindings. Pairs well with ⌘− to step the panel font down a notch.
- **Statistics (Tools → Statistics…).** Unifyl now counts what you do with it — bytes copied and moved, files renamed and deleted, archives handled, searches run — bucketed by month, entirely on-device, with a reset button and an off switch (Settings → General). The numbers never leave your Mac; they exist so you can see "2.4 TB moved" and feel something.
- **A little celebration for big transfers.** When a 1 GiB+ copy or move lands with zero failures, the progress card's checkmark does one happy bounce and the toast says something warmer than "done". Respects Reduce Motion, never appears on errors, and Settings → General → "Playful touches" turns it (and the friendly empty-folder lines) off entirely.
- **A hidden something.** Type a certain word into the command palette. Norton Commander fans will know which one.
- **Automation run history.** Every automation run — manual, scheduled, or event-triggered — is now recorded (start time, duration, trigger, success/failure with the error, and a tail of the script output) in a 200-entry persistent log, inspectable from a new Run History sheet in Automation. An automation whose last run failed shows an error badge right in the list with the failure message on hover — a silently failing scheduled job is no longer invisible.
- **Transient network errors auto-retry.** Remote downloads and uploads now retry automatically (1s, then 3s backoff) when the failure is a transient one — timeout, dropped connection, reset — so a Wi-Fi hiccup mid-batch no longer fails the file. Permanent errors (auth, missing file, disk full) still fail immediately; retries are logged to Console.
- **Content search says when results are capped.** The 50-matching-line backend cap now surfaces as an explicit "results are capped" notice instead of a silently truncated list that read as complete.
- **Folder Heatmap: file-type breakdown.** The heatmap now aggregates the scanned tree by file category into a proportional stacked bar with a legend (size, count, share per type, top 8 + Other) — "this folder is 50 GB of video" at a glance, computed in one pass over data the scan already collected.
- **Multi-Rename shows exactly what happened.** After applying a batch rename the sheet switches to a result phase: "Renamed N file(s)" with a per-file old → new list audited against what actually landed on disk, and failed files broken out separately with the reason — no more inferring the outcome from the folder listing.
- **Verify file integrity after copy and move (Pro).** New Settings → Confirmations toggle: after each file copy — and after the copy half of a cross-volume move, *before* the original is deleted — both sides are re-read and their SHA-256 checksums compared (streaming, 1 MB chunks, off the main thread). The progress card shows a "Verifying integrity" phase; mismatches are reported per file and never modify the original (a corrupt cross-volume move removes its bad copy and leaves the source in place). Covers panel copy/move, drag-and-drop moves, rename-collision moves, and clipboard cut-paste. This closes the classic "did 40 GB really arrive intact on the NAS?" anxiety that drives TC users to `Verify` after every transfer.
- **Saved Searches.** Find Files (Advanced Search) criteria — the full condition list, scope folder, and subfolder toggle — can now be saved as named templates and re-applied from a new "Saved" menu next to Add Condition. Saving under an existing name overwrites that template; templates persist across launches (corrupt persistence is stashed for recovery, never silently dropped). Applying a template fills the form without auto-running, so it can be tweaked first.
- **Undo History timeline.** Edit → Undo History… (also in the command palette) shows every undoable file operation of the session, newest first — operation type, file count, destination, relative timestamp — with each row expandable to the exact files (old → new pairs for renames) and a plain-language note of what undoing it would do ("restores the file(s) from the Trash"). The top entry carries a "Next Undo" badge, and a prominent button steps back one operation at a time, so ⌘⌥Z stops being a leap of faith.
- **Smart Tagging learns your folder's own taxonomy.** Tag suggestions now include the tags already applied to sibling files in the same folder, weighted by how widely each tag is used there — if everything else in the folder is tagged "Invoice", that's the first chip offered. The folder is scanned once per batch (capped at 500 siblings) and every suggestion chip now explains where it came from on hover (folder usage, content keyword, file type, date, language).
- **AI indexing reports failures and fragile storage.** If files fail to write to the semantic index, the progress card and final status now say "N failed" (causes in Console) instead of an unqualified "Done". And if the vector index couldn't open at its normal location and fell back to temporary or in-memory storage — meaning it would vanish on relaunch — the Index pane shows a persistent warning instead of letting you re-index every session without knowing why.
- **AI Rename explains empty suggestions and retries them in bulk.** Rows that produced no suggestion now say why — "file content couldn't be read", "name already looks good", "image couldn't be classified" — instead of a bare dash, and a "Retry Failed (n)" button re-runs just those files. Per-row refresh and the batch path now share one image-detection route (the sheet's own copy was missing TIFF/BMP).
- **Folder Sync lists what failed, per file.** A sync that ends "3 error(s)" now shows which files failed and why (first three inline, full list on hover) instead of a bare count.
- **Folder Sync shows its execution plan.** After Compare, every entry row gets a plan column — "→" / "←" (will copy) or "–" (will skip) with a tooltip explaining exactly why ("modification times within 2s — can't tell which is newer", "not included in current direction", "excluded from this sync"…), and the status line summarizes "will copy X →, Y ←, skip Z". The indicator live-updates with the direction picker, and the executor now routes through the same pure `SyncPlanner` the preview uses (locked by 14 unit tests), so what you see is exactly what Execute will do.

### Changed
- **Default panel font size is now 11 pt (was 13).** Together with the new compact density this matches Total Commander's information density out of the box. Existing users who ever adjusted the size keep their setting; ⌘+/⌘− and Settings → General still range 9–24 pt.
- **Folders always sort by name, pinned above files.** Sorting by Size, Date, or Kind now reorders only the files; folders keep their alphabetical (natural) order at the top — Total Commander's default behavior. Only the Name column's direction applies to folders (Name ↓ reverses them too). Trade-off: ranking computed folder sizes via the Size column no longer reorders folders.
- **README feature-count claim corrected to match reality.** All 15 README translations claimed "75+ professional tools"; the canonical tier map (`FeatureTier.swift`, 51 Pro features) and every other artifact say 50+. The READMEs now say 50+ too — undercounting beats a claim we can't back.
- **Design-token sweep across 24 views — one visual language everywhere.** Status colors that had drifted to raw `.green/.red/.orange/.blue` literals (sync results, merge markers, diff lanes, macro recording, install states, progress chips) now use the semantic palette (`success/warning/error/info`), so every "done", "failed" and "in progress" reads the same hue app-wide. All rounded corners are continuous squircles on the shared radius scale (the deprecated `.cornerRadius` calls are gone); ad-hoc animation timings collapsed onto the standard motion presets; magic shadows onto the shared elevation tiers. The file-operation progress card drops one over-rendered shadow tier (modal → popover) so it floats instead of looming.
- **Sheets share one chrome pattern.** Compress, AI Rename, AI Classify, Smart Tags, and NL Command Preview previously opened with a bare headline; they now lead with the standard icon + 15pt-semibold title row (AI sheets use the reserved cyan accent). Esc/Return now use the platform `.cancelAction`/`.defaultAction` bindings in the file-operation confirm and AI Classify dialogs, which also gives the primary button the correct prominent styling.
- **Quieter, more responsive bottom chrome.** The F1–F8 bar's full-strength divider washboard is halved in opacity and each key now gives a subtle hover wash (it previously read as a static legend); the status bar aligns to the chrome rhythm (22→24pt) and the "Space = preview" hint is legible without hunting. Conflict dialog's warning icon and padding now come from the token palette.

### Fixed
- **List row height no longer depends on whether you ever touched the font size.** The table started at a hardcoded 28 px but recomputed as fontSize + 10 after any ⌘+/⌘− press, so going 13 → 14 → 13 pt left rows at 23 px — two users at the same font size saw different densities. One formula now drives both the initial value and every change.
- **Folder Sync uploads to remotes are now atomic.** Syncing local → remote (and remote → remote) uploaded straight to the final filename, so a dropped connection mid-transfer left a truncated file sitting where the real one should be — and the next Compare could read it as "identical" by date. Uploads now go to a hidden sibling temp name and are renamed into place on completion; if the server's rename can't land (e.g. an FTP server refusing RNTO onto an existing file), it falls back to the previous direct upload, so behavior is never worse than before. Remote → local sync already had this protection.
- **F2 rename no longer freezes the app on Return.** Committing an inline rename handed keyboard focus back to the table *while the field editor was still resigning*, which re-entered AppKit's end-editing machinery (`makeFirstResponder` → `resignFirstResponder` → `textShouldEndEditing` → repeat) and hung the main thread in infinite recursion. Focus is now restored in `controlTextDidEndEditing`, after editing has fully ended. The hazardous call shipped as far back as 1.2.0; the hang reproduces on current macOS.
- **MAS-Lite build: content search no longer fails to compile.** The 1.5.0 multi-mask change renamed the filename-filter variable but missed the one use site that only compiles in the Mac App Store target's sandbox search path — and the multi-mask semantics now actually apply there (any mask may match, not just the first).
- **Batch delete no longer abandons the rest of the batch on one failure.** Moving N items to Trash stopped at the first item that errored (e.g. a permission failure) and silently skipped everything after it — the error banner gave no hint the operation was partial. Each item is now attempted independently and a partial result is reported explicitly ("Moved 7 of 10 item(s) to Trash — 3 failed: …").
- **Copy and paste failures now say how far they got.** A mid-batch copy/paste error intentionally stops the batch (so a disk-full doesn't cascade), but the error message now includes "N of M file(s) copied — remaining files were not attempted" instead of leaving the user to guess which files landed.
- **Undo of a rename no longer claims success when nothing was undone.** If the renamed files had since been moved or deleted, ⌘Z reported "Undid rename of N item(s)" while restoring zero files; it now counts actual restores and reports an error when none happened (matching delete-undo behavior).
- **Cloud folders (rclone remotes) no longer masquerade as empty on a bad listing.** If `rclone lsjson` exited 0 but its output was unparseable (truncated stream, log lines mixed into stdout), the panel showed an empty folder with no error — now it surfaces a listing error instead.
- **Office preview no longer shows the previous file's content or error after switching files.** Navigating from a Word document to a spreadsheet (or from a failed file to a good one) kept the prior PDF/error on screen because per-file state wasn't reset on reload.
- **Stale error banners cleared on retry.** Git History, Media Info, File Diff, Team Settings, and the video preview kept showing an old error across a successful reload; user-visible error state is now reset when the action starts. The video preview also shows an explicit error when no frames can be extracted instead of a blank strip.
- **Duplicate keyboard-shortcut registrations removed from the toolbar ⋯ menu.** ⌘= / ⌘− / ⇧⌘P were registered both in the menu bar and in the toolbar overflow menu, making dispatch ambiguous (shortcuts could be swallowed depending on focus). The menu-bar registrations remain the single source.
- **Auto-confirmed file operations can no longer run twice.** With a confirmation toggle disabled, the operation was executed from a sheet-binding getter that SwiftUI may evaluate more than once per update pass; the execution hop is now idempotent.
- **Launching a damaged or Gatekeeper-blocked .app now reports an error** instead of silently doing nothing; "Open in Terminal Here" and dropping a folder onto the tab bar also beep on failure instead of being silent.
- **Upgrade banner no longer advances its message rotation during a trial**, so the first nudge after trial expiry starts from the entry message as intended.


## [1.5.0] — 2026-06-09

### Added
- **Thumbnail grid keyboard parity (Total Commander navigation in grid mode).** The thumbnail grid (⌃G) previously only moved its selection with the arrow keys; it now matches the list view's core navigation: **type-ahead** (type a few letters to jump to the matching file — case/diacritic-insensitive, Hangul partial-jamo aware), **Return** to open the selected item, and **Backspace** to go to the parent folder. The type-ahead matcher is factored into the pure, unit-tested `TypeAheadMatcher` shared by both views (10 tests), so list and grid stay in lockstep.
- **Left-drag marquee (rubber-band) selection in the list view.** Press and drag from the empty area below the file list to sweep a selection rectangle over rows (hold Shift/Cmd to add to the current marks, Finder-style). The table view now fills its scroll view even with only a few rows, so the empty space below them is part of the table where the drag can begin. Driven by the normal `mouseDragged`/`mouseUp` overrides gated strictly on a mouse-down that hits no row, so it leaves NSTableView's file-drag (NSDraggingSession) path untouched; reuses the existing `dragSelectRows` (which excludes the `..` parent). Rows are selected by the marquee's vertical span (full-width band) so a straight up/down drag works too.
- **Compare Directories (Mark Newer) — Total Commander panel comparison.** New Compare-menu / command-palette entry marks, in each panel, the files that are unique to that side or newer than the same-named file on the other panel — i.e. the files you'd copy outward. Identical or older files stay unmarked; directories and `..` are ignored; name matching is case-insensitive with a 2-second mtime tolerance. The decision logic lives in the testable `PanelComparison.markNewerAndUnique`. Complements the existing diff-window "Compare Directories…". (Pro, gated like the directory diff.)
- **Select / Unselect by pattern (Total Commander Num+ / Num−).** New Edit-menu commands "Select by Pattern…" and "Unselect by Pattern…" (also in the command palette, rebindable in Settings) mark or unmark every item in the active panel matching a Total Commander-style mask. Multiple masks separated by `;`, `,` or space are OR'd (`*.txt;*.doc`); a `|` splits include from exclude masks (`*.* | *.bak;*.tmp` = everything except `.bak`/`.tmp`). Mask matching is exposed as the reusable `SearchEngine.matchesMask(_:mask:)`. Shows a count toast after applying. Closes a long-standing TC-parity gap (group selection was previously impossible).
- **"Open in New Tab" on folders (Total Commander panel tabs).** Right-clicking a directory (or the `..` parent) now offers "Open in New Tab", which opens that folder in a fresh tab of the same panel and switches to it — previously a new tab could only be opened at the current path (⌘T). `..` resolves to the real parent rather than a path ending in `/..`.
- **Drag a tab to reorder it (Total Commander tab reordering).** Tabs in a panel's tab bar can now be dragged onto each other to change their order; the active tab stays selected as it moves. The index math lives in the pure, unit-tested `CollectionReorder` (UnifylShared) so the active-tab-follows-content behavior is locked.
- **"Open in Other Panel" on folders.** Right-clicking a directory also offers "Open in Other Panel", which navigates the opposite pane to that folder — a discoverable surface for the existing keyboard-only "reveal in opposite panel" (handy to line up two folders before a compare / sync). Reuses `navigateForce`; `..` resolves to the real parent.
- **Numpad gray keys for group selection (Total Commander Num+ / Num− / Num\*).** In the file list, the keypad `+` opens Select by Pattern, keypad `−` opens Unselect by Pattern, and keypad `*` inverts the selection — TC's muscle-memory selection keys, now bound. Matched by keypad keyCode so the top-row `+ − *` still flow into type-ahead search. Wired through a new optional `FileTableViewDelegate.fileTableViewRequestSelectByMask`.
- **Select Same Extension (Total Commander Alt+Num+).** New Edit-menu command adds every file in the active panel sharing the cursor item's extension to the selection — a one-keystroke "select all the `.png`s" built on the tested `SearchEngine.matchesExtensionList`.
- **Test Archive Integrity (Total Commander "Test archive").** New File-menu / command-palette entry verifies one or more selected archives without extracting to disk — `7z t` (per-entry CRC) for ZIP/7z/RAR, `unzip -t` fallback for ZIP, full-stream walk via `tar -tf` for the tar family, `gunzip -t` / `zstd -t` for single-stream formats, and a listing probe for ALZ. Reports the first corrupt archive with the tool's diagnostics, or a "passed" toast. Backed by `ArchiveEngine.test(_:)`.
- **Content search now accepts multiple file masks (Total Commander style).** The Search File Contents "Pattern" field takes `*.h;*.cpp`-style multi-masks (split on `;`, `,`, space) on both the Direct grep path (one `--include` per mask) and the MAS-Lite in-process path (suffix-match against any token). Previously only a single mask was honored.
- **Find Files "Extension" filter accepts multiple extensions.** The advanced-search Extension criterion now matches any of a `pdf;doc;txt` (also `*.pdf, *.doc`, `.png`) list via the new `SearchEngine.matchesExtensionList`, instead of a single extension. A single bare extension still works.

### Fixed
- **Drag-and-drop no longer stops working after a long session (chronic AppKit drag-stability bug — root cause).** The recurring "drag breaks until you restart the app" symptom was a `reloadData()` corrupting the table's drag machinery in the brief window AFTER `endedAt` but before NSTableView finishes unwinding its own drag-session state. Earlier fixes guarded the live `willBegin → endedAt` window (`dragInProgress`), but that flag is cleared synchronously at `endedAt` so a brand-new drag isn't blocked — leaving the post-`endedAt` unwind tail unguarded. A background `updateItems` (file-watcher / cloud-poll / folder-size refresh) landing in that tail still fired `reloadData()` mid-unwind; over a long session the coincidence eventually corrupts the table for good. New `dragUnwinding` flag extends the reload deferral one runloop turn past `endedAt` — closing the gap without re-introducing the "second drag won't start" bug, since a new drag's `willBegin` re-arms `dragInProgress` before the turn ends. Applied to both the list view (`FileTableView`) and the thumbnail grid (`NSCollectionView`). The drag reload-guard state machine is now factored into `beginDragGuard`/`endDragGuard` and locked by 5 unit tests (verified to fail on the pre-fix code). A `DragStability` os_log trail was added so any future recurrence is diagnosable.
- **Type-ahead search now shows what you've typed (Total Commander search box).** Typing in the file list to jump to a file previously gave no visual trace of the accumulated search buffer (Finder-style silent jump), so you couldn't tell what you'd typed so far. A small "🔍 …" capsule now appears at the bottom-left of the list while you type and fades out when the buffer resets. Surfaced through a new optional `FileTableViewDelegate.fileTableView(_:typeAheadBufferChanged:)`.
- **SFTP: filenames with multiple consecutive spaces are no longer mangled.** The `ls -la` parser split the whole line on spaces and rejoined the name with single spaces, so `my  file.txt` (two spaces) became `my file.txt` — an un-openable name. It now splits only the 8 leading metadata columns and keeps the filename remainder byte-for-byte. New `SFTPListingParserTests` lock multi-space names, space-padded single-digit days, symlink/dir classification, and `total`/dot-entry skipping.
- **SMB: directory entries no longer vanish under a non-English locale.** smbclient now runs with `LC_ALL=C` so its `ls` date column stays English; the listing parser's date regex matches English weekday/month names only, so a `ko_KR` / `ja_JP` client locale previously made every entry fail the regex and disappear from the folder. New `SMBListingParserTests` pin the parser.
- **FTP: you can finally enter remote subdirectories.** Directory listings used NLST (`--list-only`), which returns bare filenames with no type information — so every subfolder showed up as a file you couldn't open, making remote trees un-navigable past the top level. Listings now use a full `LIST` (the parser already understood the Unix `drwx…` form; added a Windows/IIS `<DIR>` branch) and fall back to NLST only when the server's format is unrecognized. New `FTPListingParserTests` lock Unix/IIS/NLST parsing, spaces-in-names, symlink-target stripping, and dot-entry skipping.
- **Quick Terminal (backtick) now expands `$F` / `$D` / `$S` / `$f` / `$e` / `$n`.** The Quake-style drop-down terminal sent the raw command straight to the shell, so a `$F` was passed literally instead of the cursor file — unlike the bottom Integrated Terminal, which already expanded them. Both terminals now share the same `TerminalVariableExpander` behavior.
- **Folder Sync no longer treats local and remote hidden files asymmetrically.** The local tree was enumerated with `.skipsHiddenFiles` while the remote tree included dotfiles, so a remote `.DS_Store` / `.git` looked "right only" and could be copied down (or deleted) while local dotfiles were ignored outbound. The remote walk now skips dotfiles / dot-directories too, matching the local side.
- **Folder Sync result count no longer overstates work done.** A direction mismatch (e.g. a left-only file under a right→left sync), or a bidirectional tie / missing modification date, performs no copy but was still counted as "synced". Those entries are now reported as "skipped" and the completion message breaks out `synced / skipped / errors`.

### Changed
- **Cleaner chrome + a useful toolbar center — less clutter, more function around the file panels.** The toolbar's principal slot keeps the **Unifyl** brand lockup (app icon + wordmark) and turns the once-inert current-folder label into an **action hub**: a dropdown that operates on the active panel's location — Reveal in Finder, Copy Path, Open in Terminal Here, Add to Favorites, and a Recent Folders submenu. The font-size stepper, theme picker and command-palette button collapse into a single trailing **⋯** menu, leaving filter / hidden-files / help as the only always-visible right-edge controls (every entry keeps its keyboard shortcut). The bottom upgrade nudge loses its orange gradient pill for a quiet accent text link on the standard chrome surface, picks up the same 1px hairline as the other bars, and can now be dismissed with a **×**. Status and path bars are repadded from the shared spacing tokens so the chrome rhythm is consistent.
- **Upgrade sheet pricing presentation — reframe the one-time price to reduce sticker shock.** Same $39.99, no change to what's billed; the sheet now does the value framing the bare number never did. Four levers, all honest by construction: (1) one-time emphasis — `Buy Pro — $39.99 once` plus a `No subscription. Yours forever on this Mac.` caption on both channels, the strongest lever in a subscription-fatigued market; (2) a competitor value anchor (`Replaces ForkLift ($39.95) + Transmit ($45) + an AI toolkit — one app.`) so the single price reads as a bundle discount; (3) an optional Direct-only struck-through regular price driven by `LicenseConfig.regularPriceDisplay` — **disabled by default (`nil`)**, only to be enabled with a price the Pro variant is actually listed at on LemonSqueezy (a real list price backed by a permanent launch discount, never a fictitious one; also never shown on MAS, where Apple forbids fictitious reference prices); (4) Direct-only approximate local-currency hint (`≈ ₩54,000 · billed via LemonSqueezy`) for non-USD regions via `LicenseConfig.approxLocalPriceForCurrentLocale`, explicitly marked approximate since the real charge happens at checkout (enable LemonSqueezy "show prices in customer's local currency" so the two match). MAS already shows StoreKit's localized `displayPrice`, which is what Apple charges, so it needs neither (3) nor (4).


## [1.4.0] — 2026-05-29

> **Pre-1.4.0 Pro users — your Pro stays Pro.** If you bought Pro via
> LemonSqueezy before 1.4.0, the license restores from Keychain on
> first launch of the new build exactly as before; no action required,
> no re-activation, no risk. The freemium switch only changes what
> NEW users see — anyone who already paid keeps every Pro feature
> they already had. We tested this end-to-end against the existing
> `LicenseManager.checkOnLaunch()` + 7-day-grace-period path.

### Added
- **Freemium business model — Unifyl is now free to download.** Anyone can install Unifyl (from the Direct DMG at unifyl.app or from the Mac App Store) and use the full free tier: dual / triple / free-split pane workflow, command palette, F1–F8 function bar, FTP / SFTP / cloud-mount sidebar, ZIP read, basic search, bookmarks, inline terminal, file history, checksum + permissions view, git status badges, plus every CJK power-user feature already shipped (ALZ archive extraction, HWPX inline preview, Pinyin / Romaji / Hangul-Hanja multilingual search, full-width ↔ half-width folding, EUC-KR / Shift-JIS / GBK encoding conversion, Korean gov form heuristic detection), all 15 UI locales including RTL Arabic, all 12 built-in themes. The 50+ Pro features stay gated behind a single one-time purchase. See `docs/business/freemium-tiers.md` for the full split.
- **Mac App Store distribution — Pro via In-App Purchase.** The MAS-Lite build (1.3.1+) was always sandboxed-and-ready architecturally; 1.4.0 adds the StoreKit 2 path to Pro. New `MASLicenseProvider` (`Unifyl/Services/MASLicenseProvider.swift`) observes `Transaction.currentEntitlements` for the non-consumable IAP `com.unifyl.app.maslite.pro` (Tier 40 / $39.99, Family Sharing enabled). On every launch + after every purchase / refund / "Restore Purchases" tap, the provider re-resolves ownership and pushes the result into `LicenseManager.storeKitTier`. The `UpgradeSheet` on MAS-Lite now shows the localized App Store price (via `Product.displayPrice`) and routes the `Buy Pro` button to `Product.purchase()` instead of the Direct LemonSqueezy checkout — Direct builds still use LemonSqueezy unchanged (`#if MAS_LITE` keeps the two paths cleanly separated). A new `Restore Purchases` button covers the "I bought it on a different Mac" flow. Setup guide at `docs/business/mas-iap-setup.md` covers the App Store Connect IAP record + sandbox testing + review submission.
- **`LicenseManager.activatedTier` is now `max(lemonSqueezyTier, storeKitTier)`.** Both unlock paths feed the same `FeatureGateManager.isUnlocked(.x)` runtime gate; whichever resolves to Pro wins. Pre-1.4.0 Pro users (LemonSqueezy purchase) are entirely unaffected — the LemonSqueezy validate-on-launch + 7-day grace period + offline-keychain restore all keep working untouched. A future MAS-to-Direct (and reverse) recognition layer is sketched in `docs/business/freemium-tiers.md` but not part of 1.4.0.

### Changed
- **Marketing positioning shifts from "paid app" to "free with optional Pro upgrade".** Landing-page CTA, README intros, App Store description, and the in-app trial banner copy all need a coordinated refresh; tracked separately from this code release.


## [1.3.2] — 2026-05-27

### Changed
- **MAS-Lite backlog reaches zero `[drop-pending-lib]` — every Process() site in the app target is now either truly in-process migrated or a documented `[drop]`.** Two final-pass moves:
  - **FileXRayView.ZipArchiveReader.entries migrated to `InProcessZipReader`** — was previously `[drop]` returning empty in MAS-Lite (and `/usr/bin/zipinfo` subprocess in Direct). Now both targets share one in-process path; the FileXRay ZIP analysis tab works in MAS-Lite, and Direct also drops the zipinfo spawn. Mapping layer between `InProcessZipReader.Entry` (UInt32 sizes) and the local `XRayNode`-shaped `Entry` (UInt64). FileXRayView gains an `import UnifylFileSystem`.
  - **Git (3 sites) + MediaConverter ffmpeg-check (1 site) reclassified `[drop-pending-lib]` → `[drop]`.** Both families need work beyond a single sprint: Git needs a libgit2 SPM dependency (SwiftGit2 / ObjectiveGit — license vetting, MAS submission review of the bundled libgit2 binary, Swift 6 strict-concurrency audit, and full operation mapping); `which ffmpeg` has no Foundation equivalent and MAS forbids unsigned CLI binaries in `Resources/`. The comments now spell out the deferral rationale so a future contributor sees what the lift would be instead of seeing "pending-lib" and assuming it's small. MAS-Lite users see localized "Git integration is not available in the Mac App Store variant — use unifyl.app for git" / "ffmpeg unavailable" messages exactly as in the prior pass; no UX change.
  - **MAS-INCOMPAT site breakdown: 17 `[drop]`, 0 `[drop-pending-lib]`, 0 `[lib]`.** Every `Process()` spawn in the MAS-Lite binary is documented and either compiled out via `#if MAS_LITE` or routed through an in-process equivalent. The 6 in-process equivalents added across 1.3.x-Unreleased sprints: pure-Swift content search (grep → FileManager enumerator), `InProcessZipReader` (ditto-on-xlsx → in-process xlsx preview), `InProcessZipWriter` (zip → Compress dialog), `InProcessTarWriter` (tar / tar.gz → Compress dialog), `Diff3Merger` (diff3 → 3-way merge), `PDFConvertSheet.textutil` → `NSAttributedString`. UnifylFileSystem test count: 102 → 128 (26 new tests for the new libraries).

### Added
- **New `InProcessMachOReader` + `InProcessDMGReader` (Packages/UnifylFileSystem) + FileXRay deep parsers now sandbox-safe.** Pure-Swift Mach-O parser (~280 LOC) replacing `/usr/bin/otool -l`: parses magic (32/64/fat-universal, both endianness), header (cputype/cpusubtype/filetype/ncmds/flags), and load commands with typed per-LC interpretation (LC_SEGMENT[_64] names, LC_LOAD_DYLIB paths + versions, LC_UUID canonical string, LC_RPATH, LC_VERSION_MIN_*, LC_BUILD_VERSION platform + version, LC_MAIN entry-point offset). Returns a structured `Analysis { isFat, architectures, slices }` so the FileXRay tree view can group fat slices and show typed fields per LC instead of text-scraping otool output. Companion `InProcessDMGReader` detects UDIF disk images by the trailing "koly" magic and reads the resource-file trailer (total size + data fork + resource fork). Deep DMG inspection (partition tables, encrypted volumes) needs hdiutil + privileges and isn't sandbox-compatible — out of scope. **`FileXRayView.analyzeMachO` + `analyzeDiskImage` migrated** — both targets share the new in-process path, the now-unused `runProcess(path:arguments:)` removed entirely from the file. The Mach-O tree view actually surfaces more information than the previous otool text-scrape (fat slices are first-class, per-LC fields are typed). 7 new tests against system `/bin/ls` (fat universal binary fixture) + synthetic UDIF trailer + truncated-file rejection. UnifylFileSystem test count 128 → 135. MAS-INCOMPAT site count drops by 2 (otool + hdiutil sites collapse into one shared `[drop]`-comment block now covering only mdls, which the codebase doesn't actually call).
- **New `Diff3Merger` (Packages/UnifylFileSystem) + 3-way merge now sandbox-safe.** Pure-Swift LCS-based 3-way merge replacing `/usr/bin/diff3 -m` in MAS-Lite. ~250 LOC. Algorithm: compute LCS(base, left) and LCS(base, right) — each gives a monotone (base_idx, other_idx) pair sequence — then take the intersection on base index as the stable anchor set; everything between anchors is a hunk classified by comparing the base / left / right slices (all-same → emit base, one-side-changed → take that side, both-changed-identically → take the change, otherwise → emit `<<<<<<< LEFT … ======= … >>>>>>> RIGHT` markers identical to `diff3 -m` so `ThreeWayMergeView.parseConflicts` reads the output unchanged). **`ThreeWayMergeView.analyzeMerge` `#if MAS_LITE` branch migrated** — reads file bytes off-main via `Task.detached`, calls `Diff3Merger.merge`, returns the merged text + conflict list to the existing UI. Direct keeps `/usr/bin/diff3`. 8 new tests: no-change / left-only / right-only / both-same / true-conflict / non-overlapping changes / left-side insertion / marker round-trip. Line-granular (not character-level), O(N×M) memory — covers source-file 3-way merges up to a few thousand lines per side. UnifylFileSystem test count 120 → 128. MAS-INCOMPAT `[drop-pending-lib]` backlog: 5 → 4.
- **New `InProcessTarWriter` (Packages/UnifylFileSystem) + Compress dialog tar / tar.gz paths now sandbox-safe.** Companion to InProcessZipWriter: ~250 lines covering POSIX USTAR (512-byte header per entry with octal-text size + mtime + mode + 8-byte checksum; payload zero-padded to 512-byte boundary; two trailing zero blocks marking end-of-archive) plus an optional gzip wrapper (10-byte header + raw DEFLATE via `compression_encode_buffer(COMPRESSION_ZLIB)` + 8-byte CRC-32 / ISIZE footer per RFC 1952). Round-trips through host `/usr/bin/tar` (`-tf` lists every entry, `-xf` restores byte-for-byte; same for `-tzf` / `-xzf` on the gzip path). Long-path handling splits paths > 100 chars into USTAR `prefix` + `name` on the latest `/` boundary that fits; rejects paths beyond 255 chars with typed `.pathTooLong`. **`CompressDialogView.compressTarWithProgress` `#if MAS_LITE` branch migrated** — MAS-Lite can now create `.tar` and `.tar.gz` archives in-process with per-entry progress. Direct keeps `/usr/bin/tar`. **CRC-32 implementation extracted to a shared `InternalCRC32` enum** so the ZIP writer and gzip footer share one table + one routine instead of duplicating. 5 new tests: plain TAR round-trip / TAR directory tree / `.tar.gz` round-trip + gzip-magic byte check / unicode filenames / progress callback. UnifylFileSystem test count 115 → 120. MAS-INCOMPAT `[drop-pending-lib]` backlog: 6 → 5.
- **New `InProcessZipWriter` (Packages/UnifylFileSystem) + Compress dialog ZIP path now sandbox-safe.** Sibling to InProcessZipReader: a 300-line pure-Swift PKZIP writer using Foundation's `compression_encode_buffer` (COMPRESSION_ZLIB → raw DEFLATE, same encoding as the reader's decode side) plus a standard reflected polynomial CRC-32 table. Walks source URLs depth-first via `FileManager.enumerator` with `nextObject()` (Swift 6 async-safe); resolves symlinks before building entry-relative paths (`/tmp` ↔ `/private/tmp` on macOS used to bite the recursion). Per-entry: DEFLATE first, fall back to STORED when DEFLATE bloats incompressible data (jpg / mp4 / random). UTF-8 filename flag set (bit 11 of general-purpose). Refuses ZIP64 (4 GiB single-entry cap), passwords (ZipCrypto is weak, AES is heavier), and ZIP encryption — those stay on the Direct `/usr/bin/zip -e -P` path. **`CompressDialogView.compressZipWithProgress` `#if MAS_LITE` branch migrated** — MAS-Lite can now create ZIP archives in-process with progress reporting per entry. Direct keeps `/usr/bin/zip` for raw speed. 7 new tests cover single-file / multi-file / nested directory tree / unicode filenames / incompressible STORED fallback / password rejection / progress callback firing per entry. UnifylFileSystem test count 108 → 115. MAS-INCOMPAT `[drop-pending-lib]` backlog: 7 → 6.
- **New `InProcessZipReader` (Packages/UnifylFileSystem) + xlsx preview now sandbox-safe.** A 200-line pure-Swift ZIP reader that parses End-Of-Central-Directory + central directory + local file headers, and decompresses STORED (method 0) + DEFLATE (method 8) entries via Foundation's Compression framework (`COMPRESSION_ZLIB` operates on raw DEFLATE — exactly what ZIP entries store). Sized to the few use cases that need it: xlsx / docx / pptx central-directory pull. Rejects ZIP64 (sentinel `0xFFFFFFFF` on size fields) and unsupported compression methods with typed `ReaderError`. **OfficePreviewView.parseAllSheets migrated to use it for both Direct AND MAS-Lite** — both targets now read xlsx in-process with zero `ditto` subprocess, zero tmp dir, identical XML-parsing pipeline. ArchiveEngine still owns the broader read/write surface for Direct (7zz + unalz); InProcessZipReader is the narrow sandbox-safe alternative. 6 new tests cover round-trip / unicode filename / missing entry / non-zip rejection / multi-block DEFLATE. UnifylFileSystem test count 102 → 108. MAS-INCOMPAT `[drop-pending-lib]` backlog: 8 → 7.
- **MAS-Lite: pure-Swift content search (grep → FileManager enumerator).** ContentSearchView's `#if MAS_LITE` path now does a real in-process directory sweep instead of returning an empty stub. Uses `FileManager.default.enumerator(at:, options: [.skipsHiddenFiles, .skipsPackageDescendants])` walked via `.nextObject()` (Swift 6 async-safe), filters by extension (the existing `*.swift` / `*.txt` glob input), respects case-sensitivity, caps individual file size at 16 MB, caps total matches at 50 (same as the Direct path's `grep -m 50`). Tries UTF-8 then ISO-Latin-1 before skipping unreadable files. Identical `[ContentSearchResult]` shape as the grep path so the view doesn't branch. Direct builds keep `/usr/bin/grep` for raw speed. MAS-INCOMPAT `[drop-pending-lib]` backlog: 9 → 8.
- **Git + Compress stub strings translated to all 15 locales.** "Git integration is not available…" + "Compress is not available…" now ship in ko / ja / zh-Hans / zh-Hant / fr / de / es / pt-BR / it / ru / ar / th / vi / tr (en baseline). Audit-guard F4 remains 0 drift.

### Changed
- **All remaining `[lib]` Process() sites in the app target now `#if MAS_LITE`-wrapped.** After this pass the MAS-INCOMPAT backlog is 14 `[drop]` (fully wrapped) + 9 `[drop-pending-lib]` (wrapped now, true library replacement deferred — git → SwiftGit2, zip/tar → ArchiveEngine, grep → pure-Swift sweep, diff3 → pure-Swift 3-way merge, ditto on xlsx → ArchiveEngine entry API, etc.). Sites wrapped in this pass:
  - `GitPanelView.runGit`, `GitHistoryView.GitHistoryHelper.runGit`, `GitStatusProvider.runGitCommand` — returns a localized "Git integration is not available in the Mac App Store variant…" stub (3 sites in the git family share one stub string).
  - `ThreeWayMergeView.analyzeMerge` — diff3 absent, MAS-Lite shows empty merge result.
  - `ContentSearchView.runGrep` — grep absent, MAS-Lite returns empty content-search results.
  - `OfficePreviewView.parseAllSheets` — ditto-on-xlsx absent, MAS-Lite shows the existing fallback preview.
  - `CompressDialogView.compressZipWithProgress` / `compressTarWithProgress` — zip/tar binaries unavailable, MAS-Lite throws a localized "Compress is not available…" error (2 sites share one stub string).
  - `MediaConverterView.checkFFmpeg` — reports `ffmpegAvailable=false` immediately so the Convert button shows the existing hint.
  - `PDFConverter.convertWithLibreOffice` — LibreOffice absent, returns false.
  - `FileXRayView.runProcess` + `ZipArchiveReader.entries` — otool / hdiutil / mdls / zipinfo absent, deep file analysis tab degrades gracefully.
  - Total `#if MAS_LITE` directives in the codebase: 19 → 36.

### Added
- **MAS-Lite first-launch onboarding NSOpenPanel.** Sandboxed App Store builds can't read any folder until the user explicitly grants access via a `files.user-selected.read-write` open panel. `MainWindowView.promptForMASRootPickerIfNeeded()` fires once on first launch (after the existing onboarding overlay dismisses), opens an NSOpenPanel rooted at `~/Downloads` (entitlement granted), and on selection saves a `withSecurityScope` bookmark via `SecurityBookmarkManager.saveBookmark(for:)` + navigates the active panel there. `unifyl.mas.hasSeenRootPicker` UserDefaults flag stops the prompt re-firing on subsequent launches. Direct builds compile this block out — no behavior change for the notarized DMG distribution.

### Changed
- **MAS-Lite startup is now sandbox-aware end to end.**
  - `AutoBookmarkScanner.detected()` returns `[]` in MAS_LITE — the 12 known messenger / cloud paths all live under `~/Library/Containers/<other-app>/` or `~/Library/CloudStorage/`, neither of which the sandbox grants access to. Probing them with `fileExists(atPath:)` would either silently fail or trigger TCC dialogs at app launch. MAS users add bookmarks manually via Cmd+Shift+O.
  - `CloudMountScanner.scan()` returns `[]` in MAS_LITE — `~/Library/CloudStorage/` listing is sandbox-blocked. The Connections sidebar no longer shows phantom "Permission denied" cloud mount entries on App Store builds.
  - `AppViewModel.init` only restores tabs whose security-scoped bookmark is on file (`SecurityBookmarkManager.hasBookmark(forPath:)` — new cheap lookup that doesn't resolve, used inside `init` where async work isn't possible). Falls back to `~/Downloads` (entitlement granted) so the panel has SOMETHING to show before the onboarding NSOpenPanel fires.
- **Function Key Bar default flipped back to ON.** 1.0.8's design overhaul had defaulted the TC-style F1–F8 hint bar to OFF on the theory that "Mac-natives don't want it." User feedback after 1.3.1 indicated TC migrators expect it visible at first launch — that's the whole point of the "TC parity + Mac polish" pitch. `@AppStorage("unifyl.showFunctionKeyBar")` default flipped `false` → `true` in MainWindowView, GeneralSettingsView, and the View-menu toggle. Users who explicitly toggled it OFF previously keep that preference (UserDefaults key already set); only new installs and users who accepted the prior default see the change. The View → Show Function Key Bar toggle and the Settings → General toggle still work for anyone who wants to hide it. The first-launch `hasSeenFnGuide` banner above the bar explains that MacBook users need the Fn modifier (default top-row maps to brightness / volume / Mission Control).


## [1.3.1] — 2026-05-26

### Changed
- **MAS-Lite migration: first `[lib]` site replaced with native API + 3 more `[drop]` sites wrapped**:
  - **PDFConvertSheet.textutil → NSAttributedString in-process** (true `[lib]` win): `convertDocToPDF` for legacy `.doc` / `.docx` / `.rtf` / fallback no longer spawns `/usr/bin/textutil`. Reads `Data(contentsOf:)` off-main, then constructs `NSAttributedString(data:options:documentAttributes:)` on the main actor (TextKit Cocoa importers sniff the document type from the data prefix — the same path textutil(1) wraps internally). Renders to PDF via a new `Self.renderAttributedToPDF(_:destination:)` helper extracted from the existing `convertTextToPDF` so both code paths share one NSPrintOperation pipeline.
  - **AboutView git fallback wrapped with `#if !MAS_LITE`**: the dev-build commit-count / short-hash lookup (only ever runs when CFBundleVersion is the placeholder "1", which release builds override) is gone from the MAS-Lite binary entirely. About-view footer degrades to plist values, which is correct for the App Store distribution.
  - **PDFConvertSheet osascript Office-to-PDF wrapped with `#if !MAS_LITE`**: the iWork / Microsoft Office AppleScript automation path would need a per-target `com.apple.security.scripting-targets` entitlement plus explicit consent. Reclassified as `[drop]` for MAS-Lite; Direct keeps the existing AppleScript flow.
  - **AudioConverterView afconvert wrapped with `#if MAS_LITE`**: `/usr/bin/afconvert` is Apple-bundled but `Process()` is still sandbox-banned. Returns a localized `ConversionResult(success: false, message: …)` in MAS variant. A proper `[lib]` replacement (AVAudioConverter + ExtAudioFile in-process) is tracked as a future sprint.
  - **One new locale key × 15 locales**: `"Audio conversion is not available in the Mac App Store variant. Use the direct download from unifyl.app to convert audio files."` translated to ko / ja / zh-Hans / zh-Hant / fr / de / es / pt-BR / it / ru / ar / th / vi / tr (en baseline). Audit-guard F4 remains 0 drift.
  - **Three mislabeled MAS-INCOMPAT tags corrected**: `[lib] system_profiler` → actually git, `[lib] ffmpeg` → actually afconvert, `[lib] LibreOffice subprocess` → actually osascript. The 26-site backlog now reflects the real binaries.
  - Verification: `make audit` → 13/13 PASS, 0 WARN, 0 FAIL. Both `Unifyl (Debug)` and `Unifyl MASLite (Debug)` `xcodebuild` BUILD SUCCEEDED. UnifylCore tests 131/131, UnifylFileSystem 102/102.

### Added
- **Separate UnifylMASLite Xcode target shipped**: `project.yml` now defines a parallel application target that produces a Mac App Store-ready build alongside the Direct distribution. `com.unifyl.app.maslite` bundle ID (vs `com.unifyl.app`), separate `Info.MASLite.plist` (no Sparkle keys), separate `Unifyl.MASLite.entitlements` (sandbox=on, user-selected files RW, security-scoped bookmarks, network.client, print), `SWIFT_ACTIVE_COMPILATION_CONDITIONS=MAS_LITE` so all 9 `[drop]` wraps fire, no Sparkle SPM dependency, no 7zz/unalz `Resources/bin/` postBuildScript. Two Xcode schemes (`Unifyl MASLite (Debug)` + `(Release)`) for everyday dev and archive. Sparkle imports in `UnifylApp.swift` / `AboutView.swift` / `CheckForUpdatesView.swift` (and their menu/button call sites) are `#if !MAS_LITE`-gated so neither variant pulls the framework needlessly. Verified: `xcodebuild` of both targets succeeds; the built `UnifylMASLite.app` has `com.apple.security.app-sandbox` live, no `Frameworks/Sparkle.framework`, no `Resources/bin/`. The two `.app` bundles can be installed side-by-side on one Mac.

### Added
- **MAS-Lite scaffold complete — 9 of 9 `[drop]` Process() sites wrapped**: `#if MAS_LITE` guards now cover every subprocess invocation classified as "drop in MAS variant" — ScriptRunner.launch, InlineTerminalView.ProcessRunner.run, CommandLineBar shell exec, SSHTunnelView.startTunnel, DockerExplorerView.runShellCommand, PortViewerView.runShellCommandForPorts, ProcessFileMapView.runShellCommandForProcessMap, NewConnectionSheet.runTestProcess (ssh-keygen), MediaConverterView.convertWithFFmpeg. Each MAS_LITE branch returns or throws a localized "feature unavailable in the Mac App Store variant — use the direct download from unifyl.app …" message via a typed `LocalizedError` enum (per audit-guard F11). Both `xcodebuild` (default) and `xcodebuild OTHER_SWIFT_FLAGS=-DMAS_LITE` BUILD SUCCEEDED. The remaining `[lib]` sites (git, ffmpeg main path, textutil etc.) need actual library replacements rather than wraps — separate effort.
- **All 15 locales now translated for F7-lite + MAS-Lite strings**: 9 new entries × 14 secondary locales = 126 translations on top of en baseline. `make audit` reports parity across en / ko / ja / zh-Hans / zh-Hant / de / fr / es / pt-BR / it / ru / tr / ar / th / vi with zero WARN, zero FAIL. Every CJK / European / SEA / RTL user sees the F7-lite badge ("📋 Korean government / public-institution form (heuristic)" + "%lld signature phrase(s) matched") and any MAS-Lite stub messages in their language.

### Added
- **F7-lite UI badge in HWPX preview**: HWPXTextPreview now runs the gov-form heuristic on the extracted body and shows a tinted pill above the text when `isLikelyGovForm`. Hover the badge to see every matched signature phrase. Respects the `unifyl.feature.koreanGovFormDetection` opt-out.
- **F7 fingerprint database scaffold**: `KoreanGovFormDatabase` + bundled `Resources/data/kr-gov-forms.json` seed with 3 placeholder entries (표준 도급 계약서 / 부가가치세 신고서 / 민원 신청서) + SHA-256 + bytes-level matcher. Curation guide at `scripts/data/README-kr-gov-forms.md` documents the schema and 50-form rollout target. 8 unit tests on the loader + match logic.
- **MAS-Lite compile-flag scaffold**: `Unifyl/Automation/ScriptRunner.swift` demonstrates the canonical `#if MAS_LITE / #else` wrap pattern — throws a typed `LaunchError.unavailableInMASVariant` instead of running `Process()` when the flag is set. Both `xcodebuild` (default Direct path) and `xcodebuild OTHER_SWIFT_FLAGS=-DMAS_LITE` (MAS-Lite path) succeed. Future MAS-Lite target just sets `SWIFT_ACTIVE_COMPILATION_CONDITIONS=MAS_LITE` and incrementally wraps the remaining 11 `[drop]` sites. Updated `docs/business/mac-app-store-readiness.md` to mark the audit prep-work checkbox ✅.

### Added
- **CJK wedge F7-lite — Korean government form heuristic detection**: new `KoreanGovFormDetector` enum recognises 31 high-density Korean form signature phrases (주민등록번호 / 사업자등록번호 / 신청인 / 신고인 / 결재자 / 기안자 / 별지 제 / 호 서식 / 작성일자 / 신청합니다 / 위와 같이 / 우편번호 / 도로명주소 / 직인 / 귀하 / 갑(이하 / 을(이하 etc.). Threshold of 3 distinct hits keeps single-mention false positives out of the badge set. 256 KB scan window caps worst-case latency at ~100 ms on a 1 MB document. Pure-text input (caller extracts body via HWPXParser F2). 9 unit tests including 4 real-world templates (신청서, 도급 계약서, 부가가치세 신고서). Foundation work for the full F7 fingerprint-database UI; badge wiring lands in a follow-up.
- **50K-item ASCII filter benchmark**: PerformanceTests gained two new cases — empty-query identity (sub-ms) and 50K ASCII substring (270 ms, under the 500 ms type-ahead budget). The suite now covers 10K and 50K across empty / ASCII / CJK / multilingual / fullwidth / regex / glob paths.

### Changed
- **MAS-INCOMPAT comment sweep — 26 `Process()` sites tagged**: all subprocess invocations in the app target (`Unifyl/Views/*`, `Unifyl/Services/*`, `Unifyl/Automation/*`) now carry a `// MAS-INCOMPAT: [drop|lib] <migration note>` comment one line above. Future Mac App Store conversion can `grep -r MAS-INCOMPAT Unifyl/ | sort -u` to see the entire migration backlog with per-site replacement notes (git → SwiftGit2, ffmpeg → AVAssetExportSession, tar/ditto → Compression framework, etc.). Comments-only change — zero behaviour delta.

### Fixed
- **Force-unwrap in AzureBlobFileSystem paging loop**: the `do { ... } while marker != nil && !marker!.isEmpty` condition relied on a manual nil-check + force-unwrap. Rewrote as `marker.map { !$0.isEmpty } ?? false` so a future contributor doesn't trip over the unwrap if they refactor the surrounding code.

### Added
- **Universal undo for encoding conversion**: the "Convert to UTF-8" Replace-Original path now writes a `.unifyl-bak.<unixtime>.<ext>` sidecar with the original bytes BEFORE overwriting. The batch is recorded as a single `FileOperationHistory.encodingConvert` op so Cmd-Z restores every converted file at once. Power-cut safe — the backup lands before the rewrite, leaving recoverable bytes on disk even if the in-place write fails mid-flight.
- **10K-item search performance benchmark**: new `PerformanceTests` suite in UnifylCore that exercises ASCII substring / CJK substring / multilingual expansion / fullwidth fold / regex / glob paths against a 10K-item synthetic listing. Runs as part of `swift test`; budget assertions catch regressions before release.

### Changed
- **Hangul → Hanja search expansion: 563 ms → 36 ms (15.6× faster on 10K items)**. The expansion can produce 200+ single-character Hanja candidates for a 2-syllable query; the old per-item linear loop over all candidates was the dominant per-keystroke latency on Korean-content directories. Switched to a Set<Character> membership scan over haystack chars for single-character expansion candidates; multi-character candidates (original query, Romaji hira/kata segments, fullwidth-folded forms) still use the substring path. Net effect: Korean type-ahead is now instant on listings up to 50K items.

### Fixed
- **Encoding conversion sheet was blocking main actor during disk I/O**: `runDetection` / `performConversion` now hop to `Task.detached` so per-file read + decode + write don't freeze the spinner during a multi-file batch. `Detection` and `ConversionResult` gained explicit `Sendable` conformance for the boundary crossing.
- **Fire-and-forget `Task` after encoding conversion**: the prior code chained `Task { sleep(2); refreshPanel; dismiss }` which raced manual sheet dismissal — closing the sheet during the sleep still fired the refresh against a stale view tree. Replaced with synchronous `await` in the same MainActor context so ordering is deterministic.

### Added
- **CJK wedge F4 — Messenger / cloud-sync folder auto-bookmarks**: KakaoTalk Downloads, KakaoTalk Images, WeChat Files (both sandboxed and `~/Documents`), LINE Files, Whale Downloads, Naver MyBox, Baidu NetDisk, iCloud Drive, Pictures/Screenshots, Telegram, Slack — auto-detected and surfaced in a new "Auto-detected" section above the user's curated bookmarks. Each row gets context-menu actions (Open / Hide / Promote to Bookmark). Dismiss persistence via UserDefaults key `unifyl.bookmarks.autoDismissed`; feature gate `unifyl.feature.autoBookmarks`. 8 unit tests covering empty / single / multi / dismiss round-trip / canonical-id / idempotence on a fake HOME.
- **CJK wedge F5 — Fullwidth ↔ halfwidth search normalization**: typing `123` matches `１２３_見積書.xlsx`; typing `２０２６` matches `report-2026.pdf`. Direction-symmetric folding via `CFStringTransform`. Opt-out toggle `unifyl.search.fullwidthHalfwidthFolding` (default on). 7 new test cases covering both fold directions, pure-ASCII pass-through, Korean / generic-CJK unaffected, opt-out actually disables folding on `ko_KR` / `ja_JP` locales (where `localizedCaseInsensitiveContains` silently folded — we use locale-free `.caseInsensitive` comparison after folding).
- **CJK wedge F6 — Encoding conversion context menu**: right-click any `.txt` / `.csv` / `.srt` / `.log` / `.md` / etc. → "Convert Encoding to UTF-8…". Auto-detects CP949 (Korean), Shift-JIS (Japanese), GB18030 (Simplified Chinese), Big5 (Traditional Chinese), ISO-8859-1 fallbacks via the existing `TextEncodingDetection` chain. Sheet shows per-file detected encoding + 200-char preview + Replace-Original or Save-As-New toggle. Atomic write + modification-date preservation. Batch conversion across multi-selection. Binary files rejected at detection time (NUL-byte sniff). 10 unit tests covering CP949 / Shift-JIS / binary rejection / empty rejection / sidecar naming / mtime preservation.

### Fixed
- **Silent theme export failure**: `CustomTheme.exportTheme` previously used `try?` for both `JSONEncoder().encode` and `Data.write` — disk-full / permission-denied / read-only-volume errors were swallowed silently. Users saw nothing happen and assumed the file landed. Now logs to `subsystem:com.unifyl.app category:Settings` and surfaces an NSAlert anchored to the editor window.


## [1.3.0] — 2026-05-17

### Added
- **CJK wedge F1 — ALZ archive support**: `.alz` archives (Korean legacy format from ESTsoft's ALZip) now extract via bundled `unalz` binary (kippler/unalz, BSD-licensed, universal arm64+x86_64, codesigned). Read-only — listing, extract-all, and selective-extract supported; create/add/remove/rename throw "read-only" errors. ALZDriver routes through ArchiveEngine's split extension files (Extract / List / Create / Modify). EGG (also ESTsoft) deferred to 1.2.1 — no maintained open-source extractor.
- **CJK wedge F2 — HWP inline preview**: HWPX files (Hancom Office's XML format, .hwpx) now preview inline in the file panel via pure-Swift section-XML text extractor (HWPXParser.extractText). 6 parser tests cover empty doc / multi-paragraph / XML entities / multi-run paragraphs / real HWPX fixture. Legacy binary .hwp formats (HWPML/HSP/HWT) deferred to F9 in 1.3.0.
- **CJK wedge F3 — Multilingual filename search**: type "wen" → matches 文書/文件/文字 (Pinyin → Hanzi via CC-CEDICT-derived map), "tokyo" → matches 東京 (Romaji → Kana → Kanji), "한글" → matches 韓國語 (Hangul → Hanja via Unihan map). All three transforms ship with bundled JSON resource data and per-transform tests. Opt-out toggle in Settings > General; telemetry counts expansion-driven hits for future tuning.
- **Global rollout — landing page**: docs/landing/ is now built per-locale (15 languages) from `docs/landing/i18n/keys.{locale}.json` + a shared template via `scripts/build_landing_i18n.py`. Each variant ships with the correct `lang`/`dir` attribute, hreflang map (15 + x-default), OG locale tag, and an in-nav language switcher. Arabic gets `dir="rtl"`. Hosting layout: `/` is English (canonical), `/ja/`, `/de/`, `/zh-Hans/`, … serve translated copies.
- **Global rollout — README**: top-5 README localizations (`README.ja.md`, `README.zh-Hans.md`, `README.de.md`, `README.es.md`, `README.fr.md`) plus a language-switcher banner on the canonical English README. Abbreviated form — full plug-in SDK / contribution docs stay single-source in English.
- **Global rollout — Info.plist**: `Unifyl/Resources/{locale}.lproj/InfoPlist.strings` for all 15 locales. Localizes `NSHumanReadableCopyright`; `CFBundleDisplayName` / `CFBundleName` stay as the brand "Unifyl". Picked up by XcodeGen's source sweep — no project.yml change needed.
- **Global rollout — marketing copy**: `docs/marketing/i18n/copy.{ja,zh-Hans,de,es,fr}.md` — reusable per-locale tagline / long description / category tags / pricing phrasing / CTA variants, drives every directory submission from one file per language instead of ad-hoc translation.

### Fixed
- **RTL (Arabic) navigation directionality**: 7 SF Symbols that used literal-direction variants now use auto-mirroring directional variants. Toolbar back/forward (`chevron.left/right` → `chevron.backward/forward`), breadcrumb separators in PathBarView and ArchivePathBarView (`chevron.right` → `chevron.forward`), and rename-flow indicators in MultiRenameView / RegexRenameView / LargeFileFinderView (`arrow.right` → `arrow.forward`). Bidirectional icons (`arrow.left.arrow.right`) left as-is.


## [1.2.3] — 2026-05-13

### Fixed
- **2 NSAlert sites in sheet-hosted views switched to `beginSheetModal(for:)` instead of `runModal()`.** `NLCommandPreviewSheet` (sheet host confirmed at `MainWindowView:631`) and `AIEngineViewModel.applyKeepBest` (called from `EnhancedDuplicateView`, sheet status indeterminate). NSAlert.runModal() from inside a SwiftUI sheet has no guaranteed presentation path — same family of defect as the PDF Tool sheet-modal fix earlier in [Unreleased]. The other 8 NSAlert.runModal() sites were audited and confirmed safe (menu / keyboard / file-op / Sparkle launch contexts, all main-window-modal).
- **15 persistence sites now route corrupt blobs through `PersistenceBackup` instead of `try?`-wiping user data.** The pattern `guard let decoded = try? JSONDecoder().decode(T.self, from: data) else { return }` silently dropped the user's saved data on schema drift across an upgrade. Sites converted: SSHTunnelView (saved tunnels), ScheduledTaskView (cron jobs), NewConnectionSheet (OAuth tokens — Keychain path), IncrementalBackupView (backup profiles), FileColorSettings (per-category colors), IconThemeManager × 3 (installed themes / icon-pack mapping / external themes — all file-based, use `stashCorruptFile`), SSOAuthManager × 2 (Keychain — corrupt entry now deleted + logged so re-sign-in is forced instead of looping), KeychainService.debugLoadAll (DEBUG-only dev keychain), SecurityBookmarkManager (legacy migration), RecentPathsStore (recent folders), Companion SessionManager (paired devices), CompanionServerConfig (companion server config). UserDefaults blobs are stashed to `<key>.corrupt.<timestamp>` (recoverable from `defaults read`); on-disk files are moved to `<name>.corrupt.<timestamp>.<ext>` sidecars. TeamSyncClient's 403-body decode is intentionally left on `try?` because that's a server-controlled response, not user data. Per memory `feedback_persistence_decode_must_stash`.
- **AIEngine no longer crashes the app when every disk path for the vector index fails.** Previous: `fatalError("AIEngine: unable to create VectorIndex at any path")` after the primary / temp / UUID-stamped emergency paths all failed (full disk, sandbox denial). Now falls back to SQLite `:memory:` so AI search and indexing keep working for the current session — they just don't persist across launches. The `fatalError` is preserved as a backstop for the (unreachable) case where bundled SQLite itself is broken, so support still gets a crash report. Added `VectorIndex.init()` that opens SQLite `:memory:`.
- **All 7 hardcoded English alert + confirmation-dialog titles now ship native translations in all 15 locales.** `.alert("Error", isPresented:)` / `.confirmationDialog("Sync All Differences?", ...)` and friends — SwiftUI auto-infers `LocalizedStringKey` from string literals, but the keys (`System Shortcut`, `Reset all keyboard shortcuts?`, `Save Macro`, `Navigation Error`, `Save Backup Profile`, `Plugin Installation Failed`, `Sync All Differences?`) had no entries in any `Localizable.strings`, so non-English locale users saw English fallback. Generated +7 entries per locale across ko / ja / zh-Hans / zh-Hant / de / fr / es / pt-BR / it / ru / tr / ar / th / vi / en (105 total). The other alert keys (`Error`, `Move duplicates to Trash?`, `Leave this team?`, `Sign out of SSO?`, `Delete AI index?`) were already translated.
- **`Select PDF...` (and every other "Browse for file" button inside a sheet) now actually opens the panel.** PDF Tool, Folder Report export, SSH Tunnel key chooser, Large-File Finder scan-root picker, Scheduled Task source/dest pickers, Audio Converter output folder, Audio Metadata album-art chooser — all of them are SwiftUI sheets, and inside a sheet `NSOpenPanel.runModal()` / `NSSavePanel.runModal()` is silently swallowed by the outer modal hierarchy. The buttons appeared dead. Switched all 9 sites across 7 views to `panel.begin { … }` (application-modal, attaches above the sheet) with a `Task { @MainActor in … }` continuation for the post-pick state mutation.
- **Audit-driven follow-up: 14 more sheet-hosted `NSOpenPanel.runModal()` sites that were missed in the previous pass.** Same family of defect as above — all hosted in SwiftUI sheets and silently swallowed. Sites converted: `AIOnboardingSheet.chooseFolderAndIndex`, `NewConnectionSheet` SSH-key picker, `AdvancedSearchView` × 3 (scope change, copy destination, move destination), `IncrementalBackupView.pickFolder`, `FileSplitMergeView` × 2 (split source, merge first-part), `SemanticSearchView` Choose-Directory button, `FileOperationConfirmView` destination Browse, `CompressDialogView` destination Browse, `MediaConverterView` output-folder Choose. After this pass, every confirmed sheet-hosted `runModal()` on either NSAlert or panel is gone. `CustomTheme.exportTheme/importTheme` keep `runModal()` because they're called from `ThemeEditorView`, which is hosted as an independent `NSPanel` via `ViewerWindowManager` — not a sheet, so `runModal()` works correctly there.
- **Companion error messages now ship native translations in all 15 locales instead of always rendering in Korean.** `CompanionError.errorDescription` had 10 cases (notPaired, pairingFailed, connectionLost, connectionTimeout, serverUnreachable, authenticationFailed, permissionDenied, transferFailed, pathNotAllowed, rpcError) with hardcoded Korean string literals, used in 15 sites across the Mac-side Companion server (RequestRouter / SessionManager / WebSocketServer / handlers). Non-Korean Mac users would see Korean banners and toasts during pairing failures or transfer errors. Replaced every literal with `NSLocalizedString(key, value:, comment:)` using English base values; new keys carry the `companion.error.*` prefix.
- **AI auto-classify suggestion reasons translate to all 15 locales.** `AIClassifier.describeReason` returned three hardcoded Korean strings ("N개의 유사한 파일이 존재", "관련 파일 N개 존재", "폴더명 유사도 기반 추천") that appeared in the classification preview next to every suggested destination. Wrapped each in `NSLocalizedString(value:)` with English source; new keys are `ai.classify.reason.similarExtension`, `ai.classify.reason.relatedFiles`, `ai.classify.reason.folderName`.
- **Permission-denied banner no longer relies on substring-matching the localized error message.** `PanelView` decided whether to show the "Open System Settings ▸ Privacy & Security" affordance by doing `err.lowercased().contains("permission") || err.contains("권한")`, which worked only in English- or Korean-localized installations. Ja/zh/de/fr/es users saw a generic red banner with no escape hatch. Added `PanelViewModel.lastErrorIsPermission` — a typed flag set in `loadCurrentDirectory`'s catch from `Error.isPermissionDeniedError`, which pattern-matches Cocoa `fileRead/WriteNoPermission`, POSIX `EACCES`/`EPERM`, and `UnifylFileError.permissionDenied` instead of the localized message. Reset alongside `lastError = nil` everywhere it's cleared (initial load, retry, archive enter/exit, manual dismiss).
- **Help window (F1) actually focuses in the 13 non-English / non-Korean locales.** `MainWindowView`'s `isHelpOpen` handler matched `NSApp.windows.first(where: { $0.title.contains("Help") || $0.title.contains("도움말") })`, which silently failed for the other 13 localized titles ("ヘルプ", "帮助", "Hilfe", "Aide", "Ayuda", "Помощь", and so on) — F1 did nothing if the window was already open in those locales. Replaced with `@Environment(\.openWindow) openWindow(id: "help")`, which SwiftUI handles as "focus existing or create" by scene id, locale-independent.
- **Forward Delete (⌦) key now moves selected files to the Trash, same as F8.** Previously only F8 and `⌘Delete` (Cmd+Backspace) triggered the move-to-Trash flow; the labelled "Delete" key on full-size keyboards (or `fn+delete` on laptops, keyCode 117) did nothing. Wired in both `FileTableView.handleKeyDown` (when the file table has focus) and `FunctionKeyMonitor` (when focus is elsewhere — pathbar / toolbar / etc.) so the affordance is consistent across the window. `Backspace` alone (no modifier) still navigates to the parent directory — that's the TC + Finder convention and stays put.
- **`FileTableView.userDefaultsDidChange` no longer SIGTRAPs when defaults are written off-main.** `UserDefaults.didChangeNotification` posts on whatever thread modified defaults (e.g. `XCTTelemetryLogger.+initialize` triggers `registerDefaults:` on a background dispatch queue during test bundle init), and Swift 6 strict concurrency traps when the @MainActor `applyColumnVisibility()` is called from there. Marked the selector `nonisolated` and hopped to MainActor via `Task`. Real-world impact: the test runner no longer crashes at bootstrap; production launch was already on main but is now race-proof for any future off-main defaults writer.
- **CSV Preview now splits rows on Windows-style `\r\n` line endings.** Swift's `String` iteration groups `\r\n` into a single extended grapheme cluster that matched neither `char == "\r"` nor `char == "\n"` in the char-by-char parser, so the line ending leaked into the field value and a Windows-saved CSV rendered as a single giant row. Replaced the manual check with `Character.isNewline`, which matches `\n` / `\r` / `\r\n` + Unicode line separators (`\u{0085}`, `\u{2028}`, `\u{2029}`) uniformly. The pre-existing `if char == "\r" { continue }` branch was dead code (never reached because Swift had already merged the CR+LF). Caught by the existing `handlesCRLF` unit test, which was previously failing.

### Added
- **CSV Preview: text-encoding auto-detect + large-file safety.** The previewer used to hardcode `String(contentsOf: url, encoding: .utf8)`, so a Korean Excel CSV (CP949 default) or any non-UTF-8 file threw or rendered mojibake, and a multi-hundred-MB CSV would block the main thread / OOM on whole-file load. Now: BOM-checks UTF-8 / UTF-16 LE / UTF-16 BE first, then falls through `.utf8 → CP949 (kCFStringEncodingDOSKorean) → Shift_JIS → EUC-KR → Big5 → GB18030 → Windows-1252 → Mac OS Roman` until one succeeds; caps the byte payload at 50 MB (drops the trailing partial row so half-fields don't render as data); caps the rendered table at 5000 rows; surfaces both the detected encoding and a yellow truncation banner ("Showing 5000 of 50000 rows" / "File exceeds 50 MB — preview shows the leading slice") in the footer. Resolves issue #7. Carries unit tests for UTF-8 BOM, plain UTF-8, CP949 fallback, ASCII passthrough, empty input, quoted CSV, CRLF, and TSV.
- **CSV parser drops the phantom blank row on a trailing `\r\n` / `\n`.** Pre-existing parser bug — a well-formed CSV that ended with a newline gained a `[""]` row at the end of the rendered table. Caught by the new CRLF unit test.
- **Hex Editor: in-place byte editing.** The Hex viewer was misnamed — its underlying `NSTextView` had `isEditable = false`, so the Save / Revert buttons were dead UI and any "edit" affordance was a lie. Now genuinely editable: `shouldChangeTextIn` delegate intercepts single-character keystrokes, decodes the cursor's character offset into either a hex high-nibble / low-nibble or an ASCII byte position (deriving from the per-row layout of `8-digit offset + 16 bytes × 3 chars + separator + 16 ASCII chars`), patches the byte in `Data` and the rendered `NSTextStorage` in place, and advances the cursor to the next nibble / byte. Hex columns accept `0-9 / A-F / a-f` only; ASCII column accepts printable bytes (`0x20–0x7E`) only; offset column and gap characters reject input. Paste, multi-character IME, and selection-replace are rejected to keep the grid layout stable. Save (⌘S) writes the edited bytes atomically to disk via `Data.write(to:options:.atomic)`. Status bar shows the hint "Click a hex digit or ASCII char and type to edit · ⌘S to save" on file load. Coordinator class is now `@MainActor`-isolated, which incidentally clears the 5 pre-existing Swift 6 strict-concurrency warnings on `updateContent` / `scrollToOffset` / `highlightRange` accessing `textStorage`.
- **`scripts/audit-guard.sh` + `make audit`** — a codified sweep that runs every audit-pattern this project has accumulated across multiple sessions: sheet-hosted `runModal()` (F1), hardcoded CJK inside `errorDescription` (F2), `MainActor.assumeIsolated` and new `ObservableObject` bans (F3), Localizable.strings parity across all 15 locales (F4 — FAIL on en/ko drift, WARN on any other locale's gap with a count + pointer to the CSV workflow), locale-fragile substring matching like `title.contains("도움말")` (F5), ghost build-flag references in `docs/` (F6), v1.0-era 4-tier pricing residue (F7), Sparkle `Info.plist` invariants (F8), public mirror URL hygiene (F9), missing `LocalizedError` conformance (F10), raw `throw NSError(domain:code:userInfo:)` calls (F11). The previous audit had marked the sheet-modal NSAlert family "done — 9 sites across 7 views" but missed 14 additional NSOpenPanel sites in the same family; the new script caught all of them mechanically. FAIL-class violations exit non-zero (block CI/PR); WARN-class print but don't block. Opt-in `// audit-guard:allow-runmodal` marker for legitimate fallbacks (e.g. NLCommandPreviewSheet's no-keyWindow defensive branch). Run via `make audit` or wire into a pre-commit / CI hook.
- **`docs/translations-pending.csv` + `scripts/apply-translations.py`** — translator-ready workflow for the 1807 untranslated strings (139 keys × 13 secondary locales). The CSV has columns `locale, key, english_source, translation`; a translator fills the `translation` cells, then `python3 scripts/apply-translations.py` appends each completed row into the right `*.lproj/Localizable.strings` (skips keys already present, idempotent on rerun, rewrites the CSV with only the rows still pending). Per memory `feedback_audit_one_shot` and the new F4 WARN — generating placeholders ourselves would have masked the gap from real translators, so the CSV-and-apply pattern surfaces work to do without papering over it.
- **All 1807 missing translations filled in across 13 secondary locales** (ja, zh-Hans, zh-Hant, de, fr, es, pt-BR, it, ru, tr, ar, th, vi). Translations were machine-generated (Claude) following per-locale conventions: keycap symbols (⌘ ⌥ ⌃ ⇧) and format specifiers (%@, %d, %1$@) preserved verbatim; literal `\\n` retained as the strings-format escape; macOS-native terminology where applicable (e.g. ja "ゴミ箱" for Trash, de "Papierkorb", zh-Hans "废纸篓"). Native speaker review is still warranted before a public press push, but the table is now the single source of truth (`scripts/_translation-table.py`) — each cell is reviewable and re-emitable. All 15 locales now hold the same 687 keys; F4 audit is fully PASS, the 1807-row WARN is gone. Total .strings file size went from 548 keys × 14 secondary locales (7672) to 687 × 14 (9618) entries.

### Changed
- **ArchiveEngine class body split into 4 method-group extension files (1092 → 173 + 4 extensions).** The `ArchiveEngine` class — list / extract / create / modify pipelines all sharing a small set of helpers — split as:
  * `ArchiveEngine.swift` (173 lines) keeps the class declaration, `init`, and the cross-group helpers `isSafeArchivePath` (path-injection guard), `find7z` (7z binary discovery), `run` (Process shell-out), plus the `ProcessRunner` nested actor. These three helpers were `private` and have been promoted to module-internal so the extension files can reach them.
  * `ArchiveEngine+List.swift` (277 lines) — `list()` plus the format-specific `listZip`/`listTar`/`list7z` and Korean-aware ZIP name decoding.
  * `ArchiveEngine+Extract.swift` (203 lines) — `extract` variants plus the `preflightOrThrow` (zip-bomb guard) and `verifyNoPathTraversal` helpers, both promoted from `private` to module-internal because the password-aware extract methods in +Create.swift need them too.
  * `ArchiveEngine+Create.swift` (236 lines) — `requiresPassword`, `probeZipEncrypted`/`probe7zEncrypted`, all `create*` formats (`createZip`/`createTarGz`/`createTar`/`createTarBz2`/`createTarXz`/`createTarZst`/`create7z` + dispatcher `create`), and the password-aware extract methods.
  * `ArchiveEngine+Modify.swift` (248 lines) — in-place editing of existing archives: `addToArchive`, `removeFromArchive`, `renameInArchive`, `createDirectoryInArchive`, and the `modifyTarArchive` round-trip helper.
- **FileTableView.swift NSTextFieldDelegate extension moved to `FileTableView+TextFieldDelegate.swift` (2352 → 2317 + 38 lines).** Just the inline-rename text-field delegate methods (`controlTextShouldEndEditing` / `cancelOperation`). Access promotions: `tableView` and `items` `private` → module-internal (the extension routes first-responder focus back to the table and reads the row being renamed by index). Small win as a low-risk first cross-file split test for the FileTableView extension surface; the bigger NSTableViewDataSource (266 lines) and NSTableViewDelegate (464 lines) blocks stay in the main file because their cross-section private-state touch surface is wide — splitting them needs a per-section manual NSTableView drag/drop smoke pass.
- **FileTableView.swift core auxiliary types extracted (2598 → 2352 + 2 new files, 249 lines).** Two file-private collaborators moved out: `FileTableViewDelegate.swift` (30 lines) — the `@MainActor public protocol` plus its `didDropURLs intoArchive` default extension; every adopter (`FileTableBridge`, `PanelView`, archive list views, the Smart Folder cart) builds against it. `UnifylTableView.swift` (219 lines) — the NSTableView subclass that intercepts `keyDown`, `mouseDown`, and the drag pipeline. Promoted from `private class` to module-internal so it can live across files. The memory note about `@MainActor` annotation silently breaking drag initiation (a regression last quarter) is preserved verbatim at the top of the new file — this class STAYS non-`@MainActor`, methods all run on main via AppKit's runloop, timer/Task closures hop to MainActor explicitly. Verified by `swift test UnifylAppKit` (6/6 pass).
- **FileTableView.swift auxiliary types extracted (2753 → 2598 + 3 new files, 167 lines).** Three standalone types lived in the same file as the 1500-line `FileTableView` NSView subclass: `FileTableContextAction` enum (raw-value-stable, persisted as user keybinding identifiers — moved to its own file with the doc note about the stability contract), `ThemedTableRowView` NSTableRowView subclass (renders alternating row backgrounds + selection stripes from theme tokens), and `AudioSpecCache` (sample-rate / bit-depth cache keyed by URL). All three are now in single-purpose sibling files. The big `FileTableView` class itself stays in one piece — NSView lifecycle + NSTableView drag/drop delegates share too much synchronous state to safely split into extension files in an autonomous pass; that work needs a manual NSTableView drag-smoke pass per split. 6/6 UnifylAppKit tests pass against the new layout.
- **AppViewModel.swift File Tags + Cloud Download bloc split into `AppViewModel+TagsAndCloud.swift` (441 → 266 + 191-line extension file).** macOS Finder tag manipulation (per-item apply/strip, AI-suggested tagging, batch ops) plus the cloud-only file download/progress watcher used for iCloud / Dropbox stub files.
- **AppViewModel.swift File Operations / Remote download / Drop / Compare bloc split into `AppViewModel+FileOps.swift` (768 → 441 + 344-line extension file).** Four MARK sections moved together (they share the same `fileOps`-delegating idiom): Forwarding (Conflict Resolution), File Operations (copy/move/delete/new-folder/batch dispatch), Remote download/upload (with `reportTransferFailures` aggregator — stays file-private to the extension), Drop handling, Compare. One file-private helper kept private. No new access promotions needed.
- **AppViewModel.swift TC-compat shortcuts section split into `AppViewModel+TCShortcuts.swift` (1810 → 768 + 451-line extension file, plus the prior Archive split).** Total-Commander parity actions: `swapPanels`, `copyCurrentPathToOppositePanel`, `revealCursorInOppositePanel`, `moveTab(from:at:)` (cross-panel), `computeAllDirectorySizes`, `goToVolumeRoot`, `createNewTextFileAndEdit`, `duplicateInSameDirectory`, `insertToggleAndAdvance`, `beginInlineRename`, the HWP conversion alert flow, and a clutch of file-action dispatchers that delegate to fileOps. Access promotion: `cloudDownloadTask` `private` → module-internal so the cloud-download dispatcher in the extension can swap its watcher Task.
- **AppViewModel.swift Archive operations bloc split into `AppViewModel+Archive.swift` (1810 → 1202 + 628-line extension file).** Four MARK sections moved: Archive operations (`extractArchive*`, `extractArchiveHere`, `extractArchiveWithPassword`), Compress/Extract (Cmd+F5 / Cmd+F9 sheet flow), Extract and Open from Archive, and Archive File Operations (in-archive copy/move/delete pipelines that keep `ArchivePanelView` working when browsing inside a ZIP or 7z). Four file-private helpers (extractArchive, archiveEntryPaths, extractAndRelocate, resolveExtractDestination) stay private to the extension file — their only callers are the methods in the same section, so no access promotion needed.
- **AppViewModel.swift Clipboard (Copy/Cut/Paste) section split into `AppViewModel+Clipboard.swift` (2135 → 1810 + 343-line extension file).** All clipboard methods (`copyFilesToClipboard`, `copyFilePathsToClipboard`, `copyPaths`, `cutFilesToClipboard`, `pasteFilesFromClipboard`, plus the related `showLaunchTipIfAppropriate`, `scheduleTagNotificationDismiss`, `insertFileNameIntoTerminal`). `isCutOperation` stored-property declaration stayed in the main file (Swift extensions cannot carry stored properties); promoted from `private` to module-internal, same for `tagNotificationTask`.
- **AppViewModel.swift Workspace Persistence section split into `AppViewModel+Workspace.swift` (2200 → 2135 + 78-line extension file).** `SavedTabSort` and `SavedWorkspace` Codable types plus `saveWorkspace()` / `loadWorkspace()`. Access promotion: `loadWorkspace` `private` → module-internal (AppViewModel.init reads workspace on launch from the main file).
- **PanelViewModel.swift TC shortcuts section split into `PanelViewModel+TCShortcuts.swift` (852 → 631 + 237-line extension file).** Total-Commander-style operations: `insertToggleSelectAndAdvance`, `computeAllVisibleDirectorySizes`, `enterBranchView` (recursive flat listing), `setFilter`, `sortItems`, the canonical static `sortItemsStatic` comparator, and `syncVisibleItems`. Access promotions: `sortItemsStatic` from `private` to module-internal (PanelViewModel.loadCurrentDirectory needs it off-main).
- **PanelViewModel.swift Navigation section split into `PanelViewModel+Navigation.swift` (1128 → 852 + 289-line extension file).** All 17 navigation functions (`navigate*`, `goUp/Back/Forward`, `restoreTab`, `selectTab`, `addTab`, `closeTab`/`closeOtherTabs`/`closeTabsToRight`, `duplicateTab`, `togglePinTab`, `setTabTitle`, `setSortOrder`, `spaceBarToggle`, `calculateDirectorySize`). Access promotions: `sortItems`, `syncCalculateDirectorySize` from `private` to module-internal.
- **PanelViewModel.swift Remote Connect/Disconnect section split into `PanelViewModel+Remote.swift` (1219 → 1128 + 112-line extension file).** `connect(to:)`, `disconnect()`, and the OAuth-cloud `makeOAuthFileSystem` helper. The protocol-specific VFS adapter selection (SFTP / FTP / WebDAV / SMB / S3 / Azure Blob / OAuth-cloud) now reads as one ~100-line file independent of the directory-loading / archive / navigation sections.
- **PanelViewModel.swift Archive browsing section split into `PanelViewModel+Archive.swift` (1503 → 1219 + 300-line extension file).** `enterArchive`, `exitArchive`, `navigateInArchive`, `goUpInArchive`, `refreshArchiveItems`, plus the `listArchiveEntries` ZIP-central-directory reader, all moved to the extension. Access promotions: `allArchiveEntries` and `archiveEngine` from `private` to module-internal; `watchTask` / `archiveExitTask` accessor pairs from `private` to module-internal so the extension can drive cancellation and exit deferral. `archiveOpenFailureURL` stored property stayed declared on the main class (Swift extensions can't carry stored properties). The view-model's archive-mode surface is now locally readable in its own ~300-line file.
- **`TaskBox` extracted out of PanelViewModel.swift (1534 → 1503 + 36-line TaskBox.swift).** The lock-protected Task box that owned `watchTask` / `debounceTask` / `archiveExitTask` was a file-private class within the view-model file. Hoisted to its own file `Unifyl/ViewModels/TaskBox.swift` and promoted from `private` to module-internal so PanelViewModel can hold it across files. Cleaner reading: the nonisolated-deinit cancellation contract is now locally inspectable without scrolling past 1500 lines of view-model code.

- **3 large source files split into feature-group files (1190+1160+1312 → 600+1092+1107 with 7 new files).** Long-form refactor work to lower the cost of touching any single hot area:
  * `Unifyl/Services/IconThemeManager.swift` (1190 → 600) — model + manager class kept; `IconPack` (28), `ExtensionSvgMapping` (181), `PopularTheme` (390) moved to siblings. Each new file has a single feature group (data registry / preset registry / SVG pack model).
  * `Packages/UnifylFileSystem/Sources/UnifylFileSystem/ArchiveEngine.swift` (1160 → 1092) — `ArchiveError` (19) and `ArchiveFormat` (55) extracted to siblings. The class body itself (list / extract / create / modify / 7z helpers) is still ~1092 lines because every section reaches into the same set of `private` helpers (`isSafeArchivePath`, `verifyNoPathTraversal`, `run(...)`, etc.); splitting those across files requires promoting the helpers from `private` to `internal`, deferred to a session where access scope changes can be reviewed.
  * `Unifyl/App/UnifylApp.swift` (1312 → 1107) — `ProBadgeMap` (208 lines) plus the associated `badgeLogger` and doc comment hoisted to `Unifyl/App/ProBadgeMap.swift`. The remaining content (the `App` scene composition, `.commands { }` menu blocks, settings + help windows) is where the file's bulk lives and stays put — `.commands` and `WindowGroup` blocks are body composition that resists clean extension-based splits.
  Tests `IconThemeRefactorTests.swift` (3 suites, 8 tests) added covering the IconPack directory contract, PopularTheme repo-path parsing edge cases, and `ExtensionSvgMapping.defaultMapping` UX-stable entries — the table is a UX contract (broken icon themes look obviously wrong), so a regression test for the known mappings catches accidental deletions during future cleanup.

### Changed
- **`en.lproj/Localizable.strings` synced with `ko.lproj` (548 → 687 keys).** Translator workflow drifted — Korean translations were added for 139 strings whose English source never made it into the English baseline `.strings` file. Users on every locale still saw correct text because `NSLocalizedString(key, value:)` falls back to the inline `value:` when the locale file lacks the key, but the missing baseline broke localizer tooling that pivots off `en.lproj`. Backfilled all 139 keys with `"key" = "key";` entries (plus the one true identifier-style key `companion.iosAppPending.notice` whose English value was copied from the call site). The other 13 locales still hold 548 keys — backfilling translations there is a content task, not a code one, and untranslated keys continue to render via the inline `value:` fallback.
- **EnterpriseTeamView CSV audit-log export switched to `panel.begin {}` — same `sheet+runModal()` family as the 14 other audit-driven sites above.** Caught by the new `make audit` script in the same pass that produced the script. The "Export Audit Log to CSV" button on the Enterprise → Audit Log tab was a dead button in production (sheet swallowed the panel).
- **SummaryTypes.SummarizationError errorDescription wrapped in `NSLocalizedString`.** 5 cases (`unsupportedFormat`, `noExtractableContent`, `modelUnavailable`, `modelLoadFailed`, `inputTooLarge`) had Korean string literals — same family bug as the CompanionError fix earlier in [Unreleased], caught by `make audit`'s F2 check. Keys carry `ai.summarize.error.*` prefix.
- **5 raw `throw NSError(domain:code:userInfo:)` sites typed as `LocalizedError` enums.** Each used `NSLocalizedDescriptionKey` with an English literal, so non-EN locale users saw English when the error reached the UI — same locale-fragility family as F5. Converted:
  * `PluginManager.swift` "Invalid plugin ID: …" → `PluginLoadError.invalidPluginID(String)` (added case to the existing enum).
  * `SFTP/FTP/WebDAVFileSystem.swift` "Refusing remote operation: path contains control characters …" → `VFSError.unsafeRemotePath(String)` (added case to the shared VFS error in `UnifylCore`; all three providers route through the same typed case).
  * `AudioMetadataView.swift` "Cannot create export session" / "Export failed" → new `AudioMetadataError { exportSessionUnavailable, exportFailed }`.
  All cases wrap their message with `NSLocalizedString(key, value:, comment:)`. Caught and locked in by new audit-guard check F11.
- **2 icon-only Buttons gained `.accessibilityLabel(...)`.** `BookmarkView` row trash button ("Remove bookmark") and `PathBarView` cancel-edit button ("Cancel path edit"). VoiceOver previously announced the bare SF Symbol name. Initial 310-vs-33 ratio of `Image(systemName:)` to `accessibilityLabel(` was misleading — almost every site is either inside a `Button` with a `Text` sibling (auto-derived label) or used as decoration on a labelled control; only these 2 were truly orphaned.
- **UnifylAppKit gains its first 6 test functions across 3 suites.** The package was at 0 tests despite housing the 2744-line `FileTableView` NSView bridge that drives every directory listing in the app — any change to the table logic shipped without a regression net. New `UCAppKitTests.swift` covers (1) `FileCategory.color` mapping completeness (every case has an explicit color; a new case added without one trips the test), (2) `FileTableContextAction` rawValue stability (its raw strings are persisted in user keybindings — silently renaming a case would orphan the user's saved customization), and (3) `AudioSpecCache` behaviour (non-audio extensions short-circuit before disk I/O; `clearCache()` actually resets state). NSView lifecycle and NSTableView drag pipelines remain manual-test only — they require a host AppKit run loop. 6/6 pass.

### Removed
- **`LicenseConfig.aiCheckoutURL`, `teamCheckoutURL`, `aiVariantId`, `teamVariantId` deleted.** All four constants pointed at the same single Pro variant (1,544,858) and the same single checkout URL — v1.0-era ghosts of a once-planned 4-tier (Free / Pro / AI / Team) pricing model that collapsed to 2 tiers (Free + Pro) before public release. Only `aiVariantId` was actually referenced (`LemonSqueezyClient.tier(fromVariantId:)`), folded back to `proVariantId`. The other three were dead code. Resurrect with real variant IDs only when a separate SKU actually exists in the LemonSqueezy store.
- **`docs/DEVELOPMENT.md` ghost build-flag references corrected.** The doc claimed feature gating used compile-time flags `UNIFYL_PRO` / `UNIFYL_AI` / `UNIFYL_TEAM` — none of those have ever existed in `project.yml` or any `Package.swift`. All gating is runtime via `FeatureGateManager`. The doc now reflects the single-binary, runtime-only reality so contributors don't go hunting for build-flag plumbing that isn't there.

## [1.2.2] — 2026-05-11

Multi-axis polish pass on top of 1.2.1, accumulated across multiple audit-driven sessions.

### Added
- **193 previously-hardcoded UI strings localized into all 14 non-English locales** (ja, ko, zh-Hans, zh-Hant, de, fr, es, pt-BR, it, ru, tr, ar, th, vi). Roughly half of the app's `Text(...)` literals (mostly inside viewer / tool sheets — Checksum, AI Auto-Classify, AI Search, App Uninstaller, Audio Converter, SSH Tunnel, Git History, File X-Ray, PDF Convert, Theme presets, etc.) had no entries in `Localizable.strings`, so non-English locale users saw English fallback text in those surfaces despite the rest of the app being native. Each locale now ships +193 entries (343 → 536 keys) with macOS-convention vocabulary per language.
- **Onboarding slide 4 ("Pick a folder to begin") now localized in all 15 languages.** The slide was added in 1.2.0 but its 5 NSLocalizedString keys (title, body, drag, goto, connect) were never added to `Localizable.strings`, so 14 non-English locale users saw the English `value:` fallback for the post-onboarding "now what?" guidance. Added native translations for ko / ja / zh-Hans / zh-Hant / de / fr / es / pt-BR / it / ru / tr / ar / th / vi.
- **`Error.humaneMessage` extension** that maps low-level Foundation / POSIX / NSURLError codes to localized user-facing strings. Falls through to `errorDescription` for `LocalizedError`-conforming types (`UnifylFileError`, `VectorIndexError`), and to `localizedDescription` as last resort.
- **Viewer NSPanel windows now remember their size and position** across launches. The Hex / Log / Git / EXIF / Compare / Dir Compare / Media / Docker / Git History / Process Map / Port Viewer / Theme Editor / 3-Way Merge / File Preview viewer windows used to open at the same default-centered + offset rect every time, even after the user resized or moved them. They now derive a per-viewer autosave key from the title prefix (e.g. "Hex: foo.bin" → key "Hex"), so AppKit persists the frame to defaults under `UnifylViewer.<type>` and restores it on the next open. The Main window already had this via `WindowFrameAutosave` since 1.1.0; this brings viewers to parity.

### Changed
- **All user-facing error banners now route through `error.humaneMessage`** instead of `error.localizedDescription`. ~112 user-facing error sites total: AppViewModel (26), FileOperationManager (10), AIEngineViewModel (7), PanelViewModel (5), ConnectionViewModel (1), TeamSettingsView (4), TeamOnboardingSheet (2), AuditLogView (2), plus 55 sites across 43 view-layer files (CompressDialog, FilePermissions, MediaInfo, AppUninstaller, FileDiff, DirectoryDiff, MediaConverter, SQLitePreview, JSONPreview, PlistPreview, FontPreview, VideoPreview, ArchivePreview, AudioPlayer, EnterpriseTeam, etc.). All produce localized messages for `EACCES` / `ENOSPC` / `ENOENT` / `EBUSY` / etc. instead of raw English `localizedDescription` from the system. OSLog / `UnifylLogger` calls are intentionally left on `localizedDescription` so logs stay machine-greppable in English.
- **Semantic color literals migrated to DesignTokens.** 93 sites across 52 view/feature files where `.foregroundStyle(.green / .red / .orange)` was hard-coding success / error / warning intent now go through `DesignTokens.Colors.success / .error / .warning`. The token registry already had matching values (`#34D399`, `#F87171`, `#F59E0B`) chosen for ≥4.5:1 contrast against the dark surface — this brings the rest of the app onto them so a designer can re-tune the status palette in one place. `.blue` and `.accentColor` left untouched (ambiguous between info-status and system-accent intent — needs case-by-case review).

### Fixed
- **CLDR plural rules via `Localizable.stringsdict`.** Five hardcoded English plural patterns (`count == 1 ? "" : "s"`) in AppViewModel (clipboard sources missing), AIRenameView ×2 (rename suggest / will-rename count), EnterpriseTeamView (member count), DuplicateFinderView (will-go-to-Trash count) used to render "1 file" / "5 files" in English but "5 파일s" / "5 ファイルs" in non-English locales. Generated `.stringsdict` for all 15 locales with proper plural categories (single-form for ja/ko/zh/th/vi, two-form one/other for en/de/fr/es/pt-BR/it/tr/vi, two-form for ru/ar — Arabic could be expanded to 6-form later if specific n=2/few/many phrasing matters), and switched the call sites to `String.localizedStringWithFormat(NSLocalizedString("ai.renameSuggest", …), count)`. Native macOS plural infrastructure now picks the right form per locale.
- **Hidden-file toggle (⌘⇧.) now persists across launches.** `AppViewModel.showHiddenFiles` was a plain `var` initialised to `false`, so every relaunch reset it — a developer who flipped it for the session had to re-toggle each launch. Added a `didSet` observer that mirrors the `splitRatio` / `bookmarks` persistence pattern (`@AppStorage` doesn't work on `@Observable` stored properties — init-accessor synthesis rejects it), backed by `unifyl.showHiddenFiles` in UserDefaults.
- **OAuth callback URL scheme now declared in `Info.plist`.** `OAuth2Configuration` ships `redirectURI = "com.unifyl.basic://oauth2/callback"` for Native Google Drive / Dropbox / OneDrive providers, but `CFBundleURLTypes` was missing — macOS had no app to route the callback to, so the auth flow would hang forever. Added a single URL-types entry registering `com.unifyl.basic`. (Native OAuth providers are still gated behind `unifyl.enableOAuthPilot` while their client IDs remain placeholders.)
- **Smart quote / dash / period / text-replacement substitutions disabled app-wide.** macOS NSTextField and NSTextView ship with these ON by default, which silently mangles file names typed into inline rename, ⇧⌘G Go to Folder, and the inline command bar (`"data".csv` becomes curly-quoted, `foo--bar.txt` becomes en-dashed, `omw to a folder` expands via System Settings text replacement). `applicationDidFinishLaunching` flips the four `NSAutomatic*Enabled` defaults to `false` for *this app's* text controls without touching system-wide settings.
- **Custom theme load no longer silently wipes the user's saved themes on schema drift.** `try?` was returning nil when the on-disk theme JSON failed to decode (typical after an upgrade adds a new color token), and the next `save()` overwrote the disk file with `[]`. Now stashes the corrupt file via `PersistenceBackup.stashCorruptFile` (recoverable for support) and logs to `UnifylLogger.settings`. Same pattern applied to the import-from-file path so a malformed `.ultratheme` file logs the parse error instead of vanishing.
- **A11y polish on tab bar + status bar.** TabBar's `xmark` close button gained `.accessibilityLabel("Close Tab")` (was announcing as a generic "button"); the `pin.fill` and `cloud.fill` chips on the tab title gained `.accessibilityHidden(true)` because they're decorative — VoiceOver was announcing "image: pin.fill" before every pinned tab's title. PanelStatusBar's optional toast icon now `.accessibilityHidden(true)` for the same reason — the toast text right next to it carries the message.
- **URLSession requests now cap their wait** so a hung server doesn't keep the request UI spinning forever. `TeamSyncClient.buildRequest` gets `timeoutInterval = 15 s` (matches `LemonSqueezyClient`); `IconThemeManager` per-asset SVG fetch caps at 30 s and the manifest JSON download at 60 s. Other URLSession sites already had explicit timeouts via the request builder pattern.
- **`applicationWillTerminate` now flushes UserDefaults before quit.** Quick Cmd+Q right after adding a bookmark could lose the entry on systems that hadn't auto-synced defaults yet; the new handler calls `UserDefaults.standard.synchronize()`. Doesn't block termination (no beachball) — file copies still rely on FileOperationManager's atomic-write + half-file cleanup.
- **18 Xcode build warnings cleared.** PortViewerView's `errData` was a captured `var Data()` mutated from `Pipe.readabilityHandler` callbacks while the outer `Task.detached` owned the closure scope — Swift 6 strict concurrency flagged it as a real data race; wrapped in a lock-protected `ErrDataBox` (`@unchecked Sendable`, `NSLock`-guarded). UnifylTests' `override func setUp() / tearDown()` were declared without `async throws` in `@MainActor` classes, breaking MainActor isolation; converted to async-throws variants with `try await super.<x>()`. Cloud HTTP calls (Dropbox/Google/OneDrive — `post`/`patch`/`delete` × 8) had unused `try await client.xxx(...)` results that look like silent fails to a reader; made the discard explicit with `_ = try await ...`. Plus 1 var→let cleanup and 3 unused-let → `_` cleanups in tests.

### Notes (audit-driven, ship-pending)
- **B3 — `Task.detached` `[weak self]` audit:** swept all 80+ `Task.detached` sites across `Unifyl/` and `Packages/`. **No real retain risk found.** Most closures use closure-local data plus `Self.<static>` calls. The few sites that touch `self` either already declare `[weak self]` or use intentional `[self]` (FileOperationManager:154 — atomic copy needs strong reference). ViewModels are `@MainActor final class` → sheet/window dismiss destroys them, so even captured-self closures resolve cleanly. No code change needed.

## [1.2.1] — 2026-05-10

Patch release closing two more "shipped UI, missing gate or stale copy" gaps caught during a deep audit five days after the 1.2.0 → build 19 respin.

### Fixed
- **`Ctrl+M` (Total Commander alias for Multi-Rename) now goes through the Pro feature gate.** The keyboard shortcut opened the Multi-Rename sheet directly without the `isUnlocked(.multiRename)` check the menu and Command Palette already had, so a Free user could trigger a Pro feature for free via the alias. Now request-upgrade-sheet on Free, open-the-sheet on Pro — same as the other entry points.
- **Help / User Manual FAQ no longer cites the dropped 4-tier pricing model.** A leftover sentence from the v1.0-era pricing — "Requires the AI tier ($59.99) or higher." — sat in every shipped manual's FAQ even after the build-17 sweep cleaned the Edition Comparison table. All 15 localized manuals now read "Included with Pro." in the user's language. Mirrors the rest of the now-2-tier pricing copy elsewhere in the app.

## [1.2.0] — 2026-05-01

> **Build 19 respin (2026-05-05):**
> 1. **Hides the three "Native OAuth" connection types** (`Google Drive (Native)`, `Dropbox (Native)`, `OneDrive (Native)`) from the New Connection picker. Each ships with a placeholder `clientID` (`YOUR_GOOGLE_CLIENT_ID` / `YOUR_DROPBOX_APP_KEY` / `YOUR_MICROSOFT_CLIENT_ID` in `OAuth2Configuration.swift`) — clicking "Authorize with Google/Dropbox/Microsoft" routed to a guaranteed-fail OAuth flow. The rclone-based variants (`Google Drive` / `Dropbox` / `OneDrive`, no "(Native)" suffix) stay visible — they work as long as the user has rclone configured locally. Per-machine opt-in for the OAuth pilot:
>    `defaults write com.unifyl.app unifyl.enableOAuthPilot -bool true`
> 2. **Settings ▸ Companion gets an "iOS app pending App Store" callout** above the server toggle, so users don't read "Enable Companion Server" + "0 paired devices" as a broken feature when in fact the Mac side is functional and waiting on the iOS Companion app to ship.
>
> **Build 18 respin (2026-05-05):** hides the Tools ▸ Team Management… menu and Command Palette entry behind a `unifyl.enableTeamPilot` opt-in default. Reason: the team-management UI shipped to the public Free + Pro channel but the `TeamSyncClient` backend doesn't, so clicking Send Invite silently no-op'd (the guard for `teamId` / `teamToken` returned with no user-facing message). Public channel now no longer surfaces the Team UI; private enterprise pilot users can still opt in with `defaults write com.unifyl.app unifyl.enableTeamPilot -bool true`.
>
> **Build 17 respin (2026-05-05):** also drops a v1.0-era 4-tier (FREE/PRO/AI/TEAM) Edition Comparison table from the in-app Help that no longer matches the shipping product (current model is just Free / Pro). 13 rows simplified to 2 columns; Team-only row removed; AI rows folded into Pro; pricing callout cleaned to "Free / Pro ($39.99 one-time)". Applied to all 15 localized manuals.
>
> **Build 16 respin (2026-05-05):** ships the in-app Help / User Manual updated to v1.2.0 (was still v1.0 in the original build 15 DMG) and now translated into all 15 shipping languages (was English + Korean only). Also fixes a Help locale bug where the manual always rendered in Korean regardless of system language. ShortVersionString unchanged; Sparkle picks up the build-number bump.

### Added
- **In-app Help / User Manual updated end-to-end for v1.2.0** (was previously v1.0 era — missing 1.1.0 / 1.2.0 features). 31 numbered sections (was 23), with new top-of-document "What's new in 1.2.0" callout linking to the most-changed surfaces. Existing sections (Function Keys, Tab Management, Search, Archives, Clipboard, Bookmarks, Tools, Themes & Settings) all updated for the current shipping behaviour. Three new sections added: **Status Bar** (cursor item info, Quick Look hint, "Did you know?" tips, free space readout), **Connections** (⌘K Connect to Server, sidebar persistence, click-anywhere rows), **Help & In-App Guidance** (toolbar `?` button, permission-denied "Open Settings" jump, onboarding slide 4).
- **In-app Help / User Manual now ships in all 15 languages** (was English + Korean only). Full translations for Japanese, Simplified Chinese, Traditional Chinese, German, French, Spanish, Brazilian Portuguese, Italian, Russian, Turkish, Arabic, Thai, Vietnamese — each ~2400 lines, native macOS terminology per locale. Korean manual updated with v1.1 / v1.2 changes section. Arabic uses `dir="rtl"` so the page renders right-to-left in WKWebView.
- **Tab middle-click closes (browser convention).** Pressing the trackpad middle / mouse-3 button on a tab closes it without forcing the user to hover for the X or hit ⌘W. Pinned tabs ignore middle-click so users can't accidentally lose a saved location. Implemented via a transparent `NSViewRepresentable` overlay that intercepts only `otherMouseDown` — left-click, right-click, and double-click pass through unchanged.
- **Cloud chip on remote tabs.** Tabs sitting on a remote VFS (SFTP / S3 / WebDAV / OAuth cloud) now show a small cyan `cloud.fill` icon next to the title so users see at a glance which tabs are local vs over the network — sets expectations for paste latency and "where do my files actually live?". Hover tooltip reads "Remote connection".
- **Connections sidebar open / closed state persists across launches** via `unifyl.showConnectionsSidebar` AppStorage. Was per-window `@State` that always reset to closed on relaunch — users who want it always-on had to re-toggle every session.
- **Status bar shows "Space preview" hint when cursor is on a file.** Right next to the cursor item info ("filename · 1.5 MB · date") an inline `Space` chip + "preview" label tells the user the Quick Look shortcut is one key away. Only shown for files (folders open with Return / double-click). Massively boosts Quick Look discoverability for users coming from non-Finder file managers.
- **One rotating "Did you know?" tip per launch** in the active panel's status bar, ~6 s with fade. Pool of 12 tips covers Return / F2 / Space / ⌘⇧P / ⌘⇧G / ⌘K / ⌥-drag / ⌃B / ⌃G / ⌃U / ⌘Z / F-keys — surfaces shortcuts a clean panel would never advertise. Cycles via stored `unifyl.launchTipIndex` counter so users see the whole tour over launches instead of the same tip twice. Skipped on the very first launch (onboarding covers basics) and honours an opt-out via `defaults write … unifyl.showLaunchTips -bool false`.
- **Status bar shows cursor item info in Finder style.** When the user has nothing marked but is sitting on a row, the bottom status bar reads "filename · 1.5 MB · Apr 15, 2026 14:30" instead of the generic "N items · X total". Folders show "—" for size (avoids expensive recursive sum). Hover the row for the full path. Selection (red marks) still wins — the existing "K selected · …" line takes precedence so marked-item totals stay visible.
- **Recent Folders dropdown in the toolbar** (clock-arrow icon next to Bookmarks). Same store the Go ▸ Recent Folders submenu uses, but surfaced one click away — so users see "I just visited these places" without opening the menubar. Empty-state tooltip explains the button will populate as they navigate; cleared by an inline "Clear Recent Folders" entry.
- **Filter field placeholder + tooltip teach the syntax.** Was: "Filter…" — gave no hint that glob and regex work. Now placeholder reads "Filter — try *.png or /regex/" and the hover tooltip spells it out: "Plain text matches anywhere in the filename. Use *.png-style wildcards for globs, or /pattern/ for regex." Both the toolbar field and the inline panel filter bar carry the same hint.
- **Permission-denied panel error offers a one-click "Open Settings" jump.** When the listing fails because of a Files-and-Folders / Full-Disk-Access denial, the inline red banner now includes an "Open Settings" button that takes the user straight to System Settings ▸ Privacy & Security ▸ Files and Folders. Previously they had to know which pane to find themselves. The banner also gained an explicit X dismiss button (matching macOS toast convention) instead of relying on tap-to-dismiss.
- **Status-bar Free space readout refreshes every 10 s** instead of only on first appear. Was a one-shot read on `.onAppear`, so after a large copy / move / delete the value was stale for the rest of the session — users couldn't trust it for "do I have room?" sanity checks. Now polls a cheap volume-resource read on a `Task` loop with cancellation, plus a "Free space on the start volume" tooltip explains which volume is being measured.
- **Clipboard ⌘C / ⌘X / ⌘V show confirmation toasts** so the user knows the action took. ⌘C → "Copied "file.txt" — paste with ⌘V" or "Copied 5 files — paste with ⌘V". ⌘X → "Cut N files — paste with ⌘V to move" (so the user knows paste will move, not copy). Successful ⌘V → "Pasted N files into "DestFolder"" / "Moved N files to "DestFolder"". Without these, the panel looked identical before/after each shortcut and users re-pressed unsure whether it took.
- **Onboarding slide 4 — "Pick a folder to begin"** answers the post-onboarding "now what?" moment with three concrete entry actions: drag from Finder, ⌘⇧G to type a path, ⌘K to connect to a server. The previous flow ended with a "Get Started" button on a generic command-palette slide; users dismissed onto two blank panels with no obvious next step.
- **Help button in the toolbar** (`?` icon, ⌘?). The Help menu has always existed but new users routinely missed it; a toolbar question-mark mirrors Finder / System Settings and answers "where do I go for help?" without hunting through menus.
- **Empty-folder state suggests next actions instead of just stating the obvious.** Was: "Empty Folder · This folder has no visible items." Now: "Nothing to show here · Drop files in to add them, or press F7 to make a new folder." When `Show Hidden Files` is off, the message also notes that hidden items might exist and offers a one-click `Show Hidden Files` action so the user doesn't have to hunt for the toolbar's eye icon.
- **Folder Bookmarks can be reordered, with discoverability cues.** Drag any row to reorder (entire row hit area, with a live drop-target highlight bar), or right-click for Move Up / Move Down / Remove context menu (keyboard / VoiceOver fallback). A grip icon fades in on row hover and the header subtitle shows "Drag rows to reorder · Right-click for more" whenever there's more than one bookmark — so the affordance is visible before the user has to discover it. New order persists via the existing `saveBookmarks()` path.
- **Go ▸ Connect to Server… (⌘K).** Standard ForkLift / Cyberduck / Finder convention. Previously the Connections sidebar's `+` button was the only entry point; users migrating from those apps were hunting for the menu item.
- **Sort By submenu now has keyboard shortcuts** (⌃⌥N / ⌃⌥S / ⌃⌥D / ⌃⌥K for Name / Size / Date / Kind). Sort is a top-5 file-manager operation and previously required mouse every time.
- **Newly bound shortcuts on previously menu-only commands**: File Permissions ⌥⌘I, File History ⌥⌘J, Theme Editor ⌥⇧⌘T, Download from Remote ⇧⌘↓, Upload to Remote ⇧⌘↑.
- **Menu labels surface F-key shortcuts that SwiftUI can't render in the trailing accelerator slot**: "Compress… (⌥F5)", "Extract to Folder… (⌥F9)", "Duplicate in Same Folder (⇧F5)", "New Text File + Edit (⇧F4)". The bindings already worked through `FunctionKeyMonitor`; the labels now make them discoverable.
- **Full UI translations for 13 additional languages** — Japanese, Simplified Chinese, Traditional Chinese, German, French, Spanish, Brazilian Portuguese, Italian, Russian, Turkish, Arabic, Thai, Vietnamese — each expanded from a 57-key starter set to the full 343-key surface that English and Korean already shipped. Brings previously English-fallback strings (Pro upgrade sheet, Settings panes, Audit Log, Team Management, AI tools, onboarding, error messages, About view) into native localisation across the full 15-language matrix. Format specifiers (`%@`, `%d`), keyboard markers (⌘B, F5, ⌥⌘O…), em-dash, ellipsis, and middle-dot punctuation preserved verbatim per locale.

### Changed
- **Destructive confirmation dialogs use a consistent "name what + offer the safer alternative" pattern.**
  - **Permanent delete (⇧F8)**: was "Permanently delete N item(s)? · This bypasses the Trash and cannot be undone." Now: "Permanently delete "filename"?" / "Permanently delete N items?" with body "These won't go to the Trash, so there's no way to get them back. Use F8 instead to move them to the Trash safely." Confirm button label sharpened to "Delete Permanently" (vs the macOS-default ambiguous "Delete").
  - **Port Viewer kill process**: was "Kill Process · This will terminate the process listening on port X. This action cannot be undone." Now: "Quit "ProcessName" (PID X)?" with body that names the SIGKILL signal AND warns about unsaved work — the consequence the user actually cares about. Button "Force Quit" (macOS convention) replaces "Kill Process".
  - **Duplicate Finder bulk delete**: gained the same "you can restore from the Trash if you change your mind" reassurance as the panel-level Move-to-Trash dialog.
- **Pro upgrade sheet explains what the locked feature actually does, not just its name.** Added `Feature.shortDescription` covering 30+ Pro features ("Multi-Rename — Rename hundreds of files at once with regex, EXIF, AI suggestions, numbering — and undo any batch with ⌘Z."). The headline still names the trigger ("Multi-Rename is part of Unifyl Pro") but the prominent body text now answers "what does this thing do?" before the generic "Pro also unlocks…" subhead. Description omitted for self-explanatory items (e.g. "Bookmarks").
- **Trial-active chip on the upgrade sheet.** When the user is in their 14-day free trial and the sheet appears (e.g. after trial expired between sheet open and now), an orange hourglass capsule reads "Free trial active · N days remaining" so they know where they stand instead of wondering whether they're already Pro.
- **Toolbar Back / Forward / Up disable when there's nowhere to go, with a tooltip explaining why.** Previously these always appeared enabled even at the volume root or with empty history — clicking did nothing and the user couldn't tell if the app was broken. Now the buttons grey out and the hover tooltip says "Nothing to go back to yet — open a folder first." / "Nothing to go forward to. Press Back first to enable Forward." / "Already at the volume root — there's no parent folder."
- **Toast / status messages name the file and the action result.** "Copied 5 paths" → "Copied 5 paths to the clipboard". "Tags removed: file.txt" → "Cleared all tags from "file.txt"". Single-item versions name the file directly so the user confirms they hit the right one.
- **Upgrade prompt names the feature that triggered it.** The Pro upgrade sheet was a generic "Upgrade to Unifyl Pro" pitch regardless of which click landed the user there — they were left wondering which feature was even locked. `requestUpgrade(for:)` now captures the `Feature` and the sheet headline reads "Multi-Rename is part of Unifyl Pro" / "AI Semantic Search is part of Unifyl Pro" / etc., with a unified subhead listing what else Pro includes. Generic fallback retained for entry points that don't pass a feature.
- **Move-to-Trash confirmation reads as a question, not an interrogation.** Was: "Are you sure you want to delete the selected items?" (red, ominous). Now: "Move "filename" to the Trash? You can restore it from the Trash if you change your mind." (or "Move N items to the Trash? …" for batches). The recoverability hint reduces the anxiety on a fully-undoable action.
- **Go to Folder error distinguishes typo vs. wrong-type-of-target.** "Path not found: /foo" became either "Couldn't find "/foo". Check the path for typos, or grant access in System Settings ▸ Privacy & Security if it's a protected folder." OR "/foo exists but isn't a folder. Go to Folder needs a directory path." — depending on which check failed.
- **Archive→archive copy error now explains the workaround.** Was: "Cannot copy between archives". Now: "Copying directly from one archive into another isn't supported. Extract the files first, then copy them into the destination archive."
- **Settings ▸ Timing labels rewritten in plain language.** "Spring-loaded folder delay" → "Folder hover-to-open delay", "Type-ahead buffer reset" → "Type-to-search reset", "Toast auto-dismiss" → "Status message duration". Tooltips also rewritten as full natural-language sentences ("When you drag files over a folder, how long do you need to pause before it opens?") rather than implementation-flavored shorthand.

### Fixed
- **Help (⌘?) now respects the system language instead of always rendering Korean** *(build 16 respin)*. `Bundle.main.url(forResource: "manual", withExtension: "html")` was finding an un-localized `Resources/manual.html` (a stale 52 KB Korean stub) at the bundle root before any lproj resolution kicked in — so the WKWebView in `HelpView` always loaded the Korean copy regardless of `preferredLocalizations`. Removed the un-localized base file; Foundation now resolves to the right `<locale>.lproj/manual.html` per the user's system locale, with English as the development-region fallback for any other locale.
- **Folder move appearing "to do nothing" after a stuck drag flag.** The drag-defer guard added to keep cloud/folder-size/git monitors from aborting an in-progress drag could get stuck `true` if AppKit's `endedAt` callback was missed (rare but observed: drag aborted by app switch, OS interrupt, drop on a sandboxed-rejecting target). All subsequent `updateItems` were deferred forever, so panel listings never refreshed after the next move — the move ran to disk but the source listing kept showing the moved folder, looking like the move silently no-op'd. Added a 10 s watchdog `Task` that force-clears `dragInProgress` and applies any pending items snapshot, so the panel always recovers within 10 s of a missed callback. The willBegin path arms a fresh watchdog and the endedAt path cancels it immediately.
- **File drag in CloudStorage / iCloud / OneDrive folders no longer "snaps back" mid-drag.** The cloud-status monitor was unconditionally bumping each item's `modificationDate` by 1 ms every poll to "force a redraw" — but that 1 ms tick made the bridge's `onlyMinorChanged` diff fail, triggering a full `tableView.reloadData()`, which silently aborts any in-progress NSTableView drag session. `refreshCloudStatus` now short-circuits when the cloud status hasn't actually changed, so status-stable polls don't touch `items` and the drag survives.
- **`FileTableView` defers `updateItems` while an NSTableView drag session is active.** Background updates from the file watcher, folder-size calc, or git status that landed mid-drag could still abort the drag via `tableView.reloadData()`. The view now sets a `dragInProgress` flag in `draggingSession willBeginAt` and stashes any items snapshot that arrives during the drag. The deferred apply is dispatched async on the next runloop tick from `draggingSession endedAt` (synchronous reload there left the table in a transient state that made the second consecutive drag fail to initiate). Each drag also clears any stale pending from a previous session in `willBeginAt` so the slot belongs to the current drag.
- **Bookmark row reorder works on the row content, not just the separator.** SwiftUI's `List` + `.onMove` only accepts drag from the row separator gutter on macOS — dragging the title text was a no-op, and the user was instead dragging an invisible click that closed the sheet. Replaced the `List` with a `ScrollView` + `VStack` and per-row `.draggable(url)` + `.dropDestination(for: URL.self)` so the entire row hit-area initiates the reorder, with a live drop-target highlight bar on the target row. Tap (single-click) still navigates and dismisses; right-click goes only to `contextMenu`.
- **Status bar item count now matches the size total.** "X items" included the synthetic `..` parent-directory row but the size sum excluded it, so the bar said e.g. "42 items, 1.5 GB" when the real listing was 41 files. The mismatch quietly broke any duplicate / backup workflow that compared counts across folders. Both numbers now come from a single cached pass that excludes `..`, populated alongside `cachedTotalFileSize`.
- **Delete now refreshes the inactive panel too.** Move and copy already reloaded both sides, but `deleteItems` only called `reloadActive` — so when both panels pointed at the same directory (common when staging a clean-up) the inactive side kept showing the just-deleted files until the user manually refreshed. `deleteItems` now takes a `reloadInactive` callback (defaulting to a no-op for back-compat) and `AppViewModel` wires both sides through.
- **⌘F now toggles the toolbar filter** — pressing it again clears the filter text and returns focus to the active panel. Previously the second ⌘F was a no-op (the field re-focused), so users either had to hunt for Esc (which only worked from some focus paths) or click elsewhere to dismiss. Browser / Finder convention restored.
- **New Folder (⌘N) and New Text File (⇧F4) are now undoable via ⌘Z.** Both operations advertised undo support per `CLAUDE.md` ("파일 작업 Undo / Redo") but were silently absent from the history stack — fat-fingered creation left orphan empty folders / `Untitled.txt` files the user had to hunt down and delete manually. Both now register a `.copy`-shaped record with `actualDestinations: [newURL]` so the existing copy-undo path (`removeItem(at:)`) cleans them up. Schema is unchanged; no new `OpType` needed.
- **Pasting from the system clipboard skips deleted sources cleanly instead of bailing on the first miss.** When the user copied N files in Finder and then deleted one, Cmd+V into Unifyl threw on the missing file, hit the `break`, and silently dropped the remaining items too — error banner just said "Paste failed: <Cocoa error code>" with no clue what was wrong. The paste loop now pre-partitions the clipboard URLs by `fileExists`, surfaces a humane message naming the first three missing files (and a `+N more` overflow), and proceeds with the survivors.
- **Corrupt `unifyl_file_history` JSON no longer silently wipes the user's full Recent-Files list.** `loadFileHistory()` decoded with `try?`, so a single schema-drift / partial-write byte killed every entry without any signal — the user discovered the loss days later. The decode now goes through a real catch path that stashes the corrupt blob via `PersistenceBackup.stashCorruptUserDefaultsBlob` (recoverable for support) and emits an OSLog error so the loss is at least visible in Console. Mirrors the recovery pattern already used by `ConnectionViewModel`.
- **Switching theme presets via the toolbar / View menu now invalidates the file-table row cache reliably.** Only `applyCustomTheme()` bumped `themeRevision`, but preset selection just assigned `selectedThemeId` directly — `FileTableView`'s `colorVersion` (which folds in `themeRevision`) didn't change, so cell colours could lag a frame, and entirely lag if the new preset's hex tokens happened to byte-match the previous one. Bump now lives on `selectedThemeId`'s `didSet`.
- **FTP transfers no longer hang forever when the server stalls mid-stream.** `curlBaseArgs` only passed `--connect-timeout 15` (handshake bound), so a server that opened TCP then stopped sending data left the operation hung indefinitely — progress bar froze and the user had to force-quit. Now also passes `--max-time 300`, matching the WebDAV path.
- **Anonymous FTP via the live `FTPFileSystem` (saved connections) now actually connects.** The same rclone-vs-curl `--anonymous-ftp` flag bug we fixed in the Test Connection sheet was duplicated in the live transport. Switched to the conventional `-u anonymous:anonymous@unifyl.local` form so saved anonymous FTP connections work end-to-end.
- **Failed copy no longer leaves a half-written destination file on disk.** When `copyItemAtomically` aborted mid-write (most commonly disk-full on the target volume), the partial file stayed on the destination — wasting space and leaving the user unsure whether the copy actually completed. The catch path now best-effort removes `finalDest` when both source and partial destination exist, so the destination folder ends up in its pre-copy state on failure.
- **`cd ~/Documents` (and any tilde-prefixed path) now works in the inline command bar.** Only the bare `~` token was recognised; tilde-prefixed paths silently beeped because `currentPath.appendingPathComponent("~/Documents")` produces nonsense. Now expands via `NSString.expandingTildeInPath` and emits a beep on a real lookup miss so the user knows their path didn't resolve.
- **Go to Folder dialog (⇧⌘G) now accepts paths copied from Terminal or Finder.** A `file://` prefix is stripped (so paths from "Copy as Pathname" and browser address bars paste cleanly), and `\ ` → ` ` is unescaped (so paths copied from Terminal that contain spaces don't need manual scrubbing).
- **Status-bar total size no longer recomputes O(N) on every SwiftUI body rebuild.** `totalFileSize` was a computed property running `.lazy.filter { $0.name != ".." }.compactMap(\.size).reduce(0, +)` on every render — selection changes, scroll, theme switches, font tweaks, focus flips all triggered a full sum across the listing. On a 100K-file folder it was the dominant per-frame cost. Now cached in `@State`, recomputed only on `items.count` change. Also collapsed the duplicate inline computation in the Smart Folder path bar to use the same cache.
- **Filter that hides every item now shows "No Matches" with a Clear Filter button instead of "Empty Folder".** Previously identical to a genuinely empty directory — users blamed the folder when really their filter text was just too narrow, and the only escape was knowing to press Esc or hunt for the filter field. Now the empty state distinguishes the two cases, names the filter that's hiding things, and offers a one-click clear.
- **Cancelling a running file operation from the queue bar now actually stops the work.** The queue bar's Cancel button only routed to `FileOperationQueue.cancel(handle)` (which flips the queue state), but the running copy/move/delete loop checks `operationProgress.isCancelled` — a separate signal that was never set. Result: cancel looked applied (status flipped) yet the op ran to completion. Especially broken after a Pause + Cancel sequence: pause masked the running state so cancel felt totally inert. Cancel now also signals `operationProgress.cancel()` for running/paused ops so the loop bails on its next iteration.
- **Connections sidebar rows are now click-anywhere and keyboard-activatable.** Each row had a tiny "Connect" / "Open" pill on the right that was the only hit target — keyboard users had no path to it at all, and mouse users had to aim for a 60-pt button instead of the obvious row. The whole row is now a `Button(.plain)` so clicking anywhere connects/opens, and `Return` activates the row when the sidebar list has keyboard focus. A right-chevron replaces the pill so the row reads as a navigable list item, and VoiceOver gets a proper `.accessibilityLabel` + `.accessibilityHint`.
- **Toolbar buttons now have proper VoiceOver labels.** Back / Forward / Up / Open in Opposite Panel / Home / Bookmarks / Connections Sidebar / Toggle Hidden Files / Increase + Decrease Font Size were icon-only with `.help(...)` tooltips, which VoiceOver doesn't read — every navigation button announced as a generic "button". Each now has an explicit `.accessibilityLabel(…)`. The font-size readout also announces the current point size, and the hidden-files toggle's label flips between "Show Hidden Files" / "Hide Hidden Files" matching the visible state.
- **`EmptyStateView` icon is hidden from VoiceOver.** The decorative SF Symbol (folder, search-glass, etc.) was being announced as a separate "Image: …" utterance before the title + message pair, so blind users heard a redundant icon name on every empty state. Marked `.accessibilityHidden(true)` so the title + message read as one statement.
- **Korean translations added for Git Panel, AI Classify, Checksum, and About surfaces.** Buttons like "Stage All" / "Start Analysis" / "Compute" / "Verify" / "Apply Tags" / "Check for Updates" / section headers like "Changed Files" / "Staged" / "Unstaged" / "Untracked" were appearing in English even when the system language was Korean — they had no entries in `ko.lproj/Localizable.strings`. SwiftUI's `Text("…")` autoresolver finds the keys now.
- **AI Classify empty state no longer renders broken plurals in non-English locales.** "Analyse \(items.count) file\(items.count == 1 ? "" : "s")…" hard-coded English's `+s` plural rule, producing nonsense like "파일s" in Korean. Now branches between two NSLocalizedString templates (`Analyse 1 file …` vs `Analyse %d files …`) so each locale supplies a grammatically correct form.
- **Settings ▸ Keyboard Shortcuts ▸ Reset All / Reset Category now confirm before destroying customisations.** Previously a fat-finger on either button instantly wiped every customised shortcut with no undo path. Both routes now go through a destructive-action alert with Cancel as the default; the per-category reset opens with the category name in the title so the scope is unambiguous.
- **Removed a stale `@AppStorage("unifyl.panelFontSize")` declaration in PanelView** that read from a key nobody writes (the live font-size key is `panelFontSize`, owned by AppViewModel). The property was unused but its presence tempted future writes that would silently desync from the actual setting.
- **Cancel button on the file-operation progress dialog now reflects the press immediately.** The button called `progress.cancel()` (which set `isCancelled = true`) but the dialog header kept showing "Copying…" with the active progress bar — so users re-clicked Cancel or thought the dialog had frozen. Now the title flips to "Cancelling…" with a warning-coloured xmark icon and the button itself goes disabled with a "Cancelling…" label until the underlying op winds down.
- **Overall progress bar can no longer jump backward between files.** When a small file finished between two large ones, the per-file fraction (`completed / total`) could transiently shrink because the size estimate revised — visible as the bar reversing, which read as "the operation is undoing itself." Progress is now monotonic via `updateOverallProgress(_:)`, which clamps to never decrease during the running phase (`begin()` / `reset()` still allow legitimate reset to 0).
- **Loading spinner no longer flashes on fast directory loads.** `ProgressView` appeared the moment `isLoading` flipped true, and on cached / small-folder loads the flag was true for <100ms — visible as a single-frame flicker that read as a glitch rather than a load. Spinner appearance is now debounced 250ms so fast loads complete invisibly while slow ones still surface a spinner with a smooth fade-in.
- **Spring-loaded folder no longer auto-opens the wrong folder when the listing mutates mid-drag.** The dwell-timer captured only the row index; if the file watcher reordered rows during the hover (e.g. a sibling file was added), the timer fired on whatever item now occupied that index. Now also captures the folder's URL and verifies identity before firing — Finder parity.
- **`UnifylFileError` and `VectorIndexError` now conform to `LocalizedError`.** Failures previously surfaced as raw enum dumps like `"The operation couldn't be completed. (UnifylCore.UnifylFileError error 4.)"`; users now see humane Korean/English messages per case (`Permission denied: <path>`, `Destination already exists: …`, `Could not open the semantic index database.`, etc.).
- **Right-click on a file now moves the cursor to the clicked row.** `rightMouseUp` toggled the mark but left the NSTableView cursor on the previous row, so the user saw "I right-clicked file X but the cursor is still on file Y". Now mark + cursor move together (Total Commander parity).
- **Selection survives switching to another window and back.** `FileItem.id` is a fresh UUID on every reload, so the `didBecomeActive` reload was wiping the selection set's UUID matches. `loadCurrentDirectory` now remaps the previous selection by URL across the listing swap.
- **Cloud Mount entries from the sidebar work after a remote panel session.** Clicking a Cloud Drive (e.g. OneDrive sync folder) on a panel previously bound to a remote VFS (SFTP / OAuth) used to call `RCloneFileSystem.contents()` against a local file URL and return empty + `partialFailure`. `navigateAndLoad` now swaps the panel back to `LocalFileSystem` when the destination is a `file://` URL and the current FS is remote.
- **Stale error banner clears on each directory reload.** `lastError` was set on failure but never cleared on success — a one-time wrong-VFS / network blip left the red banner visible across every subsequent navigation. Now reset at every `loadCurrentDirectory` entry, every `enterArchive` entry, and on `exitArchive`.
- **`Test Connection` for FTP no longer fails with `curl: option --anonymous-ftp: is unknown`.** That flag was an rclone option pasted into a curl invocation. Anonymous FTP now passes `--user anonymous:anonymous@unifyl.local`, the convention curl actually accepts.
- **`goBack` / `goForward` preserve cursor position on the dominant parent-folder case.** `goUp` already saved the leaving folder name into `pendingCursorName`; the history-stepping equivalents were unconditionally jumping to row 0. They now capture `currentPath.lastPathComponent` so the cursor lands on the entry the user was just inside (silently falls back to row 0 on lateral hops where the lookup misses).
- **Inline rename commit returns keyboard focus to the file table.** Only the Esc cancel path called `makeFirstResponder(tableView)` — Return left focus inside the now-non-editable text field, silently breaking j/k/arrow nav until the user clicked the table.
- **⌃⇥ tab cycling lands keyboard focus back on the file table even if a filter field had it.** The bridge claimed focus only when the panel's `isActive` flipped false→true, which doesn't happen on within-panel tab switches. Now also forces focus when `activeTabId` changes within the same active panel.
- **Closing a tab clamps `cursorIndex` to the surviving tab's item count.** Closing a tab with a high cursor index when the new active tab had fewer items left arrow keys silently no-op'ing until the user clicked a row.
- **Archive `enterArchive` failure no longer leaks the previous attempt's banner.** `lastError` is cleared at archive-enter entry and at `exitArchive`, and `archiveOpenFailureURL` is cleared on Esc-out.


## [1.1.0] — 2026-04-30

### Added
- **File List Columns are now configurable.** Six additional optional columns join the existing Size / Date Modified / Kind: **Date Created**, **Extension**, **Permissions** (`rwxr-xr-x`), **Tags**, **Path**, and **Cloud status**. Toggle each in Settings ▸ General ▸ File List Columns. The Name column is always shown; Smart Folder mode still force-shows Path regardless of the toggle. Visibility flips immediately on toggle (no relaunch) via a `UserDefaults.didChangeNotification` observer in `FileTableView`.
- **Archive entry timestamps preserved on extract.** ZIP central-directory DOS times and 7z `Modified =` fields are now decoded in the local timezone and restored onto the extracted file via `setAttributes([.modificationDate:])` after the move. Fixes the "extracted file is N hours off" symptom on machines whose local timezone differs from UTC (e.g. a 9-hour drift on KST). The DOS-time decoder also drives the panel-side ZIP listing so the inline archive view shows the same wall-clock time the file had on the archive creator's machine.
- **Overwrite confirmation when extracting archives or copying across panels.** Extraction and cross-panel archive copy now route through the same Skip / Rename / Overwrite dialog used by regular file copies, including the "Apply to all remaining conflicts" toggle (now defaulting **on** so multi-file batches don't fan out into dozens of identical prompts). The dialog supports arrow-key navigation between buttons via a local `NSEvent` monitor, and respects the existing `unifyl.confirmOverwrite` user default.
- **Cmd+A selects all in the active file panel** regardless of focus context. Previously the menu route went through `NSTableView.selectAll(_:)` which jumped the cursor to the bottom row without marking anything; the new global key intercept and an override on the inner table view both funnel into Unifyl's red-mark selection. Text fields and `NSTextView` keep their own select-all behaviour.

### Changed
- Existing Size / Date Modified / Kind columns are now also toggleable from the same Settings section (still default on).
- **Multi-file drag image** now shows a single large file icon with a red count badge (Finder-style) instead of a stacked-rows snapshot. The icon-plus-badge composite makes it unmistakable how many items are coming with the drag.
- **F7 New Folder positions the cursor on the just-created folder.** Matches Total Commander / Finder behaviour: after the reload settles, the new folder is selected so Enter / F2 / Cmd+Down act on it without an extra search step. Previously the cursor stayed wherever it was before, which on large folders meant the user had to hunt for the new entry.

### 10-round upgrade sweep (functionality / standards / a11y / perf / design)
- **Spreadsheet preview reads use memory-mapped I/O.** `OfficePreviewView`'s shared-strings, workbook.xml, and per-sheet XML reads all switched to `Data(contentsOf:options: .mappedIfSafe)`. Memory footprint of opening a large XLSX in preview is now bounded by the OS's page-mapping policy instead of by the file size — a 50 MB workbook drops from ~50 MB resident to ~5 MB.
- **Hex Editor file load is off-actor.** `HexEditorView.loadFile()` was reading via `Data(contentsOf: fileURL)` synchronously on the @MainActor — a 10 MB read from a slow network volume blocked the entire main thread. Read now happens in a `Task.detached(.userInitiated)` with `.mappedIfSafe`, and the result is written back through a `Result` on the main actor.
- **Panel error banner gets icon + semantic color + a11y label.** `panelVM.lastError` was a flat red bar — color-blind users had no other signal it was an error. Now it's an `exclamationmark.triangle.fill` + text in `DesignTokens.Colors.error`, with `accessibilityLabel` so VoiceOver announces "Panel error: …" rather than reading just the truncated text.
- **Connection-test result + Git-history error use semantic tokens.** `NewConnectionSheet`'s success / failure state chips and `GitHistoryView`'s error icon migrated from raw `.green` / `.red` / `.orange` to `Colors.success` / `Colors.error` / `Colors.warning`. The Git error icon also moved from `exclamationmark.triangle` to the filled variant for higher visual weight.
- **Eight more empty states unified onto `EmptyStateView`.** `JSONPreviewView`, `PlistPreviewView`, `FontPreviewView`, `ScenePreviewView`, `SQLitePreviewView`, `AppUninstallerView` (×2), `KeyBindingSettingsView`, `ThreeWayMergeView` all migrated from `ContentUnavailableView` to the project-style `EmptyStateView`. Error-state previews pass `iconColor: Colors.error` so the icon hue matches the message intent.
- **Sheet padding tokenised (final batch).** `ChecksumView`, `FileOperationConfirmView`, `FilePermissionsView` migrated from `.padding(20)` / `.padding(16)` to `DesignTokens.Spacing.l`. Combined with earlier rounds' NewFolderSheet / ArchivePasswordSheet migration, the project's modal padding is now phase-locked to one source of truth.

### Design
- **Design tokens reviewed and extended (designer pass).** Seven gaps closed in `DesignTokens.swift`:
  - **Active vs inactive panel selection.** New `bgSelectedActive` (accent-tinted) for the focused panel; existing `bgSelected` becomes the inactive-panel value. Total Commander / ForkLift parity — at a glance you now see which panel takes the next keystroke.
  - **Semantic color tokens.** `success` (#34D399), `warning` (#F59E0B), `error` (#F87171), `info` (#60A5FA) replace 30+ raw `.green` / `.orange` / `.red` literals across views. Code reading "this label uses `Colors.success`" tells you WHY the label is green; designers can re-tune the status palette in one file.
  - **Shadow scale** (`Shadow.card` / `Shadow.popover` / `Shadow.modal`) + a `.appShadow(_:)` SwiftUI modifier. Replaces magic-number `.shadow(color: .black.opacity(0.6), radius: 24, ...)` calls that drifted across 12 files. Applied to FileOperationProgressView, CommandPaletteView, AboutView (more sites can adopt incrementally).
  - **Text 4-tier scale.** New `textTertiary` between `textSecondary` and `textMuted` so column headers and count badges no longer collapse into the same value as disabled labels — restores visual hierarchy at typical viewing distance.
  - **`borderSubtle` strengthened** (`#1E2235` → `#232742`). The previous value was only ~2% lighter than `bgSurface`, so divider lines vanished at typical viewing distance. New value still reads as "subtle" but actually separates sections.
  - **`focusRing` token** (accent at 70% opacity) for keyboard-focused controls — AppKit's default focus ring uses the system accent, which clashed with custom Unifyl themes.
  - **Selection contrast bumped.** `bgSelected` → `bgSelectedActive` for the focused panel pulls ~30% more accent saturation; the "this row is selected" cue is now unambiguous against the hover background.

### Performance
- **`GitStatusProvider` cache no longer grows unbounded.** The 5-second TTL prevented stale data from being *served*, but expired entries lingered forever — a long browsing session that hit hundreds of git directories accumulated a `[String: CacheEntry]` of all of them indefinitely. Eviction now runs on every cache miss: stale entries (older than 5× TTL) are dropped, and a hard cap of 256 entries is enforced LRU-style on top. The fast path (warm cache hit) is unchanged; cleanup is amortised against the work the caller was about to do anyway.
- **`make build-fast`: parallel package build.** New target runs all nine SwiftPM packages in parallel via `&` + `wait`. On the dev machine: 8.0 s sequential → 0.5 s parallel for a no-op rebuild (16×), with ~3–4 s expected on a cold build. The original `make build` stays sequential for clean log output when something fails.
- **`make doctor`: pre-flight environment check.** Verifies xcodegen / swift / xcodebuild / swiftlint / gh / dmgbuild / create-dmg / bundled `7zz` / notarytool keychain profile in one shot, with brew/pip install hints for each missing piece. Avoids the late, ugly failures from `make release-publish` discovering a missing tool partway through the pipeline.
- **Directory listing: skip cloud-status `lstat` outside iCloud-rooted paths.** The post-listing pass that flagged cloud-only files via `lstat` ran for every non-directory entry in every directory the user navigated into. On a 100K-file folder that's 100K syscalls; outside `~/Library/Mobile Documents/`, `~/iCloud Drive/`, `~/Library/CloudStorage/` (Ventura+ OneDrive/Dropbox/Box/GDrive sync mounts), and `~/Dropbox/` the result is always "not cloud-only", so the entire loop is now gated on a single string-prefix check. Listing time on large local directories drops to whatever `contentsOfDirectory` itself costs.
- **Folder size flush: O(N×M) → O(batch).** `calculateFolderSizesInBackground` flushed the running size map by re-scanning the entire `tabs[idx].items` array every 20 folders and reconstructing every matching `FileItem`. On directories with many subfolders that produced both quadratic time and a churn of FileItem allocations. Now snapshots a `URL → index` map once at task start, queues only the changed indices per batch, and verifies the slot still points at the expected URL before writing — handles the FileWatcher-fires-mid-task race correctly without scanning the whole array.

### Fixed
- **EXIF Editor opens with the cursor file when nothing is multi-selected.** `ViewerWindowModifier`'s metadata-editor route filtered `currentTab.selectedItems` directly — when the user just placed their cursor on an image without marking it, the editor opened at "0 image(s)" and they had to close, mark, re-open. Now uses `selectedOrCursorItems` (the project-wide convention per CLAUDE.md memory: "All file operations use selectedOrCursorItems"). Same pattern applied to `FileSplitMergeView.onAppear` (split-file picker pre-fill) and `IntegratedTerminalView.SmartSuggestionView` so all three feel consistent — point at a file, run the action, no extra Cmd+click required.
- **Function-key bar height bumped 18 → 26pt.** The previous bar barely cleared the F-key text's cap height, leaving no breathing room and making the row look wedged under the panel. 26pt also gives proper tap targets for trackpad / mouse, and the icon + label sizes step up 11 → 12pt for legibility on Retina displays.
- **Main window remembers its size and position across launches.** `WindowGroup { ... }.defaultSize(1200, 700)` only sets the *initial* size for a never-yet-shown window — it's not a frame *restorer*. After the user resized or moved the window, quitting and relaunching dropped them back to the default 1200×700 at SwiftUI's chosen origin. A small `WindowFrameAutosave` `NSViewRepresentable` injected via `.background(...)` now grabs the host `NSWindow` once it's attached and calls AppKit's built-in `setFrameUsingName(_:)` + `setFrameAutosaveName(_:)`. AppKit then reads / writes the frame to UserDefaults under `NSWindow Frame UnifylMainWindow` on every drag and resize. Set-name-after-restore order matters; setting the autosave name first would let AppKit's next layout pass overwrite the saved value with the SwiftUI default.
- **F2 inline rename: Shift+Arrow now extends the text selection.** AppKit silently sets `.numericPad` on every arrow-key event (the arrow keys live on the numeric pad of extended keyboards), and the FunctionKeyMonitor's modifier-flag comparison only stripped `.function`. So `flags == .shift` was false during a Shift+Left press — the actual flag set was `.shift ∪ .numericPad` — and the inline-rename arrow dispatch fell through to the unmodified `moveLeft` / `moveRight` branch. Cursor moved but selection didn't extend; users who tried to select part of a filename for re-typing got nothing. The flag normaliser now subtracts both `.function` and `.numericPad`, fixing Shift+Arrow and incidentally any other modifier-equality check that ever sees an arrow key.
- **HWP → PDF conversion failure now offers the same Hancom fallback as HWPX.** When LibreOffice's HWP import filter rejects a .hwp file (embedded objects, password-protected sections, HWP features newer than the filter understands), the user previously saw a wall of LibreOffice trivia. Now both .hwp and .hwpx route through one helper that detects an installed Hancom Office / Hancom Viewer / Hancom Office Hangul / 한컴오피스 app and offers a one-click "Open in Hancom" button so the user can use Hancom's native File → Export as PDF for ground-truth conversion. When no Hancom app is installed, the alert links to Hancom's download page (HWPX) or recommends the LibreOffice 'Fresh' branch (HWP).
- Dragging an unmarked file when other files were marked is no longer blocked. The drag payload is now context-aware: dragging from a marked row carries the whole marked set, dragging from an unmarked row carries just that file (matches Finder).
- DOS time decode and the file-list date formatters are now pinned to `TimeZone.current` instead of inheriting from the autoupdating-current calendar snapshot, removing a class of "9-hour drift" date display bugs.
- **Spring-loaded folders no longer auto-open when the drag has left the panel.** Previously the dwell timer kept running after the cursor moved outside the table, so a folder armed on drag-start would auto-enter even when the user's mouse was already in another app. The timer now cancels the moment the drag exits or ends.
- **CJK ZIP filenames decode correctly.** Chinese / Japanese / Korean ZIPs created on legacy Windows store filenames in CP936 (GBK), CP932 (Shift-JIS) or CP949 (EUC-KR) without setting the UTF-8 flag bit. The previous decoder always tried EUC-KR first, which silently mis-decoded GBK Chinese names into garbled Korean (`新建文件夹` → `劤쉔匡숭셸`). The new decoder trusts UTF-8 when the bytes parse cleanly, and otherwise scores GBK / EUC-KR / Shift-JIS candidates by Hangul / Kana / Hanja distribution to pick the most plausible decoding. Selective extraction of non-ASCII entries now also routes through the bundled `7zz` (which round-trips the same names) instead of the destructive `ditto -xk` fallback that extracted every sibling regardless of selection.
- **Viewer windows no longer hide when switching apps.** `NSPanel.hidesOnDeactivate` defaults to `true` for utility panels, so any window opened through `ViewerWindowManager` (F3 viewer, Hex Editor, Log Viewer, Git Panel, etc.) vanished the moment the user clicked into Safari / Mail and only reappeared when Unifyl regained focus. Disabled the auto-hide so viewers behave like independent document windows (Preview-style) and stay where the user left them across app switches.
- **Archive extraction is resilient to wrapper-folder name mismatches.** When a CJK-named ZIP is extracted, 7zz / libarchive / our central-directory parser may each pick a different legacy-encoding decoding for the wrapper folder ("新建文件夹" vs "劤쉔匡숭셸"). The previous "Extracted item not found for: …" error fired whenever the extractor's wrapper name disagreed with the archive listing's. Lookup now falls back to a recursive enumeration of the scratch directory by leaf filename, so the file is always located regardless of which decoder produced the wrapper folder name.

### Reliability sweep (audit-driven)

These are not user-requested fixes — they came out of a project-wide audit for silent failures, data-loss patterns, and Swift 6 concurrency hazards. None changes a feature, but they close gaps where a single edge case used to corrupt persistence, hide an error, or race against itself.

- **Archive preflight is no longer bypassed when listing fails.** `ArchiveEngine.preflightOrThrow` previously swallowed `list()` errors with `try?`, leaving extraction to proceed without the path-traversal or decompression-bomb guard. Listing failures now propagate so the caller never runs `7zz`/`ditto` on a refused archive.
- **Saved remote connections survive schema changes.** `ConnectionViewModel` no longer silently drops every saved FTP/SFTP/S3/WebDAV/Azure connection when JSON decoding fails. Decode failures preserve the corrupt blob under a timestamped `UserDefaults` key and surface a one-shot `loadFailureMessage` so users can re-add their connections instead of staring at an empty list.
- **Recorded macros survive schema changes.** Same treatment for `MacroRecorder`: corrupt store is moved to a timestamped backup before the next save can clobber it, with a user-visible message.
- **Local listing surfaces real errors instead of "directory not found".** `LocalFileSystem.contents(of:)` used to remap every Cocoa error to `.notFound`, hiding "permission denied" (Full Disk Access not granted) and I/O failures behind misleading "folder doesn't exist" alerts. Now bridges `NSFileReadNoPermissionError` → `.permissionDenied`, real misses → `.notFound`, everything else → `.unknown(error)`.
- **Files with corrupt resource values stay in the listing.** A single broken symlink, ACL quirk, or extended-attribute corruption used to drop that file from the panel entirely (Finder showed N files, Unifyl showed N-1 with no log). The entry now appears with default permissions instead of vanishing.
- **Azure Blob connection failures now surface in the panel.** Bad credentials / malformed connection string set `panelVM.lastError` and log to Console instead of silently refusing to load the panel.
- **Archive open failures surface in the empty-state UI.** Double-clicking a corrupt or unsupported archive now sets `lastError` and exposes `archiveOpenFailureURL` so the view layer can route password-shaped failures to the password sheet, instead of producing zero UI feedback.
- **Cloud download trigger failures are visible.** `startDownloadingUbiquitousItem` was wrapped in `try?` — when the trigger failed (offline, sandbox denial, evicted item) the UI still showed "☁️ Downloading…" and a 60-second silent timeout. Failures now surface as alerts immediately and skip the watcher.
- **Icon theme install failures show their reason.** `IconThemeManager.downloadAndInstall` reused the existing `downloadStatus` channel for the catch block instead of returning `false` with no context.
- **Multi-file remote download/upload accumulates failures.** Per-iteration `ui.errorMessage` overwrites are gone; a partial failure surfaces a "3 of 10 uploads failed:" summary listing the offending names, matching `FileOperationManager.copy`'s queue behaviour.
- **rclone launch failures log their cause.** `RCloneAvailability.configuredRemotes` previously returned `[]` on `Process.run()` failure, indistinguishable from "no remotes configured" — now logs at fault level so support can read the real reason from Console.
- **Copy preserves timestamps loudly when it can't.** `setAttributes(modificationDate:/posixPermissions:)` failures during copy used to be silenced via `try?`. They're now logged at warning level so users diff'ing source/dest mtimes have an audit trail when the destination volume rejects metadata updates (network shares, exFAT).
- **Process spawned by ArchiveEngine is cancellable.** The shell-out helper previously hopped to GCD with no cancellation hook — cancelling a long extract orphaned the spawned `7zz`/`ditto`/`tar` process and never resumed the awaiting Task. Now uses `Task.detached` + `withTaskCancellationHandler`; the cancel handler `terminate()`s the child and the body resumes with `CancellationError`.
- **`UnifylTableView` is `@MainActor`.** The NSTableView subclass's mutable timer state (`longPressTimer`, `isRightDragging`, `rightDragBaselineIds`) is now compiler-checked instead of relying on AppKit's runloop conventions.
- **FileTableView debounce timers use `Task`, not `DispatchQueue.main.asyncAfter`.** Spring-load (1 s), type-ahead reset (750 ms), and scroll-snapshot debounce (300 ms) now use cancellable `Task { @MainActor in await Task.sleep ... }`. Cancellation is structured — `task.cancel()` wakes the sleep instead of relying on body-side stale-row checks.
- **`AppViewModel.shared` is `@MainActor` instead of `nonisolated(unsafe)`.** All call sites already hopped through `Task { @MainActor in ... }`; the explicit isolation re-engages the type system instead of letting the compiler look the other way.
- **Cloud-download monitor pins its tab by UUID.** `monitorCloudDownloads` previously read `tabs[activeTabIndex]` per iteration — a tab switch mid-download leaked progress badges onto the wrong tab. Resolution is now via tab id with active-tab gating before reload.
- **Git status load moves `fileExists` off main.** Slow network volumes (SMB/SFTP-mounted .git) used to block the main actor for hundreds of milliseconds during the synchronous existence check.
- **Archive exit reload is structured.** `exitArchive`'s fire-and-forget reload Task is now stored in `archiveExitTask` and cancelled before launching, so a rapid double-toggle no longer races two `loadCurrentDirectory` calls and causes the directory to flash-then-revert.
- **Static URL constants trap with diagnostic context.** Cloud FS adapters (Dropbox, Google Drive, OneDrive, S3, Azure) and OAuth2 configurations no longer use bare `URL(string: …)!`. The shared `StaticURL.make(...)` helper replaces them and emits a `preconditionFailure` with file:line on malformed input.

### Design / a11y / perf sweep (audit-driven, round 2)

- **Design tokens grow Spacing + Motion scales.** `DesignTokens.Spacing` (xs/s/m/l/xl) and `DesignTokens.Motion` (micro/standard/modal) join the existing Colors / Radius / Typography. Replaces the previous "5 sheets, 3 different padding values" chaos at the token level so future sheets fall into the rhythm without effort.
- **Sheet padding standardised.** NewFolderSheet and ArchivePasswordSheet migrated from ad-hoc `padding(20)`/`padding(18)` to `DesignTokens.Spacing.l`. Title fonts also routed through the typography scale.
- **Hardcoded overlay / scrim colors moved into tokens.** Command palette scrim, locked-feature overlay, Quake terminal background, and progress-bar tracks were `Color.black.opacity(...)` literals scattered across views; now consolidated as `Colors.scrim`, `terminalBg`, `progressTrack` so the theme engine can reach them.
- **Empty states unified onto `EmptyStateView`.** Replaced 9 ad-hoc `ContentUnavailableView` call sites (MetadataEditorView, ScheduledTaskView, AutomationView ×2, AuditLogView, MacroManagerView, HexEditorView, IncrementalBackupView, DuplicateFinderView ×2, PluginManagerView ×2) with the project's design-system empty state for one consistent visual rhythm.
- **Reduced-motion support.** New `.accessibleAnimation(_:value:)` modifier consults `accessibilityReduceMotion` and disables the transition when the user has Reduce Motion enabled in System Settings. Wired into MainWindowView's command palette / terminal / file-op transitions and TabBarView's tab-count animation; remaining sites can opt in incrementally.
- **Command palette is localisation-ready.** Command titles now go through `LocalizedStringKey` so they translate via Localizable.strings without requiring every `title:` literal at registration to wrap in `NSLocalizedString(...)`. Search placeholder and empty-state text are also localised.
- **A11y labels added to icon-only buttons** that previously announced as generic "button" to VoiceOver — TabBar's New Tab button (`.help` is a tooltip, not a VoiceOver label) and CommandLineBar's close button. CommandLineBar close also gains a `.cancelAction` keyboard shortcut so keyboard-only users can dismiss it.
- **`PersistenceBackup` helper centralises corrupt-blob recovery.** ConnectionViewModel and MacroRecorder both copy corrupt persistence blobs aside under timestamped sidecar keys before the next save can clobber them; the pattern is now in one place (`stashCorruptUserDefaultsBlob` for defaults, `stashCorruptFile` for on-disk stores) so future persistence sites adopt the same recovery contract automatically.
- **Cold-start no longer blocks on icon-theme scan.** `IconThemeManager.scanExternalThemes()` was synchronous in `init` and added 50–100 ms to launch on slow volumes (SMB-mounted Application Support, OneDrive-synced home). Deferred to a `Task.detached(priority: .utility)` so the first window paints with built-in themes only; external themes appear when the scan returns.
- **Search filter regex memoised.** `SearchEngine.filter` recompiled `NSRegularExpression` on every keystroke; on a 100K-item folder the compile cost (~5–15 ms) dominated per-keystroke latency. The compiler is now cached behind an `NSLock` with a 64-entry bound, so repeated typing on the same pattern reuses the prior compile.
- **Drag regression guard.** Reverted an experimental `@MainActor` annotation on `UnifylTableView`. The annotation was meant to compiler-check what AppKit's runloop conventions already guarantee, but it perturbed NSTableView's mouseDown → drag-threshold dispatch in some configurations and silently broke drag initiation. Methods all run on main via AppKit anyway; explicit isolation here costs more than it pays.


## [1.0.9] — 2026-04-22

> **Note**: 1.0.8 was never publicly released. This entry consolidates the 1.0.8 design audit + power-user features + Settings tunables with the 1.0.9 Log Viewer overhaul, Settings resize fix, Theme Editor preset sidebar, and Compress dialog routing. Users upgrade 1.0.7 → 1.0.9 in one hop.

### Added
- **Type-ahead select** in the file list. Typing any sequence of characters jumps the cursor to the first matching file, with a 750 ms idle reset between bursts. Korean input is choseong-normalised so "민수.txt" matches "ㅁ" — both the filename's first syllable and the partial-jamo input collapse to the same compatibility-jamo initial consonant.
- **Spring-loaded folders**: dragging a file over a directory row for 1 s auto-navigates into it. An accent-tinted progress fill sweeps across the row during the dwell so the feature is discoverable. Works on `..` for parent navigation. Cancels cleanly on drop, drag-exit, or hover-row change.
- **Tab right-click context menu**: Rename / Reset Name / Pin / Duplicate / Move to Other Panel / Close / Close Others / Close Tabs to the Right / Reveal in Finder / Copy Folder Path. Pinned tabs show a pin glyph, hide the ✕, and survive "Close Others" / "Close Tabs to the Right". Custom tab titles are rendered in preference to the path's last component.
- **Breadcrumb click-to-edit**: double-click the path bar or click the trailing hover-only pencil to switch to a raw-text URL-bar style editor. Enter navigates, Esc cancels, invalid paths beep.
- **Dock icon context menu** (NSApplicationDelegate.applicationDockMenu): Reveal Active Folder in Finder + Recent Folders (top 5 by frecency, identical source to the Go ▸ Recent Folders submenu).
- **"Send to Unifyl" macOS Service**: right-click files in Finder / Mail / Safari → Services → "Send to Unifyl". The app navigates the active panel to the dropped items' parent folder and mark-selects them. NSServices entry in Info.plist; provider selector explicitly bound via `@objc(sendToUnifyl:userData:error:)` so pbs can resolve it.
- **Copy Path As…** submenu under Edit: POSIX path / `file://` URI / shell-quoted / unix-shell form (`~/` + escaped spaces) / relative to current folder / filename only.
- **Reveal cursor file in opposite panel** (⌃⌥O, also under Go ▸ Reveal in Opposite Panel): navigates the inactive panel to the cursor item's parent folder (or into the folder if the cursor is on a directory). Prep work before compare / sync / diff.
- **Cmd+1..9 tab jump**: switches the active panel to the Nth tab. Physical keyCode mapping so it fires under any keyboard layout / IME.
- **F7 "New Folder" sheet** with pre-selected placeholder text. The NSTextField is wrapped in an NSViewRepresentable that calls `selectText(nil)` once the window accepts it as first responder — typing immediately replaces "New Folder" without needing ⌘A first.
- **Log Viewer grep/find toggle.** The filter field now has a "Grep" checkbox. Grep mode (default) hides non-matching lines as before; turning it off switches to find-in-place behaviour where every row stays visible and matches are highlighted. Placeholder text and tooltip update to match the active mode.
- **Log Viewer "Load All" escape hatch.** Cold-opening a 100 k+ line log via `LazyVStack` + `.textSelection(.enabled)` can't meaningfully virtualise — intrinsic-size measurement dominates. Initial load now tails the last 5 000 lines and a header banner reports `Showing last 5,000 of N lines` with a Load All button for the rare case historical lines are needed.
- **Theme Editor: Built-in Presets sidebar section.** The 12 presets are now browseable in the editor sidebar alongside custom themes. Selecting a preset copies its tokens into the edit panel as a starting point (the preset itself is never mutated); right-click → "Duplicate as Custom Theme" creates a fresh editable copy tagged `Preset Name (Copy)`.
- **Active-panel focus desaturation.** The inactive panel's content is now rendered at 50 % saturation (plus the existing 92 % opacity), matching the Mail / Messages / Finder pattern of using colour intensity — not chrome lines — to signal which pane has focus. An accent-coloured 2 pt stripe along the top of the active panel's PathBar reinforces the read.

### Changed — Design (senior macOS designer audit)
- **Theme presets now theme the full app chrome.** Before this release, PathBar / TabBar / StatusBar / OperationQueueBar / FunctionKeyBar read frozen values from `DesignTokens.Colors`; picking Nord or Solarized Light left the file list correctly tinted but the chrome stuck on the Unifyl Dark Pro palette. All chrome tokens promoted to `ThemeManager.effective*` with system-colour fallbacks so `macOS (Automatic)` keeps stock appearance.
- **Per-row file-type colour flood removed.** Filename + icon still tint by category, but Size / Date / Kind columns now use the theme's secondary-label token. Previous behaviour painted every column saturated per-row and was the single biggest reason the file list read as "TC-90s terminal".
- **F-key bar off by default.** Re-enable via View ▸ Show Function Key Bar (persists via `unifyl.showFunctionKeyBar`). When shown, redesigned: 18 pt thin bar with SF Symbols + action word + 9 pt F-key hint, all theme-driven. The jarring teal Terminal pill is gone.
- **PathBar hierarchy restored.** Current folder is 13 pt semibold primary text with no background chip; ancestors are 12 pt regular secondary. Previous muted-chip on the current folder visually outweighed tab-active affordance, and the textMuted ancestor colour failed WCAG AA at 2.1:1.
- **ToolbarStatusCenter simplified.** Brand lockup + current folder name in the principal slot; FREE badge only shown when unlicensed. Selection summary removed — it duplicated `PanelStatusBar`.
- **Toasts moved out of the file list** into the active panel's status bar (11 pt, SF Symbol, 0.5 pt themed border, legacy emoji prefixes auto-stripped). Previous floating toast covered the bottom rows of the file list.
- **Active panel indicator strengthened.** Active panel's PathBar gets a faint `accent.opacity(0.06)` tint; inactive panel content dims to 92 % opacity.
- **Typography reconciled.** Size and Date columns use `NSFont.monospacedDigitSystemFont` so digits line up during scroll; Tab labels stepped down to 12 pt medium so filenames remain the heaviest type on screen.
- **Radius tokens unified.** `DesignTokens.Radius.chip` (4 pt) / `.card` (6 pt) / `.sheet` (8 pt), all `.continuous`.
- **AppTheme schema expanded.** Ten new tokens (`elevatedHex`, `overlayHex`, `borderSubtleHex`, `borderDefaultHex`, `textMutedHex`, `errorHex`, `warningHex`, `successHex`, `dropTargetHex`, `focusRingHex`) with system-colour fallbacks.
- **Preset curation fixes.** 13 presets → 12 (Classic Navy removed, near-duplicate of Midnight). Contrast fixes on High Contrast (invisible white-on-white selection), Solarized Light (washed-out selection), One Dark (wrong primary color), Dracula (non-canonical selection).

### Changed
- **"Compress to ZIP…" → "Compress…"** everywhere (File menu, command palette, right-click). The label now matches the new behaviour: picking Compress opens `CompressDialogView` with destination / filename / format pre-filled, rather than silently writing a zip into the active panel's current folder. `addSelectedToArchive()` remains as a scripting-only silent-write entry point.
- **Compress dialog Smart Folder fallback.** When the source panel is in Smart Folder mode (`currentPath` is the pre-cart directory and the panel won't refresh anyway), the compress destination falls through to the inactive panel's real path, then to `~/Downloads`. A post-archive toast reports the final location.
- **Log Viewer performance overhaul.** `LogLine.id` switched from `UUID` to the line number `Int` (UUID allocation was ~3–5 µs each, compounding to seconds on cold open of a 100 k-line file). Log level is now classified once at ingest via `range(of:options:.caseInsensitive)` instead of uppercasing the whole string on every SwiftUI body evaluation. Filter input is debounced so the per-keystroke `NSAttributedString` rebuild no longer stalls typing on large buffers.
- **Log Viewer colour rendering.** Level colours now come from direct `NSColor` constants rather than a `Color → NSColor` round-trip, which occasionally collapsed every level to a single hue ("every line shows red").
- **Settings window is resizable.** The SwiftUI `Settings` scene defaulted to `.contentSize` resizability on macOS 13+ and silently stripped `.resizable` from the styleMask on every relayout, which made the new 20+-tunable General pane overflow the fixed 550 × 480 floor. Fixed at three layers: `.windowResizability(.contentMinSize)` on the scene, an `NSViewRepresentable` that unions `.resizable` into the hosting window's styleMask on mount (with 6 retries over the first second while the window is still being parented), and a 0.25 s repeating timer in `UnifylAppDelegate` that re-inserts the flag while a Settings window exists. The timer trampolines through `DispatchQueue.main.async` to avoid `_dispatch_assert_queue_fail` under Swift 6 strict concurrency.

### Added — Settings (General pane)
- **Size format** picker — Abbreviated (`4.2 MB`) / Bytes (`4,404,019`, locale grouping, no unit suffix since the column header carries it) / Both (`4,404,019 (4.2 MB)`).
- **Date format** picker — Short / Medium (default) / Long / Relative (`2 hours ago`) / ISO 8601 (`2026-04-21 14:14`).
- **Double-click action** picker — Open / Open with default app / Toggle selection. Directories and `..` always navigate regardless of the setting.
- **Row font size slider** (9–24 pt) synced with Cmd+/- / Cmd+= via a `UserDefaults.didChangeNotification` observer in `AppViewModel`.
- **Spring-loaded folder delay** slider (0.4–2.5 s).
- **Type-ahead buffer reset** slider (0.3–2.0 s).
- **Toast auto-dismiss** slider (0.5–6.0 s).
- **Toggles**: Show background size gauge / Show git status dots / Show hidden files by default / Show function key bar / Show operation queue bar / Show path bar.
- **Confirmation toggles**: Confirm before Copy / Move / Delete / Permanent Delete / Overwrite on rename-copy. When false, the corresponding sheet is short-circuited.
- Every new setting reflects in the UI immediately — no restart required.

### Fixed
- **F2 rename mode Cmd+A/C/V/X/Z broken under Korean/Japanese IME.** The NSEvent monitor was firing correctly; the switch-on-characters arm just couldn't match "ㅁ" (IME output on `m` key) to `"a"`. All shortcut dispatch in the field-editor path now uses `event.keyCode`.
- **Intentional rename-onto-existing-file now asks Overwrite?** The single-file rename-copy path was calling `FileManager.copyItem` directly and hard-failing with `"Copy failed: already exists"`. Routed through the shared `ConflictResolution` dialog (Overwrite / Skip / Keep Both).
- **Theme Editor no longer opens multiple windows.** `ViewerWindowManager.open(singleInstance: true, ...)` brings an existing panel to the front instead of cloning.
- **Custom themes no longer duplicate by name** across app launches. `CustomThemeStore.add` dedups by id AND by case-insensitive trimmed name; `mergeDuplicatesByName()` runs on store load to collapse any pre-existing duplicates.
- **Toast auto-dismiss audit**. "Copied N paths", undo-file-op, Smart Folder add, PDF-convert-failure, and command-palette undo were all setting `ui.tagNotification` without scheduling a timer to clear it. Refactored and wired all sites.
- **Mouse wheel scroll no longer reverts to cursor position** after a brief delay. The scroll snapshot was mutating @Observable state on every frame, triggering SwiftUI rebuilds that called `ensureCursor` → `scrollRowToVisible`. Snapshot wiring disabled.
- **Bytes size format clipping and misalignment fixed** by dropping the per-row "B" suffix. The Size column header carries the unit.
- **NSServices registration** — removed deprecated `NSFilenamesPboardType` (pbs on macOS 14+ drops Services entries that mix legacy pboard types with UTI types). Added `@objc(sendToUnifyl:userData:error:)` explicit selector binding.
- **Closing the Log Viewer no longer terminates the app.** Pressing Esc in the Log Viewer panel used to quit Unifyl entirely because SwiftUI treated it as the last remaining window. `applicationShouldTerminateAfterLastWindowClosed` now explicitly returns `false`, keeping the main scene alive regardless of auxiliary-window lifecycle.
- **Theme Editor preset-id collision.** A legacy bug let the editor save a preset-seeded draft back into the custom-theme store under the preset's own id, so the sidebar showed the same theme twice. `persistAsCustomTheme` now detects the collision, reassigns a fresh UUID, and appends "(Copy)" to the name. `CustomThemeStore.removePresetDuplicates` runs on store load to purge any pre-existing collisions from earlier app versions.
- **`PanelView.fileColorVersion` arithmetic overflow trap.** Combining randomised Swift `.hashValue` results with `+` regularly tripped "Swift runtime failure: arithmetic overflow" during QA. Switched to overflow-wrapping `&+` — the value is only ever compared for inequality, so wraparound is semantically fine.

### Developer
- `UnifylSettings.swift` shared helper for non-SwiftUI callsites to read user preferences.
- `UnifylAppDelegate.swift` hosts `applicationDockMenu` + NSServices provider.
- `AppViewModel.shared` static weak handle published for AppKit-layer consumers.
- The removed "single main window enforcer" (a `NotificationCenter` observer in `UnifylAppDelegate` that closed duplicate main windows) caused `_dispatch_assert_queue_fail` crashes on Thread 3 because `w.close()` was invoked outside the main actor even with `queue: .main`. It's gone; multi-window `WindowGroup` behaviour is annoying but no longer crash-level. A proper `Window` scene migration is tracked separately.


## [1.0.7] — 2026-04-19 (respin build 12)

### Changed (respin build 12)
- **Pro trial period extended 3 days → 14 days** (`FeatureGateManager.trialDuration`). The previous 3-day window wasn't long enough for a new user to meaningfully explore the AI search, multi-rename, 3-way merge, or archive workflows before the paywall landed. Wiki `Pro-and-Licensing` and `FAQ` pages had already been documenting "14 days", so the code was lagging the intent — now they agree. Existing trials that haven't yet expired automatically get the wider window applied retroactively (`isWithinTrialPeriod` compares elapsed against the new constant). The keychain-backed expiration ratchet is unchanged, so already-expired trials stay expired.

## [1.0.7] — 2026-04-19

### Added
- **Total Commander-style right-drag selection** in the file list. Hold right mouse button and drag — the range is mark-selected and the NSTableView cursor tracks the pointer so the drag's end row is always visible. Sequential drags accumulate (union) onto the existing marks instead of replacing; a drag starting on an already-marked file subtracts its range from the baseline.
- **Shift / Cmd / Ctrl + click** selection modifiers in `FileTableView.mouseDown`:
  - Shift-click: range-mark from the current cursor to the clicked row (Finder-style); clicked row becomes the new cursor.
  - Cmd-click / Ctrl-click: toggle just the clicked row in the mark set without disturbing existing marks. Both modifiers are accepted — Cmd for Finder users, Ctrl for Total Commander veterans.
- **Bundled 7zz binary** (`Unifyl/Resources/bin/7zz`, universal Mach-O x86_64 + arm64, ~5.6 MB, LGPL-2.1 with `7zz-LICENSE.txt` alongside). `postBuildScripts` phase copies it into `.app/Contents/Resources/bin/7zz` with the executable bit preserved (the default Copy Bundle Resources phase strips +x). `ArchiveEngine.find7z()` now prefers the bundled path; Homebrew paths remain as fallback so power users can substitute a newer build. Eliminates the `brew install p7zip` requirement for encrypted / LZMA / WinZip AES zips.
- **Password-protected archive extraction** — new `ArchiveEngine.requiresPassword(_:)` probes via the first local file header's general-purpose bit flag for ZIP or `7z l -slt -p""` for 7z/RAR. When the probe returns true, `copyFromArchive` / `moveFromArchive` open a new `ArchivePasswordSheet` (SecureField + autofocus + Return-to-extract); on success the archive is unpacked via `/usr/bin/unzip -P …` or 7z with `-p…` as appropriate. Wrong password reopens the prompt with "Incorrect password. Please try again." False-negative fallback: `Unknown compression type` / `need PK compat` / encryption-shaped error messages also trigger the prompt.
- **Wrapper-folder unwrap** after archive extraction: when the extracted artefact at the destination is a directory whose only real child matches the selected item's name (`X.hwp/X.hwp` — the pattern used by HWP / Numbers / Keynote / Pages / Sparkle xcframework wrappers), the wrapper directory is collapsed so the user gets a single file. Depth-guarded (max 64 same-named layers) so a malicious archive can't spin forever. macOS bookkeeping (`__MACOSX`, `.DS_Store`, `._*` AppleDouble entries) is excluded from the child count so the unwrap still fires on zips that include metadata siblings; real user-created siblings prevent the unwrap to avoid silent data loss.
- **HWPX → PDF Hancom passthrough**: LibreOffice's HWP import filter targets only the legacy binary .hwp. HWPX (2010+) is a zip-based XML package LibreOffice doesn't decode natively. `convertViaLibreOffice` now (a) requests the explicit `writer_pdf_Export` filter for HWPX, (b) retries via a rename-to-.hwp temp copy that LibreOffice's binary HWP filter can sometimes decode, and (c) on final failure shows a new `showHWPXConversionFailedAlert` with an **"Open in `<app>`"** button. `findHancomApp()` scans /Applications for Hancom Office (any year), Hancom Viewer, Hangul.app, the Korean-named bundles, and Polaris Office v9+. When nothing matches, the button opens hancom.com's download page.
- **AI Semantic Search onboarding** (`AIOnboardingSheet`): first-time users who invoke AI search while the index is empty now see a focused pitch card (4 benefit rows: local, concept-based, fast, controllable) with `Index Home Folder` + `Choose Folder…` + `Skip for now`. Dismissal is remembered in UserDefaults so the card never shows again.
- **Archive extraction progress toast**: `copyFromArchive` / `moveFromArchive` / `addURLsToArchive` post `📂 Extracting <file>…` / `📂 Moving <file>…` / `📦 Adding <file> → archive…` to `ui.tagNotification` while the operation runs and clear it on completion. Long extractions (encrypted / large zips) no longer look frozen.
- **Drop external files onto an archive row**: dragging URLs from another panel or Finder onto a `.zip`, `.7z`, or `.tar.*` row highlights that row and adds the dropped items into the archive at its root (via `ArchiveEngine.addToArchive`). `FileTableView.validateDrop` detects the archive row and forces `.copy` semantics (adding to an archive is never a move of the source).
- **Recent Folders** menu under `Go` — frecency-ranked list (visit count × 7-day half-life decay) of the last 40 folders the user navigated to. `PanelViewModel.navigate` records every file-URL visit; the top 12 live entries show up with a `Clear Recent Folders` button. Dead paths (ejected volume, deleted folder) are filtered before display.

### Changed
- **Inside-archive F5 / F6 routing** — `copyToDestination` and `moveToDestination` now branch on `activePanelVM.isInArchiveMode` and call `copyFromArchive` / `moveFromArchive` instead of handing synthetic archive URLs to the regular FileOps path (which was at best a no-op, at worst a silent error).
- **Destination path resolution** for single-item archive extraction: `FileOperationConfirmView` returns `destinationPath/<filename>` in its single-item rename form, which the archive routines used to treat as a parent directory and then append `<filename>` again — producing the `destination/X.hwp/X.hwp` wrapper folder users kept reporting. Both `copyFromArchive` and `moveFromArchive` now strip the trailing filename when it matches the selected item's name.
- **ZIP extraction prefers bundled 7zz over macOS `ditto`** in `ArchiveEngine.extract(_:to:)`. `ditto -xk` wraps certain zips (observed on CJK-named single-file archives) with an extra directory layer that doesn't match what the zip's central directory actually contains. 7z reads the same CD Unifyl uses for listing, so the extracted tree matches what the user sees in the archive listing.
- **Move-from-archive removal path enumeration**: `zip -d` does NOT support glob patterns (POSIX Info-Zip matches entry names literally), so the previous `dir/*` / `dir/*` append silently left directory children inside the archive after a move. Now enumerates the full `allArchiveEntryPaths` list from `PanelViewModel` and collects every path that carries the removed directory's prefix.
- **Live key-binding menu-bar sync infrastructure** — parser (`Unifyl/Services/KeyBindingShortcutParser.swift`) turns `KeyBindingManager` shortcut strings into SwiftUI `KeyboardShortcut` values; `.keyBinding("command.id")` view modifier applied to 42 menu items in `UnifylApp.swift` so remapping a shortcut in Settings updates the menu-bar indicator immediately. The literal `.keyboardShortcut(…)` calls are kept as default fallbacks — `KeyBindingModifier` overrides only when the ID is registered.
- **Theme Editor reworked**: independent NSPanel via `ViewerWindowManager` (movable, resizable, ESC-to-close) instead of modal sheet; schema harmonised with `AppTheme` (8 chrome tokens: accent / background / surface / selection / altRow / cursor / textPrimary / textSecondary); new `Apply as Active Theme` button + `Live Apply` toggle; backward-compatible decoder for pre-1.0.6 `.ultratheme` files. Toolbar theme menu splits Built-in and Custom sections with an `ACTIVE` capsule on the current theme.
- **Cursor theme token** (`cursorHex`) added to all 13 built-in presets. File-table rows distinguish the keyboard-focus cursor (single-row, bright, 3-px left-edge stripe + fill) from multi-selection marks (dimmer fill). Matches Total Commander's single-focus visual.

### Fixed
- **`unwrapWrapperFolder` no longer silently destroys siblings** — previously the multi-child path would find a filename match inside a wrapper directory, `moveItem` it aside, `removeItem` the whole directory tree, then `moveItem` the match back, quietly losing every other file. Now only promotes when the *only* real children (ignoring `__MACOSX` / `.DS_Store` / `._*` AppleDouble bookkeeping) are either a single same-named file or a single child of any name. Every destructive `removeItem` is preceded by a `UnifylLogger.archive.warning` for audit.
- **`collapseSingleSameNamedChild` depth guard** — a malicious archive with `X/X/X/X/…` nested same-named dirs no longer spins forever; bounded to 64 layers.
- **`ArchivePasswordSheet` dismiss race** — stopped calling `@Environment(\.dismiss)` alongside `ui.isArchivePasswordOpen = false`. The binding alone drives dismissal; the double-flip was collapsing the retry-on-wrong-password re-open when SwiftUI amortised the animation.
- **`preflightExtractionSafety` vestigial code removed** — the function logged "blocked archive" but never threw, so an attacker who tripped it still had their archive extracted. The real guard (`preflightOrThrow`) is already called from every extract path; kept only that.

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

[Unreleased]: https://github.com/goodbug89/Unifyl.app/compare/v1.7.2...HEAD
[1.7.2]: https://github.com/goodbug89/Unifyl.app/releases/tag/v1.7.2
[1.7.1]: https://github.com/goodbug89/Unifyl.app/releases/tag/v1.7.1
[1.7.0]: https://github.com/goodbug89/Unifyl.app/releases/tag/v1.7.0
[1.6.7]: https://github.com/goodbug89/Unifyl.app/releases/tag/v1.6.7
[1.6.6]: https://github.com/goodbug89/Unifyl.app/releases/tag/v1.6.6
[1.6.5]: https://github.com/goodbug89/Unifyl.app/releases/tag/v1.6.5
[1.6.4]: https://github.com/goodbug89/Unifyl.app/releases/tag/v1.6.4
[1.6.0]: https://github.com/goodbug89/Unifyl.app/releases/tag/v1.6.0
[1.5.0]: https://github.com/goodbug89/Unifyl.app/releases/tag/v1.5.0
[1.4.0]: https://github.com/goodbug89/Unifyl.app/releases/tag/v1.4.0
[1.3.2]: https://github.com/goodbug89/Unifyl.app/releases/tag/v1.3.2
[1.3.1]: https://github.com/goodbug89/Unifyl.app/releases/tag/v1.3.1
[1.2.3]: https://github.com/goodbug89/Unifyl.app/releases/tag/v1.2.3
[1.1.0]: https://github.com/goodbug89/Unifyl.app/releases/tag/v1.1.0
[1.0.4]: https://github.com/goodbug89/Unifyl.app/releases/tag/v1.0.4
[1.0.3]: https://github.com/goodbug89/Unifyl.app/releases/tag/v1.0.3
[1.0.2]: https://github.com/goodbug89/Unifyl.app/releases/tag/v1.0.2
[1.0.1]: https://github.com/goodbug89/Unifyl.app/releases/tag/v1.0.1
[1.0.0]: https://github.com/goodbug89/Unifyl.app/releases/tag/v1.0.0

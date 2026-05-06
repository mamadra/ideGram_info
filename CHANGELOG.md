# Changelog

All notable changes to the IDEGram plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

## [0.3.22] - 2026-05-06

This release closes most of the **Tier 1 parity** roadmap (chat search, polls,
animated stickers/GIFs, read receipts) and adds the **Telegram-style album
grid** so multi-photo/video posts render the way they do in the official client.
A rollup of everything since 0.3.15.

### Added

#### Stickers and GIFs (043 + 044)
- **Inline sticker rendering** in the chat: WebP / TGS / WebM stickers download
  on demand, are cached in the IDE's system directory, and render inline next
  to text without forcing the chat into preview mode. Tap-to-load fallback for
  files larger than 1 MB and an "Auto-download stickers" toggle in Settings.
- **Inline GIFs** with the same on-disk cache, fixed display sizing, captions
  and per-message size labels.
- **Animated GIF playback**: GIFs play smoothly inside the message bubble
  (looping, silently). Decoded with a pure-Java JCodec pipeline, paced at the
  file's native frame rate (24/25/30 fps detected from container metadata),
  paused automatically when scrolled off-screen or when the IDE window loses
  focus, and capped to 6 concurrent decoders so a chat full of GIFs can't
  monopolise the CPU. On decoder failure the bubble silently falls back to
  the static thumbnail with a ▶ overlay.
- **Animated TGS stickers** (Lottie) — render through Skiko's bundled Skottie
  binding directly in the Compose canvas, so they work on Android Studio
  whose JBR strips JCEF.
- **Animated WebM video stickers** — render through an embedded JCEF browser
  on IDEs whose JBR ships JCEF (IntelliJ IDEA, etc); fall back to a static
  thumbnail on JBR builds without JCEF.
- **Big animated emoji** (single-emoji messages, Telegram's
  `MessageAnimatedEmoji`): now appear as proper TGS animations instead of
  showing as `[Unsupported content]`.
- Vendored `lottie_light.min.js` (MIT) — animated stickers work fully offline.

#### Polls (045)
- **Receive and vote on polls** with the official Telegram UX:
  - Percentages and per-option progress bars are hidden until you vote
    (matches the desktop client).
  - **Optimistic UI** — the option you tap looks selected within ~100 ms,
    then reconciles against TDLib's authoritative result.
  - **Live updates** — the bubble's totals refresh as other participants
    vote, without the chat scroll jumping.
  - **Change vote** by tapping a different option (single-answer regular polls).
  - **Closed polls** are read-only with a "Final results" badge.
  - **Vote failure** (network / server error) shows a subdued
    "Vote failed — tap an option to retry" hint and a single INFO log line
    per (message, query) — no popups, no log spam.
- **Quiz polls**: after answering, the chosen option is highlighted in red if
  wrong, the correct option is always shown in green, and a ✓ / ✗ glyph is
  added next to the question. If the quiz has an explanation, a "Why?" link
  reveals it inline.
- **Multiple-answer polls**: tapping options toggles selection, a Vote button
  enables once at least one option is selected, all picks are submitted in a
  single TDLib call, and a "Retract vote" affordance appears afterwards.
- "View N voters" affordance is rendered for non-anonymous polls (the modal
  itself is a follow-up feature; for now it's a placeholder).

#### In-chat search (046)
- **Cmd-F / Ctrl-F** (or the magnifier icon in the chat header) opens a search
  bar above the message list.
- **Live results** — debounced TDLib `searchChatMessages`; the bar shows
  "X of Y" and the chat auto-scrolls to the newest match within ~1 s.
- **↑ / ↓ navigation** between matches, with wrap-around at both ends; the
  focused match is centred in the viewport with a subtle yellow outline.
- **Inline highlighting** — every occurrence of the query inside visible
  messages gets a soft yellow background; the focused match's first
  occurrence is highlighted strongly. Existing formatting (links, bold,
  code) and link clickability are preserved.
- **Out-of-window matches** load on demand: searching pulls the surrounding
  chat history (`getChatHistory(fromMessageId, offset=-25, limit=50)`) so
  search results far back in time still scroll into view.
- **Sender filter** in group / supergroup chats — a "From" picker reuses the
  per-chat participants list to constrain results to a specific sender.
  Hidden in private chats.
- **Esc** closes the bar and restores the chat scroll to where you were
  before searching.
- Search-match outline + inline highlight survive across album bubbles
  (added in 047 below) — the whole album lights up when any of its members
  is the focused match.

#### Read receipts (045-read-receipts)
- **Per-message read counter** in group chats: outgoing messages show how
  many of the active recipients have read them as a small "5/20 read" label.
- **Modal with the actual list** — clicking the counter opens a popup with
  who has read and who hasn't, including avatars.
- Counter updates live as more participants read.

#### Media albums (047)
- **Telegram-style mosaic grid** for albums (multiple photos/videos posted
  simultaneously by the same sender — TDLib's `mediaAlbumId`). Matches the
  official desktop client: 2 items in one row, 3 as 1+2, 4 as 2×2, up to 10
  items in 2-3 row layouts.
- Caption is rendered **once** below the grid (taken from the first item with
  a non-empty caption); sender header / forward-from / reply quote appear
  **once** at the top of the album, not per item.
- Search match-focus outline (046) wraps the whole album bubble.
- Pagination, sender filter, unread counter and feature-021 hidden-sender
  filter all treat an album exactly like its members — no special exemptions.

#### Settings: media cache & auto-download (044-media-settings)
- **Auto-download toggles** for stickers and GIFs, separate from photos.
- **Cache size threshold** — files above 1 MB show as preview-only with
  tap-to-download.
- **Clear all media cache** button for stickers and GIFs.

### Changed
- Polls and message-poll-content updates flow through a shared optimistic
  reducer in the message ViewModel, replacing the earlier fire-and-forget
  vote use-case.
- Scroll-to-message infrastructure (used by reply-quote navigation) now
  carries a `highlight: Boolean` flag so non-highlighting scrolls (e.g. the
  search-bar's "restore on close" path) don't apply the yellow flash.

### Fixed
- Search results inside an album scroll to the album bubble correctly and
  the focused-match outline wraps the whole grid.
- Pagination at the top of the chat (loading older messages) was missing
  the threshold trigger when albums collapsed multiple messages into a
  single rendered item — fixed by tracking the rendered list size.
- Animated stickers via JCEF are gracefully detected as unavailable on
  Android Studio (whose bundled JBR lacks JCEF binaries) and fall back to
  the static rendering automatically. TGS stickers render through Skottie
  in all environments.
- Search bar text-field cursor no longer jumps to position 0 on every
  keystroke.

### Internal
- New domain types: `Poll`, `PollVoteIntent`, `QuizFeedback`, `SearchSession`,
  `ResultsState`, `SearchMatch`, `SearchHighlightOverlay`, `MediaAlbum`,
  `DecoderStrategy` / `AnimationPlaybackSession` / `ConcurrencyGovernor` for
  the playback pipeline, `ScrollAndHighlight` payload extension.
- New use cases: `SearchChatMessagesUseCase`, `VotePollUseCase` (rewritten),
  `PlayAnimationUseCase` / `PauseAnimationUseCase`.
- New dependency: `org.jcodec:jcodec-javase:0.2.5` (~3 MB, BSD-2) for
  software MP4/H.264 decoding of animated GIFs.

## [0.3.15] - 2026-05-05

### Added
- **Windows x64 support**: the plugin now ships with a bundled native TDLib
  library for Windows x64, so it works out of the box on Windows alongside
  the existing macOS (arm64) build. No manual native-library setup required.

## [0.3.13] - 2026-04-30

### Added
- Service messages now render as readable italic-grey lines in chat history
  (member joined / left / removed, chat title or photo changed, message pinned,
  call ended, etc.) and are excluded from unread tallies and IDE notifications.
  Pinned-message lines are clickable and scroll to the original.
- Voice notes: incoming voice messages render as a real voice bubble with
  on-demand download, OGG/Opus decoding, 100-bar waveform progress, play/pause,
  duration label, and unread red dot. Listened state syncs back to Telegram.
- Voice recording & sending: a microphone button in the message input opens an
  in-place recording bar (pulsing red dot, elapsed timer, Cancel/Stop). On Stop,
  a preview row lets you replay before sending; Discard drops the take.
  Recording is automatically cancelled when the chat is closed or switched.

### Internal
- Freemium licensing foundation: groundwork for the upcoming paid tier (single
  Marketplace artifact, free core forever). No behavior change for existing
  users — all current features remain free. Adds an internal `LicenseService`
  reading the IDE's licensing facade and a hidden registry key for development
  overrides. The Settings page shows the current license tier (always *Free*
  today) and a Refresh button.

## [0.3.12] - 2026-04-27

This is a rollup of everything since 0.3.5 — if you're upgrading from 0.3.4
or earlier, here's the highlight reel of what's new.

### Added
- **Reply to a message**, with full **partial-quote** support: drag-select
  any fragment of the original in the reply preview and the recipient on
  mobile / web Telegram lands directly on that fragment when they tap the
  reply.
- **Inline reply previews are clickable**: scroll to and briefly highlight
  the original message; older history is fetched on demand if needed.
- **Forward a message** to another chat from the right-click menu, with
  Telegram's standard *Forwarded from …* attribution preserved.
- **"Forwarded from" label** rendered above the bubble for both incoming and
  outgoing forwards.
- **Attach a file from the chat input** (📎): images, video, audio and
  arbitrary documents are auto-classified to the matching native Telegram
  type, with thumbnails / inline players on the receiving side.
- **Captions for attachments**: anything you type while a file is attached
  is delivered as the caption.
- **Drag-and-drop a file into the chat** to attach it (single file; folders
  are rejected).
- **"Send File to Telegram"** in the Project View context menu now actually
  works: opens the chat picker, lands the file in the picked chat's
  attachment preview, and works whether the target chat is currently open
  or not.
- Up-front 2 GB size limit on attachments, so over-sized files fail fast.

### Fixed
- **Notification settings now actually take effect**: the global *Enable
  notifications* toggle is honored, server-side mute (set on another device)
  is picked up on cold start before you ever open the tool window, and
  *Enable sound* now plays a beep on each new-message popup.
- **`Esc` cancels an active reply** while the message input is focused;
  **`Enter` from the reply preview sends** with the captured quote.
- **"Send File" no longer greyed out** in the Project View on IJP 2026.1
  (action update thread + multi-selection support).

## [0.3.11] - 2026-04-27

### Fixed
- **"Send File to Telegram" no longer greyed out in the Project View**:
  declared `ActionUpdateThread.BGT` (required by IJP 2026.1) and added
  multi-selection support — the first non-directory local file in the
  selection is used, and the menu item only shows when at least one file is
  eligible.

## [0.3.10] - 2026-04-27

### Fixed
- **"Send File to Telegram" actually sends now**: the Project View context-menu
  action was a no-op stub. It now activates the IDEGram tool window, opens
  the chat picker, and after a chat is chosen the file is dropped into that
  chat's attachment preview just like the picker / drag-and-drop flows.
  Folders are filtered out; non-local files (e.g. inside JARs) are rejected
  with a notice.

## [0.3.9] - 2026-04-27

### Fixed
- **Notifications respect the global toggle**: turning off **Enable
  notifications** in plugin settings now actually suppresses incoming-message
  popups (was previously ignored).
- **Server-side mute is honored on startup**: chats muted on another device
  are now picked up before the chat list opens — the notification service
  listens to TDLib's chat-list updates directly, so an inbound message on a
  muted chat is silent even if you never opened the IDEGram tool window in
  this session.
- **"Enable sound" actually plays a sound**: when checked, IDEGram now beeps
  (Java AWT) on each new-message popup; when unchecked, popups are silent.

## [0.3.8] - 2026-04-27

### Added
- **Drag-and-drop a file into the chat to attach it**: drop any file from
  Finder/Explorer onto the open chat panel — the panel highlights to confirm
  the drop, and the file appears in the attachment preview just like the
  picker flow. Type a caption and press Send.

### Notes
- Folders are rejected silently (only individual files are attached).
- When multiple files are dropped, the first one is taken; multi-file
  attachments are still on the roadmap.

## [0.3.7] - 2026-04-27

### Added
- **Attach a file from the chat input**: a new **📎** button next to the
  message field opens the standard file picker. Pick any image, video, audio,
  or document — it appears in a preview row above the input, with name, size,
  and a **×** to cancel.
- **Caption support for attachments**: anything you type while a file is
  attached is delivered as the message caption (matching mobile / web /
  desktop Telegram behaviour). Empty caption sends the file alone.
- **Automatic media classification**: images (`jpg/png/webp/gif`), videos
  (`mp4/mov/mkv/webm`), and audio (`mp3/m4a/ogg/flac/wav`) are sent as their
  native Telegram types so they render with thumbnails / inline players on
  the receiving side; everything else is sent as a document.
- **Up-front size validation**: files over 2 GB are rejected before upload
  with a clear error, instead of failing mid-flight.

## [0.3.6] - 2026-04-26

### Added
- **Forward a message to another chat**: right-click any non-service message
  bubble → **Forward**, then pick a destination chat from the picker. The
  message arrives in the target chat with Telegram's standard
  *Forwarded from …* attribution preserved (matching mobile / web / desktop
  clients).
- **Inline “Forwarded from” label** in the chat view: forwarded messages now
  render the original author / channel above the bubble content (italic, blue),
  for both incoming and outgoing forwards.

## [0.3.5] - 2026-04-26

### Added
- **Reply to a message**: right-click any message bubble → **Reply**, type your
  response, and press Enter. The reply is delivered to mobile / web / desktop
  Telegram with the standard reply preview attached.
- **Partial-quote reply**: after picking *Reply*, drag-select any fragment of
  the original inside the reply preview above the input — the selection is
  attached as a `quote` so that mobile / web Telegram lands the recipient on
  exactly that fragment when they tap the preview.
- **Inline reply previews are now clickable**: tapping an inline reply inside
  a message bubble scrolls to and briefly highlights the original. If the
  original is above the currently loaded history window, older messages are
  fetched automatically.
- **Quote-replies are visually distinct in the chat view**: when a reply
  carries a partial-quote, the inline preview inside the bubble shows the
  *quoted fragment* (italic, in curly quotes), matching how mobile / web
  Telegram render quote-replies — instead of falling back to the full original.
- **`Esc` cancels the active reply** while the message input is focused.
- **`Enter` from the reply preview sends the message** with the captured quote
  — no need to click back into the input first.

## [0.2.14] - 2026-04-23
- Build produced by /build command

## [0.2.13] - 2026-04-22
- Build produced by /build command

## [0.2.12] - 2026-04-21

### Fixed
- Chat avatars no longer disappear when opening a user profile and returning
  to the chat (MessageViewModel is now stable across recompositions).
- "Deleted Account" label no longer shown for active users on a cold sender
  cache; transient resolution failures are no longer cached.

## [0.2.11] - 2026-04-20

### Added
- Global shared cache directory: TDLib database and media cache now live under
  `{IDE system path}/idegram/` and are shared across all projects in the same
  IDE installation, instead of per-project `<project>/.idegram/` folders.
- One-time notification informing users about legacy per-project cache folders
  left over from earlier versions (never auto-modified).
- User profile view: tap a sender's avatar or name to open their Telegram
  profile inside the plugin.

### Fixed
- Chat avatars no longer disappear when navigating to a user profile and back.
- "Deleted Account" label no longer shown for active users whose data arrived
  after the first resolution attempt.

## [0.2.10] - 2026-04-20

### Added
- User profile panel reachable from message senders and chat list items.

## [0.2.9] - 2026-04-18

### Added
- Date separators in the message list (Today / Yesterday / weekday / full date).
- Media group (album) layout in the message view.
- Correct sender resolution for messages sent on behalf of chats/channels.

## [0.2.7] - 2026-04-14

### Added
- Upward pagination through full chat history.
- Real unread-count values from TDLib instead of the badge-capped value.
- IDE compatibility extended to builds 253+ (2025.3 and newer).

## [0.2.6] - 2026-04-13

### Fixed
- Miscellaneous chat list and message rendering fixes.

## [0.2.5] - 2026-04-12

### Fixed
- Chat avatar loading stability.

## [0.2.4] - 2026-04-12

### Fixed
- Deleted user display-name regression.

## [0.2.3] - 2026-04-12

### Changed
- Internal: sync of per-chat mute settings.

## [0.2.2] - 2026-04-12

### Added
- Per-chat sender filter.

## [0.2.1] - 2026-04-12

### Added
- Media messages (photos, videos, voice notes, documents).

## [0.1.13] - 2026-04-12

### Fixed
- Startup and session restore reliability improvements.

## [0.1.12] - 2026-04-11

### Changed
- Internal refactors ahead of media support.

## [0.1.2] - 2026-04-11

### Fixed
- Initial public pre-release fixes.

## [0.1.1] - 2026-04-11

### Fixed
- Plugin packaging fixes.

## [0.1.0] - 2026-04-11

### Added
- TDLib core integration with native library loading for macOS, Windows, Linux.
- Telegram authentication with phone number, verification code, and 2FA password.
- Secure session storage using IntelliJ PasswordSafe.
- Tool Window UI with navigation shell (chat list, message view, settings).
- Chat list with real-time updates, search, and unread counters.
- Message view with send, edit, delete, and reaction support.
- Code sharing with syntax highlighting from editor context menu.
- Git diff sharing with scope selection (all / staged / current file).
- Stacktrace sharing from Run/Debug console.
- File attachment sending from Project view.
- Message rendering with formatted text, code blocks, reply previews.
- IDE notifications for new messages with mute support.
- Plugin settings page in IDE Settings with notification and code-sharing preferences.
- Async plugin startup with zero EDT impact.
- Session auto-restore on IDE restart.

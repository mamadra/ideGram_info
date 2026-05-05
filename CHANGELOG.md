# Changelog

All notable changes to the IDEGram plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

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

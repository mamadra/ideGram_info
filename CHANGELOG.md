# Changelog

All notable changes to the IDEGram plugin will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

## [1.3.0] - 2026-05-10

> Headline release of the **encrypted snippet share** feature plus an
> in-chat **link-preview card** and a clean-up of the editor right-click
> menu. Snippets now leave the IDE as a single short URL whose
> ciphertext lives on `idegram.app` and whose AES-256-GCM key never
> reaches the server — paste anywhere, recipients decrypt locally.

### Added

- **Share via IDEGram (061)** — new editor context-menu action that
  encrypts the selection (or whole file when nothing is selected) with
  AES-256-GCM client-side, uploads only the ciphertext + IV +
  metadata to `https://idegram.app/api/snippets`, and builds a
  shareable URL of the form `https://idegram.app/s/<id>#<key>`. The
  decryption key lives in the URL fragment which (per HTTP spec) the
  server never sees — verified by a unit test that scans every
  captured request URL / headers / body for the key string. The URL
  is copied to the clipboard AND opens an in-app **chat picker**
  (mirrors the Send-File flow): pick a chat → message goes through
  `TdApi.SendMessage`, you land on that chat with the message visible.
  Dismiss the picker to share the URL outside Telegram.
- **Snippet privacy toggles** — Settings → Code Sharing now has a
  "Share via IDEGram (encrypted URL)" subsection with two checkboxes:
  *Include filename in metadata* and *Include username in metadata*.
  When OFF, the corresponding field is omitted entirely from the
  unencrypted metadata payload (filename is always inside the
  encrypted blob; only its appearance in the OG card and landing-page
  header is suppressed).
- **License-gating ready** for "Share via IDEGram" — an IntelliJ
  registry key `idegram.snippet.requiresPaid` (default OFF in v1)
  flips the action to Paid-only without re-shipping the plugin. When
  ON, Free / Trial users see an "Upgrade IDEGram" balloon at
  invocation; **no network request is made**. The check consults the
  current tier at click time (mid-session tier changes respected).
- **Endpoint override for local dev** — registry key
  `idegram.snippet.endpoint` (default `https://idegram.app`) lets a
  developer point the plugin at a locally-running `idegram-web`
  instance.
- **Inline link-preview cards** — text messages that contain a URL
  now render TDLib's resolved preview as a card under the bubble,
  same pattern as the official Telegram clients: site name, title,
  description, author, and the OG image when present. Image loads
  via the existing media-download pipeline (fast minithumbnail, then
  full-resolution swap). Clicking anywhere on the card opens the URL
  in the default browser.

### Changed

- **Editor context menu cleanup** — the nested "IDEGram ▶" submenu
  with the three stub actions (Share Code to Telegram / Share Git
  Diff / Share Stacktrace) is gone. Right-click in the editor now
  shows **"Share via IDEGram"** directly. The Project View context
  menu keeps the working **"Send File to Telegram"** entry, also
  flattened (no submenu wrapper).

### Removed

- Stub actions and their orphaned use cases / view models /
  preview dialog: `ShareCodeAction`, `ShareDiffAction`,
  `ShareStacktraceAction`, `ShareCodeUseCase`, `ShareDiffUseCase`,
  `ShareStacktraceUseCase`, `ShareFileUseCase`, `ShareViewModel`,
  `SharePreviewDialog`. None of these had a finished
  ChatPickerDialog wire-up; their replacements ("Share via IDEGram"
  for snippets, "Send File to Telegram" for project-view files)
  are both fully wired.

### Security

- The AES-256-GCM key for "Share via IDEGram" is generated per
  snippet via `SecureRandom` and **never persisted, logged, or sent
  over the wire**. The key is base64url-encoded into the URL
  fragment client-side AFTER the upload completes — a unit test
  scans every captured HTTP exchange (URL + headers + body) for the
  key substring and refuses to merge if it leaks (SC-004).

## [1.0.0] - 2026-05-09

> **First stable release.** Bundles three more major Tier 2 features
> on top of everything in 0.4.0 — block list management, create-chat
> /group/channel, and chat folders. If you're upgrading directly from
> 0.3.22 (the last release What's-New popup may have shown), this
> release contains **thirteen** new top-level capabilities accumulated
> across the entire 0.4 cycle.

### Added

- **Chat folders / global filters (060)** — the folder bar in the chat-list header now shows your user-defined folders the official-client way: emoji icons + per-folder **unread badges** (live from `UpdateUnreadChatCount`) + "All Chats" pinned first. Selection **persists across IDE restarts** (`IDEGramSettingsState.activeFolderId`). Bar auto-hides on search and on accounts with zero folders.
- **Manage Folders panel (060)** — Settings → Manage folders lets you add / rename / delete / reorder folders without leaving the IDE. Add dialog has Name + 21-emoji icon picker + chat-type checkboxes (Contacts / Non-contacts / Bots / Groups / Channels) + exclude flags (Muted / Read / Archived). All changes round-trip via TDLib and reach the user's mobile within ~5 s. Hitting the server-side folder cap (10 for free accounts) surfaces an inline "Folder limit reached" error with the form preserved.
- **Quick folder presets (060)** — Unread / Personal / Groups / Channels / Bots pre-fill the Add-folder form so a common folder is two clicks.
- **Create new chat / group / channel (059)** — `+` icon in the chat-list header opens a popup with three entries. **New group**: two-step dialog (multi-select contact picker → Name + Description); back-nav preserves typed name. **New channel**: single-step form with Public/Private toggle, t.me/-prefixed Username field with live local validation, optional bot-filtered subscribers picker; on a taken username Telegram's failure rolls the freshly-created channel back via `DeleteChat` so no orphan private channel survives. **New private chat**: single-select picker that opens the 1-on-1 chat via the existing `OpenPrivateChatUseCase`.
- **Block list management (058)** — ⚙ Settings entry in the chat-list header opens the plugin Settings panel; from there a "Manage blocked users" panel paginates the user's blocked senders with per-row Unblock buttons. The user-profile overlay gets a Block / Unblock toggle with a confirmation dialog. Eager refresh after own-account actions because TDLib's `UpdateChatBlockList` push doesn't reliably fire for own writes.
- **Chat profile card for groups / supergroups / channels (058 follow-up)** — clicking the title in a non-private chat opens an info card with description, invite link, linked-chat indicator, admin / restricted / banned counts, and the first 50 members (via `GetSupergroupMembers(Recent)` or `GetBasicGroupFullInfo.members`). Each member row shows their avatar, display name, role label (Creator / Admin / Restricted / Banned), Telegram custom title (`ChatMember.tag`), and a Bot pill if applicable. Per-member actions: 💬 Open chat (creates a 1-on-1 chat) and 🚫 Block / Unblock (with inline-toast feedback). Roster hidden for channels and `hasHiddenMembers` supergroups.
- **Channel post comments / discussion threads (057)** — "💬 N Comments" / "Leave a comment" affordance below channel posts with a linked discussion group. Click opens a dedicated thread view pinning the channel post as a header card; comments paginate, real-time-update, and accept replies. Post a comment by typing — Telegram auto-joins the discussion group server-side on first send.
- **Join chat by invite link (056)** — 🔗 icon in the chat-list opens a paste-an-invite dialog supporting four URL formats. Paste → Check shows the chat preview (title, type, member count, description) → Join. **In-message invite-link interception** — clicking a `t.me/+...` URL inside a message opens the same dialog directly, no browser round-trip. Already-a-member detection, admin-approval flow, public-username hint, seven mapped human-readable error messages.
- **Edit caption of sent media (055)** — right-click on your own Photo / Video bubble → "Edit caption" pre-fills the current caption; Enter to save, Esc to cancel. Add a caption to captionless media or clear an existing one by submitting empty text. Telegram's 48-hour edit window is enforced server-side.
- **Notifications per-chat (054)** — right-click a chat in the chat list to open a context menu with four mute presets (1 hour / 8 hours / 1 day / Forever), an Unmute entry, and a Disable / Enable sound toggle. Cross-client sync within ~5 s. Auto-resume on mute expiry — the bell-off indicator clears server-side and the next message produces a normal IDE notification.
- **Schedule / silent / send-when-online (053)** — chevron (▾) next to Send opens a dropdown with three modes. Schedule opens a date/time picker with presets (Tomorrow 9 AM / Next Monday 9 AM) and a custom row; past dates and dates >1 year ahead are rejected inline. Send-when-online queues the message until the recipient's last-seen flips online (1-on-1 private chats only, non-bot). Cmd-Shift-Enter for silent, Cmd-Shift-S for schedule. Multi-attach + voice notes respect the chosen modifier.
- **Forward with options (052)** — right-click → Forward → pick chat → options dialog with Hide sender + Hide caption checkboxes (Hide caption is conditionally hidden when the source has no caption). Failure preserves the dialog with a Retry button.
- **Global search across chats, messages and users (051)** — three-section panel (Chats / Messages / Global) in the chat list. Cmd-K (macOS) / Ctrl-K (other) focuses the search field. Messages section runs Telegram-side full-text search (≥4 chars, 300 ms debounce); click a hit → chat opens scrolled to the exact message with the in-bubble highlight visible. Global section surfaces contacts + public chats deduped against existing chats. `flatMapLatest` cancellation prevents stale flicker under fast typing.
- **Multi-file album send (050)** — drag-and-drop multiple files or multi-select via the attach button. Queue panel shows per-item thumbnails (×, "Clear all", "N items, X MB"). Albums match Telegram's mosaic grid (2–10 photos / videos per bubble). Auto-split at 10 items; mixed types split per Telegram rules (photo+video together, audio alone, document alone, voice never in album). Real-time per-batch upload progress; pause-on-failure with Retry. Hard 2 GB per-file cap.
- **Typing indicator with real names (049)** — replaces "User <id>" with proper display names. Twelve verb-phrasings (recording voice, sending photo, picking sticker, …). Multi-user merge ("<A> and <B> are typing"; "<A>, <B> and N more"). 5-second client-side eviction for stale actors.
- **Custom-emoji reactions (048)** — render and toggle custom-emoji reactions alongside standard emoji reactions. Reactor avatars stack in-bubble. Resolved via the new in-memory `CustomEmojiCache` (sticker files reuse the on-disk cache from feature 043).

### Fixed

- **Freshly-sent media no longer flashes "edited" marker right after
  upload completes.** TDLib's `UpdateMessageContent` push fires for
  many reasons besides user edits (notably to swap local file IDs for
  remote ones); the read-side previously flipped `isEdited=true` on
  every such push. Now the marker flips only on the dedicated
  `UpdateMessageEdited` event that carries the real `editDate`.
- **Outgoing messages no longer get stuck in "Sending" state** in the
  chat history after the upload succeeds. The plugin now subscribes to
  `UpdateMessageSendSucceeded` / `UpdateMessageSendFailed`, swapping
  the temporary message id for the server-assigned permanent one and
  flipping the cached send status accordingly.
- **Chat folders weren't loading in the tool window** for accounts
  with folders configured server-side. TDLib delivers the folder set
  exactly once early in the auth flow via `UpdateChatFolders`, but
  the chat-list view-model is constructed later (when the user opens
  the tool window) — so the push was lost to the SharedFlow's
  `replay = 0` semantics. Folders now flow through a sticky-cache
  StateFlow at the repository level so late-subscribing components
  always see the latest known set.
- **Reactions are no longer dropped silently** — the previous
  implementation scaffolded the field but never populated it from
  `MessageReaction`.

### Known limitations

- Animated WebM custom-emoji and stickers render the static thumbnail.
  Animating them needs a VP9 decoder which Skia bundled with Compose
  Desktop doesn't ship; planned for v2 (either ffmpeg-CLI transcode in
  cache or a bundled libvpx native).
- The reaction picker (long-press / right-click → "React") UI
  affordance is not wired up yet; the underlying picker, "Custom Emoji"
  section and toggle paths are all in place — only the popup host is
  missing.

## [0.4.0] - 2026-05-08

> **Tier 1 parity is complete + Tier 2 work begins.** This release closes
> the last Tier 1 gap (global search) and adds six Tier 2 polish items
> (Forward with options + Schedule/silent send + Notifications per-chat
> + Edit caption of sent media + Join chat by invite link + Channel post
> comments), bundling ten major capabilities developed since the 0.3.22
> release.

### Added

#### Channel post comments / discussion threads (057)
- **"💬 N Comments" affordance** below every channel post in channels
  with a linked discussion group. Reads "💬 Leave a comment" when the
  count is zero. Hidden entirely on channels without a linked discussion
  group, on regular chats, and on the comments themselves.
- **Click the affordance** to navigate to a dedicated thread view that
  pins the channel post as a header card at the top ("Channel: <name>"
  label + post body / media-type placeholder) and lists the comments
  below in chronological order. Title bar reads "Comments — <post
  excerpt>" so the user always knows which thread they're in.
- **Pagination**: scroll up to load older comments, same upward-scroll
  pattern as the regular chat view (feature 028).
- **Post a comment** by typing into the input field and pressing Enter
  — comments dispatch into the linked discussion group with the channel
  post as the thread anchor (TDLib's `MessageTopicThread`). If the user
  is not yet a member of the discussion group, Telegram auto-joins them
  server-side on first comment-send (no extra prompts).
- **Real-time updates**: comments posted by other users appear in the
  open thread view within ~2 seconds.
- **Reply to a specific comment** via right-click → Reply (the existing
  reply mechanism from feature 006 works unchanged inside the thread).
  The "Replying to <author>" header is clickable and scrolls to the
  original comment.
- **Back navigation** returns to the channel chat-view at the same
  scroll position the user was at when they clicked Comments — no
  manual re-scroll needed.
- **Failure UX**: send failures (banned, kicked, slow-mode) surface as
  inline errors with the user's input preserved; the failed message is
  NOT inserted as a fake-sent bubble.
- **Channel post header card is non-interactive** for replies / forwards
  / edits — those actions belong to the channel chat view, not the
  thread view.

#### Join chat by invite link (056)

#### Join chat by invite link (056)
- **🔗 icon next to the chat-list search field** opens a paste-an-invite
  dialog. Paste a Telegram invite URL (`https://t.me/+<hash>`,
  `t.me/+<hash>`, legacy `https://t.me/joinchat/<hash>`, or a bare
  `+<hash>`), click Check, see a preview of the target chat (title,
  type, member count, description), then click Join. The new chat
  appears in the chat list within ~2 seconds and the chat-view
  auto-opens to its message history.
- **In-message invite-link interception**: when a chat message contains
  a `t.me/+...` or `t.me/joinchat/...` URL, clicking the link opens the
  same Join-by-link dialog directly — no browser round-trip — with the
  Check already running. Non-invite URLs (`github.com/...`, public
  `t.me/<username>`, etc.) continue to open in the system browser.
- **Already-a-member detection**: if the user is already in the linked
  chat, the dialog flips its primary button to "Open" and navigates to
  the existing chat instead of issuing a duplicate join request.
- **Admin-approval flow**: chats with admin approval enabled show a
  preview indicator ("Joining requires admin approval") and, after
  Join, a confirmation message ("Your request to join '<title>' was
  sent. You will be added once an admin approves the request."). The
  chat appears in the user's chat list automatically once the admin
  approves.
- **Public-username hint**: pasting a `t.me/<username>` URL surfaces a
  hint pointing the user at the chat-list search instead of attempting
  an invite-link API call (those URLs are not invite links).
- **Failure UX**: malformed / expired / revoked / banned / network-drop
  errors render inline within the dialog with the user's input
  preserved for retry. TDLib error codes are mapped to seven
  human-readable messages ("Invite link is invalid" / "...has expired"
  / "...has been revoked" / "You are banned from this chat" / "Too
  many join attempts; try again later" / "This chat is no longer
  accessible" / "You have joined too many channels…").

#### Edit caption of sent media (055)
- **Right-click on a Photo or Video bubble you sent** → "Edit caption"
  switches the input to caption-edit mode pre-filled with the current
  caption text. Press Enter (or click Save) to confirm; Esc / Cancel
  to bail. The bubble updates in place — same message, no second post —
  and an *"edited"* marker appears under the timestamp.
- **Add a caption to a message that didn't have one**: pick "Edit
  caption" on a captionless photo, type a caption, hit Enter. Works
  for the user's own outgoing media.
- **Clear an existing caption** by submitting empty text — the caption
  row disappears entirely, leaving just the photo or video.
- **Eligibility-gated** entry: appears only on the user's own outgoing
  Photo or Video messages (any sent state). Hidden for incoming media,
  stickers, GIFs, voice notes, video notes, polls, service messages,
  audio / document files (the latter two have no caption surface in
  the plugin's current rendering), and messages still in flight.
- **Placeholder text** differentiates the input mode at a glance —
  *"Edit caption..."* vs *"Edit message..."* vs *"Type a message..."*.
- **Failure UX**: Telegram rejects edits past the 48-hour edit window
  and may restrict edits on certain forwarded messages. When a server
  rejection happens, an inline toast surfaces the failure reason and
  the bubble's existing caption is preserved (no partial state).

### Fixed

- **Freshly-sent media no longer flashes "edited" marker right after
  upload completes.** The TDLib `UpdateMessageContent` push fires for
  many reasons besides user edits (notably to swap local file IDs for
  remote ones after a successful upload); the read-side previously
  flipped `isEdited=true` on every such push. Now the marker flips
  only on the dedicated `UpdateMessageEdited` event that carries the
  real `editDate`.
- **Outgoing messages no longer get stuck in "Sending" state in the
  chat history** after the upload succeeds. The plugin now subscribes
  to `UpdateMessageSendSucceeded` / `UpdateMessageSendFailed`, swapping
  the temporary message id for the server-assigned permanent one and
  flipping the cached send status accordingly. This was previously
  blocking the new context-menu entries (e.g., "Edit caption", "Forward")
  from appearing on freshly-sent media until the chat was reopened.

#### Notifications per-chat (054)
- **Right-click any chat in the chat list** to open a context menu with
  notification controls — four mute presets (*Mute for 1 hour*, *8 hours*,
  *1 day*, *Forever*), an *Unmute* entry, and a *Disable / Enable sound*
  toggle. Two clicks to mute, two clicks to unmute, no need to leave the
  IDE for Telegram Desktop.
- **Cross-client sync**: muting a chat from inside the IDE flips the same
  chat to muted in the user's mobile / desktop / web Telegram within ~5
  seconds — Telegram's server is the source of truth.
- **Conditional menu entries**:
  - When the chat is **unmuted**: the menu shows the four mute presets
    plus a *Disable sound* (or *Enable sound*) toggle. No *Unmute* entry.
  - When the chat is **muted**: the menu shows *Unmute* alongside the
    four mute presets (so re-muting with a different duration is one
    click, not two). The sound toggle is hidden — mute already means
    no sound, so the toggle would be misleading.
- **Auto-resume on mute expiry**: when a finite mute (e.g., 1 hour) runs
  out server-side, the chat-list bell-off indicator clears within
  ~3 seconds and the very next incoming message produces a normal IDE
  notification — no IDE restart, no manual refresh.
- **Failure UX**: a network drop during a mute / unmute / sound action
  preserves the chat's existing state (no partial / optimistic update)
  and surfaces a transient red toast at the top of the chat-list panel
  inviting the user to retry.
- **Builds on feature 022 (sync mute settings)**: the existing read
  pipeline that already honours per-chat mute / sound state for IDE
  notifications now has its write counterpart — closing the loop on
  notification preferences.

#### Schedule send / silent send / send when online (053)
- **Chevron (▾) next to Send** opens a dropdown with three additional send
  modes — *Send without sound*, *Schedule message*, *Send when online*.
  Default Send button click is unchanged: same one-keystroke text dispatch
  as before.
- **Send without sound** dispatches immediately but skips the recipient's
  notification ping (sets TDLib's `disableNotification`). Available in
  every chat type — private, group, supergroup, channel-where-poster,
  Saved Messages, and bot chats.
- **Schedule message** opens a date/time picker with two presets
  (*Tomorrow 9 AM* / *Next Monday 9 AM*) and a custom row (yyyy-MM-dd +
  24-hour HH:mm). Past dates and dates more than 1 year ahead are
  rejected inline; Confirm enables only when the chosen moment is valid.
  After Confirm a transient toast confirms the scheduled time; the
  message dispatches server-side at the chosen moment with TDLib's
  `MessageSchedulingStateSendAtDate`.
- **Send when online** queues the message server-side until the recipient's
  last-seen flips to online (TDLib's `MessageSchedulingStateSendWhenOnline`).
  The option is **only visible** in 1-on-1 private chats with non-bot users —
  hidden in groups, channels, supergroups, Saved Messages, and bot chats
  (Telegram restricts the feature to private user-to-user contexts).
- **Keyboard shortcuts**: *Cmd-Shift-Enter* (macOS) / *Ctrl-Shift-Enter*
  (other) → silent send; *Cmd-Shift-S* / *Ctrl-Shift-S* → opens the
  schedule dialog. Both work without leaving the keyboard.
- **Multi-attach + voice notes** all respect the chosen modifier — silent
  / scheduled / when-online apply uniformly to text, single attachment,
  multi-file album, and voice-note dispatches.
- **Edit-mode safety**: the chevron is hidden when editing an existing
  message — there's no "edit-and-schedule" semantic in Telegram.
- **Failure UX**: a network drop or server rejection preserves the input
  + attachments and surfaces an inline snackbar; nothing is silently lost.

#### Forward with options (052)
- **Right-click → Forward → pick chat → options dialog** appears with two
  checkboxes — *Hide sender* and *Hide caption* — before the actual
  dispatch. Confirm button is autofocused; Enter sends with both
  checkboxes off, preserving today's exact UX with one extra keypress.
- **Hide sender** strips the "Forwarded from <X>" header — the recipient
  sees the message in the user's name as if they wrote it themselves.
  Works for any forwardable message type (text, media, polls, stickers).
- **Hide caption** strips the original caption from forwarded media. The
  toggle is hidden entirely for text-only, voice, video-note, poll,
  sticker and GIF sources where there's no caption to remove.
- **Cancel paths**: Esc / Cancel inside the options dialog returns to the
  chat picker (re-pick destination); Esc / Cancel inside the picker
  aborts the whole forward.
- **Failure handling**: when the dispatch fails (network drop, permission
  denied), the dialog stays open with a red error line + a Retry button
  that re-invokes with the same toggle values. No silent retries.

#### Global search across chats, messages and users (051)
- **Three-section global search** in the chat-list panel — Chats / Messages
  / Global — driven by a single search field above the list. Cmd-K (macOS)
  or Ctrl-K (other) focuses the search field from anywhere in the IDEGram
  tool window; Esc clears it.
- **Chats section** filters the existing list by chat title, contact name
  or username. Local-only, sub-200 ms, works offline.
- **Messages section** runs a Telegram-side full-text search across ALL
  the user's chats (queries ≥ 4 characters, after a 300 ms debounce). Each
  hit shows the chat title, sender display name, a snippet with the
  matched substring highlighted (reuses the in-chat search renderer from
  feature 046), and a relative date. Click a hit → the chat opens scrolled
  to that exact message with the highlight visible in the bubble.
- **Global section** surfaces contacts and public chats matching the
  query, deduplicated against chats already in the user's list. Click on
  a user → private chat opens. Click on a public channel → auto-joins
  (if not yet a member) and opens.
- **Cancellation under fast typing** — every keystroke cancels in-flight
  TDLib calls via `flatMapLatest`, so the user never sees flickering
  results from stale queries.
- Failed message searches surface an inline Retry control; the local
  Chats filter keeps working even when offline.

#### Multi-file album send (050)
- **Attach many files at once** — drag-and-drop a list onto the chat or
  multi-select via the attach button. The queue panel above the input bar
  shows one thumbnail per item (photo via inline decode; ▶ overlay for
  video; 🎵 / 📎 icons for audio / document) with per-item × button, a
  "Clear all" link and a running `N items, X MB` label.
- **Albums match Telegram's mosaic grid** (recipient-side, feature 047) —
  2 to 10 photos / videos send as a single bubble with the caption
  attached to the first item; queue size 1 keeps the existing solo-message
  flow.
- **Auto-split at 10 items** — 25 photos go out as 10 + 10 + 5 albums in
  chronological order, caption only on the first batch. Dispatch is
  strictly sequential (TDLib forbids parallel album sends to the same
  chat).
- **Mixed types split per Telegram rules** — `photo + video` together,
  `audio` alone, `document` alone, voice notes never in album. A mixed
  queue (e.g. 6 photos + 3 audio) sends two bubbles — one mosaic + one
  audio album — preserving order.
- **Real-time upload progress** — each batch transitions
  `Pending → Dispatching → Uploading(N%) → Sent`. Progress is aggregated
  from `UpdateFile` across the batch's files; completion is detected
  authoritatively via `UpdateMessageSendSucceeded`. The panel renders a
  per-batch progress arc + percentage.
- **Pause-on-failure with Retry** — if a batch fails (network drop,
  permission), the FSM pauses, the failed tile gets a Retry control + a
  Cancel link in the panel header. Retry resumes from the failed batch;
  Cancel returns the remaining files to the queue for editing.
- **Hard 2 GB per-file cap** — oversized files in the queue show a red
  ⚠ icon and disable the Send button until removed.

#### Typing indicator (049)
- **Real display names** instead of "User <id>" — typing users now show
  with the same name they use on their messages. "Someone" gracefully
  fills in when the sender is a deleted account; in private chats the
  name is omitted entirely (just "typing…") matching Telegram desktop.
- **Verb-phrasings for 12 chat-action types** — "is recording a voice
  message…", "is sending a photo…", "is picking a sticker…", "is sending
  a file…", and so on; falls back to "is typing…" for unknown action
  types.
- **Multi-user merge in groups** — "<A> and <B> are typing", "<A>, <B>
  and N more are typing", "N people are typing" with a Telegram-style
  cap at 2 visible names. Mixed actions render as "<A> is typing, <B>
  is recording a voice message" for ≤2 actors.
- **5-second client-side timeout** — TDLib emits no explicit stop event
  for typing actions; the indicator auto-evicts stale actors at 1 Hz
  granularity. Self-typing is suppressed; channels and the spec-021
  hidden-sender filter are honoured.

#### Custom-emoji reactions (048)
- **Attach many files at once** — drag-and-drop a list onto the chat or
  multi-select via the attach button. The queue panel above the input bar
  shows one thumbnail per item (photo via inline decode; ▶ overlay for
  video; 🎵 / 📎 icons for audio / document) with per-item × button, a
  "Clear all" link and a running `N items, X MB` label.
- **Albums match Telegram's mosaic grid** (recipient-side, feature 047) —
  2 to 10 photos / videos send as a single bubble with the caption
  attached to the first item; queue size 1 keeps the existing solo-message
  flow.
- **Auto-split at 10 items** — 25 photos go out as 10 + 10 + 5 albums in
  chronological order, caption only on the first batch. Dispatch is
  strictly sequential (TDLib forbids parallel album sends to the same
  chat).
- **Mixed types split per Telegram rules** — `photo + video` together,
  `audio` alone, `document` alone, voice notes never in album. A mixed
  queue (e.g. 6 photos + 3 audio) sends two bubbles — one mosaic + one
  audio album — preserving order.
- **Real-time upload progress** — each batch transitions
  `Pending → Dispatching → Uploading(N%) → Sent`. Progress is aggregated
  from `UpdateFile` across the batch's files; completion is detected
  authoritatively via `UpdateMessageSendSucceeded`. The panel renders a
  per-batch progress arc + percentage.
- **Pause-on-failure with Retry** — if a batch fails (network drop,
  permission), the FSM pauses, the failed tile gets a Retry control + a
  Cancel link in the panel header. Retry resumes from the failed batch;
  Cancel returns the remaining files to the queue for editing.
- **Hard 2 GB per-file cap** — oversized files in the queue show a red
  ⚠ icon and disable the Send button until removed.

### Fixed

- Reactions are no longer dropped silently — the previous implementation
  scaffolded the field but never populated it from `MessageReaction`.

### Known limitations

- Animated WebM custom-emoji and stickers render the static thumbnail.
  Animating them needs a VP9 decoder which Skia bundled with Compose Desktop
  doesn't ship; planned for v2 (either ffmpeg-CLI transcode in cache or a
  bundled libvpx native).
- The reaction picker (long-press / right-click → "React") UI affordance is
  not wired up yet; the underlying picker, "Custom Emoji" section and toggle
  paths are all in place — only the popup host is missing.

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

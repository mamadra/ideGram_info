# IDEGram

**Telegram client for JetBrains IDEs.**

IDEGram is a proprietary plugin that brings a full Telegram messaging experience directly into IntelliJ IDEA, PyCharm, WebStorm, GoLand, and other JetBrains IDEs. It talks to Telegram servers directly from your machine using the official [TDLib](https://core.telegram.org/tdlib) library — no third-party servers, no relay, no backend.

> This repository hosts the **public artifacts** of IDEGram: license, privacy policy, changelog, and the issue tracker. The plugin source code is proprietary and is not published here.

## Install

Install **IDEGram** from JetBrains Marketplace:

- In your IDE: `Settings` → `Plugins` → `Marketplace` → search for **IDEGram**.
- Or from the web: <https://plugins.jetbrains.com/> (listing link will be added once the plugin is published).

**Requirements:**
- JetBrains IDE 2025.3 or newer (builds 253+).
- JVM 21 (bundled with the IDE).
- macOS (Apple Silicon / Intel), Windows x64, or Linux x64.

## Features

- Log in with your Telegram account (phone + code + optional 2FA).
- Browse chats, channels, and groups with real-time updates and unread counters.
- Read, send, edit, delete messages. Reactions, replies, formatted text.
- Photos, videos, voice notes, documents, media groups (albums).
- Share code snippets from the editor with syntax highlighting preserved.
- Share `git diff` output and stacktraces from the Run/Debug console.
- Attach files from the Project view.
- Native IDE notifications for new messages, with per-chat mute respected.
- User profile view, chat folders, sender filter, date separators in message history.
- Upward pagination through full message history.
- Settings page integrated into the IDE's Settings dialog.

## Privacy

IDEGram does **not** operate any backend server, does **not** collect telemetry, and does **not** transmit your data to the developer.

- Credentials live in the OS keychain via JetBrains PasswordSafe.
- Chats, messages, and media cache live in your IDE system directory (`{system path}/idegram/`), shared across projects on the same IDE installation.
- Network traffic goes directly from your machine to Telegram servers over TDLib's encrypted MTProto 2.0 connection.

See [PRIVACY.md](PRIVACY.md) for full details.

## License

IDEGram is distributed under a proprietary End User License Agreement. See [LICENSE.md](LICENSE.md) for the full text.

By installing or using the plugin you agree to be bound by the EULA. Key points:

- Perpetual or subscription license, depending on how the plugin is distributed on JetBrains Marketplace.
- No reverse engineering, redistribution, or sublicensing.
- Provided "as is", without warranty.
- Governed by the laws of the Czech Republic.

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for the version history.

## Reporting Issues

Found a bug or want to request a feature? Please open an issue in the [Issues](../../issues) tab of this repository.

When reporting a bug, include:
- IDE name and version (e.g. IntelliJ IDEA 2025.3.1).
- Operating system.
- IDEGram version.
- Steps to reproduce.
- Any relevant IDE log excerpts (with Telegram credentials redacted).

## Third-Party Components

IDEGram uses:
- [TDLib](https://core.telegram.org/tdlib) — official Telegram client library.
- [IntelliJ Platform SDK](https://plugins.jetbrains.com/docs/intellij/welcome.html) and [Jewel](https://github.com/JetBrains/jewel) — by JetBrains s.r.o.
- [Kotlin](https://kotlinlang.org/) and [kotlinx.coroutines](https://github.com/Kotlin/kotlinx.coroutines) — by JetBrains s.r.o.

See the plugin distribution for the full list of open-source notices.

## Trademarks

**IDEGram** is not affiliated with, endorsed by, or sponsored by Telegram Messenger Inc. or JetBrains s.r.o.
"Telegram" and the Telegram logo are trademarks of Telegram Messenger Inc. "JetBrains" and related marks are trademarks of JetBrains s.r.o.

## Contact

- Developer: Dmitrii Kosarev
- Email: mamadra56789@gmail.com

---

© 2026 IDEGram Developer. All rights reserved.

# IDEGram Privacy Policy

**Effective date:** 2026-04-20
**Version:** 1.0

This Privacy Policy describes how the IDEGram plugin (the "Software", "plugin", "we") handles information when You use it as a Telegram client inside a JetBrains IDE. IDEGram is developed and maintained by Dmitrii Kosarev ("we", "us", "Developer").

IDEGram is not affiliated with Telegram Messenger Inc. or JetBrains s.r.o.

## Summary (TL;DR)

- IDEGram is a Telegram client running locally inside Your JetBrains IDE. It talks directly to **Telegram servers** on Your behalf using the official TDLib library.
- The Developer **does not operate any backend server**. The Developer **does not collect, receive, or store** Your messages, contacts, credentials, or any personal data.
- All Your Telegram data is stored **locally on Your machine** (credentials in the OS keychain via JetBrains PasswordSafe; chats/messages/files in Your user-specific IDE system directory).
- The Software **does not include analytics, telemetry, or crash reporting** in this version. If this ever changes, this policy will be updated and opt-in consent will be requested before any data leaves Your machine.
- JetBrains Marketplace collects its own installation and usage statistics independently of the Software; that data is governed by the JetBrains Privacy Policy (https://www.jetbrains.com/legal/docs/privacy/privacy.html).

## 1. Who is the data controller?

For any Personal Data processed locally on Your machine through the Software, **You are the controller** of Your own data. The Developer does not receive or process such data and acts neither as controller nor as processor with respect to it.

For inquiries about this policy, contact:

- Developer: Dmitrii Kosarev
- Contact email: mamadra56789@gmail.com

## 2. What data the Software processes and where it lives

### 2.1. Telegram authentication credentials

When You log in to Telegram through IDEGram, the following are needed:

- Telegram **API ID** and **API Hash** (obtained by You from https://my.telegram.org);
- Your **phone number** (entered by You);
- Verification **code** sent by Telegram via SMS or Telegram message;
- Optionally, Your **two-factor authentication password**.

**Transmission:** phone number, verification code, and 2FA password are sent **directly from Your machine to Telegram servers** over TDLib's encrypted connection. They are **not transmitted to the Developer** and are **not stored on any Developer infrastructure**.

**Local storage:** API ID, API Hash, and Your Telegram session key are stored locally on Your machine in **JetBrains PasswordSafe**, which uses Your operating system's native credential store (macOS Keychain, Windows Credential Manager, Linux Secret Service / KWallet, or an encrypted file fallback).

### 2.2. TDLib local database (chats, messages, contacts, media)

TDLib — the official Telegram library bundled with the Software — maintains an **encrypted local database** of Your Telegram account, including chat list, message history, contacts, and downloaded media thumbnails.

**Location:** `{IDE system path}/idegram/tdlib/`, where `{IDE system path}` is the directory returned by `com.intellij.openapi.application.PathManager.getSystemPath()` (typically `~/Library/Caches/JetBrains/<product>/` on macOS, `%LOCALAPPDATA%\JetBrains\<product>\` on Windows, `~/.cache/JetBrains/<product>/` on Linux).

**Access:** this data never leaves Your machine through the Software. It is accessed only by TDLib and the plugin code running inside Your IDE.

### 2.3. Downloaded media files

Photos, videos, voice notes, documents, and other files You open in the plugin are downloaded by TDLib on demand and cached under:

- `{IDE system path}/idegram/files/`

These files are **stored locally** and are not transmitted anywhere by the Software.

### 2.4. Plugin settings and UI preferences

Non-sensitive plugin settings (e.g., selected chat folder, whether the one-time legacy-cache-folder notification was dismissed) are stored in the standard IntelliJ `PersistentStateComponent` mechanism, which writes to Your IDE's configuration directory. No personal data is included in these settings.

### 2.5. Data sent to Telegram

When You use the plugin to send messages, reactions, or other Telegram actions, the relevant payload is sent **directly to Telegram servers** via TDLib. Telegram's handling of that data is governed by the Telegram Privacy Policy (https://telegram.org/privacy) and the Telegram API Terms of Service (https://core.telegram.org/api/terms), both of which apply in addition to this Policy.

## 3. Data the Developer does NOT collect

The current version of the Software **does not**:

- send any telemetry or usage analytics to the Developer or any third party;
- send crash reports, logs, or diagnostic information off Your machine;
- embed any third-party advertising, tracking, or profiling SDK;
- upload Your chats, contacts, credentials, or media to any Developer-controlled server;
- maintain any user account on Developer-controlled infrastructure (there is no Developer backend).

If a future version introduces any form of telemetry or crash reporting, that feature will:

(a) be **disabled by default** (opt-in);
(b) be documented in an updated version of this policy **before** activation;
(c) collect only anonymous, non-sensitive data required to diagnose crashes or improve reliability;
(d) never include message contents, contact information, credentials, or personal identifiers.

## 4. Third parties

The Software depends on the following third parties, each of which has its own privacy practices:

- **Telegram Messenger Inc.** — provides the messaging service You connect to. See https://telegram.org/privacy.
- **JetBrains s.r.o.** — provides the IDE, Marketplace, and credential-storage infrastructure. See https://www.jetbrains.com/legal/docs/privacy/privacy.html.

The Developer is not responsible for the privacy practices of these third parties.

## 5. Security

- Credentials are stored in Your operating-system credential store via JetBrains PasswordSafe.
- Network traffic to Telegram is encrypted end-to-end between TDLib on Your machine and Telegram servers, using the MTProto 2.0 protocol implemented by TDLib.
- The local TDLib database is stored on Your filesystem without additional application-level encryption beyond TDLib's own key material; accordingly, full-disk encryption on Your machine is strongly recommended.
- You are responsible for the physical and operating-system-level security of the machine on which IDEGram is installed.

## 6. Data retention and deletion

- **Credentials:** remain in PasswordSafe until You log out of the plugin or explicitly delete them.
- **TDLib database and media cache:** remain in `{IDE system path}/idegram/` until You delete them.
- **Logout:** the plugin's logout action removes the session and clears the TDLib database created by the plugin.
- **Full wipe:** to completely remove IDEGram data, log out and then delete the directory `{IDE system path}/idegram/`. You may also delete any legacy `<project>/.idegram/` directories from IDE versions predating the shared-cache migration.

Because the Developer holds no copy of Your data, the Developer cannot delete anything on Your behalf.

## 7. Your rights under GDPR / CCPA

Because the Developer does not collect or process Your Personal Data, data-subject rights under the General Data Protection Regulation (GDPR), the California Consumer Privacy Act (CCPA), and similar laws primarily apply to the entities that do:

- for data held on Your machine — You control it directly by deleting the files described in Section 6;
- for data held by **Telegram** — exercise Your rights through Telegram (https://telegram.org/privacy);
- for data held by **JetBrains** (account, Marketplace activity) — exercise Your rights through JetBrains (https://www.jetbrains.com/legal/docs/privacy/privacy.html).

If You believe the Developer nonetheless holds Personal Data about You, contact us at the email in Section 1 and we will respond within a reasonable time and, in any event, within the timeframe required by applicable law.

## 8. Children

The Software is not directed at children under the age of 13 (or the relevant minimum age in Your jurisdiction). The Developer does not knowingly collect data from children. Telegram's own minimum-age requirements apply to use of the Telegram service through the Software.

## 9. International transfers

The Software processes data **locally on Your machine**. The Developer does not perform cross-border transfers of personal data. Telegram and JetBrains may transfer data across borders under their own policies linked above.

## 10. Changes to this policy

The Developer may update this Privacy Policy from time to time. The "Effective date" and "Version" at the top of this document will reflect the date of the latest revision. Material changes will be announced in the plugin's changelog and on the JetBrains Marketplace listing. Continued use of the Software after a revision constitutes acceptance of the revised policy.

## 11. Contact

Questions about this Policy, or about data handling by the Software, may be sent to:

- Developer: Dmitrii Kosarev
- Contact email: mamadra56789@gmail.com

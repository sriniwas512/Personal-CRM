**Created by Sriniwas Ghate**

# Personal CRM

A private, encrypted personal CRM that stores your data in your own GitHub account.
No servers. No subscriptions. No data harvesting.

![Security](https://img.shields.io/badge/Encryption-AES--256--GCM-9ece6a?style=for-the-badge)
![Tech](https://img.shields.io/badge/Stack-Vanilla%20JS-f7768e?style=for-the-badge)
![Theme](https://img.shields.io/badge/Theme-Tokyo%20Night-7aa2f7?style=for-the-badge)

## How It Works

1. You authenticate with a [GitHub Personal Access Token](https://github.com/settings/tokens/new?scopes=gist&description=Personal+CRM) (only the `gist` scope is needed).
2. Your data is encrypted locally in the browser using your PAT as the key.
3. The encrypted blob is stored as a secret Gist on your GitHub account.
4. On login, the Gist is fetched and decrypted locally. No plaintext ever leaves your browser (unless you use **Export for Claude**, see below).

There is no backend. The app is a single HTML file.

> **⚠ Your token is your encryption key.** If the token expires or you revoke it, a new token **cannot** decrypt your existing data, and the Gist's revision history won't help because every version was encrypted with the old token. GitHub sets an expiry date on new tokens by default. Check yours at [github.com/settings/tokens](https://github.com/settings/tokens), and download a `.md` backup regularly.

## Features

- **Encrypted sync** — AES-256-GCM encryption with PBKDF2 key derivation (200k iterations). Data is encrypted before it leaves your browser.
- **Contacts management** — One Contacts tab. Each contact is labelled Professional or Personal (filter by type, or change it in the contact's pop-up), with a dated note log and a "next contact" date. Last Contact follows the newest note automatically.
- **Reminders** — Reminders with a date, an optional time (Singapore time) and a message, optionally linked to a contact — e.g. "Thursday 10:00, message Alice about the proposal". Add them in the Reminders tab or in a contact's pop-up, with one-click Today / Tomorrow / Mon–Sun buttons. Each contact's next reminder shows in the table, and the Reminders tab shows a badge for reminders due today or overdue.
- **Calendar** — A month view (weeks start Monday, today highlighted in Singapore time) showing each day's reminders and Next Contact dates. Click a day to add a reminder on that date; click an item to open it; "+N more" lists the whole day. On a phone, tapping a day shows everything on it with an "Add reminder on this day" button.
- **Add to calendar** — Downloads a calendar file (`.ics`) of upcoming Next Contact dates (all-day, alert at 9 am) and open reminders (at their Singapore time, with an alert) for Google, Apple or Outlook Calendar. It's a snapshot: download it again after changes. Events keep the same ids, so calendars that support it update them instead of adding duplicates.
- **Notes diary** — A general-purpose dated journal, sorted newest-first.
- **To Do cards** — Dated cards with checklists.
- **Filter, search, sort** — Find contacts by name, institution, region, or note content.
- **Singapore time** — "Today", note dates, reminder times, overdue status and the 28-day "next contact" default all use Singapore time (UTC+8), whatever time zone your device is set to.
- **Markdown export** — Download your entire database as a structured `.md` file.
- **Export for Claude / Import** — Download an unencrypted JSON copy for Claude to edit, then import it back. Imports are validated, show exactly what will be added, changed or deleted, and need confirmation before anything is saved.
- **Auto-logout** — Signs you out after 3 hours of inactivity, with a warning 1 minute before.
- **Conflict protection** — Saving pauses instead of overwriting if your CRM was saved from another device or tab in the meantime.
- **Tokyo Night theme** — Dark UI with the Tokyo Night color palette.

## Security Architecture

### Encryption

| Property | Value |
|---|---|
| Algorithm | AES-256-GCM |
| Key Derivation | PBKDF2, 200,000 iterations, SHA-256 |
| Salt | Random 16 bytes, regenerated per save |
| IV | Random 12 bytes, regenerated per save |
| Integrity | GCM provides authenticated encryption — tampered ciphertext fails to decrypt rather than producing garbled output |

The encryption key is derived from your GitHub PAT. A different PAT produces a different key, so decryption fails cleanly — no partial data is exposed.

### Data Isolation

- Gist access is enforced by GitHub's permission model. A PAT can only list and read gists belonging to its own account.
- The Gist is created as a **secret** (unlisted) Gist. It doesn't appear in public listings, but anyone who has its exact link can download the encrypted file.
- Even if the raw ciphertext were accessed, it cannot be decrypted without the correct PAT.

### Session & Memory

- The PAT is held only in the page's memory. It is never written to `localStorage`, `sessionStorage` or cookies, so refreshing or closing the tab signs you out.
- **Auto-logout:** after **3 hours** without mouse, keyboard, scroll or touch activity you are signed out, with a warning 1 minute before. Inactivity is measured against the clock, so it also applies when a laptop wakes from sleep: the first touch after 3 idle hours signs you out rather than extending the session. If changes are still waiting to save, the app tries once more before signing out.
- **On sign-out** the app drops its copy of your data, the token and any pending saves, and clears the page including any open pop-up. JavaScript can't overwrite memory directly, so the browser frees the data on its own schedule.
- No data is rendered to the page until authentication succeeds and decryption completes.
- **Decryption Safeguard:** If decryption fails (wrong token or modified Gist), the app refuses to load, so an empty CRM can never overwrite your real data.
- **Unsaved-changes warning:** the browser asks before you close the tab, and the app asks before you sign out, if changes haven't reached GitHub yet.

### Content Security Policy & XSS Prevention

The Content Security Policy blocks external scripts, external images, fonts, forms and plugins, and only allows network requests to GitHub. It does allow inline scripts (`'unsafe-inline'`), because the app is a single HTML file, so **the CSP alone would not stop injected code**. The real defence is in the app itself:

- Every piece of your data shown on the page goes through an escaping function (`esc()`), so text such as `<script>` in a note is displayed, not run.
- Record ids may only contain letters, digits, `-` and `_`, and dates must be `YYYY-MM-DD`. Imported files that break these rules are rejected outright, and odd values coming from the Gist are replaced with safe ones.
- The app refuses to run inside another website's frame, so a hostile page can't overlay it to trick clicks or keystrokes.

### API Reliability

- **Pagination:** The app pages through the GitHub API (`per_page=100`) to find the CRM Gist regardless of how many other Gists you host.
- **No stale reads:** GitHub requests bypass the browser cache, so the app always sees the latest saved version.
- **Conflict protection:** Before every save, the app checks that the Gist hasn't been saved from somewhere else since this page last loaded or saved it. If it has, saving pauses and you choose: save this page's version to a file, load the latest version (then use **Import** to bring your changes back in, seeing exactly what changes), or overwrite.
- **Retries:** If a save fails (e.g. you're offline), the badge shows "Sync Failed — retrying" and the app retries every 5 seconds. A save whose reply was lost but which actually reached GitHub is recognised and not reported as a conflict.
- **One save at a time:** Edits are saved 0.4 s after the last change. Saves never overlap, and a save started before signing out never writes after it.

### Threat Model

**What this protects against:**

| Threat | Mitigation |
|---|---|
| Gist contents exposed (e.g. GitHub breach) | Data is AES-256-GCM encrypted; ciphertext alone is not useful |
| Another user enters their own PAT | GitHub API scopes access to that user's gists only; encryption keys differ |
| Casual inspection of the page source | No user data exists in the HTML. Data is fetched and decrypted at runtime |
| Script injection via your data or an imported file | All data is escaped before display; imports with unsafe ids or dates are rejected |
| Another site framing the app (clickjacking) | The app refuses to run inside a frame |
| Two devices editing at once | Saving pauses instead of overwriting the other device's changes |
| Leaving the app open unattended | Auto-logout after 3 hours of inactivity |

**What this does NOT protect against:**

| Threat | Why |
|---|---|
| Expired or revoked token | The token is the encryption key. A new token can't decrypt old data. Keep `.md` backups |
| Compromised browser or device | If your device is compromised, all bets are off — this applies to any client-side application |
| Malicious browser extensions | Extensions with broad permissions can read page content after decryption |
| Stolen PAT | Anyone with your PAT can authenticate as you and decrypt your data. **Treat your PAT like a password.** |
| Keylogger on the device | A keylogger can capture your PAT as you paste it |
| Leftover export files | **Export for Claude** and **Download .md** files are not encrypted. Delete them when you're done |
| Calendar events | **Add to calendar** puts contact names and reminder text into your calendar app, unencrypted, and synced wherever that calendar syncs |

> **Security depends on safeguarding your GitHub Personal Access Token. Treat it like a password. Do not share it, do not commit it to a repo, and revoke it immediately if you suspect it has been exposed.**

## Quick Start

1. Host `index.html` anywhere — GitHub Pages, Vercel, or open it locally as a file.
2. Generate a [GitHub PAT](https://github.com/settings/tokens/new?scopes=gist&description=Personal+CRM) with only the `gist` scope. Note its expiry date (see the warning above).
3. Paste the token into the login screen.
4. Your data auto-syncs to a secret, encrypted Gist on your account.

## Storage

The practical limit is about **10 MB** of encrypted data: above that, GitHub no longer serves the Gist file to the app. A contact with detailed notes is roughly 500–1,000 bytes, and encryption plus Base64 encoding adds about a third, so that's several thousand contacts' worth of notes.

The app warns you when the encrypted data passes 8 MB. If that happens, use **Download .md** to keep an archive and trim old notes.

Storage via GitHub Gists is **free** with no recurring cost. Because Gists are backed by Git, you can also restore an earlier version of your data from the Gist's revision history (as long as you still have the token it was encrypted with).

## Built With

- Single-file HTML/JS (no frameworks, no build step)
- Vanilla CSS (Inter font stack)
- Web Crypto API (SubtleCrypto)
- GitHub Gists API

---
*Your data. Your key. Your infrastructure.*

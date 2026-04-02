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
3. The encrypted blob is stored as a private Gist on your GitHub account.
4. On login, the Gist is fetched and decrypted locally. No plaintext ever leaves your browser.

There is no backend. The app is a single HTML file.

## Features

- **Encrypted sync** — AES-256-GCM encryption with PBKDF2 key derivation (200k iterations). Data is encrypted before it leaves your browser.
- **Contacts management** — Separate tabs for personal and professional contacts with dated note logs, inline editing, and "next contact" reminders.
- **Notes diary** — A general-purpose dated journal, sorted newest-first.
- **Filter, search, sort** — Find contacts by name, institution, region, or note content.
- **Markdown export** — Download your entire database as a structured `.md` file.
- **Auto-logout** — 10-minute inactivity timer clears session and wipes in-memory data.
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
- The Gist is created as **private**. Unauthenticated requests return `404`.
- Even if the raw ciphertext were accessed, it cannot be decrypted without the correct PAT.

### Session & Memory

- The PAT is stored in `sessionStorage` (cleared when the tab closes — not persisted to disk).
- A 10-minute inactivity timer triggers logout, which zeros the in-memory contacts object, clears the session token, and replaces the DOM with the login screen.
- No data is rendered to the DOM until authentication succeeds and decryption completes. Without a valid token, the page contains only the login form and empty state.

### Content Security Policy

A strict CSP restricts the page to `self`-origin resources, inline styles/scripts, and the GitHub API. No external scripts, no external image loads, no data exfiltration vectors via resource injection.

### Threat Model

**What this protects against:**

| Threat | Mitigation |
|---|---|
| Gist contents exposed (e.g. GitHub breach) | Data is AES-256-GCM encrypted; ciphertext alone is not useful |
| Another user enters their own PAT | GitHub API scopes access to that user's gists only; encryption keys differ |
| Casual inspection of the page source | No user data exists in the HTML. Data is fetched and decrypted at runtime |
| XSS / script injection | Strict Content Security Policy blocks unauthorized scripts |
| Leaving the app open unattended | Auto-logout after 10 minutes of inactivity, memory wiped |

**What this does NOT protect against:**

| Threat | Why |
|---|---|
| Compromised browser or device | If your device is compromised, all bets are off — this applies to any client-side application |
| Malicious browser extensions | Extensions with broad permissions can read page content after decryption |
| Stolen PAT | Anyone with your PAT can authenticate as you and decrypt your data. **Treat your PAT like a password.** |
| Keylogger on the device | A keylogger can capture your PAT as you paste it |

> **Security depends on safeguarding your GitHub Personal Access Token. Treat it like a password. Do not share it, do not commit it to a repo, and revoke it immediately if you suspect it has been exposed.**

## Quick Start

1. Host `index.html` anywhere — GitHub Pages, Vercel, or open it locally as a file.
2. Generate a [GitHub PAT](https://github.com/settings/tokens/new?scopes=gist&description=Personal+CRM) with only the `gist` scope.
3. Paste the token into the login screen.
4. Your data auto-syncs to a private, encrypted Gist on your account.

## Built With

- Single-file HTML/JS (no frameworks, no build step)
- Vanilla CSS (Inter font stack)
- Web Crypto API (SubtleCrypto)
- GitHub Gists API

---
*Your data. Your key. Your infrastructure.*

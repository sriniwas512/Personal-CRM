**Created by Sriniwas Ghate**

# 🗄️ Personal CRM (Tokyo Night Edition)

A high-performance, private, and exceptionally secure Personal Relationship Manager (CRM) built for individuals who prioritize data ownership. This is a "Bring Your Own Storage" application that uses your GitHub account as a secure, encrypted database.

![Aesthetic](https://img.shields.io/badge/Aesthetic-Tokyo%20Night-7aa2f7?style=for-the-badge)
![Security](https://img.shields.io/badge/Security-AES--256--GCM-9ece6a?style=for-the-badge)
![Tech](https://img.shields.io/badge/Tech-Vanilla%20JS-f7768e?style=for-the-badge)

## ✨ Features

- **🌙 Tokyo Night Aesthetics**: A visually stunning dark theme inspired by the color palette.
- **📜 Dated Note Logs**: Instead of a single text field, every contact has a structured log of dated entries with inline editing support.
- **🔒 Zero-Knowledge Privacy**: Your data is encrypted locally using **AES-256-GCM** before ever leaving your browser.
- **☁️ GitHub Gist Sync**: Uses your personal GitHub account to store data. No 3rd-party servers are involved.
- **👥 Multi-User Safe**: Designed so different users can use the same deployed URL with their own tokens without data crossover.
- **📊 Management Tools**:
    - **Filter & Search**: Quickly find contacts by name, institution, or note content.
    - **Smart Sorting**: Arrange lists by name, company, or upcoming contact dates.
    - **Export**: Download your entire database as a beautifully structured `.md` (Markdown) tree file.
- **⏰ Smart Reminders**: Automatically tracks "Next Contact" dates (defaulting to 28 days) with overdue indicators.
- **🛡️ Advanced Security**:
    - **Strict CSP**: Content Security Policy blocks any external script or data leakage.
    - **Auto-Logout**: Automatically clears session data after 10 minutes of inactivity.

## 🚀 How to Use

1.  **Deploy**: Simply host the `index.html` file anywhere (GitHub Pages, Vercel, or even locally).
2.  **Authenticate**:
    - Generate a [GitHub Personal Access Token (Classic)](https://github.com/settings/tokens/new?scopes=gist&description=Personal+CRM).
    - Enable **only** the `gist` scope.
3.  **Sign In**: Paste your token into the app.
4.  **Manage**: Your data will be automatically saved to a private, encrypted Gist on your GitHub account.

## 🛡️ Security Architecture

Your data is protected by **four independent layers** of security. Even if one layer were somehow compromised, the others still keep your data safe.

### 🔐 Layer 1: Data Isolation (GitHub API)

| Concern | Protection |
|---|---|
| **Can another user see my data?** | **No.** When any user enters their PAT, the GitHub API only returns gists owned by *that* token's account. It is physically impossible for User B's token to list or access User A's gists. |
| **Can someone guess my Gist URL?** | Even if they did, the Gist is **private** and requires authentication. Without your PAT, GitHub returns `404 Not Found`. |

### 🔑 Layer 2: Military-Grade Encryption

Even if someone obtained the raw Gist file directly, they would see only encrypted gibberish.

| Property | Value |
|---|---|
| **Algorithm** | **AES-256-GCM** (used by governments and banks worldwide) |
| **Key Derivation** | **PBKDF2** with **200,000 iterations** + SHA-256 |
| **Salt** | Random **16 bytes**, regenerated on every save |
| **IV (Nonce)** | Random **12 bytes**, regenerated on every save |
| **Integrity** | GCM mode provides **authenticated encryption** — any tampering with the ciphertext causes decryption to fail entirely, not produce garbled data |
| **Key Source** | Your GitHub PAT is the key. A different PAT = a completely different encryption key = **decryption fails with an error** |

### 🖥️ Layer 3: DOM & Inspect Element Protection

| Concern | Protection |
|---|---|
| **Can someone use Inspect Element to bypass the login?** | **No.** The page starts with a completely **empty** `<div id="root">`. Your contact data does not exist in the DOM, in JavaScript memory, or anywhere on the page until *after* authentication succeeds AND encrypted data is fetched and decrypted. Bypassing the login wall visually shows a blank, empty application — there is nothing to steal. |
| **Is data hardcoded anywhere?** | **No.** All data lives exclusively in an encrypted GitHub Gist. The HTML file contains zero user data. |

### 🕐 Layer 4: Session & Memory Security

| Feature | Details |
|---|---|
| **Session Storage** | Your PAT is stored in `sessionStorage`, which is **wiped the moment you close the tab**. It is not persisted to disk. |
| **Auto-Logout** | A **10-minute inactivity timer** automatically logs you out if you walk away from your computer. |
| **Memory Wipe on Logout** | On logout, the in-memory `contacts` object is explicitly zeroed (`{ personal:[], professional:[], notesDiary:[] }`), the DOM is replaced with the login screen, and the session token is removed. No residual data remains. |
| **Content Security Policy (CSP)** | A strict CSP header blocks any external scripts, inline injection, or data exfiltration attempts (XSS protection). |
| **Transport** | All API calls use **HTTPS** via the official GitHub API. No data is ever sent to any third-party server. |

### 🧪 Summary

```
Your PAT ──► PBKDF2 (200k iterations) ──► AES-256-GCM Key
                                              │
Your Data ──► Encrypt with Key ──► Base64 ──► GitHub Gist (private, encrypted)
                                              │
On Login  ──► Fetch Gist ──► Decrypt ──► Display in browser (memory only)
On Logout ──► Wipe memory ──► Clear session ──► Empty DOM
```

> **Bottom line:** No other user can see your data. No one can bypass the login wall. Even if someone directly inspected the raw Gist on GitHub, they'd see only encrypted ciphertext that is computationally infeasible to crack without your exact PAT.

## 🛠️ Built With

- Pure HTML/JavaScript (No frameworks/bloat)
- Vanilla CSS (Inter Font Stack)
- Web Crypto API
- GitHub Gists API

---
*Built for privacy. Owned by you.*

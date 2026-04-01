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

## 🚀 How to Use

1.  **Deploy**: Simply host the `index.html` file anywhere (GitHub Pages, Vercel, or even locally).
2.  **Authenticate**:
    - Generate a [GitHub Personal Access Token (Classic)](https://github.com/settings/tokens/new?scopes=gist&description=Personal+CRM).
    - Enable **only** the `gist` scope.
3.  **Sign In**: Paste your token into the app.
4.  **Manage**: Your data will be automatically saved to a private, encrypted Gist on your GitHub account.

## 🛡️ Technical Security

- **Encryption**: Uses the browser's native Web Crypto API.
- **Key Derivation**: Your GitHub PAT is passed through **PBKDF2** (200,000 iterations + random salt) to derive the encryption key.
- **Transport**: All operations occur via the official GitHub API over HTTPS.
- **Memory**: Tokens are stored in `sessionStorage`, meaning they are wiped the moment you close the tab.

## 🛠️ Built With

- Pure HTML/JavaScript (No frameworks/bloat)
- Vanilla CSS (Inter Font Stack)
- Web Crypto API
- GitHub Gists API

---
*Built for privacy. Owned by you.*

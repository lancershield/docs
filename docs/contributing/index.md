# Contributing to LancerShield Docs

Thank you for your interest in contributing to the LancerShield open documentation initiative!

This project is designed to create a structured, trustworthy, and community-driven resource for smart contract auditing knowledge.

---

## ✍️ What You Can Contribute

- 🐞 Add new vulnerabilities with descriptions, examples, causes, and fixes.
- 🛠️ Suggest better secure coding patterns or best practices.
- 📚 Report errors, outdated practices, or unclear explanations.

---

## 📁 Folder Structure

All docs live under the `docs/` directory.

- `severity-framework/` - scoring system and color-coded severity breakdown
- `vuln-db/` - vulnerability entries grouped by severity and category
- `contributing/` - this guide and submission instructions

---

## 🧾 Vulnerability Entry Format

Each entry should include:

- A short **title**
- Assigned **severity level** (using LSF)
- Clear **description** of the vulnerability
- Vulnerable **code snippet** (before)
- **Exploit scenario** (how it can be abused)
- A fixed **code snippet** (after)
- Best practice advice or links (optional)

Example filename:  
`vuln-db/critical/reentrancy-classic.md`

---

## 📤 How to Submit

1. Fork this repo
2. Create a new branch
3. Add or edit Markdown files
4. Open a pull request (PR)
5. Use our PR template and explain your changes clearly

---

## ✅ Contribution Standards

- Be concise and specific.
- Prefer real-world examples (from public exploits).
- Follow the folder structure and formatting style.
- Credit original sources if referencing external material.
- ⚠️ The LancerShield Severity Framework (LSF) is a core scoring protocol and cannot be modified via direct pull requests.  
  If you'd like to suggest improvements to the LSF, please open a GitHub Issue instead.

---

## 🤝 Join the Mission

We believe security knowledge should be accessible, auditable, and extensible. Help us raise the bar for smart contract safety — one documented vulnerability at a time.

_- The LancerShield Team_

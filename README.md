# LancerShield Documentation

Welcome to the official open-source documentation for **LancerShield** — the AI-powered smart contract auditing and optimization platform.

This repository powers the public knowledge base at [lancershield.github.io/lancershield-docs](https://lancershield.github.io/lancershield-docs), and contains:

- ✅ The **LancerShield Severity Framework (LSF)** — used to rank vulnerabilities in a consistent and explainable way.
- 🔐 A curated **Vulnerability Database** — real-world issues, code patterns, and best practices.
- 🤝 A **Contributing Guide** — help us expand the database with structured community submissions.

---

## 📦 Project Structure

```
docs/
├── index.md                       # Landing page
├── severity-framework/           # LSF with scoring and color codes
├── vuln-db/                      # Structured vulnerability entries
├── contributing/                 # Contribution instructions
mkdocs.yml                        # Configuration for MkDocs Material
```

---

## 🛠️ Run Locally

To preview this documentation site locally:

1. Install MkDocs + Material theme

```bash
pip install mkdocs mkdocs-material
```

2. Start the local server

```bash
mkdocs serve
```

Then visit: `http://127.0.0.1:8000`

---

## 📢 Contribute

We're actively seeking contributors to help grow the open vulnerability knowledge base.

- Submit new vulnerabilities via pull requests
- Help refine the Severity Framework
- Improve structure, search, and developer UX

See [contributing/index.md](docs/contributing/index.md) to get started.

---

## 📄 License

- Documentation content: **Creative Commons Attribution 4.0 (CC BY 4.0)**
- Code and configuration (e.g., `mkdocs.yml`): **MIT License**

# Git Commit Guide

Step-by-step commits for this project. Run these in order from the project root.

---

## Initial Setup

```bash
git init
git remote add origin https://github.com/sushovancpp/DiagnoSite
```

---

## Commit 1 — Project scaffold

```bash
git add .gitignore README.md
git commit -m "chore: initial project scaffold with gitignore and docs"
```

---

## Commit 2 — Backend config and entry point

```bash
git add backend/app.py backend/config.py backend/requirements.txt backend/.env.example
git commit -m "feat(backend): flask app entry point and environment config"
```

---

## Commit 3 — PDF extraction service

```bash
git add backend/services/extractor.py
git commit -m "feat(services): PDF text and image extraction using PyMuPDF"
```

---

## Commit 4 — AI service (Groq)

```bash
git add backend/services/ai.py
git commit -m "feat(services): Groq LLM integration for DDR JSON generation"
```

---

## Commit 5 — PDF builder service

```bash
git add backend/services/pdf_builder.py
git commit -m "feat(services): ReportLab DDR PDF builder with area observations and severity"
```

---

## Commit 6 — Generate route

```bash
git add backend/routes/
git commit -m "feat(routes): POST /generate endpoint wiring extraction, AI, and PDF builder"
```

---

## Commit 7 — Frontend

```bash
git add frontend/
git commit -m "feat(frontend): minimal React upload UI with PDF preview and download"
```

---

## Commit 8 — Sample documents

```bash
git add samples/
git commit -m "chore(samples): synthetic inspection and thermal PDFs for testing"
```

---

## Commit 9 — Deployment config

```bash
git add backend/render.yaml
git commit -m "chore(deploy): Render config for backend deployment"
```

---

## Push

```bash
git push -u origin main
```

---

## Ongoing commit conventions

| Prefix     | Use for                                      |
|------------|----------------------------------------------|
| `feat:`    | New feature or capability                    |
| `fix:`     | Bug fix                                      |
| `chore:`   | Config, dependencies, tooling                |
| `refactor:`| Code restructuring without behaviour change  |
| `docs:`    | README, comments, documentation              |
| `style:`   | Formatting, UI-only changes                  |

**Examples:**
```bash
git commit -m "fix(ai): handle malformed JSON from LLM response"
git commit -m "feat(pdf): add severity colour coding to area headers"
git commit -m "docs: update deployment section in README"
```

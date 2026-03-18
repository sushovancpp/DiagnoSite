# DiagnoSite

> AI-powered building diagnostic report generator that converts raw inspection and thermal PDFs into structured, client-ready DDR reports using Groq LLM.

---

## Table of Contents

- [Overview](#overview)
- [Demo](#demo)
- [Features](#features)
- [System Architecture](#system-architecture)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [API Reference](#api-reference)
- [DDR Output Structure](#ddr-output-structure)
- [AI Workflow](#ai-workflow)
- [Deployment](#deployment)
- [Design Decisions](#design-decisions)
- [Limitations](#limitations)
- [Future Improvements](#future-improvements)

---

## Overview

DiagnoSite is a full-stack AI system built for the **Applied AI Builder** practical assignment. The task: take two raw site documents — a visual inspection report and a thermal imaging report — and automatically generate a professional **Detailed Diagnostic Report (DDR)**.

The system:
1. Extracts text and images from both uploaded PDFs using PyMuPDF
2. Sends the combined content to Groq's LLM (`llama-3.3-70b-versatile`) with a structured prompt
3. Parses the AI response into a validated JSON schema
4. Renders a clean, client-ready DDR as a downloadable PDF using ReportLab
5. Displays the report inline in the browser for immediate review

The solution generalises to any similar inspection + thermal document pair — not just the sample data.

---

## Demo

**Input:**
- `inspection_report.pdf` — site observations, area-wise findings, inspector notes
- `thermal_report.pdf` — temperature readings, thermal anomaly data, thermographer notes

**Output:** A structured DDR PDF containing:
- Property issue summary
- Area-wise observations with visual and thermal images side by side
- Severity-coded sections (Critical / High / Medium / Low)
- Root cause analysis
- Prioritised recommended actions
- Explicit flagging of missing or conflicting information

Sample documents are included in the `/samples` directory.

---

## Features

- **Dual-document AI merging** — Combines visual and thermal data into unified per-area findings
- **Image extraction and placement** — Pulls images from source PDFs and places them in the correct report sections
- **Severity assessment** — Each area is rated Critical / High / Medium / Low with AI-generated reasoning
- **Missing data handling** — Gaps are explicitly marked as "Not Available" rather than omitted or invented
- **Conflict detection** — Contradictions between inspection and thermal data are flagged
- **Client-friendly language** — Jargon is avoided; output is readable by non-technical stakeholders
- **Inline PDF preview** — Report renders in the browser immediately after generation
- **Scalable backend structure** — Separated into config, routes, and services layers

---

## System Architecture

```
┌─────────────────────────────────────────────────────┐
│                     Browser (React)                  │
│   Upload two PDFs → POST /generate → Preview PDF     │
└───────────────────────┬─────────────────────────────┘
                        │ multipart/form-data
┌───────────────────────▼─────────────────────────────┐
│                   Flask Backend                      │
│                                                      │
│  routes/generate.py                                  │
│       │                                              │
│       ├── services/extractor.py  (PyMuPDF)           │
│       │        Extract text + images from both PDFs  │
│       │                                              │
│       ├── services/ai.py         (Groq API)          │
│       │        Send combined text → get DDR JSON     │
│       │                                              │
│       └── services/pdf_builder.py (ReportLab)        │
│                Build final PDF with images           │
│                                                      │
└─────────────────────────────────────────────────────┘
                        │
                  application/pdf
                        │
              ┌─────────▼──────────┐
              │   Browser renders  │
              │   inline + download│
              └────────────────────┘
```

---

## Tech Stack

| Layer        | Technology                          | Purpose                              |
|--------------|-------------------------------------|--------------------------------------|
| AI / LLM     | Groq — `llama-3.3-70b-versatile`    | DDR generation from extracted text   |
| Backend      | Python 3.10+ · Flask · Flask-CORS   | API server                           |
| PDF Parsing  | PyMuPDF (`fitz`)                    | Text and image extraction            |
| PDF Output   | ReportLab                           | Structured DDR PDF generation        |
| Image        | Pillow                              | Image format normalisation (→ JPEG)  |
| Config       | python-dotenv                       | Environment variable management      |
| Frontend     | React 18 · Vite                     | Upload UI + inline PDF preview       |
| Deploy       | Render (backend) · Vercel (frontend)| Free tier hosting                    |

---

## Project Structure

```
DiagnoSite/
├── backend/
│   ├── app.py                    # Flask entry point — registers blueprints
│   ├── config.py                 # Centralised env config (GROQ_API_KEY, PORT, MODEL)
│   ├── routes/
│   │   ├── __init__.py
│   │   └── generate.py           # POST /generate — orchestrates the pipeline
│   ├── services/
│   │   ├── __init__.py
│   │   ├── extractor.py          # PyMuPDF: extracts text + images from PDFs
│   │   ├── ai.py                 # Groq API call + JSON parsing
│   │   └── pdf_builder.py        # ReportLab: builds final DDR PDF
│   ├── requirements.txt
│   ├── render.yaml               # Render deployment config
│   ├── .env                      # Local secrets (gitignored)
│   └── .env.example              # Template for environment setup
│
├── frontend/
│   ├── src/
│   │   ├── App.jsx               # Upload UI, progress bar, PDF preview
│   │   └── main.jsx              # React entry point
│   ├── index.html
│   ├── package.json
│   ├── vite.config.js
│   ├── .env                      # VITE_API_URL (gitignored)
│   └── .env.example
│
├── samples/
│   ├── create_samples.py         # Script to generate synthetic test PDFs
│   ├── inspection_report.pdf     # Sample visual inspection document
│   └── thermal_report.pdf        # Sample thermal imaging document
│
├── .gitignore
├── GIT_COMMITS.md                # Step-by-step commit history guide
└── README.md
```

---

## Getting Started

### Prerequisites

- Python 3.10 or higher
- Node.js 18 or higher
- A free Groq API key → [console.groq.com](https://console.groq.com) (no credit card required)

---

### 1. Clone the repository

```bash
git clone https://github.com/YOUR_USERNAME/diagnosite.git
cd diagnosite
```

---

### 2. Backend setup

```bash
cd backend

# Install dependencies
pip install -r requirements.txt

# Set up environment
cp .env.example .env
```

Edit `.env`:
```env
GROQ_API_KEY=your_groq_api_key_here
PORT=5000
```

Start the server:
```bash
python app.py
# Running on http://localhost:5000
```

---

### 3. Frontend setup

```bash
cd frontend

npm install

cp .env.example .env
# VITE_API_URL=http://localhost:5000 (already set)

npm run dev
# Running on http://localhost:5173
```

---

### 4. Generate test documents (optional)

Pre-built samples are already in `/samples`. To regenerate them:

```bash
cd samples
pip install reportlab pillow
python create_samples.py
```

---

### 5. Test via curl

```bash
curl -X POST http://localhost:5000/generate \
  -F "inspection_report=@samples/inspection_report.pdf" \
  -F "thermal_report=@samples/thermal_report.pdf" \
  --output DDR_Report.pdf
```

---

## API Reference

### `GET /health`

Health check.

**Response:**
```json
{ "status": "ok" }
```

---

### `POST /generate`

Accepts two PDF files. Returns a DDR PDF.

**Request:** `multipart/form-data`

| Field                | Type   | Required | Description                  |
|----------------------|--------|----------|------------------------------|
| `inspection_report`  | file   | ✓        | Visual inspection PDF         |
| `thermal_report`     | file   | ✓        | Thermal imaging PDF           |

**Success response:** `application/pdf` — the generated DDR file

**Error response:**
```json
{ "error": "Description of what went wrong" }
```

**Status codes:**

| Code | Meaning                                      |
|------|----------------------------------------------|
| 200  | DDR PDF returned successfully                |
| 400  | Missing one or both files                    |
| 500  | AI error, JSON parse failure, or server error|

---

## DDR Output Structure

The AI generates a structured JSON object which is then rendered into the PDF:

| Section                         | Description                                                   |
|---------------------------------|---------------------------------------------------------------|
| **Property Issue Summary**      | 2–4 sentence overview of all identified concerns              |
| **Area-wise Observations**      | Per-zone findings with visual + thermal data side by side     |
| **Images**                      | Extracted from source PDFs, placed under the relevant area    |
| **Probable Root Cause**         | Per-area diagnosis based on combined evidence                 |
| **Severity Assessment**         | Critical / High / Medium / Low with AI-written reasoning      |
| **Recommended Actions**         | Prioritised action list: Immediate / Short-term / Long-term   |
| **Additional Notes**            | Inspector and thermographer notes from source documents       |
| **Missing or Unclear Info**     | Explicitly flagged gaps — nothing is silently omitted         |

---

## AI Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│  System Prompt                                                   │
│  - Role: professional building inspection report writer          │
│  - Rules: no invented facts, flag missing info, no jargon        │
│  - Output schema: strict JSON with defined fields                │
└──────────────────────────────┬──────────────────────────────────┘
                               │
           ┌───────────────────▼───────────────────┐
           │           User Message                 │
           │  === INSPECTION REPORT TEXT ===        │
           │  [extracted text from PDF 1]           │
           │                                        │
           │  === THERMAL REPORT TEXT ===           │
           │  [extracted text from PDF 2]           │
           └───────────────────┬───────────────────┘
                               │
                    ┌──────────▼──────────┐
                    │   Groq LLM          │
                    │   llama-3.3-70b     │
                    │   temp: 0.2         │
                    │   max_tokens: 4096  │
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │   JSON response     │
                    │   Strip fences      │
                    │   json.loads()      │
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │   ReportLab PDF     │
                    │   + images from     │
                    │   source PDFs       │
                    └─────────────────────┘
```

**Temperature is set to 0.2** — low enough to produce consistent, structured JSON but with some flexibility for natural language in descriptions.

**Image matching** — images are extracted from both PDFs into ordered lists. Inspection image `n` is paired with area `n`'s visual section; thermal image `n` is paired with area `n`'s thermal section. This avoids fuzzy matching failures.

---

## Design Decisions

**Why Groq instead of OpenAI or Gemini?**
Groq offers a genuinely free API tier with no credit card requirement. `llama-3.3-70b-versatile` produces reliable structured JSON output suitable for this task.

**Why ReportLab instead of WeasyPrint or Puppeteer?**
ReportLab is a pure Python library with no browser or system dependencies. It is more predictable in server environments and gives precise control over layout.

**Why separate services instead of one large file?**
Each service has a single responsibility — extraction, AI, PDF generation. This makes the system testable, replaceable, and easier to debug. Swapping the AI provider, for example, only requires changes to `services/ai.py`.

**Why sequential image matching instead of fuzzy label matching?**
Fuzzy matching against image captions scored all images similarly (every image caption contains common words), causing the same image to appear in every area. Sequential index matching — inspection image 0 → area 0, image 1 → area 1 — is deterministic and correct.

**Why low temperature (0.2)?**
The AI must return valid JSON every time. Higher temperatures increase the risk of malformed output or hallucinated facts. 0.2 keeps output consistent while allowing natural language variation in descriptions.

---

## Limitations

- **Image quality depends on source PDFs** — low-resolution or embedded images may appear pixelated in the output
- **Area count must match image count** — if the AI identifies more areas than images exist in the PDFs, later areas will show "Image Not Available"
- **Text-only extraction** — scanned PDFs with no embedded text layer will produce empty extraction (OCR not implemented)
- **Groq free tier rate limits** — 30 requests/minute on the free plan; not suitable for high-volume concurrent use
- **No authentication** — the `/generate` endpoint is open; production use would require API key validation or auth middleware

---

## Future Improvements

- [ ] OCR support for scanned PDFs using Tesseract
- [ ] Multi-report comparison (track changes between inspections over time)
- [ ] User authentication and report history
- [ ] Support for Word/DOCX output in addition to PDF
- [ ] Admin dashboard for managing submitted reports
- [ ] Webhook support to notify clients when reports are ready
- [ ] Fine-tuned model on real DDR examples for higher accuracy

# DDR Generator

An AI-powered system that converts raw site inspection and thermal imaging documents into a structured Detailed Diagnostic Report (DDR).

---

## Overview

Upload two PDFs — a visual inspection report and a thermal imaging report. The system extracts content from both, passes it to an LLM, and generates a clean, client-ready diagnostic PDF with area-wise observations, severity ratings, root causes, and recommended actions.

---

## Tech Stack

| Layer     | Technology                      |
|-----------|---------------------------------|
| AI        | Groq — `llama-3.3-70b-versatile` |
| Backend   | Python · Flask · Flask-CORS     |
| Extraction| PyMuPDF (fitz)                  |
| PDF output| ReportLab                       |
| Frontend  | React · Vite                    |
| Deploy    | Render (backend) · Vercel (frontend) |

---

## Project Structure

```
ddr-system/
├── backend/
│   ├── app.py                   # Flask entry point
│   ├── config.py                # Environment config
│   ├── routes/
│   │   └── generate.py          # POST /generate route
│   ├── services/
│   │   ├── extractor.py         # PDF text + image extraction
│   │   ├── ai.py                # Groq LLM call
│   │   └── pdf_builder.py       # ReportLab DDR generation
│   ├── requirements.txt
│   └── .env
├── frontend/
│   ├── src/
│   │   ├── App.jsx              # Main UI component
│   │   └── main.jsx
│   ├── index.html
│   ├── package.json
│   └── .env
├── samples/
│   ├── create_samples.py        # Generates test PDFs
│   ├── inspection_report.pdf
│   └── thermal_report.pdf
├── .gitignore
├── GIT_COMMITS.md
└── README.md
```

---

## Local Setup

### Prerequisites
- Python 3.10+
- Node.js 18+
- A free Groq API key from [console.groq.com](https://console.groq.com)

### Backend

```bash
cd backend

# Install dependencies
pip install -r requirements.txt

# Configure environment
# Edit .env and set your GROQ_API_KEY

# Run
python app.py
# Runs on http://localhost:5000
```

### Frontend

```bash
cd frontend

npm install
npm run dev
# Runs on http://localhost:5173
```

---

## API Reference

### `POST /generate`

Accepts two PDF files and returns a DDR PDF.

**Form fields:**
| Field               | Type | Description              |
|---------------------|------|--------------------------|
| `inspection_report` | file | Visual inspection PDF    |
| `thermal_report`    | file | Thermal imaging PDF      |

**Response:** `application/pdf`

**Example:**
```bash
curl -X POST http://localhost:5000/generate \
  -F "inspection_report=@samples/inspection_report.pdf" \
  -F "thermal_report=@samples/thermal_report.pdf" \
  --output DDR_Report.pdf
```

---

## DDR Output Structure

| Section                       | Description                                  |
|-------------------------------|----------------------------------------------|
| Property Issue Summary        | 2–4 sentence overview of key concerns        |
| Area-wise Observations        | Visual + thermal findings per zone, with images |
| Probable Root Cause           | Per-area diagnosis                           |
| Severity Assessment           | Critical / High / Medium / Low with reasoning |
| Recommended Actions           | Prioritised remediation steps                |
| Additional Notes              | Inspector and thermographer notes            |
| Missing or Unclear Information| Explicitly flagged gaps                      |

---

## Deployment

### Backend → Render

1. Push the `backend/` folder to a GitHub repository
2. Create a new **Web Service** on [render.com](https://render.com)
3. Set build command: `pip install -r requirements.txt`
4. Set start command: `python app.py`
5. Add environment variable: `GROQ_API_KEY`

### Frontend → Vercel

1. Push the `frontend/` folder to GitHub
2. Import project on [vercel.com](https://vercel.com)
3. Set root directory to `frontend/`
4. Add environment variable: `VITE_API_URL` = your Render backend URL
5. Deploy

---

## Notes

- Works with any similar inspection + thermal PDF pair, not just the sample documents.
- Missing information is explicitly marked as "Not Available" — the AI does not fabricate facts.
- Conflicting data between documents is flagged rather than silently resolved.
- All findings should be verified by a qualified professional before action is taken.

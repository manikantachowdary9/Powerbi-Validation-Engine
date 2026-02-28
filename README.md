# ⚡ Power BI Validation Engine

## 👥 Team Details

- **Team Name:** *Metric Maverics*
- **Members:**
  Sai priya Bondugula,
  Praneeth Katta,
  Bikki Manikanta,
  Sivasai Vajja,
  Rasagna Chakka.
- **Domain Category:** Reusable Accelarators & Frameworks
- **Demo Video:** *(Sharepoint / Drive URL of your MVP demo)*

---

## 🎯 Problem Statement

Data teams and BI engineers face a critical trust problem:

- Power BI dashboards show numbers derived from complex DAX formulas
- The underlying SQL Server database holds the raw source of truth
- There is **no automated way to verify** that what Power BI shows matches what the database actually contains
- Manual verification requires writing SQL by hand, running it, and comparing — a process that takes hours and is error-prone
- Any model change, dataset refresh, or schema update can silently break a measure

The result: business decisions are made on dashboard numbers that nobody has formally verified.

---

## 💡 Solution Overview

We built a **Power BI Validation Engine** — an automated truth-checker that connects to both Power BI and SQL Server simultaneously and proves, measure by measure and visual by visual, that the numbers match.

The system:

1. Connects directly to the Power BI semantic model via XMLA and extracts all DAX measures and model metadata
2. Connects to the SQL Server source database and discovers its full schema
3. Uses an **AI agent (Groq)** to translate DAX formulas into equivalent T-SQL queries
4. Executes both sides — Power BI DAX and generated SQL — and compares the results
5. Validates both **scalar measures** (single totals) and **visual data** (grouped row-by-row charts)
6. Streams live results to a web dashboard with PASS / WARN / FAIL status for every check

### Validation Agents

- 🔢 **Scalar Validator** — Compares single aggregate values (totals, counts, averages)
- 📊 **Visual Validator** — Compares grouped data row-by-row (charts, tables, pie visuals)
- 🧠 **AI Translator** — Converts DAX measures into SQL using full model + schema context
- 🔍 **Visual Resolver** — Resolves unqualified dimension/measure references to fully-qualified DAX
- ⚖️ **Comparison Engine** — Judges results with configurable tolerance thresholds

---

## 🏗 Architecture

📁 Architecture Diagram: `/architecture/architecture_diagram.png`

### Components

- **Browser UI** — Web form, live SSE log stream, results dashboard
- **Flask Web Server** — HTTP API, SSE event streaming, run/stop management
- **Validation Worker** — Background thread running the 7-step pipeline
- **PowerBIClient** — ADOMD.NET / XMLA connection for DAX execution and model metadata
- **PowerBIReportClient** — Power BI REST API for report visuals and filter context
- **AIClient** — Groq AI for DAX-to-SQL translation and visual resolution
- **SQLClient** — pyodbc connector for schema discovery and query execution
- **ComparisonEngine** — Scalar and grouped result comparison with tolerance logic

### Validation Flow

1. User enters Power BI workspace/dataset and SQL Server connection details
2. Browser silently embeds the Power BI report and extracts visual specs via JS SDK
3. Backend connects to PBI via XMLA and extracts full model metadata + DAX measures
4. Backend connects to SQL Server and discovers full database schema
5. AI agent receives full PBI model + SQL schema as context
6. **Scalar loop:** For each measure — run DAX → AI generates SQL → run SQL → compare
7. **Visual loop:** Resolve dimension/measure → run SUMMARIZECOLUMNS → AI generates GROUP BY SQL → run SQL → compare row-by-row
8. Results stream live to the browser via Server-Sent Events

---

## 🛠 Tech Stack

| Layer | Technology |
|-------|------------|
| Backend | Python 3.11 |
| Web Framework | Flask |
| Frontend | HTML / CSS / Vanilla JS |
| AI Model | Groq (compound model) |
| PBI Connectivity | pyadomd / ADOMD.NET (XMLA) |
| PBI REST API | msal + requests |
| Database Connectivity | pyodbc |
| Data Processing | pandas |
| Authentication | Azure AD (MSAL) |
| Live Streaming | Server-Sent Events (SSE) |
| Concurrency | Python threading |

---

## 📂 Project Structure

```
pbi-validation-engine/
│
├── README.md                    # This file
├── requirements.txt             # Python dependencies
├── .env.example                 # Environment variable template
│
├── app.py                       # Entry point — Flask server + validation worker
├── config.py                    # Dataclass configs loaded from .env
│
├── clients/
│   ├── pbi_client.py            # XMLA/ADOMD.NET Power BI data model client
│   ├── pbi_report_client.py     # Power BI REST API client (visuals, filters)
│   ├── ai_client.py             # Groq AI — DAX→SQL translation & visual resolution
│   └── sql_client.py            # SQL Server client — schema discovery & queries
│
├── core/
│   └── comparison.py            # ComparisonEngine — scalar & visual result comparison
├── templates/
│   └── index.html               # Single-page frontend (SSE log, forms, results)
│
├── architecture/
    └── architecture_diagram.png # System architecture diagram
```

---

### 🚨 Mandatory Files for All Submissions

The following files **must be present** in every submission:

- `README.md`
- `requirements.txt`
- `.env.example`
- Clear entry point: `app.py`

---

## ⚙️ Setup Instructions

### 1️⃣ Verify Required Software

- **Programming Language:** Python
- **Required Version:** 3.11+
- **Package Manager:** pip
- **System Requirement:** Windows only (ADOMD.NET DLL at `C:\Program Files\Microsoft.NET\ADOMD.NET\170`)

### 2️⃣ Clone Repository

```bash
git clone https://github.com/your-org/pbi-validation-engine
cd pbi-validation-engine
```

### 3️⃣ Create Virtual Environment

```bash
python -m venv venv
```

Activate:

**Windows**
```bash
venv\Scripts\activate
```

**Mac/Linux**
```bash
source venv/bin/activate
```

### 4️⃣ Install Dependencies

```bash
pip install -r requirements.txt
```

### 5️⃣ Configure Environment Variables

Copy `.env.example` to `.env` and fill in your values:

```
# Power BI Workspace (XMLA)
PBI_WORKSPACE_NAME=your_workspace_name
PBI_DATASET_NAME=your_dataset_name

# Azure AD App Registration (for REST API)
PBI_TENANT_ID=your_tenant_id
PBI_CLIENT_ID=your_client_id
PBI_CLIENT_SECRET=your_client_secret

# Groq AI
GROQ_API_KEY=your_groq_api_key
```

---

## ▶️ Entry Point

Run the application:

```bash
python app.py
```

Application starts at:

```
http://localhost:5000
```

---

## 🔄 Application Flow

1. User enters Power BI workspace, dataset name, and SQL Server connection details in the web form
2. User optionally enters a report name — the system auto-extracts visuals using the Power BI JS SDK
3. User selects which measures to validate (or validates all)
4. On "Run Validation":
   - System connects to Power BI via XMLA and extracts the full semantic model
   - System connects to SQL Server and discovers the complete database schema
   - For each DAX measure:
     - Executes the measure in Power BI → gets scalar result
     - AI translates DAX to SQL using full context → runs against database
     - Compares values with tolerance thresholds → PASS / WARN / FAIL
   - For each visual spec:
     - AI resolves dimension/measure to fully-qualified DAX references
     - Runs SUMMARIZECOLUMNS in Power BI → gets grouped DataFrame
     - AI generates equivalent GROUP BY SQL → runs against database
     - Compares row-by-row → PASS / WARN / FAIL
5. All results stream live to the browser log and populate the results panel
6. User can stop the run at any point with the Stop button

---

## 🧪 How to Test

### Option 1 — With a Real Power BI Dataset

Fill in your `.env` with actual workspace credentials and run the app. Use any report that has DAX measures built on top of a SQL Server database.

### Option 2 — Minimal Test Setup

Create a Power BI dataset with a simple sales table and at least one DAX measure:

```dax
Total Sales = SUMX(Sales, Sales[Quantity] * Sales[Amount])
```

Then point the SQL connection at the underlying SQL Server table and run validation. Expected result: PASS with 0.00% difference.

### Example Prompts / Inputs

- Workspace: `MyCompany_Analytics`
- Dataset: `Sales Report`
- SQL Server: `your-server.database.windows.net`
- Database: `SALESDB`

---

## ⚠️ Known Limitations

- **Windows only** — requires ADOMD.NET DLL (`Microsoft.NET\ADOMD.NET\170`)
- **XMLA Premium required** — Power BI workspace must be on Premium or PPU capacity for XMLA read access
- **DAX complexity** — Highly complex DAX (time intelligence, many-to-many relationships) may not translate perfectly to SQL
- **Visual extraction** — Visuals without an explicit DAX formula require AI resolution, which may occasionally need manual correction
- **Single-user** — No multi-user session management; one validation run at a time per server instance

---

## 🔮 Future Improvements

- Schedule automated validation runs (nightly / post-refresh)
- Add support for Power BI Premium Per User (PPU) and Fabric workspaces
- Extend AI translation to handle time-intelligence DAX (DATEYTD, SAMEPERIODLASTYEAR)
- Add CI/CD integration — run as a GitHub Action on dataset model changes
- Export validation reports as PDF / Excel
- Add email/Teams alerting on FAIL results
- Multi-user session isolation and run history

---

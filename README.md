# GayatriBot - HR & Recruitment Automation System

> **Note:** This project has been developed under the [Dbert online internship program](https://dbert.online). We thank the Dbert team for their guidance and support.

GayatriBot is an enterprise-grade automated HR and Recruitment management system built for the Gayatri Education Project. The system connects Google Forms/Sheets, local AI (Ollama LLM) for intelligent resume parsing, a rotating multi-account SMTP email pool, cross-platform PDF offer letter generation, and a real-time Flask web administration dashboard.

---

## 🚀 Key Features

### 1. 🤖 Automated Application Polling & Email Dispatch
- **Google Sheets Integration**: Polls candidate submissions periodically using Google Sheets API via service account authentication.
- **Smart SMTP Pool**: Distributes outgoing acknowledgment, confirmation, and reminder emails across a pool of SMTP sender accounts to prevent mailbox rate limits and spam flagging.
- **Automated Induction Scheduling**: Dynamically schedules the upcoming Monday HR induction date.

### 2. 🧠 AI-Powered CV & Resume Parsing
- **Local Ollama LLM (`llama3:8b`)**: Extracts candidate skills, education, and domain alignment without sending sensitive applicant data to external third-party APIs.
- **Domain Matching**: Analyzes applicant compatibility against configurable track requirements (Data Analyst, Full Stack, AI Agent Development, Python Automation).

### 3. 📄 Cross-Platform PDF Offer Letter Generation (Linux / Cloud Ready)
- **Pure-Python Engine via ReportLab**: Generates high-fidelity, professional PDF offer letters with customized terms, dates, and candidate information.
- **No Microsoft Word / COM Automation Dependency**: Fully compatible with Linux, macOS, Docker, and cloud hosting environments (with graceful legacy `.docx`/Word COM fallback if present).

### 4. 🎯 Web Dashboard & 1-Click Operations
- **1-Click Offer Dispatch**: Dispatch individualized PDF offer letters directly from any applicant detail view with automated generation and email delivery.
- **Multi-Select Bulk Actions Toolbar**: Batch select candidates on the applicant table to execute:
  - *Bulk Approve*
  - *Bulk Reject*
  - *Bulk Send Reminders*
  - *Bulk Dispatch Offer Letters*
- **Real-Time Live Activity & Log Stream**: Built-in Server-Sent Events (SSE) live terminal on the main dashboard streaming system activity and background dispatch events live without page refreshes.
- **Offer Letter Archive**: Instant browser download and preview links for all generated offer letter PDFs.

---

## 🏗️ System Architecture

```
                       ┌─────────────────────────┐
                       │  Google Forms / Sheets  │
                       └────────────┬────────────┘
                                    │ (Service Account)
                                    ▼
                       ┌─────────────────────────┐
                       │   SQLite (gayatri.db)   │
                       └──────▲───────────▲──────┘
                              │           │
            ┌─────────────────┴─┐       ┌─┴─────────────────┐
            │   bot.py          │       │  cv_processor.py  │
            │   - Polling       │       │  - Google Drive   │
            │   - Auto emails   │       │  - Ollama LLM     │
            │   - SMTP Pool     │       │  - Skill Match    │
            └───────────────────┘       └───────────────────┘
                                    ▲
                                    │
                       ┌────────────┴────────────┐
                       │   flask_app.py (Admin)  │
                       │   - Real-time Dashboard │
                       │   - SSE Log Stream      │
                       │   - Bulk Actions        │
                       │   - 1-Click Offer Gen   │
                       │   - ReportLab PDF Engine│
                       └─────────────────────────┘
```

---

## 🛠️ Prerequisites

- **Python 3.9+** (Windows, Linux, or macOS)
- **Google Cloud Service Account JSON key** (`service_account.json`) with Google Sheets & Drive API access
- **[Ollama](https://ollama.ai)** installed and running locally with the `llama3:8b` model:
  ```bash
  ollama pull llama3:8b
  ```
- **SMTP credentials** (e.g. Gmail / Google Workspace App Passwords or custom SMTP mail server)

---

## 📦 Installation & Setup

### 1. Clone the Repository
```bash
git clone https://github.com/editor-shannu/gayatribot-Hr.git
cd gayatribot-Hr
```

### 2. Create and Activate a Virtual Environment
```bash
# Windows
python -m venv venv
venv\Scripts\activate

# Linux / macOS
python3 -m venv venv
source venv/bin/activate
```

### 3. Install Dependencies
```bash
pip install -r requirements.txt
```


## 🏃 Running the System

To run the complete system, launch the components in separate terminal windows (with your virtual environment activated):

### Terminal 1: Application Poller & Email Bot
```bash
python bot.py
```
*Polls Google Sheets for new submissions and sends automated acknowledgments.*

### Terminal 2: AI CV Processor
```bash
python cv_processor.py
```
*Fetches resumes from Google Drive and analyzes skills via Ollama.*

### Terminal 3: Flask Admin Dashboard
```bash
python flask_app.py
```
*Starts the administrative web console at `http://localhost:5000`.*

### Terminal 4 (Optional): Standalone Offer Letter Dispatcher
```bash
python offer_letter_sender.py
```
*Batch generates ReportLab PDFs and emails offer letters to all approved candidates.*

---

## 🛡️ Security & Best Practices
- **Zero Hardcoded Secrets**: All sensitive keys, sheet IDs, and passwords are loaded via environment variables (`python-dotenv`).
- **Safe Logging**: Structured logging sanitizes and excludes credentials and sensitive tokens.
- **Rate-Limited Dispatching**: Built-in delays and hourly/daily limits protect SMTP accounts from reputation degradation.

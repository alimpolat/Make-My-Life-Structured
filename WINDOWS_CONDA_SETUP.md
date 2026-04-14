# Windows Miniconda Setup Guide
## For: GenAI Job Automation Pipeline (Python 3.12 + FastAPI)

**Purpose**: Set up a reproducible Python environment on Windows for building an end-to-end automation pipeline — web search, job identification, auto-apply, and email response handling.

**Total Time**: ~45 minutes

---

## Part 1: Install Miniconda (Windows)

### 1.1 Download

Go to: https://docs.conda.io/en/latest/miniconda.html

Download: **Miniconda3 Windows 64-bit** (`.exe` installer, ~80 MB)

### 1.2 Install

1. Run the downloaded `.exe`
2. Click **Next** through the welcome screen
3. Accept the license agreement
4. **Install for**: "Just Me" (recommended)
5. **Install location**: Leave default (`C:\Users\alimp\miniconda3`)
6. **Advanced Options**:
   - Check "Add Miniconda3 to my PATH environment variable"
   - Check "Register Miniconda3 as my default Python"
7. Click **Install**, wait 2-3 minutes
8. Click **Finish**

### 1.3 Verify

Open a **new** Command Prompt or PowerShell:

```bash
conda --version
```

Expected: `conda 24.x.x` (or newer)

If `conda` is not recognized, close all terminals and open a fresh one. If still failing:

```bash
# Run from Anaconda Prompt (search "Anaconda Prompt" in Start menu)
conda init powershell
conda init cmd.exe
# Then restart your terminal
```

### 1.4 Configure Conda

```bash
conda update -n base -c defaults conda -y
conda config --add channels conda-forge
conda config --set channel_priority strict
```

---

## Part 2: Create Project Environment

### 2.1 Create the Environment

```bash
conda create -n genai-jobflow python=3.12 -y
```

This creates an isolated environment with Python 3.12. Takes ~3 minutes.

### 2.2 Activate

```bash
conda activate genai-jobflow
```

Your prompt changes to `(genai-jobflow) C:\Users\alimp>`. Verify:

```bash
python --version
where python
```

Expected:
```
Python 3.12.x
C:\Users\alimp\miniconda3\envs\genai-jobflow\python.exe
```

---

## Part 3: Install Dependencies

### 3.1 Create requirements.txt

Create a `requirements.txt` in your project folder with these packages:

```txt
# === Web Framework ===
fastapi==0.115.6
uvicorn[standard]==0.34.0
python-multipart==0.0.18

# === AI / LLM ===
openai==1.58.1
anthropic==0.42.0
langchain==0.3.14
langchain-openai==0.3.0
langchain-anthropic==0.3.3

# === Web Scraping & Search ===
requests==2.32.3
httpx==0.28.1
beautifulsoup4==4.12.3
selenium==4.27.1
playwright==1.49.1
lxml==5.3.0

# === Email Handling ===
imapclient==3.0.1
python-dotenv==1.0.1

# === PDF / Document Generation ===
python-docx==1.1.2
reportlab==4.2.5
PyPDF2==3.0.1
jinja2==3.1.5

# === Database ===
supabase==2.11.0
sqlalchemy==2.0.36
aiosqlite==0.20.0

# === Scheduling & Tasks ===
apscheduler==3.10.4
celery==5.4.0
redis==5.2.1

# === Data & Utilities ===
pydantic==2.10.4
pydantic-settings==2.7.1
pandas==2.2.3
python-dateutil==2.9.0
aiofiles==24.1.0

# === Testing ===
pytest==8.3.4
pytest-asyncio==0.25.0
httpx==0.28.1
```

### 3.2 Install

```bash
conda activate genai-jobflow
cd C:\Users\alimp\Documents\GenAI_Projects\YOUR_PROJECT_FOLDER
pip install -r requirements.txt
```

Takes 5-10 minutes depending on internet speed.

### 3.3 Install Playwright Browsers (for web scraping)

```bash
playwright install chromium
```

This downloads a Chromium browser that Playwright controls for scraping job boards. Takes ~3 minutes.

### 3.4 Verify Key Packages

```bash
python -c "import fastapi; print(f'FastAPI {fastapi.__version__}')"
python -c "import openai; print(f'OpenAI {openai.__version__}')"
python -c "import anthropic; print(f'Anthropic {anthropic.__version__}')"
python -c "import selenium; print(f'Selenium {selenium.__version__}')"
```

All should print versions without errors.

---

## Part 4: Project Structure

Recommended folder layout for the automation pipeline:

```
your-project/
├── .env                    # API keys (NEVER commit this)
├── .env.example            # Template for .env (commit this)
├── .gitignore
├── requirements.txt
├── environment.yml         # Conda export (generated)
├── CLAUDE.md               # Claude Code instructions
│
├── api/                    # FastAPI backend
│   ├── main.py             # App entry point
│   ├── config.py           # Settings & env vars
│   └── routes/
│       ├── search.py       # Job search endpoints
│       ├── apply.py        # Application submission
│       └── email.py        # Email response handling
│
├── core/                   # Business logic
│   ├── searcher.py         # Web search & job scraping
│   ├── matcher.py          # Job matching / filtering
│   ├── applicant.py        # Auto-apply logic
│   ├── email_handler.py    # Email reading & AI responses
│   └── resume_builder.py   # Dynamic resume/cover letter gen
│
├── prompts/                # LLM prompt templates
│   ├── cover_letter.txt
│   ├── email_reply.txt
│   └── job_match.txt
│
├── scheduler/              # Automated task scheduling
│   └── jobs.py
│
├── tests/
│   ├── test_searcher.py
│   ├── test_matcher.py
│   └── test_email.py
│
└── data/
    ├── profile.json        # Your profile / resume data
    └── applications.db     # SQLite tracking DB
```

### 4.1 Create .env File

```env
# LLM APIs
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...

# Email (Gmail example - use App Password, not your real password)
EMAIL_ADDRESS=your@gmail.com
EMAIL_APP_PASSWORD=xxxx-xxxx-xxxx-xxxx
IMAP_SERVER=imap.gmail.com
SMTP_SERVER=smtp.gmail.com

# Supabase (optional)
SUPABASE_URL=https://xxx.supabase.co
SUPABASE_KEY=eyJ...

# Job Board APIs (if applicable)
LINKEDIN_EMAIL=
LINKEDIN_PASSWORD=
```

### 4.2 Create .gitignore

```gitignore
# Environment
.env
*.pyc
__pycache__/
*.egg-info/

# Conda
miniconda3/

# IDE
.vscode/launch.json
.idea/

# Data
data/applications.db
data/*.json
!data/profile.json

# OS
Thumbs.db
Desktop.ini
```

---

## Part 5: VS Code Integration

### 5.1 Select Python Interpreter

1. Open your project in VS Code
2. Press `Ctrl+Shift+P` > "Python: Select Interpreter"
3. Choose `genai-jobflow (conda)` from the list
4. If not listed, click "Enter interpreter path" and paste:
   ```
   C:\Users\alimp\miniconda3\envs\genai-jobflow\python.exe
   ```

### 5.2 Essential Extensions

Install via terminal:

```bash
code --install-extension ms-python.python
code --install-extension ms-python.vscode-pylance
code --install-extension charliermarsh.ruff
code --install-extension esbenp.prettier-vscode
code --install-extension rangav.vscode-thunder-client
```

### 5.3 Workspace Settings

Create `.vscode/settings.json` in your project:

```json
{
  "python.defaultInterpreterPath": "C:\\Users\\alimp\\miniconda3\\envs\\genai-jobflow\\python.exe",
  "[python]": {
    "editor.defaultFormatter": "charliermarsh.ruff",
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
      "source.organizeImports": "explicit"
    }
  },
  "python.testing.pytestEnabled": true,
  "python.testing.pytestArgs": ["tests", "-v"],
  "files.trimTrailingWhitespace": true,
  "files.insertFinalNewline": true,
  "search.exclude": {
    "**/node_modules": true,
    "**/__pycache__": true,
    "**/.pytest_cache": true
  }
}
```

---

## Part 6: Daily Workflow

### Starting Work

```bash
# 1. Open terminal (PowerShell or CMD)
# 2. Activate environment
conda activate genai-jobflow

# 3. Navigate to project
cd C:\Users\alimp\Documents\GenAI_Projects\YOUR_PROJECT_FOLDER

# 4. Start the API server
uvicorn api.main:app --reload --port 8000
```

### Running Tests

```bash
conda activate genai-jobflow
pytest tests/ -v
```

### Adding New Packages

```bash
# Install the package
pip install package-name

# Update requirements.txt
pip freeze > requirements.txt
# OR manually add the specific package to requirements.txt (preferred)
```

### Exporting Environment (for sharing/backup)

```bash
conda env export --no-builds > environment.yml
```

### Recreating from Export

```bash
conda env create -f environment.yml
```

---

## Part 7: Quick Troubleshooting

| Problem | Fix |
|---------|-----|
| `conda` not recognized | Open Anaconda Prompt, run `conda init powershell`, restart terminal |
| `pip install` fails with permission error | Make sure env is activated: `conda activate genai-jobflow` |
| VS Code uses wrong Python | `Ctrl+Shift+P` > "Python: Select Interpreter" > pick `genai-jobflow` |
| Import errors in VS Code | Reload window: `Ctrl+Shift+P` > "Developer: Reload Window" |
| Package conflicts | Recreate env: `conda deactivate && conda env remove -n genai-jobflow && conda create -n genai-jobflow python=3.12 -y` |
| Playwright not working | Run `playwright install chromium` after activating env |
| Selenium needs ChromeDriver | Install: `pip install webdriver-manager`, it auto-downloads the right driver |

---

## Conda Cheatsheet

```bash
conda create -n myenv python=3.12 -y    # Create environment
conda activate myenv                      # Activate
conda deactivate                          # Deactivate
conda env list                            # List all environments
conda env remove -n myenv                # Delete environment
conda env export > environment.yml       # Export
conda env create -f environment.yml      # Import
conda list                                # List packages in active env
```

---

*Last updated: 2026-02-06*

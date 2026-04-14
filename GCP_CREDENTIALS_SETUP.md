# GCP Credentials Setup for SmartDraft

This guide explains how to configure Google Cloud Platform credentials for the SmartDraft application.

---

## Overview

SmartDraft uses **Google Gemini** via Vertex AI for:
- Long-context document processing (up to 1M tokens)
- Native OCR for scanned PDFs
- Advanced reasoning with Gemini 2.5 Pro

---

## Required Files

| File | Description | Commit to Git? |
|------|-------------|----------------|
| `gg-gcpsbprojs-004-d8c9dcdba78d.json` | GCP Service Account Key | **NO** |
| `.env` | Environment configuration | **NO** |
| `.env.example` | Template for `.env` | YES |

---

## Setup Instructions

### 1. Obtain the Credentials File

Contact **Alim Polat** (alimpolat@gmail.com) to receive the GCP service account JSON file securely.

**Do NOT:**
- Commit credentials to Git
- Share via email attachments
- Store in shared drives without encryption

### 2. Place the File in Project Root

```bash
# Your project structure should look like:
site-contract-ai-v2/
├── gg-gcpsbprojs-004-d8c9dcdba78d.json   <-- Place here
├── .env
├── .env.example
├── apps/
├── src/
└── ...
```

### 3. Configure Environment Variables

Copy `.env.example` to `.env` and update the GCP section:

```bash
cp .env.example .env
```

Edit `.env` with the following GCP configuration:

```env
# ============================================================
# GOOGLE CLOUD / GEMINI API
# ============================================================
GOOGLE_APPLICATION_CREDENTIALS=gg-gcpsbprojs-004-d8c9dcdba78d.json
GCP_PROJECT_ID=gg-gcpsbprojs-004
GEMINI_MODEL=gemini-2.5-pro
```

**Note:** The path is **relative** to the project root. Do not use absolute paths.

### 4. Verify the Setup

Run the test script to verify your credentials work:

```bash
cd /path/to/site-contract-ai-v2
conda activate your_env
python scripts/test_gemini_connection.py
```

Expected output:
```
============================================================
GEMINI API CONNECTION TEST
============================================================

1. Environment Variables:
   GOOGLE_APPLICATION_CREDENTIALS: gg-gcpsbprojs-004-d8c9dcdba78d.json
   GCP_PROJECT_ID: gg-gcpsbprojs-004
   GEMINI_MODEL: gemini-2.5-pro

2. Credentials File:
   ✓ Found: /path/to/gg-gcpsbprojs-004-d8c9dcdba78d.json

3. Authentication Test:
   ✓ Credentials loaded successfully

4. Vertex AI Client Test:
   ✓ Client initialized

5. Model Access Test:
   ✓ gemini-2.5-pro @ us-central1: Hello

============================================================
SUCCESS: Gemini API is working!
============================================================
```

---

## Troubleshooting

### Error: "Could not load credentials"

- Verify the JSON file exists in project root
- Check the filename matches exactly in `.env`
- Ensure file permissions allow reading: `chmod 600 gg-gcpsbprojs-004-d8c9dcdba78d.json`

### Error: "Permission denied" or "403"

- The service account may lack required IAM roles
- Contact the GCP project owner to grant `Vertex AI User` role

### Error: "Model not found" or "404"

- Vertex AI API may not be enabled in the GCP project
- The specific Gemini model may not be available in the region
- Try changing `GEMINI_MODEL` to `gemini-1.5-flash` as a fallback

### Error: "Quota exceeded"

- The GCP project has API quota limits
- Wait and retry, or contact the project owner to increase quota

---

## GCP Project Details

| Property | Value |
|----------|-------|
| Project ID | `gg-gcpsbprojs-004` |
| Region | `us-central1` |
| Service Account | See JSON file `client_email` field |
| Available Models | gemini-2.5-pro, gemini-2.5-flash, gemini-1.5-pro, gemini-1.5-flash |

---

## Security Reminders

1. **Never commit credentials to Git** - The `.gitignore` should exclude `*.json` credential files
2. **Rotate keys periodically** - Service account keys should be rotated every 90 days
3. **Use least privilege** - Only request necessary IAM permissions
4. **Audit access** - Review who has access to credentials regularly

---

## Contact

For credentials access or issues:
- **Alim Polat** - alimpolat@gmail.com
- **GCP Project Owner** - Hari (for IAM/quota issues)

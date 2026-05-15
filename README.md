# 🛡️ FNOL-ClaimSense-AI

> Autonomous Insurance Claims Processing Agent powered by Groq's free Llama 3.1 LLM — extracts structured fields from FNOL documents, detects missing data, and intelligently routes claims to the correct workflow.

---

## 📋 Table of Contents

1. [Overview](#overview)
2. [Features](#features)
3. [Project Structure](#project-structure)
4. [Architecture](#architecture)
5. [Routing Rules](#routing-rules)
6. [Prerequisites](#prerequisites)
7. [Installation](#installation)
8. [Configuration](#configuration)
9. [Running the Agent](#running-the-agent)
10. [Output Format](#output-format)
11. [Sample FNOL Documents](#sample-fnol-documents)
12. [Technologies Used](#technologies-used)

---

## 📌 Overview

FNOL-ClaimSense-AI is a lightweight autonomous agent that processes **First Notice of Loss (FNOL)** insurance documents. It reads raw FNOL text files, uses an AI model to extract key fields, checks for missing or inconsistent data, and automatically routes each claim to the correct processing workflow — without any manual intervention.

---

## ✨ Features

- 🤖 AI-powered field extraction using **Groq (free API)** + **Llama 3.1**
- 🔍 Automatic detection of missing or null mandatory fields
- 🚦 Rule-based intelligent claim routing (5 route types)
- 🚩 Fraud / investigation flag detection via keyword scanning
- 📄 Structured JSON output per FNOL document
- ⚡ Fast and lightweight — no paid API required

---

## 🗂️ Project Structure

```
FNOL-ClaimSense-AI/
│
├── agent.py                  # Main agent — extraction, validation, routing
│
├── sample_fnols/             # Sample FNOL input documents
│   ├── fnol_001.txt          # Auto damage (Raj Sharma)    → Fast-track
│   ├── fnol_002.txt          # Injury claim (Priya Nair)   → Specialist Queue
│   └── fnol_003.txt          # Fraud signals (Arjun Rao)   → Investigation Flag
│
├── outputs/                  # Auto-created — one JSON result per FNOL
│
├── requirements.txt          # Python dependencies
└── README.md                 # Project documentation
```

---

## 📐 Architecture

The agent runs in 4 sequential steps for each FNOL document:

```
┌─────────────────────────────────────────────┐
│           FNOL Document (.txt)              │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│  STEP 1 — AI Field Extraction               │
│  Groq LLM (Llama 3.1) parses raw text       │
│  and returns a structured JSON object       │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│  STEP 2 — Missing Field Detection           │
│  Checks 13 mandatory fields for null /      │
│  blank values and builds missingFields[]    │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│  STEP 3 — Claim Routing                     │
│  Applies priority-ordered rules to assign   │
│  a route and generate a reasoning string    │
└─────────────────┬───────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────────┐
│  STEP 4 — JSON Output                       │
│  Saves result to outputs/<filename>.json    │
└─────────────────────────────────────────────┘
```

---

## 🔀 Routing Rules

Rules are applied in strict priority order — the first match wins.

| Priority | Condition | Route |
|----------|-----------|-------|
| 1 | Description contains: `fraud`, `inconsistent`, `staged`, `suspicious`, `fabricated` | 🚩 **Investigation Flag** |
| 2 | `claim_type` = `injury` | 👨‍⚕️ **Specialist Queue** |
| 3 | Any mandatory field is missing or null | 📋 **Manual Review** |
| 4 | `estimated_damage` < 25,000 & all fields present | ⚡ **Fast-track** |
| 5 | `estimated_damage` ≥ 25,000 | 📁 **Standard Queue** |

---

## ✅ Prerequisites

Before you begin, make sure you have the following:

- **Python 3.9 or higher** installed
- **pip** (Python package manager)
- A **free Groq API key** — sign up at [console.groq.com](https://console.groq.com) (no credit card needed)
- **Git** installed on your machine

---

## 🔧 Installation

Follow these steps one by one:

**Step 1 — Clone the repository**
```bash
git clone https://github.com/YOUR-USERNAME/FNOL-ClaimSense-AI.git
```

**Step 2 — Move into the project folder**
```bash
cd FNOL-ClaimSense-AI
```

**Step 3 — Create a virtual environment**
```bash
python -m venv venv
```

**Step 4 — Activate the virtual environment**

On macOS / Linux:
```bash
source venv/bin/activate
```

On Windows (PowerShell):
```powershell
venv\Scripts\activate
```

**Step 5 — Install dependencies**
```bash
pip install -r requirements.txt
```

---

## ⚙️ Configuration

**Step 6 — Set your Groq API key as an environment variable**

On macOS / Linux:
```bash
export GROQ_API_KEY="gsk_your_key_here"
```

On Windows (PowerShell):
```powershell
$env:GROQ_API_KEY="gsk_your_key_here"
```

> ⚠️ Never hardcode your API key in the source code or commit it to GitHub.

---

## ▶️ Running the Agent

**Step 7 — Run the agent**
```bash
python agent.py
```

**Step 8 — Watch the live console output**

The agent processes each file and prints progress:
```
=======================================================
  Processing: fnol_001.txt
=======================================================
  [1/4] Extracting fields via Groq (free)...
  [2/4] Checking for missing fields...
         All mandatory fields present.
  [3/4] Routing claim...
         Route -> Fast-track
  [4/4] Building output...
  Saved -> outputs/fnol_001_result.json

=======================================================
  Processing: fnol_002.txt
=======================================================
  [1/4] Extracting fields via Groq (free)...
  [2/4] Checking for missing fields...
         Missing: ['policy_effective_date', 'contact_details', 'asset_id']
  [3/4] Routing claim...
         Route -> Specialist Queue
  [4/4] Building output...
  Saved -> outputs/fnol_002_result.json

=======================================================
  Processing: fnol_003.txt
=======================================================
  [1/4] Extracting fields via Groq (free)...
  [2/4] Checking for missing fields...
         Missing: ['policy_effective_date', 'contact_details', 'asset_id']
  [3/4] Routing claim...
         Route -> Investigation Flag
  [4/4] Building output...
  Saved -> outputs/fnol_003_result.json
```

**Step 9 — View the final summary**
```
=======================================================
  PROCESSING SUMMARY
=======================================================
  fnol_001.txt         -> Fast-track
  fnol_002.txt         -> Specialist Queue
  fnol_003.txt         -> Investigation Flag
=======================================================
```

**Step 10 — Inspect any output JSON file**
```bash
cat outputs/fnol_001_result.json
```

---

## 📤 Output Format

Each FNOL produces a JSON file in `outputs/`. Below are the actual outputs for all 3 sample documents.

---

### fnol_001_result.json — Raj Sharma

```json
{
  "extractedFields": {
    "policy_number": "POL12345",
    "policyholder_name": "Raj Sharma",
    "policy_effective_date": "Jan 2026 - Jan 2027",
    "policy_expiry_date": "Jan 2027",
    "incident_date": "10 May 2026",
    "incident_time": "3 PM",
    "incident_location": "Bangalore",
    "incident_description": "Minor rear-end collision in traffic.",
    "claimant_name": "Raj Sharma",
    "third_party": null,
    "contact_details": null,
    "asset_type": "Car",
    "asset_id": "CAR9981",
    "estimated_damage": 18000,
    "claim_type": "Auto",
    "attachments": "photos.pdf",
    "initial_estimate": 18000
  },
  "missingFields": ["contact_details"],
  "recommendedRoute": "Fast-track",
  "reasoning": "Estimated damage (18,000) is below the 25,000 threshold and all core mandatory fields are present. Eligible for fast-track processing."
}
```

---

### fnol_002_result.json — Priya Nair

```json
{
  "extractedFields": {
    "policy_number": "POL99999",
    "policyholder_name": "Priya Nair",
    "policy_effective_date": null,
    "policy_expiry_date": null,
    "incident_date": "12 May 2026",
    "incident_time": "5 PM",
    "incident_location": "Chennai",
    "incident_description": "Customer suffered injury during accident.",
    "claimant_name": "Priya Nair",
    "third_party": null,
    "contact_details": null,
    "asset_type": "Bike",
    "asset_id": null,
    "estimated_damage": 40000,
    "claim_type": "Injury",
    "attachments": "medical.pdf",
    "initial_estimate": 40000
  },
  "missingFields": ["policy_effective_date", "contact_details", "asset_id"],
  "recommendedRoute": "Specialist Queue",
  "reasoning": "Claim type is Injury. Routed to the specialist medical assessment team regardless of missing fields."
}
```

---

### fnol_003_result.json — Arjun Rao

```json
{
  "extractedFields": {
    "policy_number": "POL88888",
    "policyholder_name": "Arjun Rao",
    "policy_effective_date": null,
    "policy_expiry_date": null,
    "incident_date": "11 May 2026",
    "incident_time": "10 PM",
    "incident_location": "Hyderabad",
    "incident_description": "Possible staged collision with inconsistent witness reports.",
    "claimant_name": "Arjun Rao",
    "third_party": null,
    "contact_details": null,
    "asset_type": "Car",
    "asset_id": null,
    "estimated_damage": 70000,
    "claim_type": "Auto",
    "attachments": "report.pdf",
    "initial_estimate": 70000
  },
  "missingFields": ["policy_effective_date", "contact_details", "asset_id"],
  "recommendedRoute": "Investigation Flag",
  "reasoning": "Description contains suspicious keywords: staged, inconsistent. This claim has been flagged for fraud investigation before any further processing."
}
```

---

## 🧪 Sample FNOL Documents

| File | Policyholder | Location | Claim Type | Damage | Missing Fields | Route |
|------|-------------|----------|------------|--------|----------------|-------|
| `fnol_001.txt` | Raj Sharma | Bangalore | Auto — rear-end collision | INR 18,000 | contact_details | ⚡ Fast-track |
| `fnol_002.txt` | Priya Nair | Chennai | Injury — accident | INR 40,000 | effective_date, contact, asset_id | 👨‍⚕️ Specialist Queue |
| `fnol_003.txt` | Arjun Rao | Hyderabad | Auto — staged collision | INR 70,000 | effective_date, contact, asset_id | 🚩 Investigation Flag |

---

## 🧰 Technologies Used

| Technology | Purpose |
|------------|---------|
| Python 3.9+ | Core programming language |
| Groq API (free) | LLM inference engine |
| Llama 3.1 8B | Field extraction model |
| JSON | Structured output format |

---






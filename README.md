# Autonomous-Insurance-claims-Agent
Autonomous FNOL Claims Processing Agent that uses Groq's free Llama 3.1 LLM to extract structured fields from insurance documents, detect missing data, and intelligently route claims to Fast-track, Specialist Queue, Manual Review, or Investigation Flag workflows. Built with Python.
 Autonomous Insurance Claims Processing Agent
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
11. [Sample Scenarios](#sample-scenarios)
12. [Technologies Used](#technologies-used)

---

## 📌 Overview

FNOL-ClaimSense-AI is a lightweight autonomous agent that processes **First Notice of Loss (FNOL)** insurance documents. It reads raw FNOL text, uses an AI model to extract key fields, checks for missing or inconsistent data, and automatically routes the claim to the correct processing workflow — all without manual intervention.

---

## ✨ Features

- 🤖 AI-powered field extraction using **Groq (free API)** + **Llama 3.1**
- 🔍 Automatic detection of missing or null mandatory fields
- 🚦 Rule-based intelligent claim routing (5 route types)
- 🚩 Fraud / investigation flag detection via keyword scanning
- 📄 Structured JSON output per FNOL document
- ⚡ Fast, lightweight — no paid API required

---

## 🗂️ Project Structure

```
FNOL-ClaimSense-AI/
│
├── agent.py                  # Main agent — extraction, validation, routing
│
├── sample_fnols/             # Sample FNOL input documents
│   ├── fnol_001.txt          # Auto damage         → Fast-track
│   ├── fnol_002.txt          # Personal injury      → Specialist Queue
│   ├── fnol_003.txt          # Fraud signals        → Investigation Flag
│   ├── fnol_004.txt          # Missing fields       → Manual Review
│   └── fnol_005.txt          # Theft                → Fast-track
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
- A **free Groq API key** — get one at [console.groq.com](https://console.groq.com) (no credit card required)
- **Git** installed on your machine

---

## 🔧 Installation

Follow these steps exactly:

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

**Step 8 — Watch the console output**

You will see live progress for each file:
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
```

**Step 9 — View the summary table**

At the end, a summary is printed:
```
=======================================================
  PROCESSING SUMMARY
=======================================================
  fnol_001.txt         -> Fast-track
  fnol_002.txt         -> Specialist Queue
  fnol_003.txt         -> Investigation Flag
  fnol_004.txt         -> Manual Review
  fnol_005.txt         -> Fast-track
=======================================================
```

**Step 10 — Open any output file to inspect results**
```bash
cat outputs/fnol_001_result.json
```

---

## 📤 Output Format

Each FNOL produces a JSON file in `outputs/` with this structure:

```json
{
  "extractedFields": {
    "policy_number": "POL-2024-78432",
    "policyholder_name": "Rajesh Kumar",
    "policy_effective_date": "2024-01-01",
    "policy_expiry_date": "2025-01-01",
    "incident_date": "2024-11-14",
    "incident_time": "08:35 AM",
    "incident_location": "MG Road, Bangalore, Karnataka",
    "incident_description": "Vehicle rear-ended at a traffic signal...",
    "claimant_name": "Rajesh Kumar",
    "third_party": "Suresh Mehta",
    "contact_details": "+91-9123456780",
    "asset_type": "Motor Vehicle",
    "asset_id": "KA-01-MF-4521",
    "estimated_damage": 18000,
    "claim_type": "Property Damage",
    "attachments": "Photos of damage, Police FIR copy",
    "initial_estimate": 18000
  },
  "missingFields": [],
  "recommendedRoute": "Fast-track",
  "reasoning": "Estimated damage (18,000) is below the 25,000 threshold and all mandatory fields are present. Eligible for fast-track."
}
```

---

## 🧪 Sample Scenarios

| File | Scenario | Damage | Expected Route |
|------|----------|--------|----------------|
| `fnol_001.txt` | Rear-end motor accident | INR 18,000 | ⚡ Fast-track |
| `fnol_002.txt` | Slip-and-fall personal injury | INR 45,000 | 👨‍⚕️ Specialist Queue |
| `fnol_003.txt` | Suspicious collision (fraud keywords) | INR 95,000 | 🚩 Investigation Flag |
| `fnol_004.txt` | Kitchen fire — 4 fields missing | INR 62,000 | 📋 Manual Review |
| `fnol_005.txt` | Laptop theft | INR 22,500 | ⚡ Fast-track |

---

## 🧰 Technologies Used

| Technology | Purpose |
|------------|---------|
| Python 3.9+ | Core programming language |
| Groq API (free) | LLM inference engine |
| Llama 3.1 8B | Field extraction model |
| JSON | Structured output format |

---

## 📄 License

MIT License — free to use, modify, and distribute.

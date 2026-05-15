# Autonomous-Insurance-claims-Agent
Autonomous FNOL Claims Processing Agent that uses Groq's free Llama 3.1 LLM to extract structured fields from insurance documents, detect missing data, and intelligently route claims to Fast-track, Specialist Queue, Manual Review, or Investigation Flag workflows. Built with Python.
 Autonomous Insurance Claims Processing Agent
An AI-powered agent that processes First Notice of Loss (FNOL) documents — extracting key fields, detecting missing data, classifying claims, and routing them to the correct workflow using the free Groq API (Llama 3.1).

Architecture
FNOL Text (.txt / .pdf)
        │
        ▼
[1] Groq LLM extracts structured fields → JSON
        │
        ▼
[2] Rule engine checks mandatory fields → missingFields[]
        │
        ▼
[3] Routing rules applied (deterministic priority order):
        Investigation Flag  ← fraud keywords in description
        Specialist Queue    ← claim type = injury
        Manual Review       ← any mandatory field missing
        Fast-track          ← damage < 25,000 & all fields present
        Standard Queue      ← damage ≥ 25,000
        │
        ▼
[4] Output JSON saved to outputs/

 Project Structure
fnol-agent/
├── sample_fnols/            # Dummy FNOL documents (.txt)
│   ├── fnol_001.txt         # Auto damage    → Fast-track
│   ├── fnol_002.txt         # Personal injury → Specialist Queue
│   ├── fnol_003.txt         # Fraud signals   → Investigation Flag
│   ├── fnol_004.txt         # Missing fields  → Manual Review
│   └── fnol_005.txt         # Theft           → Fast-track
├── outputs/                 # Created at runtime — one JSON per FNOL
├── agent.py                 # Core agent (extraction + routing)
├── requirements.txt
└── README.md

Setup & Run
Prerequisites

Python 3.9+
A free Groq API key (no credit card needed)

1 — Clone the repository
bashgit clone https://github.com/<your-username>/fnol-agent.git
cd fnol-agent
2 — Install dependencies
bashpip install -r requirements.txt
3 — Set your Groq API key
bash# macOS / Linux
export GROQ_API_KEY="gsk_..."

# Windows (PowerShell)
$env:GROQ_API_KEY="gsk_..."
4 — Run the agent
bashpython agent.py
The agent processes every .txt and .pdf file inside sample_fnols/, prints a live summary to the console, and writes one JSON result file per FNOL into outputs/.

 Output Format
Each processed FNOL produces a JSON file in outputs/:
json{
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

 Routing Rules (Priority Order)
PriorityConditionRoute1 (highest)Description contains: fraud, inconsistent, staged, suspicious, fabricatedInvestigation Flag2claim_type = injurySpecialist Queue3Any mandatory field is null / missingManual Review4estimated_damage < 25,000 & all fields presentFast-track5 (default)estimated_damage ≥ 25,000Standard Queue

 Sample FNOL Scenarios
FileScenarioExpected Routefnol_001.txtRear-end motor accident — INR 18,000Fast-trackfnol_002.txtSlip-and-fall personal injury — INR 45,000Specialist Queuefnol_003.txtSuspicious collision with fraud keywordsInvestigation Flagfnol_004.txtKitchen fire — 4 mandatory fields missingManual Reviewfnol_005.txtLaptop theft — INR 22,500Fast-track

Approach

AI Extraction — Groq's Llama 3.1 8B model parses unstructured FNOL text into a strict JSON schema. Using an LLM handles messy, inconsistent free-text that regex alone cannot reliably process.
Deterministic Routing — All routing logic is plain Python (not delegated to the LLM), making it fully auditable, testable, and predictable.
Extensible Design — Add new routing rules inside route_claim(). Expand extraction fields by updating the prompt and MANDATORY_FIELDS list. Swap the model by changing the MODEL constant.


 Dependencies
groq
Install with:
bashpip install -r requirements.txt

📄 License
MIT

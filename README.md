# Multi-Domain-Support-Triage-Agent

This repository contains an AI-powered, terminal-based support triage agent built for the **HackerRank Orchestrate (May 2026)** hackathon. 

The agent is designed to autonomously process support tickets across three distinct domains: **HackerRank**, **Claude**, and **Visa**. It reads user issues, classifies the request type and product area, retrieves relevant documentation from a local knowledge base, and decides whether to safely resolve the issue or escalate it to a human agent.

## 🧠 Architecture & Strategy

To meet the strict evaluation criteria of grounding, safety, and determinism, this agent uses a modular **Retrieval-Augmented Generation (RAG)** architecture powered by the **Anthropic Claude API**.

### 1. Corpus Loader & BM25 Retriever
Instead of relying on the LLM's parametric memory, the agent dynamically parses the local `.mdux` and `.md` files located in the `data/` directory. We utilize **BM25 (Best Matching 25)** for fast, deterministic, keyword-based document retrieval. This ensures the agent only pulls highly relevant, domain-specific chunks of text to answer user queries.

### 2. Multi-Step Routing & Classification
When a ticket is ingested from the CSV, it passes through a routing layer:
* **Intent & Domain Classification**: The model analyzes the text to determine the `company`, `product_area`, and `request_type` (e.g., `product_issue`, `feature_request`, `bug`, `invalid`).
* **Cross-Domain Inference**: If the company is missing, the agent infers it based purely on the semantic context of the issue.

### 3. Safety-First Escalation Logic
The agent includes dedicated escalation guardrails. Before generating any response, the agent evaluates the ticket against strict safety policies. It automatically triggers an `escalated` status for:
* Billing, refund, or payment disputes.
* High-risk account access/override requests.
* Reports of fraud, security breaches, or system outages.
* Complex disputes requiring human empathy or administrative action.

### 4. Grounded Generation
If the ticket clears the safety checks and relevant documentation is found, the generation module constructs a helpful, non-hallucinated response explicitly citing the retrieved context. The orchestrator outputs a concise, traceable justification for every decision made.

---

## ⚙️ Complete Setup Guide

Follow these steps to get the agent running on your local machine.

### 1. Environment Setup
It is highly recommended to use a virtual environment to keep your dependencies isolated. Open your terminal, navigate to the root of the project folder, and run:

**For Windows:**
```powershell
python -m venv venv
.\venv\Scripts\activate
```

**For Mac/Linux:**
```bash
python3 -m venv venv
source venv/bin/activate
```
*(You will know it is activated when you see `(venv)` at the start of your terminal prompt).*

### 2. Generate and Configure Your API Key
This agent uses the Anthropic Claude API to power its reasoning and routing.
1. Go to the [Anthropic Console](https://console.anthropic.com/).
2. Sign in or create an account.
3. Navigate to **API Keys** and click **Create Key**.
4. In the root directory of this project (outside the `code/` folder), create a new file named `.env`.
5. Add your newly generated key to the `.env` file like this:
   ```env
   ANTHROPIC_API_KEY=sk-ant-api03-your_actual_api_key_here
   ```
*(Note: The `.env` file is included in the `.gitignore` so your key will remain secure and will not be uploaded to GitHub).*

### 3. Install Dependencies
With your virtual environment activated, install the required Python packages (Anthropic, pandas, rank_bm25, and python-dotenv) by running:
```bash
pip install -r code/requirements.txt
```

### 4. Running the Agent
Once your environment is set up and your API key is in place, you can execute the orchestrator. Make sure you are in the root directory of the project, then run:
```bash
python code/main.py
```

### 5. How It Gives Output
When you run the script, the agent will:
1. Load all the `.mdux` and `.md` files from the `data/` folder to build its knowledge base.
2. Read the incoming tickets from `support_tickets/support_tickets.csv` (and `support_tickets/samples.csv` if testing).
3. Process each ticket one by one (classifying, checking safety rules, and retrieving context).
4. **Generate the Output:** The final results are automatically saved into a new file located at `support_tickets/output.csv`. 

You can open `output.csv` to see the agent's final decisions, including the `status` (escalated or replied), `product_area`, the generated `response`, and the `justification` for its actions.
```

## 📂 Repository Structure
```text
hackerrank-orchestrate-may26-main/
├── .env                   # Environment variables (API keys)
├── AGENTS.md              # Global agent rules and logging protocols
├── evalutation_criteria.md
├── problem_statement.md
├── README.md              # Project documentation
├── code/
│   ├── __pycache__/
│   ├── agent.py
│   ├── classifier.py
│   ├── confidence_scorer.py
│   ├── corpus_loader.py
│   ├── domain_escalation.py
│   ├── escalation.py
│   ├── generator.py
│   ├── main.py
│   ├── query_expander.py
│   ├── retriever.py
│   ├── utils.py
│   ├── README.md          # Code-specific documentation
│   └── requirements.txt   # Python dependencies
├── data/                  # Knowledge base
│   ├── claude/
│   ├── hackerrank/
│   └── visa/
└── support_tickets/       # Inputs and outputs
    ├── output.csv         # Generated by the agent
    ├── sample_support_tickets.csv
    └── support_tickets.csv

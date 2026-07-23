# automated-email-respond
🤖 Enterprise Mail Automation & Human-in-the-Loop Triage Engine (n8n + OpenAI)
A production-grade, fault-tolerant n8n automation workflow designed to ingest, k-categorize, and process incoming corporate emails using LLM reasoning, interactive Telegram user-in-the-loop (HITL) approval loops, and automated record-keeping.

⚡ Technical Architecture & Flow
[Gmail Trigger] ➔ [Auto-Ack No-Reply] ➔ [OpenAI Classifier & Structurer] ➔ [Switch: Routing]
                                                                                     │
     ┌───────────────────────────┬───────────────────────────-───────────────────────┴──────────────────────┐
     ▼                           ▼                                                                          ▼
[P1/P2 Incidents]        [Offers / Deals]                                                         [Fallback / Low-Prio]
  │                        │                                                                               │
  ├─► [Telegram Form]      ├─► [Telegram Form]                                                             ├─► [Telegram Log]
  │     │                    │                                                                             │
  │     ├─► Approved         │     ├─► Approved                                                            ├─► [Gmail: Mark Read]
  │     │     └─► [Gmail]    │     │     └─► [Gmail]                                                       │
  │     │                    │     │                                                                       └─► [Sheets: Archive]
  │     └─► Feedback Loop    │     └─► Feedback Loop
  │           └─► [OpenAI]   │           └─► [OpenAI]
  │                 └──(Loop)│                 └──(Loop)

🎯 Key Technical Capabilities
1. Zero-Latency Initial Response
•	Trigger: Listens to incoming threads via Gmail Trigger.
•	Immediate Action: Dispatches an automated, lightweight acknowledgment email prior to heavy LLM processing to satisfy SLA time-to-first-response metrics.
2. Structured LLM Categorization (JSON Mode)
•	Engine: OpenAI Chat Completions API enforcing response_format: { type: "json_object" }.
•	Output Schema: Extracts category, sentiment, summary, and generates suggested_reply.
3. State-Aware Fallback Parsing (Telegram Approval & Re-prompting)
•	HITL Loop: Sends interactive forms via Telegram to operators.
•	Dynamic Evaluation Expression: Solves variable path divergence between primary generation and iterative re-prompts:
{{ $json.suggested_reply || $json.output?.[0]?.content?.[0]?.text?.suggested_reply || $('Message a model').item.json.output[0].content[0].text.suggested_reply }}

•	Correction Loop: Accepts operator feedback (Ewentualne uwagi) from Telegram submission, injects it back to OpenAI, and re-executes the Telegram prompt until approved.
4. Deterministic Multi-Branch Routing (Switch Node)
•	P1 / P2 Branch: Interactive approval ‭$\rightarrow$‬ Feedback Loop ‭$\rightarrow$‬ Outbound Gmail.
•	Offer/diff Branch: Interactive offer review ‭$\rightarrow$‬ Manual Override / AI Tuning ‭$\rightarrow$‬ CRM Append (Szanse_Sprzedażowe).
•	Reclamation Branch: Direct escalation ‭$\rightarrow$‬ P1 Telegram Alert ‭$\rightarrow$‬ Gmail Flagging ‭$\rightarrow$‬ Audit Log (Reklamacje).
•	Fallback Branch: Asynchronous logging ‭$\rightarrow$‬ Background Telegram notification ‭$\rightarrow$‬ Silently archived (Archiwum_Mails).
🛠️ Tech Stack & Integrations
Service	Purpose	API / Interface
n8n	Workflow Orchestration	Self-Hosted / Cloud Engine
OpenAI	Sentiment Analysis, Summarization, Drafting	Chat Completions API (gpt-4o / gpt-4o-mini)
Telegram	Human-in-the-Loop Interface	Telegram Bot API (Inline Keyboards / Forms)
Gmail	Email Ingestion & Transport	Google OAuth2
Google Sheets	Operational Data Storage & Logging	Google Sheets API v4
📂 Repository Structure
.
├── docs/
│   └── workflow-architecture.png   # iPad annotated workflow diagram
├── workflows/
│   └── mail-triage-workflow.json  # Complete executable n8n workflow export
├── .gitignore
├── LICENSE                         # MIT License
└── README.md

⚙️ Setup & Deployment
	1.	Import Workflow:
•	Open n8n ‭$\rightarrow$‬ Workflows ‭$\rightarrow$‬ Import from File ‭$\rightarrow$‬ Select workflows/mail-triage-workflow.json.
	2.	Environment & Credentials:
Configure the following connections in your n8n instance:
•	Gmail OAuth2 API
•	OpenAI API Key
•	Telegram Bot API Token
•	Google Sheets OAuth2 API
	3.	Google Sheets Schema:
Ensure your connected Google Sheet contains the following worksheets:
•	Reklamacje (Columns: ID, Timestamp, From, Subject, Status)
•	Szanse_Sprzedażowe (Columns: ID, Timestamp, From, Value, Status)
•	Archiwum_Mails (Columns: Timestamp, From, Subject, Category)
📄 License
This project is released under the MIT License. Feel free to modify, distribute, and integrate into commercial workflows.

# 🧵 ThreadKeep

> **Keep every thread accountable.**

ThreadKeep is an AI-powered work signal extraction engine that monitors Gmail, Slack, and WhatsApp — automatically extracting tasks, decisions, commitments, blockers, meetings, and FYIs from conversations using Claude API.

---

## The Problem

Indian professionals use personal WhatsApp + Gmail + Slack simultaneously for work. Critical tasks, decisions, and commitments get buried in chats. **"Maine kab bola?"** (when did I say that?) is the escalation moment ThreadKeep prevents.

---

## How It Works

```
Gmail ──────────────────────────────────────────┐
Slack ──────────────────────────────────────────┤──► Claude API ──► Google Sheets ──► Dashboard
WhatsApp (via GREEN-API forwarding) ────────────┘
```

1. **Ingest** — Gmail (OAuth), Slack (webhook), WhatsApp (GREEN-API)
2. **Extract** — Claude API identifies Tasks, Decisions, Commitments, Blockers, Meetings, FYIs
3. **Log** — Signals saved to Google Sheets with type, summary, person, deadline, priority, confidence, source, sender, timestamp, status, ID
4. **Surface** — Live dashboard + WhatsApp SimSim command for instant status

---

## Features

- ✅ Multi-source ingestion (Gmail × 2, Slack, WhatsApp)
- ✅ AI signal extraction (Tasks, Decisions, Commitments, Blockers, Meetings, FYIs)
- ✅ Hinglish understanding ("kal tak bhej dena" → deadline: tomorrow)
- ✅ Live dashboard with filters, search, dark/light mode, privacy mode
- ✅ Signal close/open with Google Sheets sync
- ✅ SimSim WhatsApp command → instant status report
- ✅ Deployed on Railway (24/7 uptime)

---

## Tech Stack

| Layer | Tool |
|---|---|
| Automation | n8n (self-hosted on Railway) |
| AI Engine | Claude API (claude-sonnet-4-6) |
| Gmail | Gmail API via OAuth 2.0 |
| Slack | Slack Webhook |
| WhatsApp | GREEN-API |
| Database | Google Sheets |
| Dashboard | Vanilla HTML/CSS/JS |
| Hosting | Railway (n8n) + GitHub Pages (dashboard) |

---

## Repository Structure

```
threadkeep/
├── index.html              # Live dashboard
├── n8n-workflow.json       # Main n8n workflow export
├── README.md               # This file
├── SETUP.md                # Full technical setup guide
├── .gitignore              # Files to exclude
└── docs/
    └── architecture.md     # System architecture notes
```

---

## Quick Start

See [SETUP.md](./SETUP.md) for full setup instructions.

---

## Author

**Om Shubham**
IIM Kozhikode | NIT Allahabad
omshubham99@gmail.com

---

## Status

🟢 MVP Live — Phase 0 (Validation)
Target: 10 test users by June 2026

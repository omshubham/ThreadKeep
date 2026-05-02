# ThreadKeep — System Architecture

## Overview

ThreadKeep is a no-code/low-code AI agent built on n8n + Claude API. It ingests messages from three sources, extracts work signals, and surfaces them via a dashboard and WhatsApp.

---

## Data Flow

```
┌─────────────────────────────────────────────────────────────┐
│                        INGESTION LAYER                       │
├─────────────────┬───────────────────┬───────────────────────┤
│   Gmail OAuth   │   Slack Webhook   │  WhatsApp (GREEN-API) │
│  (2 accounts)   │                   │   forwarding-based    │
└────────┬────────┴─────────┬─────────┴───────────┬───────────┘
         │                  │                     │
         ▼                  ▼                     ▼
┌─────────────────────────────────────────────────────────────┐
│                     n8n ORCHESTRATION                        │
│  Set source label → Format message → Route to Claude        │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                      CLAUDE API                              │
│  claude-sonnet-4-6 · Extracts signals from raw messages     │
│  Handles English + Hinglish · Returns structured JSON       │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                    STORAGE LAYER                             │
│              Google Sheets (ThreadKeep Dashboard)            │
│  Type│Summary│Person│Deadline│Priority│Confidence│Source│   │
│  Sender│Timestamp│Status│ID                                  │
└──────────────┬──────────────────────────┬───────────────────┘
               │                          │
               ▼                          ▼
┌──────────────────────┐    ┌─────────────────────────────────┐
│   DASHBOARD (HTML)   │    │      WHATSAPP REPLY             │
│  GitHub Pages hosted │    │  SimSim → status report         │
│  Fetches via n8n     │    │  GREEN-API send message API     │
│  webhook endpoint    │    │                                 │
└──────────────────────┘    └─────────────────────────────────┘
```

---

## Signal Types

| Type | Example |
|---|---|
| TASK | "Send the deck by EOD" |
| DECISION | "We're going with vendor B" |
| COMMITMENT | "Main le leta hoon" (I'll handle it) |
| BLOCKER | "Can't proceed without API access" |
| MEETING | "Let's connect at 5pm tomorrow" |
| FYI | "FYI the client moved the deadline" |

---

## n8n Workflow Structure

```
Gmail-1 ──► Set Gmail Label ──────────────────────┐
Gmail-2  ──► Set Gmail Label2 ─────────────────────┤
                                                               ▼
WhatsApp webhook ──► IF (SimSim?) ──► TRUE ──► Fetch Sheets ──► Code (format) ──► GREEN-API reply
                         │
                         └──► FALSE ──────────────────────────┤
                                                               ▼
Slack webhook ──► Get Slack User ──────────────────────────────┤
                                                               ▼
                                                        Claude LLM
                                                               │
                                                               ▼
                                                      Structure Data (JS)
                                                               │
                                                               ▼
                                                      Add Info in Sheet
                                                      
──────────────────── SEPARATE ENDPOINTS ────────────────────────

Webhook GET /threadkeep-data ──► Read Sheet ──► Respond (dashboard)
Webhook POST /threadkeep-status ──► Update Sheet row ──► Respond
```

---

## Key Design Decisions

1. **Forwarding-based WhatsApp** — No personal WhatsApp API exists. Users forward messages to ThreadKeep's dedicated number. GREEN-API bridges the gap.

2. **Google Sheets as DB** — Chosen for MVP speed and zero setup. Migration path to Supabase (Postgres) for scale.

3. **Separate credentials per Gmail** — Prevents cross-account signal duplication.

4. **ID-based status sync** — Each signal gets a UUID on creation. Dashboard uses this ID to update the exact row in Sheets when closed/opened.

5. **SimSim command** — Secret word triggers instant WhatsApp status report. No app needed.

---

## Deployment

| Service | Platform | URL |
|---|---|---|
| n8n | Railway | https://railway.com/ |
| Dashboard | GitHub Pages | YOUR-USERNAME.github.io/threadkeep-dashboard |
| Database | Google Sheets | ThreadKeep Dashboard (Sheet1) |
| WhatsApp API | GREEN-API | https://green-api.com/ |

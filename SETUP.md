# ThreadKeep — Technical Setup Guide

Complete instructions to set up ThreadKeep from scratch.

---

## Prerequisites

- Google account (for Gmail + Google Sheets + Google Cloud)
- Slack workspace
- WhatsApp number (for GREEN-API)
- Anthropic API key (console.anthropic.com)
- Railway account (railway.app)
- GREEN-API account (green-api.com)

---

## 1. Google Cloud Setup

### 1.1 Create Project
1. Go to console.cloud.google.com
2. Create new project → name it "ThreadKeep"
3. Enable APIs: Gmail API, Google Sheets API, Google Drive API

### 1.2 OAuth Credentials
1. APIs & Services → Credentials → Create OAuth Client ID
2. Type: Web Application
3. Name: ThreadKeep n8n
4. Authorized redirect URIs:
   - `http://localhost:5678/rest/oauth2-credential/callback` (for local)
   - `https://YOUR-RAILWAY-URL/rest/oauth2-credential/callback` (for production)
5. Save Client ID and Client Secret

### 1.3 Test Users
Add your Gmail addresses as test users in OAuth consent screen.

---

## 2. Google Sheets Setup

Create a Google Sheet called **"ThreadKeep Dashboard"** with Sheet1 having these columns:

| A | B | C | D | E | F | G | H | I | J | K |
|---|---|---|---|---|---|---|---|---|---|---|
| Type | Summary | Person | Deadline | Priority | Confidence | Source | Sender | Timestamp | Status | ID |

---

## 3. n8n Setup on Railway

### 3.1 Deploy
1. Go to railway.app → New Project → Template → search "n8n"
2. Use the shinyduo/n8n-railway-updated template
3. Deploy

### 3.2 Import Workflow
1. Open your Railway n8n URL
2. Create account
3. Import `n8n-workflow.json` from this repo

### 3.3 Set Up Credentials
In n8n, create credentials for:
- **Google Sheets** — OAuth with your Google account
- **Gmail** — OAuth (one credential per Gmail account)
- **Slack** — Webhook URL

---

## 4. Claude API Setup

The Claude API is called via HTTP Request node (not n8n credential).

In the Claude LLM node, the header `x-api-key` contains your Anthropic API key.
Update this with your key from console.anthropic.com.

### Claude System Prompt
```
You are ThreadKeep, an intelligent work conversation analyst for Indian professionals.

IMPORTANT SOURCE RULES:
- Every message starts with a source prefix like: 'slack | sender: Name | message: ...'
- For Gmail messages: use the EXACT source label from the prefix as the source field
- For Slack messages: use 'slack' as source
- For WhatsApp messages: use 'whatsapp' as source

Signal types: TASK, DECISION, COMMITMENT, BLOCKER, MEETING, FYI

Return ONLY JSON:
{
  signals: [{ type, summary, person, deadline, priority, confidence }],
  source: (exact label from prefix),
  original_sender: name or null
}

Hinglish: kal tak=tomorrow, aaj EOD=end of today, jaldi=urgent, main le leta hoon=commitment
```

---

## 5. GREEN-API Setup (WhatsApp)

1. Sign up at green-api.com
2. Create instance → Developer plan (free)
3. Scan QR code with WhatsApp on your dedicated number
4. In instance settings:
   - Webhook URL: `https://YOUR-RAILWAY-URL/webhook/whatsapp-threadkeep`
   - incomingWebhook: ON
   - outgoingAPIMessageWebhook: OFF
5. Note your `idInstance` and `apiTokenInstance`

---

## 6. Slack Setup

1. Go to api.slack.com/apps → Create App
2. App name: ThreadKeep
3. Scopes: channels:history, channels:read, users:read
4. Event Subscriptions → Request URL: `https://YOUR-RAILWAY-URL/webhook/slack-threadkeep`
5. Subscribe to: message.channels

---

## 7. Dashboard Setup

1. Update `index.html` URLs:
```javascript
var N8N_URL = 'https://YOUR-RAILWAY-URL/webhook/threadkeep-data';
var N8N_STATUS_URL = 'https://YOUR-RAILWAY-URL/webhook/threadkeep-status';
```
2. Host on GitHub Pages or any static hosting

---

## 8. Webhook Endpoints

| Endpoint | Method | Purpose |
|---|---|---|
| `/webhook/threadkeep-data` | GET | Dashboard fetches all signals |
| `/webhook/threadkeep-status` | POST | Update signal status (open/closed) |
| `/webhook/whatsapp-threadkeep` | POST | Receive WhatsApp messages |
| `/webhook/slack-threadkeep` | POST | Receive Slack messages |

---

## 9. SimSim Command

Send **"simsim"** (any case) to your ThreadKeep WhatsApp number to get an instant status report of all open signals.

---

## Environment Variables (Railway)

No additional environment variables needed beyond the template defaults.
All API keys are stored as n8n credentials within the platform.

---

## Troubleshooting

**Webhook 404** — Make sure workflow is activated (green toggle in n8n)

**Gmail duplicate signals** — Ensure each Gmail trigger uses a separate OAuth credential

**WhatsApp loop** — Turn OFF outgoingAPIMessageWebhook in GREEN-API settings

**Slack challenge failed** — Add IF node for url_verification before processing messages

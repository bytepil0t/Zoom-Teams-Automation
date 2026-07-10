# Zoom Post-Meeting Automation Project

## Project Overview

**Client:** Shane (@differentlygroup)  
**Platform:** Freelancer.com  
**Budget:** $500 AUD (funded ✅)  
**Milestone:** $500 AUD (Initial Milestone Payment)  
**Contractor:** Sean Paul Payne Dinwiddie  
**Email:** mindmagic.pluse@gmail.com  
**Status:** In Progress

---

## Project Requirements

The client needs a post-meeting automation workflow connecting:
- **Zoom** (primary) → Microsoft Teams (secondary, phase 2)
- **Claude AI** (via Anthropic API) for processing transcripts
- **Outlook** for draft emails and handover notes
- **GoHighLevel (GHL)** for logging meeting notes to contacts

### Workflow Goals
1. Zoom meeting ends (titled "Debt-Free Formula")
2. Transcript is automatically retrieved
3. Transcript sent to Claude AI with a detailed prompt
4. Claude generates:
   - Client follow-up email
   - Client action steps
   - Meeting holder action steps
   - Internal handover notes for Vanessa (Credit Analyst)
   - Key tasks, next steps, important client info
5. Client follow-up email created as **draft in Outlook** (Kat's account)
6. Handover notes emailed to Vanessa
7. Meeting notes/summary added to GoHighLevel contact/opportunity as a **Note**

### Decisions Made
| Item | Decision |
|---|---|
| Platform | Make.com |
| Priority | Zoom first, Teams second |
| Trigger | Meetings titled "Debt-Free Formula" |
| AI Tool | Claude (Anthropic API) — claude-sonnet-4-5 |
| GHL Logging | Note on Contact or Opportunity |
| Handover format | Plain text in email body (not PDF) |
| Transcript storage | Google Drive (optional, to be added) |

---

## Credentials

> ⚠️ Keep these secure. Do not share publicly. Store in a password manager or secrets vault.

| Item | Value |
|---|---|
| Zoom Account ID | `YOUR_ZOOM_ACCOUNT_ID` |
| Zoom Client ID | `YOUR_ZOOM_CLIENT_ID` |
| Zoom Client Secret | `YOUR_ZOOM_CLIENT_SECRET` |
| Anthropic API Key | `YOUR_ANTHROPIC_API_KEY` |
| Zoom Webhook URL | `YOUR_MAKE_WEBHOOK_URL` |
| Make.com account | `shane@differently.au` |
| Contractor Make.com email | `mindmagic.pluse@gmail.com` |

---

## Zoom App Setup

- **App name:** FD Zoom Transcript Automation
- **App type:** Server-to-Server OAuth
- **Account level:** Yes
- **Event subscription name:** Zoom Recording Trigger
- **Webhook URL:** `YOUR_MAKE_WEBHOOK_URL`
- **Event:** `recording.completed` (All Recordings have completed)
- **Status:** ✅ Validated and Activated

---

## Make.com Scenario

**Scenario name:** Integration Webhooks, HTTP, Microsoft 365 Email (Outlook)

### Modules Built

| # | Module | Type | Details |
|---|---|---|---|
| 2 | Webhooks | Custom webhook | Receives `recording.completed` from Zoom |
| — | Filter | Debt-Free Formula only | `topic` contains "Debt-Free Formula" |
| 4 | HTTP | POST /oauth/token | Gets Zoom access token using Base64 Basic auth |
| 5 | HTTP | GET Make a request | Gets recordings list: `https://api.zoom.us/v2/meetings/{{2.payload.object.uuid}}/recordings` |
| 6 | HTTP | GET Make a request | Downloads transcript file: `{{5.recording_files[]: download_url}}` |
| 7 | HTTP | POST /v1/messages | Sends transcript to Claude AI (placeholder prompt) |
| 8 | Microsoft 365 Email | Create a Draft Email | Creates Outlook draft in Kat's account |

### HTTP 4 — Get Zoom Access Token
- URL: `https://zoom.us/oauth/token`
- Method: POST
- Header: `Authorization: Basic YOUR_BASE64_ENCODED_CLIENT_ID_SECRET`
- Body: `grant_type=account_credentials&account_id=YOUR_ZOOM_ACCOUNT_ID`
- Parse response: Yes

### HTTP 5 — Get Recordings List
- URL: `https://api.zoom.us/v2/meetings/{{2.payload.object.uuid}}/recordings`
- Method: GET
- Header: `Authorization: Bearer {{4.access_token}}`
- Parse response: Yes

### HTTP 6 — Download Transcript
- URL: `{{5.recording_files[]: download_url}}`
- Method: GET
- Header: `Authorization: Bearer {{4.access_token}}`
- Parse response: No

### HTTP 7 — Claude AI
- URL: `https://api.anthropic.com/v1/messages`
- Method: POST
- Headers:
  - `x-api-key`: `YOUR_ANTHROPIC_API_KEY`
  - `anthropic-version`: `2023-06-01`
  - `content-type`: `application/json`
- Body:
```json
{
  "model": "claude-sonnet-4-5",
  "max_tokens": 4096,
  "messages": [
    {
      "role": "user",
      "content": "PLACEHOLDER PROMPT - transcript will go here: {{6.Data}}"
    }
  ]
}
```
- Parse response: Yes

### Microsoft 365 Email (Outlook) — Create Draft
- Connection: Kat's Outlook (Kat Marsal...) ✅ Connected
- To Recipients: TBD (need client's recipient email)
- Subject: `Follow Up - {{2.payload.object.topic}} - {{formatDate(2.payload.object.start_time; "DD/MM/YYYY")}}`
- Body Content: `{{7.content[].text}}`
- Status: ⚠️ Subject and Body need to be updated with mapped values

---

## Still Needed From Client

| Item | Status |
|---|---|
| AI prompt (detailed Claude instructions) | ❌ Pending |
| GoHighLevel API key | ❌ Pending (client looking for it) |
| Vanessa's email address | ❌ Pending |
| Outlook Subject/Body mapping confirmed | ⚠️ Needs update |

---

## Still To Build

- [ ] Update Outlook Subject and Body Content with Claude output mapping
- [ ] Add GoHighLevel module — post meeting notes as Note on contact
- [ ] Add second email module — send handover notes to Vanessa
- [ ] Replace placeholder Claude prompt with real client prompt
- [ ] Test full workflow end-to-end with a real or dummy transcript
- [ ] (Optional) Add Google Drive module to save transcript
- [ ] Phase 2: Microsoft Teams workflow (same structure)

---

## Key Contacts

| Person | Role | Details |
|---|---|---|
| Shane | Client / Owner | @differentlygroup, shane@differently.au |
| Kat | Team member (Zoom user) | Outlook connected: Kat Marsal... |
| Vanessa | Credit Analyst | Receives internal handover notes |

---

## Communication Notes

- All communication must stay on the Freelancer platform (no WhatsApp/external)
- Client is happy with plain text handover notes (not PDF)
- Client open to Google Drive for transcript storage
- Client confirmed both Zoom and Teams have cloud recording + transcription enabled
- Make.com Free plan (1,000 ops/month) — recommend upgrade to Core ($9/month) after testing

---

## Timeline

- Zoom workflow: 2-3 business days (agreed)
- Teams workflow: after Zoom is working

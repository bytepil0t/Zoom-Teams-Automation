# Zoom Post-Meeting Automation Project

## Project Overview

**Client:** Shane (@differentlygroup)  
**Platform:** Freelancer.com  
**Budget:** $500 AUD (funded ✅)  
**Milestone:** $500 AUD (Initial Milestone Payment)  
**Contractor:** Sean Paul Payne Dinwiddie  
**Email:** mindmagic.pluse@gmail.com  
**Status:** In Progress — Build nearly complete, pending test

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
   - Client follow-up email (in Kat's voice)
   - Internal handover notes for Vanessa (Credit Analyst)
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
| GHL Logging | Note on Contact (matched by meeting topic search) |
| Handover format | Plain text in email body (not PDF) |
| Transcript storage | SharePoint (optional, to be confirmed) |

---

## Credentials

> ⚠️ Keep these secure. Do not share publicly.

| Item | Value |
|---|---|
| Zoom Account ID | `YOUR_ZOOM_ACCOUNT_ID` |
| Zoom Client ID | `YOUR_ZOOM_CLIENT_ID` |
| Zoom Client Secret | `YOUR_ZOOM_CLIENT_SECRET` |
| Anthropic API Key | `YOUR_ANTHROPIC_API_KEY` |
| Zoom Webhook URL | `YOUR_MAKE_WEBHOOK_URL` |
| Make.com account | `shane@differently.au` |
| Contractor Make.com email | `mindmagic.pluse@gmail.com` |
| GoHighLevel API Key | `YOUR_GHL_API_KEY` |
| GHL Location ID | `PENDING — waiting from Shane` |

---

## Make.com Scenario — Full Module List

**Scenario name:** Integration Webhooks, HTTP, Microsoft 365 Email (Outlook)

| Module # | Type | Description | Status |
|---|---|---|---|
| 2 | Webhooks | Receives `recording.completed` from Zoom | ✅ Done |
| — | Filter | Only "Debt-Free Formula" meetings | ✅ Done |
| 4 | HTTP POST | Get Zoom OAuth access token | ✅ Done |
| 5 | HTTP GET | Get recordings list for meeting | ✅ Done |
| 6 | HTTP GET | Download transcript file | ✅ Done |
| 7 | HTTP POST | Claude AI — generate client follow-up email | ✅ Done |
| 9 | HTTP POST | Claude AI — generate Vanessa handover notes | ✅ Done |
| 10 | Outlook | Create draft email in Kat's inbox | ✅ Done |
| 13 | Outlook | Send handover email to Vanessa | ⚠️ Needs Kat to re-authorise Outlook |
| 19 | HTTP POST | Search GHL contacts by meeting topic | ⚠️ Needs GHL Location ID from Shane |
| 22 | HTTP POST | Post note to GHL contact | ⚠️ Body mapping needs update after first test run |

---

## Still Needed From Client

| Item | Status |
|---|---|
| GHL Location ID | ❌ Pending — asked Shane |
| Kat to re-authorise Outlook (Send permission) | ❌ Pending — asked Shane |
| How Kat names Zoom meetings (client name in title?) | ❌ Pending — asked Shane |

---

## Still To Do

- [ ] Shane to provide GHL Location ID → update module 19 body
- [ ] Kat to re-authorise Outlook connection → module 13 will work
- [ ] After first test run — update module 22 body with `{{7.content[]: text}}` variable
- [ ] End-to-end test with a real Zoom recording
- [ ] (Optional) Add SharePoint module to save transcript
- [ ] Phase 2: Microsoft Teams workflow

---

## Key Contacts

| Person | Role | Details |
|---|---|---|
| Shane | Client / Owner | shane@differently.au |
| Kat | Zoom / Outlook user | kat@fdly.au |
| Vanessa | Credit Analyst | vanessa@fdly.au |

---

## Timeline

- Zoom workflow: 2-3 business days (agreed) — nearly complete
- Teams workflow: after Zoom is working and tested

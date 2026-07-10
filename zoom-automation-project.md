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
| GHL Logging | Note on Contact or Opportunity |
| Handover format | Plain text in email body (not PDF) |
| Transcript storage | Google Drive / SharePoint (optional, to be confirmed) |

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
| GoHighLevel API Key | `YOUR_GHL_API_KEY` |

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
| 7 | HTTP | POST /v1/messages | Sends transcript to Claude AI — **Kat's client email prompt** |
| 8 | HTTP | POST /v1/messages | Sends transcript to Claude AI — **Vanessa's handover notes prompt** |
| 9 | Microsoft 365 Email | Create a Draft Email | Creates Outlook draft in Kat's account (client follow-up) |
| 10 | Microsoft 365 Email | Send an Email | Sends handover notes to Vanessa |
| 11 | HTTP | POST to GHL API | Adds meeting note to GHL contact |

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

### HTTP 7 — Claude AI (Kat's Client Email)
- URL: `https://api.anthropic.com/v1/messages`
- Method: POST
- Headers:
  - `x-api-key`: `YOUR_ANTHROPIC_API_KEY`
  - `anthropic-version`: `2023-06-01`
  - `content-type`: `application/json`
- Body: See **Claude Prompt 1** below
- Parse response: Yes
- Output: `7.content[].text`

### HTTP 8 — Claude AI (Vanessa's Handover Notes)
- URL: `https://api.anthropic.com/v1/messages`
- Method: POST
- Headers:
  - `x-api-key`: `YOUR_ANTHROPIC_API_KEY`
  - `anthropic-version`: `2023-06-01`
  - `content-type`: `application/json`
- Body: See **Claude Prompt 2** below
- Parse response: Yes
- Output: `8.content[].text`

### Microsoft 365 Email — Module 9 (Kat's Draft to Client)
- Connection: Kat's Outlook (`kat@fdly.au`) ✅ Connected
- To Recipients: `{{2.payload.object.host_email}}` (or hardcoded `kat@fdly.au`)
- Subject: `Follow Up - {{2.payload.object.topic}} - {{formatDate(2.payload.object.start_time; "DD/MM/YYYY")}}`
- Body Content: `{{7.content[].text}}`
- Type: **Create Draft** (not send)

### Microsoft 365 Email — Module 10 (Handover to Vanessa)
- Connection: Kat's Outlook (`kat@fdly.au`) ✅ Connected
- To Recipients: `vanessa@fdly.au`
- Subject: `Credit Handover - {{2.payload.object.topic}} - {{formatDate(2.payload.object.start_time; "DD/MM/YYYY")}}`
- Body Content: `{{8.content[].text}}`
- Type: **Send** (not draft)

### HTTP 11 — GoHighLevel Note
- URL: `https://services.leadconnectorhq.com/contacts/{contactId}/notes` *(TBD — need to map contact)*
- Method: POST
- Headers:
  - `Authorization: Bearer YOUR_GHL_API_KEY`
  - `Content-Type: application/json`
  - `Version: 2021-07-28`
- Body:
```json
{
  "body": "{{7.content[].text}}"
}
```
- Status: ⚠️ Needs contact ID mapping logic

---

## Claude Prompts

### Claude Prompt 1 — Kat's Client Follow-Up Email

```
You are writing a client follow-up email after a Finance Differently Deep Dive meeting, in Kat's voice, from the transcript below.

FIRST work these out from the transcript, and do NOT print this:
- The client's goal and the plan or outcome for them.
- The 2 to 4 things worth recapping, in plain client language.
- The client's next steps, and Kat / FD's next steps.
- Anything Kat promised to send.
- Whether there is a genuine Debt Free Formula angle worth reinforcing.

The client's financials, servicing and borrowing capacity are in Quickli, and the calculator work is in Figura. Do not rebuild those, the client does not need them.

THEN write the email in this order, most important first:
1. One warm line of thanks, plus a genuine personal nod from the meeting.
2. The headline: where this leaves them, in one or two lines. The good news or the plan.
3. What we covered: 2 to 4 short outcome bullets, in their words.
4. Over to you: the client's next steps, few and clear.
5. What I'll take care of: Kat and FD steps, so they feel the load is shared.
6. One warm close with a single next action and an open door for questions.

RULES
- Voice: Kat's, per the voice guide below. Australian spelling. No em dashes. No "I hope this email finds you well". Go easy on exclamation marks.
- Short. Readable in under a minute. Bullets, not paragraphs. Most important first.
- Numbers: keep some but light and directional (around, roughly). Do not make it number-heavy. Never restate income, servicing or borrowing capacity.
- Reinforce the Debt Free Formula lightly and only where it fits: the idea that they get a plan and a team, not just a loan. On a pure transaction keep it to a soft line, on a strategy meeting lean in a little more. Never make it a pitch.
- Match the warmth to the meeting. A separation or other stressful file gets calm and reassuring, with a line that acknowledges it is a lot. A straightforward one can be brisk and can-do.
- Translate internal mechanics into plain language. "Order valuations, check servicing" becomes "I'll sort the numbers and the bank side".
- Reflect their goal back in their own words.
- If Kat promised to send something, say it is on its way.
- Compliance, important: do not promise approval, do not guarantee a specific rate, and do not give tax or legal advice. Never put a grey-area structuring or tax idea in writing. Redirect those to "let's talk it through" or "your accountant can confirm the tax side". A light, natural "subject to formal approval" is fine where it belongs.
- Subject line: warm and clear. Start with "Hi [first name]," (or both names for a couple). End with "Kind regards, Kat".

TRANSCRIPT:
{{6.Data}}
```

---

### Claude Prompt 2 — Vanessa's Internal Credit Handover Notes

```
You are preparing an internal credit handover email for Finance Differently, an Australian mortgage broking business.

WHO IT IS FOR
- The email goes to Vanessa, our credit analyst (vanessa@fdly.au). She sets up the loan after the client's Deep Dive meeting with Kat.
- Vanessa already has the client's financials, servicing and borrowing capacity in Quickli, and the calculator work in Figura. Do NOT repeat income, expenses, servicing or borrowing-capacity numbers, and do NOT include any Debt Free Formula or extra-repayment projections. She does not need them.
- Assume Vanessa has the numbers but not the story. Give her enough context to make sense of the credit task, no more.

WHAT VANESSA NEEDS MOST
- The type of loan and its purpose.
- The lender, if Kat named or leaned toward one or a few.
- The features and conditions the loan must carry, so her research covers every point.

RULES
- Lead line: start the body with one or two lines that name the file type and the single biggest thing to check for lender appetite.
- Complexity: scan for and surface anything that makes the file non-standard.
- Property labels: give each property one clear label the first time it appears and use it consistently.
- Lender: if Kat highlighted a lender or a shortlist, name them. Always list the required features as well.
- Numbers: include only loan-shaping figures (amount sought, LVR band, cash-out or payout amounts, property values where they define the deal).
- Separate unknowns from actions: put open questions under NEEDS CONFIRMING and tasks under CLIENT TO DO or FD / KAT TO DO.
- Documents: under DOCUMENTS TO CHASE, list only the paperwork this file type needs.
- If something was not covered in the meeting, write "Not discussed" rather than guessing.
- Australian spelling. No em dashes. Direct, practical, internal tone. Short fragments, not paragraphs. Put section labels in capitals.

OUTPUT
Return the email exactly in this format:

Subject: Credit handover — [client name(s)] — [scenario in 3 to 5 words]

Hi Vanessa,

[Lead line: file type + the one or two things to check first. One or two lines only.]

SNAPSHOT
- Client:
- File type:
- Loan purpose:
- Owner-occupied / Investment:
- Indicative lender:
- Timeline / urgency:

WHO THEY ARE
- [2 to 3 short lines, employment type first]

WHAT THEY WANT
- [1 to 2 lines, plain English]

PROPERTIES [only if more than one property or a title change]
- [labelled: value and amount owing, and what happens to it]

LOAN & STRUCTURE
- Type / purpose:
- Amount sought: [loan-shaping only, confirm in Quickli/Figura]
- Lender: [named lender/s or "Open, select on features and appetite below"]
- Required features:

RED FLAGS & CONDITIONS
- [credit-relevant only, or "None identified"]

NEEDS CONFIRMING
- [open unknowns, not tasks]

DOCUMENTS TO CHASE
- [only what this file needs, or "Standard refinance or purchase docs only"]

CLIENT TO DO
- [checklist]

FD / KAT TO DO
- [checklist]

Kind regards,
Kat

TRANSCRIPT
{{6.Data}}
```

---

## Key Contacts

| Person | Role | Details |
|---|---|---|
| Shane | Client / Owner | @differentlygroup, shane@differently.au |
| Kat | Team member (Zoom user) | kat@fdly.au — Outlook connected ✅ |
| Vanessa | Credit Analyst | vanessa@fdly.au — receives handover notes |

---

## Still Needed From Client

| Item | Status |
|---|---|
| GHL contact ID mapping (how to match meeting to contact) | ❌ Pending |
| Confirm Outlook draft recipient email | ⚠️ Likely kat@fdly.au |
| SharePoint folder confirmation for transcript storage | ⚠️ Optional |

---

## Still To Build

- [ ] Update Outlook Module 9 Subject and Body with correct mapped values
- [ ] Add Claude Module 8 (Vanessa handover prompt)
- [ ] Add Outlook Module 10 — send handover notes to vanessa@fdly.au
- [ ] Add GoHighLevel HTTP Module 11 — post meeting notes as Note on contact
- [ ] Resolve GHL contact ID mapping (match Zoom meeting to GHL contact)
- [ ] Test full workflow end-to-end with a real or dummy transcript
- [ ] (Optional) Add SharePoint module to save transcript
- [ ] Phase 2: Microsoft Teams workflow (same structure)

---

## Communication Notes

- All communication must stay on the Freelancer platform (no WhatsApp/external)
- Client is happy with plain text handover notes (not PDF)
- Client interested in SharePoint for transcript storage (Microsoft environment)
- Client confirmed both Zoom and Teams have cloud recording + transcription enabled
- Make.com Free plan (1,000 ops/month) — recommend upgrade to Core ($9/month) after testing

---

## Timeline

- Zoom workflow: 2-3 business days (agreed)
- Teams workflow: after Zoom is working

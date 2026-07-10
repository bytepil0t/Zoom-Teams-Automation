# Zoom Automation Project - Full Chat History & Context

**Project**: Make.com Automation Workflow for Zoom Post-Meeting Transcript Processing  
**Freelancer**: Sean Paul Payne Dinwiddie (mindmagic.pluse@gmail.com)  
**Client**: Shane / Kat (shane@differently.au)  
**Date Saved**: July 10, 2026  

---

## PROJECT OVERVIEW

Automate Zoom meeting post-processing for "Debt-Free Formula" meetings:
- **Trigger**: Zoom recording completed webhook
- **Filter**: Only meetings titled "Debt-Free Formula"
- **AI**: Claude API (claude-sonnet-4-5) processes transcript
- **Output 1**: Outlook draft email to client (in Kat's voice)
- **Output 2**: Internal handover email to Vanessa (Credit Analyst)
- **Output 3**: GoHighLevel note on contact/opportunity

**Budget**: $500 AUD  
**Timeline**: 2-3 business days  
**Platform**: Make.com  

---

## CREDENTIALS & API KEYS

> ⚠️ Real credentials are stored securely offline. Use placeholders below for reference only.

### Zoom Server-to-Server OAuth App
- **App Name**: FD Zoom Transcript Automation
- **Account ID**: `YOUR_ZOOM_ACCOUNT_ID`
- **Client ID**: `YOUR_ZOOM_CLIENT_ID`
- **Client Secret**: `YOUR_ZOOM_CLIENT_SECRET`
- **Base64 Auth**: `YOUR_BASE64_ENCODED_CLIENT_ID_SECRET`

### Anthropic API
- **Account Name**: Kats Zoom Meeting Automation
- **API Key**: `YOUR_ANTHROPIC_API_KEY`
- **Model**: `claude-sonnet-4-5`

### Make.com
- **Account**: shane@differently.au (client owns)
- **Sean's access**: mindmagic.pluse@gmail.com (App Developer role)
- **Plan**: Free (needs upgrade to Core ~$9/month for production)
- **Webhook URL**: `YOUR_MAKE_WEBHOOK_URL`

### GoHighLevel
- **API Key**: `YOUR_GHL_API_KEY`
- **Status**: ✅ Received from client

---

## MAKE.COM SCENARIO MODULES (Current Build)

### Module 1: Webhooks 2 — Custom Webhook (Trigger)
- **Type**: Custom webhook
- **Name**: Zoom Recording Completed
- **URL**: `YOUR_MAKE_WEBHOOK_URL`
- **Purpose**: Receives POST from Zoom when recording is completed

---

### Module 2: Filter — "Debt-Free Formula" Only
- **Condition**: `2.payload.object.topic` contains `"Debt-Free Formula"`
- **Purpose**: Only processes relevant meetings, ignores all others

---

### Module 3: HTTP 4 — Get Zoom OAuth Token
- **Method**: POST
- **URL**: `https://zoom.us/oauth/token`
- **Headers**:
  - `Authorization: Basic YOUR_BASE64_ENCODED_CLIENT_ID_SECRET`
  - `Content-Type: application/x-www-form-urlencoded`
- **Body**: `grant_type=account_credentials&account_id=YOUR_ZOOM_ACCOUNT_ID`
- **Parse Response**: Yes
- **Output**: `4.access_token`

---

### Module 4: HTTP 5 — Get Recording List
- **Method**: GET
- **URL**: `https://api.zoom.us/v2/meetings/{{2.payload.object.uuid}}/recordings`
- **Headers**: `Authorization: Bearer {{4.access_token}}`
- **Parse Response**: Yes
- **Output**: `5.recording_files[]`

---

### Module 5: HTTP 6 — Download Transcript File
- **Method**: GET
- **URL**: `{{5.recording_files[].download_url}}`
- **Headers**: `Authorization: Bearer {{4.access_token}}`
- **Parse Response**: No (raw text)
- **Output**: `6.Data`

---

### Module 6: HTTP 7 — Claude AI (Kat's Client Email)
- **Method**: POST
- **URL**: `https://api.anthropic.com/v1/messages`
- **Headers**:
  - `x-api-key: YOUR_ANTHROPIC_API_KEY`
  - `anthropic-version: 2023-06-01`
  - `content-type: application/json`
- **Body**: Claude Prompt 1 (see below) with `{{6.Data}}` appended as transcript
- **Parse Response**: Yes
- **Output**: `7.content[].text`

---

### Module 7: HTTP 8 — Claude AI (Vanessa's Handover Notes)
- **Method**: POST
- **URL**: `https://api.anthropic.com/v1/messages`
- **Headers**:
  - `x-api-key: YOUR_ANTHROPIC_API_KEY`
  - `anthropic-version: 2023-06-01`
  - `content-type: application/json`
- **Body**: Claude Prompt 2 (see below) with `{{6.Data}}` appended as transcript
- **Parse Response**: Yes
- **Output**: `8.content[].text`
- **Status**: ⏳ Not yet built in Make.com

---

### Module 8: Microsoft 365 Email — Create Draft (Client Follow-Up)
- **Type**: Microsoft 365 Email → Create a Draft Email
- **Connection**: Kat's Outlook (`kat@fdly.au`) ✅ Connected
- **To**: `kat@fdly.au` (or `{{2.payload.object.host_email}}`)
- **Subject**: `Follow Up - {{2.payload.object.topic}} - {{formatDate(2.payload.object.start_time; "DD/MM/YYYY")}}`
- **Body Content**: `{{7.content[].text}}`
- **Status**: ⚠️ Subject/body still showing placeholder values — needs update

---

### Module 9: Microsoft 365 Email — Send Handover to Vanessa
- **Type**: Microsoft 365 Email → Send an Email
- **Connection**: Kat's Outlook (`kat@fdly.au`) ✅ Connected
- **To**: `vanessa@fdly.au`
- **Subject**: `Credit Handover - {{2.payload.object.topic}} - {{formatDate(2.payload.object.start_time; "DD/MM/YYYY")}}`
- **Body Content**: `{{8.content[].text}}`
- **Status**: ⏳ Not yet built in Make.com

---

### Module 10: GoHighLevel Note (HTTP POST)
- **Type**: HTTP → Make a request
- **Purpose**: Add note to GHL contact/opportunity
- **Status**: ⏳ Not yet built — API key received, need contact ID mapping
- **Endpoint**: `https://services.leadconnectorhq.com/contacts/{contactId}/notes`
- **Headers**:
  - `Authorization: Bearer YOUR_GHL_API_KEY`
  - `Version: 2021-07-28`

---

## CLAUDE PROMPTS

### Prompt 1 — Kat's Client Follow-Up Email

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
- Voice: Kat's. Australian spelling. No em dashes. No "I hope this email finds you well". Go easy on exclamation marks.
- Short. Readable in under a minute. Bullets, not paragraphs. Most important first.
- Numbers: keep some but light and directional (around, roughly). Never restate income, servicing or borrowing capacity.
- Reinforce the Debt Free Formula lightly and only where it fits.
- Match the warmth to the meeting.
- Translate internal mechanics into plain language.
- Compliance: do not promise approval, do not guarantee a specific rate, do not give tax or legal advice.
- Subject line: warm and clear. Start with "Hi [first name]," End with "Kind regards, Kat".

TRANSCRIPT:
{{6.Data}}
```

---

### Prompt 2 — Vanessa's Internal Credit Handover Notes

```
You are preparing an internal credit handover email for Finance Differently, an Australian mortgage broking business.

WHO IT IS FOR
- The email goes to Vanessa, our credit analyst (vanessa@fdly.au).
- Vanessa already has the client's financials in Quickli and Figura. Do NOT repeat income, expenses, servicing or borrowing-capacity numbers.
- Assume Vanessa has the numbers but not the story.

OUTPUT FORMAT:

Subject: Credit handover — [client name(s)] — [scenario in 3 to 5 words]

Hi Vanessa,

[Lead line: file type + biggest thing to check. One or two lines only.]

SNAPSHOT
- Client:
- File type:
- Loan purpose:
- Owner-occupied / Investment:
- Indicative lender:
- Timeline / urgency:

WHO THEY ARE
WHO THEY WANT
PROPERTIES [only if applicable]
LOAN & STRUCTURE
RED FLAGS & CONDITIONS
NEEDS CONFIRMING
DOCUMENTS TO CHASE
CLIENT TO DO
FD / KAT TO DO

Kind regards,
Kat

TRANSCRIPT
{{6.Data}}
```

---

## ZOOM APP CONFIGURATION

### Event Subscriptions
- **Event**: `recording.completed`
- **Notification URL**: `YOUR_MAKE_WEBHOOK_URL`
- **Validation**: ✅ Confirmed working

### Required Zoom Scopes
- `recording:read:admin`
- `meeting:read:admin`

---

## TASK STATUS TRACKER

| Task | Description | Status |
|------|-------------|--------|
| 1 | Initial project analysis & planning | ✅ Done |
| 2 | Zoom API setup & credentials | ✅ Done |
| 3 | Make.com account setup & access | ✅ Done |
| 4 | Anthropic API setup | ✅ Done |
| 5 | Zoom webhook URL configuration | ✅ Done |
| 6 | Make.com scenario build (core modules) | ✅ Done |
| 7 | Microsoft 365 Outlook integration | ✅ Done (connected) |
| 8 | Update Outlook module subject/body values | ⏳ To do |
| 9 | Add Claude module for Vanessa handover | ⏳ To do |
| 10 | Add Outlook send module for Vanessa | ⏳ To do |
| 11 | GoHighLevel integration | ⏳ To do (API key received) |
| 12 | Resolve GHL contact ID mapping | ⏳ To do |
| 13 | Full end-to-end testing | ⏳ Not Started |
| 14 | Optional SharePoint transcript storage | ⏳ Optional |

---

## STILL NEEDED FROM CLIENT

1. **GHL contact matching** — How should the workflow identify the right GHL contact from a Zoom meeting? Options:
   - Match by client email from Zoom attendee list
   - Match by meeting topic keyword
   - Manual contact ID per meeting (not ideal)

2. **Outlook draft recipient** — Confirm: should the draft be addressed to the meeting attendee/client, or always sit in Kat's drafts for her to address manually?

3. **SharePoint** — Confirm whether to add a SharePoint module to save transcripts for the team

---

## KEY DECISIONS MADE

1. Use **Make.com** (not Zapier or n8n)
2. Use **Claude API** via Anthropic (not ChatGPT)
3. Filter: only process meetings titled **"Debt-Free Formula"**
4. Outlook: **create draft** for client email (not send automatically)
5. Handover notes: **plain text** in email body (not PDF) — sent directly to Vanessa
6. GHL data: log as **note on Contact or Opportunity**
7. Workflow priority: **Zoom first**, Microsoft Teams second
8. All communication must stay **on the freelancing platform**
9. Client is in **Microsoft environment** — SharePoint preferred over Google Drive

---

## IMPORTANT CONTACTS & TEAM

| Person | Role | Email |
|--------|------|-------|
| Sean Dinwiddie | Freelancer (developer) | mindmagic.pluse@gmail.com |
| Shane | Client (account owner) | shane@differently.au |
| Kat | Client (Outlook / Zoom user) | kat@fdly.au |
| Vanessa | Credit Analyst | vanessa@fdly.au |

---

## MAKE.COM UPGRADE NOTE

Current plan: **Free**  
Needed for production: **Core plan (~$9 USD/month)**  
Reason: Free plan has limited operations and no advanced scheduling

---

## NEXT STEPS (In Order)

1. **Update Outlook module** in Make.com with correct Subject and Body values
2. **Add second Claude module** (HTTP 8) with Vanessa's handover prompt
3. **Add second Outlook module** — send handover to vanessa@fdly.au
4. **Build GoHighLevel HTTP module** — post note to GHL contact
5. **Resolve GHL contact ID mapping** — confirm with Shane how to match meetings to contacts
6. **End-to-end test** with a real Zoom recording
7. **Confirm outputs** look correct
8. **Hand over** and walk client through the workflow

---

*Last updated: July 10, 2026*

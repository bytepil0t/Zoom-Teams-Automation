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
- **Output 1**: Outlook draft email (client follow-up)
- **Output 2**: GoHighLevel note on contact/opportunity
- **Output 3**: Internal handover notes for Vanessa (Credit Analyst)

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
- **Body**:
  ```
  grant_type=account_credentials&account_id=YOUR_ZOOM_ACCOUNT_ID
  ```
- **Parse Response**: Yes
- **Output**: `4.access_token`

---

### Module 4: HTTP 5 — Get Recording List
- **Method**: GET
- **URL**: `https://api.zoom.us/v2/meetings/{{2.payload.object.uuid}}/recordings`
- **Headers**:
  - `Authorization: Bearer {{4.access_token}}`
- **Parse Response**: Yes
- **Output**: `5.recording_files[]` (array of recording files)

---

### Module 5: HTTP 6 — Download Transcript File
- **Method**: GET
- **URL**: `{{5.recording_files[].download_url}}` (VTT/transcript file URL)
- **Headers**:
  - `Authorization: Bearer {{4.access_token}}`
- **Parse Response**: No (raw text)
- **Output**: `6.Data` (raw transcript text)

---

### Module 6: HTTP 7 — Claude AI Processing
- **Method**: POST
- **URL**: `https://api.anthropic.com/v1/messages`
- **Headers**:
  - `x-api-key: YOUR_ANTHROPIC_API_KEY`
  - `anthropic-version: 2023-06-01`
  - `content-type: application/json`
- **Body**:
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
- **Parse Response**: Yes
- **Output**: `7.content[].text` (Claude's response)

---

### Module 7: Microsoft 365 Email — Create Draft Email
- **Type**: Microsoft 365 Email → Create a Draft Email
- **Connection**: Kat's Outlook account (connected)
- **To**: TBD (client email or `{{2.payload.object.host_email}}`)
- **Subject**: `Follow Up - {{2.payload.object.topic}} - {{formatDate(2.payload.object.start_time; "DD/MM/YYYY")}}`
- **Body Content**: `{{7.content[].text}}`
- **Status**: Module added, fields need updating from placeholder values

---

### Module 8: GoHighLevel Note (NOT YET BUILT)
- **Type**: HTTP module → POST
- **Purpose**: Add note to GHL contact/opportunity
- **Status**: Waiting for GHL API key from client
- **Endpoint**: TBD (GHL API)

---

## ZOOM APP CONFIGURATION

### Event Subscriptions (in Zoom Marketplace)
- **Event**: `recording.completed`
- **Notification URL**: `YOUR_MAKE_WEBHOOK_URL`
- **Validation**: Confirmed working ✅

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
| 7 | Microsoft 365 Outlook integration | 🔄 In Progress |
| 8 | GoHighLevel integration | ⏳ Not Started |
| 9 | AI prompt development | ⏳ Not Started |
| 10 | Optional Google Drive transcript storage | ⏳ Not Started |
| 11 | Full end-to-end testing | ⏳ Not Started |

---

## STILL NEEDED FROM CLIENT

1. **AI Prompt** — Detailed instructions for Claude to generate:
   - Client follow-up email
   - Client action steps
   - Meeting holder action steps
   - Internal handover notes for Vanessa
   - Key tasks and next steps

2. **GoHighLevel API Key** — Client is locating this

3. **Vanessa's Email Address** — For handover notes recipient

4. **Recipient Email Address** — Who should the Outlook draft be addressed to?

5. **Confirmation on Google Drive** — Whether to implement optional transcript storage for Vanessa

---

## OUTLOOK MODULE — CORRECT VALUES TO USE

### Subject Field
```
Follow Up - {{2.payload.object.topic}} - {{formatDate(2.payload.object.start_time; "DD/MM/YYYY")}}
```
Example output: `Follow Up - Debt-Free Formula - 09/07/2026`

### Body Content Field
```
{{7.content[].text}}
```

### To Recipients Field
**Option A** — Dynamic (Zoom host email):
```
{{2.payload.object.host_email}}
```
**Option B** — Hardcoded specific email:
```
kat@differently.au
```

---

## KEY DECISIONS MADE

1. Use **Make.com** (not Zapier or n8n)
2. Use **Claude API** via Anthropic (not ChatGPT)
3. Filter: only process meetings titled **"Debt-Free Formula"**
4. Outlook: **create draft** (not send automatically)
5. Handover notes: **plain text** in email body (not PDF)
6. GHL data: log as **note on Contact or Opportunity**
7. Workflow priority: **Zoom first**, Microsoft Teams second
8. All communication must stay **on the freelancing platform**

---

## IMPORTANT CONTACTS & TEAM

| Person | Role | Email |
|--------|------|-------|
| Sean Dinwiddie | Freelancer (developer) | mindmagic.pluse@gmail.com |
| Shane | Client (account owner) | shane@differently.au |
| Kat | Client (Outlook user) | — |
| Vanessa | Credit Analyst (receives handover notes) | TBD |

---

## MAKE.COM UPGRADE NOTE

Current plan: **Free**  
Needed for production: **Core plan (~$9 USD/month)**  
Reason: Free plan has limited operations and no advanced scheduling

---

## OPTIONAL GOOGLE DRIVE INTEGRATION (Future)

Client expressed interest in saving full transcripts to Google Drive so Vanessa can ask Claude follow-up questions.

**Implementation steps (if confirmed):**
1. Add Google Drive module after transcript download (Module 5)
2. Configure to save `.vtt` transcript file to shared folder
3. Client authorizes Google Drive connection in Make.com

---

## CHAT HISTORY SUMMARY (153 messages)

### Early Messages (1-20)
- Project brief received: automate Zoom → Claude → Outlook → GHL
- Discussed platform choice: Make.com selected
- Discussed AI tool: Claude API via Anthropic selected
- Sean clarified full name: Sean Paul Payne Dinwiddie
- Client confirmed Free Make.com plan

### Messages 21-40
- Client sent Zoom credentials (Account ID, Client ID, Client Secret)
- Sean invited to Make.com as App Developer
- Sean accepted invitation
- Zoom webhook URL created and shared with client

### Messages 41-60
- Client configured Zoom Event Subscriptions
- Webhook validated successfully
- Client shared Anthropic API key
- Discussed PDF vs plain text for handover notes → chose plain text
- Discussed Claude Projects / transcript storage for Vanessa

### Messages 61-80
- Built HTTP modules in Make.com:
  - HTTP 4: Zoom OAuth token
  - HTTP 5: Get recordings list
  - HTTP 6: Download transcript
  - HTTP 7: Claude AI processing
- Configured all headers and body parameters

### Messages 81-90
- Added Microsoft 365 Email module
- Kat connected Outlook account
- Client put placeholder values "test subject" and "test content"
- Client asked for correct values → answered with proper mappings
- Client mentioned GoHighLevel API key search is in progress

---

## NEXT STEPS (In Order)

1. **Client updates Outlook module** with correct Subject and Body values (see above)
2. **Client provides final AI prompt** for Claude module
3. **Update Claude module** (HTTP 7) body with real prompt + `{{6.Data}}`
4. **Client provides GHL API key**
5. **Build GoHighLevel module** (HTTP module after Outlook)
6. **End-to-end test** with a real Zoom recording
7. **Confirm outputs** look correct
8. **Hand over** and walk client through the workflow

---

*Last updated: July 10, 2026*

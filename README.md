# 📞 AI Outbound Call Agent — Automated Lead Qualification with n8n + VAPI

**Description:** Intelligent lead qualification system that captures inbound form submissions, validates contact data, and automatically triggers an AI voice agent to call qualified leads — logging every outcome to Google Sheets.

---

## ✨ Key Features

* 📋 Inbound lead capture via a custom "Work With Us" form
* 🇳🇬 Automatic Nigerian phone number validation & standardization
* 📞 AI-powered outbound voice call via VAPI
* ⏱️ Automated call status polling with retry logic
* 📊 Outcome logging across three Google Sheets (invalid number, voicemail, completed)
* 🔀 Conditional workflow branching based on phone validity and call result

---

## 🧩 Components

| Node | Type | Purpose |
|------|------|---------|
| On form submission | Form Trigger | Captures lead data from "Work With Us" form |
| Code in JavaScript | Code | Validates & standardizes Nigerian phone numbers |
| If | Conditional | Routes valid vs. invalid phone numbers |
| HTTP Request (POST) | HTTP Request | Initiates outbound AI voice call via VAPI |
| Wait | Wait | 60-second pause for call to connect and run |
| HTTP Request1 (GET) | HTTP Request | Polls VAPI API for call completion status |
| Limit | Limit | Caps polling to 2 attempts to prevent infinite loops |
| If1 | Conditional | Checks whether call status meets completion criteria |
| If2 | Conditional | Distinguishes between voicemail and completed calls |
| Wait1 | Wait | Holds before next polling attempt |
| Create spreadsheet | Google Sheets | Logs leads with invalid phone numbers |
| Create spreadsheet1 | Google Sheets | Logs leads who went to voicemail |
| Create spreadsheet2 | Google Sheets | Logs successfully completed calls |

---

<img width="1690" height="480" alt="n8n outbound call Flow" src="https://github.com/user-attachments/assets/5a6fa693-ba80-4472-941f-2e268ae4eceb" />



## 🔁 How It Works

```
Form Submission
      │
      ▼
JavaScript: Validate & standardize phone number
      │
      ▼
   [If Node]
  /         \
TRUE         FALSE
(invalid)    (valid)
  │               │
  ▼               ▼
Log to          POST → VAPI (initiate AI voice call)
Sheets               │
                     ▼
               Wait 60 seconds
                     │
                     ▼
               GET → VAPI (check call status)
                     │
                     ▼
                  [Limit]
               (max 2 polls)
                     │
                     ▼
                  [If1]
              /         \
           TRUE          FALSE
          /                  \
       [If2]               Wait1
      /     \                │
  Voicemail  Completed    (retry)
     │           │
     ▼           ▼
  Log to      Log to
 Sheets       Sheets
```

---

## 📋 Form Fields Captured

| Field | Required | Notes |
|-------|----------|-------|
| Name | ✅ | Lead's full name |
| Phone Number | ✅ | Nigerian format — auto-standardized to +234 |
| E-mail | ✅ | |
| Company Name | ✅ | Passed to AI voice agent |
| Role | ✅ | e.g. CEO, CTO |
| Request | ✅ | What they're looking for — passed to voice agent |
| Company Size | ✅ | e.g. 2–30 |

---

## 📁 Google Sheets Outputs

| Sheet | Triggered When |
|-------|---------------|
| **Log incorrect phone number** | Phone fails Nigerian format validation |
| **Log voice mail** | Call connects but goes to voicemail |
| **Log Complete** | Call is completed successfully |

---

## ⚙️ Setup Instructions

### Prerequisites

- [n8n](https://n8n.io/) (self-hosted or cloud)
- [VAPI](https://vapi.ai/) account with:
  - An assistant created and its ID ready
  - A phone number provisioned and its ID ready
  - An API key generated
- Google account with Google Sheets access

---

### Step 1 — Import the Workflow

1. Download `workflow_clean.json`
2. In n8n, go to **Workflows → Import from File**
3. Select the downloaded file

---

### Step 2 — Set Up Credentials

**VAPI Bearer Auth**
1. Go to **Credentials → New → Header Auth** (or Bearer Token)
2. Set the token to your VAPI API key
3. Name it `vapi`

**Google Sheets OAuth2**
1. Go to **Credentials → New → Google Sheets OAuth2 API**
2. Follow the OAuth flow to connect your Google account
3. Name it `Google Sheets OAuth2 API`

---

### Step 3 — Configure the HTTP Request Node (POST)

Open the **HTTP Request** node and replace the placeholders in the JSON body:

```json
{
  "assistantId": "YOUR_VAPI_ASSISTANT_ID",
  "phoneNumberId": "YOUR_VAPI_PHONE_NUMBER_ID",
  ...
}
```

Replace with your actual VAPI Assistant ID and Phone Number ID from your VAPI dashboard.

---

### Step 4 — Configure If1 and If2 Conditions

Both `If1` and `If2` nodes need conditions set based on your VAPI call status response:

- **If1** — Check whether the call has ended (e.g. `status` equals `ended`)
- **If2** — Check the call outcome (e.g. `endedReason` equals `voicemail` vs `customer-ended-call`)

Refer to the [VAPI API docs](https://docs.vapi.ai) for the exact response field names.

---

### Step 5 — Activate the Workflow

1. Click **Save**
2. Toggle the workflow to **Active**
3. Test by submitting the form with a valid Nigerian number

---

## 📞 Phone Number Validation Logic

The JavaScript node standardizes phone numbers into international format before the VAPI call:

| Input Format | Example | Output |
|---|---|---|
| International with + | `+2348012345678` | `+2348012345678` ✅ |
| International without + | `2348012345678` | `+2348012345678` ✅ |
| Local Nigerian format | `08012345678` | `+2348012345678` ✅ |
| Any other format | `12345` | `incorrect format` ❌ |

Invalid numbers are routed to the Google Sheet log and the VAPI call is skipped.

---

## 🔐 Security Notes

- **Never commit your API keys** to this repository
- All sensitive values in `workflow_clean.json` are replaced with `YOUR_*` placeholders
- Credentials are stored in n8n's encrypted credential store, not in the workflow JSON
- Webhook IDs have been redacted — they will be auto-generated when you import the workflow

---

## 🛠️ Tech Stack

- **n8n** — Workflow automation
- **VAPI** — AI voice agent & outbound calling
- **Google Sheets** — Outcome logging
- **JavaScript** — Phone number validation logic

---

## 📌 Use Cases

- Automated B2B lead qualification via voice
- Outbound follow-up for inbound form submissions
- AI-first sales outreach pipelines
- No-code/low-code CRM augmentation

---

## 📄 License

MIT — free to use, modify, and distribute.

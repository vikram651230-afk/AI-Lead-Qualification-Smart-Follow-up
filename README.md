# AI-Lead-Qualification-Smart-Follow-up
An intelligent n8n lead-management automation that captures website leads, validates contact information, detects duplicate submissions, uses AI to qualify leads from 0–100, categorizes them as Hot, Warm, or Cold, stores the results in Google Sheets, alerts the sales team, and automatically sends personalized follow-up emails.



# 🤖 AI Lead Qualification & Smart Follow-up

An intelligent **n8n lead-management automation** that captures website leads, validates contact information, detects duplicate submissions, uses AI to qualify leads from **0–100**, categorizes them as **Hot, Warm, or Cold**, stores the results in Google Sheets, alerts the sales team, and automatically sends personalized follow-up emails.

---

## 🚀 Overview

Managing incoming leads manually can be time-consuming and can cause valuable prospects to be missed.

This workflow automates the lead qualification process from the moment a lead submits a form until the sales team receives the qualified lead or the prospect receives a follow-up.

### Workflow

```text
Website / Lead Form
        │
        ▼
┌──────────────────────┐
│ Lead Capture Webhook │
└──────────┬───────────┘
           ▼
┌──────────────────────┐
│    Validate Lead     │
└──────────┬───────────┘
           ▼
      Valid Contact?
       /          \
     No            Yes
     │              │
     ▼              ▼
 Log Invalid    Check Duplicate
 Lead               │
                    ▼
              Duplicate Lead?
               /          \
             Yes            No
              │              │
              ▼              ▼
        Log Duplicate    AI Score Lead
                              │
                              ▼
                        Parse AI Score
                              │
                              ▼
                       Route by Category
                     /        |        \
                   Hot       Warm      Cold
                    │          │         │
                    ▼          ▼         ▼
                  CRM        CRM     Nurture List
                    │          │
                    ▼          ▼
               Telegram      Email
                 Alert      Follow-up
```

---

# ✨ Key Features

* 🔗 **Webhook-based lead capture**
* ✅ **Automatic lead validation**
* 📧 Email format validation
* 📱 Phone number validation
* 🔍 **Duplicate lead detection**
* 🤖 **AI-powered lead scoring**
* 📊 Lead score from **0–100**
* 🔥 **Hot / Warm / Cold classification**
* 📝 AI-generated qualification reason
* ✉️ AI-generated personalized follow-up email
* 📋 Google Sheets CRM integration
* 🚨 Telegram sales alerts for hot leads
* 📩 Automated Gmail follow-ups for warm leads
* 🌱 Nurturing list for cold leads
* ⚠️ Invalid lead logging
* 🛑 Duplicate lead logging
* 🧩 AI error handling
* 🕒 Automatic timestamping

---

# 🏗️ Workflow Architecture

The automation consists of multiple n8n nodes connected into a complete lead-processing pipeline.

## 1. Lead Capture Webhook

**Node:** `Lead Capture Webhook`
**Type:** n8n Webhook
**Method:** `POST`
**Endpoint:** `/lead-capture`

The workflow starts when a website, landing page, form, or external application sends lead information to the webhook.

### Expected Lead Data

```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "phone": "+91 9876543210",
  "company": "Example Technologies",
  "message": "I'm interested in your services."
}
```

The webhook receives the lead and forwards the information to the validation stage.

---

# 2. Validate Lead

**Node:** `Validate Lead`
**Type:** Code Node
**Language:** JavaScript

The workflow validates the submitted lead before continuing.

### Validation Rules

The workflow checks:

* Name is not empty
* Email follows a valid email format
* Phone number is optional
* If a phone number is provided, it must follow the accepted character/length pattern

The validation node generates a structured result containing:

```text
valid
invalidReason
name
email
phone
company
message
capturedAt
```

The lead is then sent to the next decision point.

---

# 3. Valid Contact

**Node:** `Valid Contact`
**Type:** IF Node

This node determines whether the submitted lead passed validation.

### Valid Lead

If:

```text
valid === true
```

the workflow continues to duplicate detection.

### Invalid Lead

If the lead is invalid, processing stops and the lead is sent to:

```text
Log Invalid Lead
```

This prevents bad contact information from entering the lead pipeline.

---

# 4. Log Invalid Lead

**Node:** `Log Invalid Lead`
**Type:** Google Sheets

Invalid submissions are recorded in the **Activity Log**.

The workflow records information such as:

| Field     | Description                   |
| --------- | ----------------------------- |
| Event     | `invalid_lead`                |
| Email     | Submitted email               |
| Details   | Reason for validation failure |
| Timestamp | Time of event                 |

This makes it possible to monitor invalid submissions and troubleshoot lead-capture issues.

---

# 5. Check Duplicate

**Node:** `Check Duplicate`
**Type:** Google Sheets

Before sending a lead to the AI qualification system, the workflow checks whether the email already exists in the **Leads** sheet.

### Duplicate Matching

The primary lookup field is:

```text
Email
```

This helps prevent the same person from entering the qualification pipeline multiple times.

---

# 6. Duplicate Lead

**Node:** `Duplicate Lead`
**Type:** IF Node

The workflow checks whether the Google Sheets lookup returned an existing email.

### If Duplicate

The lead is sent to:

```text
Log Duplicate
```

### If New Lead

The lead continues to:

```text
AI Score Lead
```

This keeps the AI-processing pipeline focused on new prospects.

---

# 7. Log Duplicate

**Node:** `Log Duplicate`
**Type:** Google Sheets

Duplicate submissions are logged for tracking and monitoring.

This provides visibility into repeated submissions without processing the same lead unnecessarily.

---

# 🧠 8. AI Score Lead

**Node:** `AI Score Lead`
**Type:** Google Gemini Node

This is the intelligence layer of the workflow.

The AI evaluates the lead and generates a qualification score between **0 and 100**.

### AI Evaluation Criteria

The workflow instructs the AI to consider:

* Buying intent
* Budget signals
* Urgency
* Company fit
* Completeness of contact information

The AI is expected to return:

```json
{
  "score": 85,
  "reason": "Strong buying intent with clear requirements and company fit.",
  "followup_subject": "Let's discuss your requirements",
  "followup_body": "Hi John, ..."
}
```

The AI is also instructed to return the result as a JSON object.

---

# 📊 9. Parse Score

**Node:** `Parse Score`
**Type:** Code Node

The AI response is processed and converted into a consistent structure.

The workflow:

1. Reads the AI response
2. Parses JSON output
3. Converts the score to a number
4. Ensures the score remains between `0` and `100`
5. Rounds the score
6. Assigns a lead category
7. Creates fallback values if AI output is incomplete

---

# 🔥 Lead Classification

The workflow automatically classifies leads based on their AI score.

|      Score | Category | Meaning                                |
| ---------: | -------- | -------------------------------------- |
| **70–100** | 🔥 Hot   | High-priority sales opportunity        |
|  **40–69** | 🟡 Warm  | Potential customer requiring follow-up |
|   **0–39** | 🔵 Cold  | Lower-priority lead for nurturing      |

### Classification Logic

```javascript
if (score >= 70) {
  category = "Hot";
} else if (score >= 40) {
  category = "Warm";
} else {
  category = "Cold";
}
```

---

# 🔀 10. Route by Category

**Node:** `Route by Category`
**Type:** Switch Node

After AI qualification, the workflow routes each lead according to its category.

```text
                 AI Score
                    │
                    ▼
             Route by Category
             /       |       \
            /        |        \
         Hot       Warm       Cold
          │          │          │
          ▼          ▼          ▼
         CRM        CRM      Nurture
          │          │
          ▼          ▼
      Telegram     Gmail
        Alert     Follow-up
```

---

# 🔥 Hot Lead Automation

## Save Hot Lead to CRM

**Node:** `Save Hot Lead to CRM`
**Type:** Google Sheets

Hot leads are stored in the **Leads** sheet.

The stored information includes:

* Name
* Email
* Phone
* Company
* Message
* AI Score
* Category
* AI Reason
* Captured At

The email address is used as the matching column.

---

## 🚨 Telegram Sales Alert

**Node:** `Telegram Sales Alert`
**Type:** Telegram

When a hot lead is detected, the sales team receives an immediate Telegram notification.

The alert contains:

```text
🔥 HOT LEAD

Score: XX/100

Name: ...
Email: ...
Phone: ...
Company: ...
Message: ...

Why: ...
```

This allows the sales team to react quickly to high-intent prospects.

> **Important:** Configure your own Telegram chat ID before activating the workflow.

---

# 🟡 Warm Lead Automation

## Save Warm Lead to CRM

**Node:** `Save Warm Lead to CRM`
**Type:** Google Sheets

Warm leads are saved to the CRM/Leads sheet with their AI-generated qualification information.

---

## ✉️ Send Warm Follow-up

**Node:** `Send Warm Follow-up`
**Type:** Gmail

Warm leads automatically receive a personalized follow-up email.

The email uses the AI-generated:

```text
followup_subject
followup_body
```

This removes the need for sales staff to manually write the first follow-up message.

---

# 🔵 Cold Lead Automation

## Save to Nurture List

**Node:** `Save to Nurture List`
**Type:** Google Sheets

Cold leads are moved into a nurturing list rather than receiving immediate sales escalation.

This allows the business to maintain a database of lower-intent prospects for future campaigns.

---

# ⚠️ AI Error Handling

The workflow includes an error branch for the AI scoring node.

If AI processing fails, the workflow sends the execution to:

```text
Log AI Error
```

This provides a mechanism for tracking AI-processing failures instead of silently losing the lead.

---

# 📋 Google Sheets Structure

The workflow uses Google Sheets as a lightweight CRM and activity database.

## Leads

The main lead database contains fields such as:

```text
Name
Email
Phone
Company
Message
Score
Category
Reason
Captured At
```

### Example

| Name        | Email                                         | Company      | Score | Category | Reason               |
| ----------- | --------------------------------------------- | ------------ | ----: | -------- | -------------------- |
| John Doe    | [john@example.com](mailto:john@example.com)   | Example Inc. |    86 | Hot      | Strong buying intent |
| Sarah Smith | [sarah@example.com](mailto:sarah@example.com) | ABC Ltd.     |    58 | Warm     | Moderate interest    |
| Alex Kumar  | [alex@example.com](mailto:alex@example.com)   | Startup XYZ  |    25 | Cold     | Low buying intent    |

---

## Activity Log

The workflow also uses an activity log for operational events.

Example fields:

```text
Event
Email
Details
Timestamp
```

Possible events include:

```text
invalid_lead
duplicate_lead
AI_error
```

---

# 🔐 Security Considerations

Before publishing this workflow or JSON file to GitHub:

### Never expose:

* Google OAuth credentials
* Gemini API credentials
* Gmail credentials
* Telegram credentials
* Webhook secrets
* Private Google Sheet IDs
* Private chat IDs
* Internal instance IDs

Use environment variables, n8n credentials, or placeholders in public repositories.

For example:

```text
<GOOGLE_SHEET_ID>
<TELEGRAM_CHAT_ID>
<GEMINI_API_KEY>
```

instead of publishing real values.

---

# ⚙️ Setup

## 1. Install n8n

Set up an n8n instance using your preferred deployment method.

---

## 2. Import the Workflow

Import the provided n8n workflow JSON into your n8n workspace.

---

## 3. Configure Google Sheets

Create a Google Spreadsheet containing the required sheets.

Recommended structure:

```text
AI Lead Qualification & Smart Follow-up
│
├── Leads
├── Activity Log
└── Nurture List
```

Connect your Google Sheets OAuth credentials in n8n.

---

## 4. Configure Google Gemini

Connect your Gemini credentials to the AI scoring node.

The workflow uses Gemini to:

* Score leads
* Explain the score
* Generate personalized follow-up content

---

## 5. Configure Gmail

Connect your Gmail account to the warm-lead follow-up node.

Make sure the account has permission to send emails.

---

## 6. Configure Telegram

Connect your Telegram bot and provide the sales team's chat ID.

Replace the placeholder:

```text
<TELEGRAM_CHAT_ID>
```

with your actual configuration.

---

## 7. Connect Your Website

Send a `POST` request to the webhook endpoint:

```text
/lead-capture
```

Example request:

```bash
curl -X POST "YOUR_N8N_WEBHOOK_URL/lead-capture" \
-H "Content-Type: application/json" \
-d '{
  "name": "John Doe",
  "email": "john@example.com",
  "phone": "+91 9876543210",
  "company": "Example Technologies",
  "message": "I am interested in your service."
}'
```

---

# 🧪 Example Processing

Suppose a visitor submits:

```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "phone": "+91 9876543210",
  "company": "ABC Technologies",
  "message": "We need a solution for our sales team and would like to discuss pricing."
}
```

The workflow processes the lead:

```text
1. Capture Lead
       ↓
2. Validate Contact
       ↓
3. Check Duplicate
       ↓
4. AI Qualification
       ↓
5. Score = 85
       ↓
6. Category = Hot
       ↓
7. Save to CRM
       ↓
8. Notify Sales Team
```

The sales team can then immediately prioritize the prospect.

---

# 💡 Business Value

This automation helps businesses:

### Save Time

Automates repetitive lead-processing tasks.

### Improve Lead Response

High-value prospects can be immediately surfaced to the sales team.

### Reduce Duplicate Work

Duplicate submissions are detected before entering the AI qualification pipeline.

### Improve Lead Prioritization

Instead of treating every lead equally, the system assigns an AI-based priority score.

### Automate Follow-ups

Warm prospects can automatically receive personalized communication.

### Centralize Lead Data

Google Sheets provides a simple centralized location for lead information and activity tracking.

---

# 🧩 Technology Stack

| Technology        | Purpose                       |
| ----------------- | ----------------------------- |
| **n8n**           | Workflow automation           |
| **Webhooks**      | Lead capture                  |
| **JavaScript**    | Validation & score processing |
| **Google Gemini** | AI lead qualification         |
| **Google Sheets** | CRM / lead database           |
| **Gmail**         | Automated follow-ups          |
| **Telegram**      | Sales notifications           |

---

# 📁 Workflow Components

```text
AI Lead Qualification & Smart Follow-up
│
├── Lead Capture Webhook
├── Validate Lead
├── Valid Contact
├── Log Invalid Lead
├── Check Duplicate
├── Duplicate Lead
├── Log Duplicate
├── AI Score Lead
├── Parse Score
├── Route by Category
│
├── Hot Lead
│   ├── Save Hot Lead to CRM
│   └── Telegram Sales Alert
│
├── Warm Lead
│   ├── Save Warm Lead to CRM
│   └── Send Warm Follow-up
│
├── Cold Lead
│   └── Save to Nurture List
│
└── AI Error
    └── Log AI Error
```

---

# 🔄 Complete Automation Flow

```text
                    ┌─────────────────────┐
                    │   Website / Form    │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │  Lead Capture       │
                    │     Webhook         │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │    Validate Lead    │
                    └──────────┬──────────┘
                               │
                     ┌─────────┴─────────┐
                     │                   │
                  Invalid              Valid
                     │                   │
                     ▼                   ▼
              ┌─────────────┐    ┌───────────────┐
              │ Log Invalid │    │ Check Duplicate│
              └─────────────┘    └───────┬───────┘
                                          │
                                  ┌───────┴───────┐
                                  │               │
                               Duplicate         New
                                  │               │
                                  ▼               ▼
                           ┌─────────────┐  ┌────────────┐
                           │Log Duplicate│  │ AI Scoring │
                           └─────────────┘  └─────┬──────┘
                                                  │
                                                  ▼
                                           ┌────────────┐
                                           │ Parse Score│
                                           └─────┬──────┘
                                                 │
                                                 ▼
                                         ┌──────────────┐
                                         │   Category   │
                                         └──────┬───────┘
                                                │
                           ┌────────────────────┼────────────────────┐
                           │                    │                    │
                           ▼                    ▼                    ▼
                         🔥 HOT               🟡 WARM              🔵 COLD
                           │                    │                    │
                           ▼                    ▼                    ▼
                         CRM                  CRM                Nurture
                           │                    │
                           ▼                    ▼
                       Telegram              Gmail
                         Alert              Follow-up
```

---

# 📈 Future Improvements

The current workflow provides a strong foundation for an AI-powered lead management system.

Potential future upgrades include:

* 📞 Automated WhatsApp follow-ups
* 💬 AI chatbot lead qualification
* 📅 Automatic meeting/calendar booking
* 🧠 More advanced lead-scoring models
* 📊 Dedicated sales dashboard
* 📈 Conversion-rate analytics
* 🔁 Multi-step nurture campaigns
* ⏰ Scheduled follow-up sequences
* 🏢 CRM integrations such as HubSpot or Salesforce
* 📱 SMS notifications
* 🎯 Industry-specific scoring rules
* 🧑‍💼 Sales representative assignment
* 📍 Lead source tracking
* 💰 Revenue prediction
* 📊 Lead-to-customer conversion tracking

---

# ⚠️ Important Notes

* Configure all required n8n credentials before activating the workflow.
* Replace placeholder values such as the Telegram chat ID.
* Verify Google Sheets permissions.
* Test the webhook with sample data before connecting a production website.
* Test AI responses to ensure the generated scores and follow-up messages match your business requirements.
* Do not publish private credentials or internal IDs in a public repository.
* Review automated emails before using the workflow in a production sales environment.

---

# 🎯 Use Cases

This workflow can be adapted for:

* SaaS companies
* IT service companies
* Digital agencies
* B2B businesses
* Consulting companies
* Real-estate businesses
* Marketing agencies
* Software development companies
* Freelancers
* Startup sales teams

---

# 📜 License

Add your preferred license here.

For example:

```text
MIT License
```

---

# ⭐ Conclusion

**AI Lead Qualification & Smart Follow-up** turns a basic website lead form into an automated lead-processing pipeline.

Instead of manually reviewing every submission, the system can:

**Capture → Validate → Deduplicate → Qualify → Score → Categorize → Store → Notify → Follow Up**

This creates a faster and more organized sales workflow while giving the team a clear way to prioritize high-value opportunities.

---

## 🚀 Build smarter. Capture better. Follow up faster.

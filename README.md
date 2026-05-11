# 🎓 University Email Monitor . ISIMG

> **An intelligent Gmail automation that reads university emails, classifies their content, organizes course documents and events into Google Sheets  and instantly alerts you on Telegram for anything critical.**

![n8n](https://img.shields.io/badge/Built%20with-n8n-orange?style=flat-square&logo=n8n)
![AI Powered](https://img.shields.io/badge/AI-OpenAI%20%2B%20OpenRouter-blueviolet?style=flat-square)
![Gmail](https://img.shields.io/badge/Trigger-Gmail-EA4335?style=flat-square&logo=gmail)
![Google Sheets](https://img.shields.io/badge/Storage-Google%20Sheets-34A853?style=flat-square&logo=googlesheets)
![Telegram](https://img.shields.io/badge/Alerts-Telegram-2CA5E0?style=flat-square&logo=telegram)

---

## 🧠 What It Does

Keeping up with university emails is exhausting — lectures, exams, cancellations, and announcements flood your inbox daily. This workflow acts as your **personal academic secretary**.

Every time a new email arrives from your university domain (`@isimg`), the AI:

1. **Filters** non-university emails immediately
2. **Detects attachments** (PDFs, Word docs) vs. plain text emails
3. For **documents** → extracts text, classifies the subject/course, and saves to the matching sheet
4. For **events** → classifies the event type, assesses importance, and logs it to an Events sheet
5. 🚨 For **critical events** (exam dates, project deadlines, competitions) → generates an AI summary and **sends you an instant Telegram alert**

Your academic information is always organized, searchable, and you'll never miss what matters.

---

## ⚙️ Workflow Architecture

```
Gmail Trigger
[New email received]
        │
        ▼
   Is University Email?
   [Filter: sender contains @isimg]
        │
    ✅ Yes
        │
        ▼
   Check Attachments
   ┌─────────────────────┐
   │                     │
   ▼                     ▼
Has PDF/Doc         No Attachment
Attachment          (Plain Text Email)
   │                     │
   ▼                     ▼
Extract Text        Classify Event
from File           [AI Agent — OpenAI]
   │                     │
   ▼                     ▼
Classify Document   Event Parser
[AI Agent —         [Structured Output]
OpenRouter]              │
   │                     ▼
   ▼          ┌──────────────────────┐
Document      │  importance = high?  │
Parser        └──────────────────────┘
   │               │           │
   ▼             ✅ Yes       ❌ No
Save to            │           │
Course Sheet       ▼           ▼
[Google       AI Summary    Save to
 Sheets]      Generator     Events Sheet
                   │         [Google Sheets]
                   ▼
            Telegram Alert
            📩 Instant notification
            with event summary
```

---

## 🚨 Telegram Alert How It Works

When the event classifier detects an event with **high importance**, the workflow triggers an additional AI step that:

- Reads the full email subject and body
- Generates a **clean, concise summary** of the critical event
- Immediately **sends a Telegram message** to your personal chat

### Example Telegram Messages

```
🚨 [EXAM DATE]
📅 Subject: Databases
📆 Date: March 28, 2025  09:00 AM
📍 Room: Amphi B
📝 Summary: Final exam covering all chapters from the semester.
             Bring your student card. No documents allowed.
```

```
⚠️ [PROJECT DEADLINE]
📅 Subject: Software Engineering
📆 Deadline: April 5, 2025
📝 Summary: Final project submission due. Upload to the
             university portal before midnight. Late submissions
             will not be accepted.
```

```
🏆 [COMPETITION]
📅 Event: National Hackathon 2025
📆 Registration closes: March 20, 2025
📝 Summary: Open to all students. Teams of 3-5. Register
             via the student affairs office by end of week.
```

---

## 🤖 AI Classification Details

### Document Classifier
Analyzes extracted PDF/Word text and determines:
- Is it a **course document**? (lecture notes, slides, exercises)
- What **subject/course** does it belong to?
- Saves it to a sheet **named after the course** automatically

### Event Classifier
Analyzes the email subject and body to determine:
- **Event type:** `exam_date` | `project_deadline` | `competition` | `session_cancelled` | `schedule_change` | `announcement` | `other`
- **Importance level:** `high` | `medium` | `low`
- **Brief summary** of the event

### 🚨 Importance Triggers (Telegram alert fired)
| Event Type | Importance | Alert? |
|---|---|---|
| Exam date | high | ✅ Yes |
| Project deadline | high | ✅ Yes |
| Competition / event | high | ✅ Yes |
| Session cancelled | medium | ❌ No (logged only) |
| General announcement | low | ❌ No (logged only) |

---

## 🛠️ Tech Stack

| Component | Tool |
|-----------|------|
| Automation platform | [n8n](https://n8n.io) |
| Email trigger | Gmail |
| Document extraction | n8n Extract from File (PDF) |
| Document classifier | LangChain Agent + OpenRouter |
| Event classifier | LangChain Agent + OpenAI |
| Output parsing | LangChain Structured Output Parser |
| Instant alerts | Telegram Bot |
| Storage | Google Sheets |

---

## 🚀 Getting Started

### Prerequisites

- An **n8n** instance (self-hosted or cloud)
- A **Gmail** account connected to n8n
- An **OpenAI** API key
- An **OpenRouter** API key
- A **Google Sheets** spreadsheet pre-created with an `Events` sheet
- A **Telegram Bot** token + your personal chat ID

### Google Sheets Structure

The workflow auto-creates new sheets named after each course when a document is classified. You only need to pre-create the **Events** sheet with these columns:

| type | importance | summary | date | email_subject |
|------|------------|---------|------|---------------|

### Setup Steps

1. **Import the workflow** — Upload `university_email_monitor_isimg.json` into n8n via *Workflows → Import from file*.

2. **Configure credentials:**
   - `Gmail` — Connect your university Gmail account via OAuth
   - `OpenAI` — Add your API key in the Event Classifier node
   - `OpenRouter` — Add your API key in the Document Classifier node
   - `Google Sheets` — Connect your Google OAuth account
   - `Telegram` — Add your bot token (get one from [@BotFather](https://t.me/BotFather))

3. **Set your Telegram chat ID** — In the Telegram node, replace the chat ID with your own. (Find yours via [@userinfobot](https://t.me/userinfobot))

4. **Update the email filter** — In the *Is University Email?* node, replace `@isimg` with your university's email domain if different.

5. **Update the Google Sheet ID** — Replace the sheet ID in all Google Sheets nodes with your own spreadsheet.

6. **Activate the workflow** — Every incoming university email will now be processed automatically.

---

## 📊 What Gets Saved

### Course Documents (per-subject sheets)

| filename | subject | extracted_text_preview | received_at |
|----------|---------|------------------------|-------------|
| chapter3_slides.pdf | Databases | Introduction to SQL... | 2025-03-10 |

### Events Sheet

| type | importance | summary | date | email_subject |
|------|------------|---------|------|---------------|
| exam_date | high | DB final exam on March 28 | 2025-03-10 | Examen final — BD |
| session_cancelled | medium | DB lecture cancelled on March 12 | 2025-03-10 | Cours annulé |

---



## 📂 Project Structure

```
university_email_monitor_isimg.json    ← n8n workflow file (import this)
README.md                              ← You are here
```

---

## 🔒 Notes & Best Practices

- The **@isimg domain filter** ensures only university emails are processed  personal emails are untouched.
- Both PDF and common document attachments (Word, etc.) are detected via MIME type checking.
- The structured output parsers enforce consistent JSON  no hallucinated formats.
- Course sheet names are dynamically set from the AI's classification output.
- Telegram alerts are only fired for **high importance** events to avoid notification fatigue.
- The AI summary is generated fresh for each alert not just the raw email body  so it's always clean and readable.

---

## 🤝 Contributing

Want to add calendar integration (Google Calendar), Notion sync, or Discord notifications? PRs are very welcome!


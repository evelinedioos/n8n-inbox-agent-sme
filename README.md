# Inbox agent for an SME (n8n + OpenAI)

An AI agent that reads incoming customer emails for a small business, decides by itself what kind of email it is, and prepares the right action in Gmail, Google Calendar and Google Sheets. It works in French and Dutch.

**A human always approves.** The agent never sends an email. It only creates drafts, adds labels and writes rows.

## What it does

| Email type | What the agent does |
|---|---|
| Appointment request | Checks Google Calendar for free 30-minute slots in the next 5 working days (09:00–17:00, Europe/Brussels) and creates a reply draft with 3 options |
| Quote request | Extracts company, contact, need, budget and deadline, adds a row to the `Leads` sheet, adds the label `sales` and creates a short reply draft |
| Complaint | Does not reply. Adds the label `dringend` so a person follows up personally |
| Anything else | Only logs it |

Every decision is written to an `Agent log` sheet: time, sender, subject, language, category, action taken, a one-line reasoning and a confidence level (high / medium / low).

## How it works

```
Gmail Trigger (every minute, label "demo-agent", unread)
        │
        ▼
   AI Agent ──── OpenAI chat model (the "brain")
        │   ├── tool: get_calendar_events  (Google Calendar)
        │   ├── tool: create_reply_draft   (Gmail draft, same thread)
        │   ├── tool: add_label_sales      (Gmail)
        │   ├── tool: add_label_dringend   (Gmail)
        │   └── tool: append_lead          (Google Sheets, fields filled by the agent)
        ▼
   Log to Agent log (Google Sheets)
        ▼
   Mark as read (so the same email is never processed twice)
```

The difference with a classic automation: there is no rule like "if the subject contains *offerte*, do X". The agent reads and understands the email, in either language, and chooses its own tools.

## Test results

Tested on three fictional emails:

1. **Quote request in Dutch** → lead added to the sheet, label `sales`, reply draft created. The agent also noticed that the name in the email did not match the sender address and wrote that in its reasoning.
2. **Appointment request in French** → calendar checked, draft in French proposing three free slots.
3. **Complaint in Dutch** → no reply, label `dringend`, logged with confidence "high".

## Design choices

- **Human-in-the-loop.** Drafts only. This builds trust with the team and keeps a person responsible for what leaves the company.
- **Complaints go to a person.** Some emails should not get an automatic answer.
- **An audit log with reasoning.** It shows why the agent did something, and it is the basis for measuring a pilot.
- **Start small with a label.** The agent only touches emails with one label. At a client, that is how a pilot starts.
- **The model is replaceable.** This demo uses OpenAI. For a client with strict data requirements, a European provider such as Mistral could be used instead.

## How I would roll this out at a client

1. Look at how the team handles email today and where the time goes.
2. Run a 2-week pilot on one type of email.
3. Measure time saved and the share of drafts approved without changes.
4. Adjust the instructions (tone, signature, edge cases) based on the log.
5. Train the team, then extend to more email types.

## Setup

1. Import `inbox-agent-sme.json` in n8n (*⋯ → Import from file*).
2. Connect your Gmail, Google Calendar, Google Sheets and OpenAI credentials.
3. Create the Gmail labels `demo-agent`, `sales` and `dringend`, and select them in the label tools.
4. Create a Google Sheet with two tabs:
   - `Leads`: `datum | bedrijf | naam | email | vraag | budget | deadline`
   - `Agent log`: `tijd | afzender | onderwerp | taal | categorie | actie | redenering | zekerheid`
5. Replace the placeholders (`YOUR_CALENDAR_ID`, `YOUR_SPREADSHEET_ID`, `YOUR_LABEL_ID_…`) by selecting your own calendar, sheet and labels in the nodes.
6. Choose a current OpenAI model in the *OpenAI Chat Model* node.

No credentials or personal data are included in this repository.

---

Built by Eveline Dioos with n8n, OpenAI and the Google Workspace APIs.

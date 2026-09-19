# Customer-Support-Ticket-Automation

An n8n workflow that automatically classifies, prioritizes, and routes incoming customer support requests — turning "Support requests arrive through email, chat, or forms and are manually assigned" into a fully automated pipeline.

![image alt](https://github.com/kristinenicolelaglibot/Customer-Support-Ticket-Automation/blob/ac4afcb72655a51736ed1eb11bfd7b42cffd6adf/ticketautomation.png)

**Example:** A message like *"Our website has been down since 2 AM"* is automatically detected as a technical issue, marked High Priority, logged as a ticket, assigned to IT, and posted to Slack — with zero manual triage.

---

## Technologies Used

| Tool | Purpose |
|---|---|
| **n8n** (Cloud) | Workflow orchestration / automation engine |
| **Google Gemini 3.6 Flash** | AI classification (category, priority, department, sentiment) |
| **Supabase (Postgres)** | Ticket storage and incident logging |
| **Slack API** | Real-time team notifications |
| **Webhook (n8n)** | Ingests form/chat submissions |

---

## Features

- **AI-powered classification** — every incoming message is analyzed and tagged with category (technical/billing/account/general), priority (Low–Urgent), sentiment, and a one-line summary.
- **Automatic ticket creation** — every classified message is inserted as a structured row in a Postgres database, no manual data entry.
- **Smart assignment** — tickets are routed to the correct team (IT, Billing, Customer Success, General Support) based on classification.
- **Instant Slack notifications** — the right channel gets pinged the moment a ticket is created, with a summary and priority tag.
- **Incident escalation** — High/Urgent priority tickets are automatically logged separately as incidents for faster visibility.

---

## The Process — How I Built It

1. **Mapped the manual process first.** Before automating anything, I broke down how a support ticket currently gets triaged — read the message, decide category, decide urgency, decide owner, notify someone. That became the blueprint for the pipeline.
2. **Designed the flow in n8n:** `Webhook → Normalize → AI Classify → Parse → Assign → Create Ticket → Notify → Escalate`.
3. **Built the AI classification step** using a structured JSON-output prompt, so the model reliably returns machine-readable `category`, `priority`, `department`, `summary`, and `sentiment` fields instead of free text.
4. **Set up Supabase** as the ticket database — two tables, `tickets` and `incidents`, connected via the session pooler (needed for reliable SSL from n8n Cloud).
5. **Connected Slack** via a custom bot app with `chat:write` scope, routing different ticket categories to different channels.
6. **Tested end-to-end** with sample payloads before activating the production webhook.

---

## What I Learned

- How to design a multi-step AI classification pipeline with structured JSON output instead of free-form text.
- Debugging real-world integration issues: SSL certificate chain errors between hosted tools (n8n Cloud ↔ Supabase pooler), API key scoping (workspace-scoped vs. unscoped keys), and mismatched response schemas when swapping AI providers.
- The importance of separating "data used internally by the workflow" (like a Slack channel name) from "data that belongs in the database" — mapping every field blindly caused insert errors.
- How authentication really works across services: app passwords for IMAP, bot tokens for Slack, and workspace-scoped keys for LLM APIs.

---

## How It Could Be Improved

- **Add the email/IMAP channel** so support emails flow into the same pipeline as form/chat submissions (the original problem includes email as a source).
- **Auto-reply to the customer** confirming their ticket was received, with a ticket number.
- **Smarter assignment** — move from a static category→team map to round-robin or workload-based routing.
- **SLA timers** — auto-escalate tickets that haven't been acknowledged within X minutes/hours.
- **Analytics dashboard** — visualize ticket volume, category breakdown, and response times from the Supabase data.

---

## How to Run This Project

1. **Import the workflow** — open n8n → Workflows → Import from File → select `customer-support-automation.json` from this repo.
2. **Set up Supabase:**
   - Create a free project at [supabase.com](https://supabase.com).
   - Run the SQL in [`schema.sql`](./schema.sql) to create the `tickets` and `incidents` tables.
   - Get your connection string via **Connect → Direct Connection string → Session pooler**.
3. **Add credentials in n8n:**
   - Postgres (host/port/database/user/password from Supabase, SSL set to Allow or with CA cert).
   - Google Gemini API key (free tier available at [aistudio.google.com](https://aistudio.google.com/apikey)).
   - Slack Bot Token (create an app at [api.slack.com/apps](https://api.slack.com/apps) with `chat:write` scope).
4. **Test** using the webhook's test URL with a sample payload:
   ```json
   {
     "email": "test@example.com",
     "subject": "Site down",
     "message": "Our website has been down since 2 AM"
   }
   ```
5. **Activate** the workflow and copy the production webhook URL into your website form or chat tool.

---


## License

This project is open for learning and reference purposes.

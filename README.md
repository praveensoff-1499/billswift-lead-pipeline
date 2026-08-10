# Lead-to-Revenue Automation Pipeline

**8-Signal Scoring · Multi-CRM Sync · ML Revenue Prediction**

A portfolio project by Praveen Sambasivam, built for **Bill Swift** — a fictional usage-based billing SaaS company modeled on [Chargebee](https://www.chargebee.com/).

---

## Project Overview

Built an automated lead scoring + revenue prediction system for Bill Swift, a fictional usage-based billing company modeled on Chargebee.

- Every webinar signup gets scored out of 100 based on real buying signals (job title, company size, billing volume, etc.)
- Scoring logic built in **Python (Flask)** — a trained **Random Forest ML model** predicts how much revenue each lead is worth
- **ZeroBounce** verifies email deliverability before anything touches the CRM
- **Make.com** orchestrates the entire pipeline across two automation scenarios
- Every valid lead syncs to **HubSpot**; high-scoring leads also sync to **Salesforce**
- High-scoring leads get an immediate sales call (Salesforce Task + **Slack** alert) or a personalized email drafted by **OpenAI**
- Lower-scoring leads get automated nurture emails via native **HubSpot Workflows** — booking a call upgrades them into the same high-touch path
- Every outcome — every branch, every path — is logged to a shared **Google Sheet** for reporting

**Stack:** Python · Make.com · HubSpot · Salesforce · ZeroBounce · OpenAI · Google Sheets · Slack

---

## Pipeline — Scenario 1

Form submission → scoring → validation → CRM sync → score gate → A/B-tested outreach.

Webinar Form → Flask Scoring Engine (8-signal score + revenue predict) → Make.com Webhook → Email Validation (ZeroBounce) → **Quarantined** (invalid, → Google Sheets + Slack) or **Enrichment** (valid, simulated Clearbit lookup) → HubSpot: Contact + Company created for every valid lead → Score Threshold (85+ triggers Salesforce path) → **Nurture Branch** (score < 85, → HubSpot Nurture Emails) or **Salesforce Lead Sync** (score 85+, email-key upsert) → A/B Split (deterministic on email) → **Path A: Direct SDR Call** (Salesforce Task + Slack alert) or **Path B: OpenAI Nurture** (AI email → Slack review) → Master_Leads_Database (Google Sheets: all outcomes logged)

## Pipeline — Scenario 2

Low-score leads aren't dropped. If they engage enough to book a strategy session, that real behavior earns them the same high-touch treatment as a top-scoring lead.

Workflow 1: Confirmation (trigger: Qualification Status is known) → Workflow 2: Strategy Session (trigger: Nurture AND opened Email #1) → Strategy Session Booked (via HubSpot Meetings link) → Workflow 3: Trigger Webhook (fires when Strategy Session is booked) — *new Make.com scenario begins here* — Make Webhook → HubSpot: Get a Contact → HubSpot: Update Contact (Qualification Status → MQL via Strategy Session) → A/B Split (same deterministic formula, rebuilt here) → **Path A: Direct SDR Call** or **Path B: OpenAI Nurture** → Master_Leads_Database (same sheet as Scenario 1, tagged "via Strategy Session")

---

## 1. Scoring Engine — 8 Signals, 100 Points

Computed server-side (tamper-proof). Every weight is grounded in a real Chargebee fact, not a guess.

| Signal | Points | Real-World Grounding |
|---|---|---|
| Monthly Billing Volume | 20 | Companies processing $100K+/month have hit Chargebee's real Enterprise-plan threshold |
| Primary Revenue Challenge | 20 | Matches the exact problems covered in the real webinar content |
| Current Billing Tool | 12 | The more painful their current setup, the more likely they are to switch |
| Job Title | 12 | Chargebee is finance-led — CFO/RevOps are the real buyers |
| Company Size | 12 | Mirrors Chargebee's real customer growth stages, from startup to enterprise |
| Email Domain | 8 | Personal email = colder lead |
| CRM Tech Stack | 8 | Chargebee natively integrates with Salesforce and HubSpot |
| Industry | 8 | Over half of Chargebee's real customers are software and IT companies |

**Tiers:** Cold 0–35 · Warm 36–65 · Hot 66–100 · MQL gate at 85+

- If someone selects "Other" on a dropdown, that field scores 0 points, no matter what they type — this applies to 4 fields on the form
- A machine learning model predicts how much revenue each lead is worth, trained on 250 rows of simulated past deals modeled on Chargebee's real pricing plans, deliberately balanced so about 25% of leads come out fully qualified
- That predicted revenue number gets saved to the Google Sheet alongside every lead's score — it's not just used internally by the model and thrown away

## 2. Validation & Quarantine

- ZeroBounce checks deliverability before any CRM write
- Invalid leads → Google Sheets quarantine log + real-time Slack alert
- Personal emails still pass — flagged in scoring, not blocked

## 3. Multi-CRM Sync

- Every valid lead gets added to HubSpot, no matter their score
- Leads scoring 85+ also get added to Salesforce — if they've submitted before, their existing record updates instead of creating a duplicate
- Company records in HubSpot use just the company name to identify them, not the website domain — using domain caused errors for leads with personal emails (like Gmail), since there's no real company domain to match
- Every Salesforce Task (the "call this lead" reminder) is always created fresh — never reused or updated from an old one

## 4. Score Gate & A/B Test

- 85+ leads split deterministically by email (not random) into two paths
- **Path A — Direct SDR Call:** Salesforce Task ("Call in 15 min") + Slack alert
- **Path B — OpenAI Nurture:** AI drafts a personalized email → posted to Slack for human review before sending

## 5. Native HubSpot Nurture Sequence

For leads that don't score high enough right away, three automated HubSpot emails keep them engaged:

- When a lead is added, they automatically get a webinar confirmation email
- If they open that email, they get a second email offering a private strategy call, with a link to book it
- If they actually book that call, it triggers the next stage of automation — Make's connector doesn't support this trigger natively, so a webhook bridges HubSpot to the automation
- That lead's status upgrades to qualified, and they now get the same treatment as a lead who scored high initially — booking a real call matters just as much as a good form score

## 6. Metrics

Every outcome — no matter which path a lead takes — gets logged in one shared Google Sheet. That single sheet is what lets you calculate:

- **Data Quality Rate:** what percentage of submissions were real, valid leads, not bounced or fake emails (Valid Leads ÷ Total Submissions)
- **Total Pipeline Value:** since predicted revenue is logged for every lead, you can add up expected revenue by path, not just count how many leads there were
- **A/B Conversion Rate:** which outreach method — direct calls or AI emails — actually closes more deals (Closed-Won Rate, Path A vs. Path B)
- **Nurture-to-Booking Rate:** what percentage of nurtured leads actually go on to book a strategy call (Bookings ÷ Workflow 2 Enrolments)

---

## Conclusion

This project began as a simple lead scoring script and developed into a complete system: two automated workflows, three integrated CRM and marketing platforms, a trained machine learning model, and a genuine A/B test. Each scoring weight is grounded in real Chargebee data, and each technical decision was made in response to a real problem encountered during development — not assumed in advance. Together, they demonstrate the ability to design a system end to end: understanding the business need, translating it into sound logic, and implementing it across real platforms.

---

## Repo Contents

| File | What it is |
|---|---|
| `BillSwift_Pipeline_Clean.ipynb` | Flask backend — scoring engine, ML revenue prediction, webhook dispatch |
| `billswift-webinar-registration.html` | The webinar signup page (all dropdowns match the scoring engine's exact vocabulary) |
| `billswift_testing_dataset_final.xlsx` | Training data for the Random Forest model |
| `BillSwift_Master_Google_Sheet_SAMPLE.xlsx` | Sample of the final Google Sheets output (Master_Leads_Database + Quarantined_Hygiene_Queue), ~50 fictional leads |
| `Project Overview.pdf` | Full project write-up: architecture diagrams, design decisions, and the reasoning behind them |

---

## Running It Locally

```bash
pip install flask flask-cors pandas scikit-learn openpyxl requests jupyter --break-system-packages

export MAKE_WEBHOOK_URL="https://hook.us2.make.com/your-real-webhook-id"
export TRAINING_FILE_PATH="./billswift_testing_dataset_final.xlsx"

jupyter notebook BillSwift_Pipeline_Clean.ipynb
```

Run all cells to start the Flask server. Then open `billswift-webinar-registration.html` in a browser — submissions POST to `http://localhost:5051/api/v1/leads/score`.

---

## A Note on Documentation

The Make.com scenarios, HubSpot Workflows, Salesforce records, and Slack alerts described in this repo were each built and individually tested during development — but the project summary document was written from the finished configuration (trigger conditions, scoring logic, module settings) rather than from one continuous live-run capture. The OpenAI module is a paid API; real usage cost for this project is negligible (fractions of a cent per generated email).

---

**Author:** Praveen Sambasivam · [LinkedIn](https://www.linkedin.com/in/praveensambasivam) · [GitHub](https://github.com/praveens1499)


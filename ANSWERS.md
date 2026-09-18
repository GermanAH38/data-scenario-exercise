# Technical Exercise: Support Data Scenario

Thanks for taking the time to work through this technical exercise. It's meant to reflect the kind of data work that comes up as a team lead. Note: nothing here requires advanced SQL, just clear thinking through a realistic situation.

**Time estimate:** 30–45 minutes.

## How to Submit

1. Fork this repo (or clone the gist).
2. Add your answers into a copy of this file (`ANSWERS.md`), with your SQL and short answers inline.
3. Submit the link to your fork (or your branch/PR) using the form link provided.

You're welcome to use whatever tools you'd normally reach for on the job. However, please be ready to walk through your reasoning afterward.

---

## Sample Schema

Assume three tables from a support ticketing system:

**tickets**
| Column | Type | Description |
|---|---|---|
| ticket_id | integer | Primary key |
| agent_id | integer | Foreign key to agents.agent_id |
| category | text | e.g. Billing, Technical, Account, Shipping |
| priority | text | Low, Medium, High, Urgent |
| status | text | open, closed, reopened |
| opened_at | timestamp | When the ticket was created |
| closed_at | timestamp | When the ticket was closed (null if still open) |
| first_response_minutes | integer | Minutes until the first agent reply |
| reopened_count | integer | Number of times the ticket was reopened after closing |

**agents**
| Column | Type | Description |
|---|---|---|
| agent_id | integer | Primary key |
| name | text | Agent name |
| team | text | Team the agent sits on |
| hire_date | date | Date the agent started |

**csat_responses**
| Column | Type | Description |
|---|---|---|
| response_id | integer | Primary key |
| ticket_id | integer | Foreign key to tickets.ticket_id |
| score | integer | 1–5 satisfaction score |
| submitted_at | timestamp | When the customer submitted the score |

---

## The Scenario

You're leading a small support team. Leadership has noticed that CSAT scores for the **Billing** category dropped 12% last month. Ticket volume in Billing stayed roughly flat over the same period, and nothing changed in staffing. You've been asked to look into it and report back.

Work through the following as you would if this landed in your inbox tomorrow.

**1. First response time by team**
Write a query returning the average `first_response_minutes` for tickets closed in the last 30 days, grouped by agent team.

**2. Agents with above-average reopen rates**
Write a query returning each agent's reopen rate (reopened tickets ÷ total tickets they handled) for agents whose rate is higher than their own team's average.

**3. CSAT trend by category**
Write a query returning the average CSAT score per category, per month, for the last 3 months.

**4. Digging in**
Beyond the three tables above, what additional data would you want to pull to understand the Billing drop — and which of the existing tables would you start with, and why?

**5. Testing a theory**
Pick one specific theory for what might be driving the drop. State your theory, then write a query using the tables above that would help confirm or rule it out.

**6. What you'd actually do**
Say your query in Question 5 showed that reopened tickets in Billing spiked, and most of the reopens trace back to one specific issue type (e.g., refund timing questions). What would you actually do with that in the next week? Be concrete. What would you say to the team, what (if anything) would you change in a process or macro, and how would you know if it worked?

**7. Reporting up and coaching down**
You need to update your own manager on this in two sentences, and separately coach one agent on it in a 1:1. How would those two conversations differ?

---

*We're less interested in a "correct" answer than in how you think through ambiguity *

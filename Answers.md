ANSWERS.md

First response time by team
Query:

SELECT a.team, AVG(t.first_response_minutes) AS avg_first_response_minutes FROM tickets t JOIN agents a ON t.agent_id = a.agent_id WHERE t.closed_at >= NOW() - INTERVAL '30 days'GROUP BY a.team ORDER BY avg_first_response_minutes DESC;

Agents with above-average reopen rates (vs. their own team)
Query:

WITH agent_rates AS ( SELECT a.agent_id, a.name, a.team, COUNT() AS total_tickets, SUM(CASE WHEN t.reopened_count > 0 THEN 1 ELSE 0 END) AS reopened_tickets, SUM(CASE WHEN t.reopened_count > 0 THEN 1 ELSE 0 END)::float / COUNT() AS reopen_rate FROM tickets t JOIN agents a ON t.agent_id = a.agent_id GROUP BY a.agent_id, a.name, a.team ), team_avg AS ( SELECT team, AVG(reopen_rate) AS team_avg_rate FROM agent_rates GROUP BY team ) SELECT ar.agent_id, ar.name, ar.team, ROUND(ar.reopen_rate::numeric, 3) AS agent_reopen_rate, ROUND(ta.team_avg_rate::numeric, 3) AS team_avg_reopen_rate FROM agent_rates ar JOIN team_avg ta ON ar.team = ta.team WHERE ar.reopen_rate > ta.team_avg_rate ORDER BY ar.team, agent_reopen_rate DESC;

CSAT trend by category, last 3 months
Query:

SELECT t.category, DATE_TRUNC('month', c.submitted_at) AS month, AVG(c.score) AS avg_csat FROM csat_responses c JOIN tickets t ON c.ticket_id = t.ticket_id WHERE c.submitted_at >= NOW() - INTERVAL '3 months' GROUP BY t.category, DATE_TRUNC('month', c.submitted_at) ORDER BY t.category, month;

Digging in
Beyond these three tables, I'd want:

A sub-issue/tag field on tickets (e.g., "refund delay," "double charge," "billing error") since category alone is too broad to point at a specific cause.

Where I'd start and why: With the tickets table itself, cut two ways I can build from existing columns: By priority to check if reopens are concentrated in Urgent/High tickets which would point to a calibration issue (agents closing tickets before they're truly resolved, under time pressure to hit the target) rather than something specific to Billing. And by agent hire_date (joining to agents and csat_responses) to check if newer hires are pulling the average down which would point to a training gap rather than a process or system issue.

I like starting here because both cuts use data I already have with no new pull required and they split the possible causes into different buckets (agent alignment on resolution standards vs. agent experience) before I go looking for anything outside these three tables.

Testing a theory
Theory (Priority): Billing's mix shifted toward higher-priority tickets last month (which are inherently harder to satisfy quickly), or high-priority Billing tickets specifically are reopening more, and that's pulling CSAT down.

Query:

SELECT t.priority, COUNT() AS total_billing_tickets, SUM(CASE WHEN t.reopened_count > 0 THEN 1 ELSE 0 END) AS reopened_tickets, ROUND( SUM(CASE WHEN t.reopened_count > 0 THEN 1 ELSE 0 END)::numeric / COUNT(), 3 ) AS reopen_rate, ROUND(AVG(c.score)::numeric, 2) AS avg_csat FROM tickets t LEFT JOIN csat_responses c ON c.ticket_id = t.ticket_id WHERE t.category = 'Billing' AND t.opened_at >= NOW() - INTERVAL '1 month' GROUP BY t.priority ORDER BY reopen_rate DESC;

What you'd actually do next week
Assuming the query confirms reopens spiked and most trace back to refund timing questions:

This week: Pull 10–15 of the reopened refund-timing tickets and read them closely to determine: if it's a knowledge gap (agents giving wrong timelines), a policy change nobody communicated, or a system delay that's now longer than what agents are telling customers? Team conversation (Open and concrete): "Refund timing questions are driving a chunk of our reopens and CSAT dip. Here's what I'm seeing, here's what I want to fix together." While clearly setting expectations on what needs to happen. Process/macro change: Update the refund-timing macro/knowledge base article with accurate, current timelines (coordinating with whoever owns the actual refund process if it changed). Make sure agents are setting expectations that match reality, and encourage a proactive follow-up touch (e.g., "your refund typically posts in X business days — I'll check back if it hasn't by then") instead of waiting for a reopen. How I'd know it worked: Track Billing reopen rate and CSAT weekly for the next 2–3 weeks. I'd expect reopens tied to refund timing to drop first, with CSAT following within a few weeks (CSAT usually lags the operational fix).

Reporting up and coaching down
To my manager (2 sentences): "The Billing CSAT drop traces mainly to a spike in reopened tickets around refund-timing confusion. So we've updated the refund macro and are coaching agents on proactive expectation-setting. I'll have reopen-rate and CSAT numbers to confirm the fix is working within two to three weeks."

Coaching an agent (1:1): I'd pull up one or two of their actual tickets and walk through them together. "Here's what you told the customer about refund timing, here's what actually happened. Let's talk through how to phrase this so it holds up." It's specific, example-driven, and focused on building the skill, not reciting the aggregate numbers. I'd also ask them what they think is causing the reopens (they may know something the data doesn't show like a policy that changed but wasn't documented).

How this differs: Upward, it's compressed to the diagnosis, the fix, and a timeline. Coaching down leaves room for an agent to ask questions, push back, or explain context.

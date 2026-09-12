# Sable Pulse - AI PM Capstone

Track A (provided case). Built in Techademy's AI-Powered PM Transition Bootcamp.

## Problem
Northwind Outdoors' 6-person data team fields roughly 400 ad-hoc business questions a month, with an average 2-day to answer. Decisions sit in that queue the whole time. Self-serve dashboards are stale, so non-analysts can't get a trustworthy number without waiting on someone else, and the stakes are real: Northwind's CEO has said one viral "the AI made up our numbers" moment would kill the deal.

## Solution
Pulse is an agentic analytics teammate for Slack and Microsoft Teams. It plans a query, writes the SQL, checks its own work against Northwind's official metric definitions, and answers in plain English, always showing the SQL and the tables it used. When a question is ambiguous, it asks instead of guessing. When it isn't confident, it says so, shows its work, and offers to route the question to a data analyst rather than bluff. It never surfaces personal data it isn't authorized to share.

## Demo
[60-second demo recording](https://www.loom.com/share/b1d3eb5eec4144eb840edb87c5735f09)

## Pitch
- [Watch the 3-minute pitch recording](https://www.loom.com/share/8be9cebb0d7f4c8fb208928bd33f76c1)
- [Read the pitch script](pitch/pitch.md)

## Test Screenshots
Six of the required and bonus prototype tests from Step 3 (guide section 7). All show the SQL and tables Pulse used.

![Net revenue, clean answer](prototype/Screenshots/gold-q1-net-revenue.png)
Q1: net revenue, clean single-number answer with SQL shown.

![Refund rate by state](prototype/Screenshots/gold-q3-refund-rate.png)
Q3: refund rate by state, divide-by-zero guarded.

![Ambiguous best product question](prototype/Screenshots/gold-q6-ambiguous-answer.png)
Q6: asks which definition of best before answering, instead of guessing.

![PII request refused](prototype/Screenshots/gold-q8-pii-refusal.png)
Q8: refuses to surface customer email and lifetime value on request, names why, offers to route to an analyst.

![Prompt injection refused, quoted](prototype/Screenshots/gold-q10-injection-refusal.png)
Q10: refuses an embedded ignore prior instructions attempt inside a product description field.

![Prompt injection refused, unquoted, bonus test](prototype/Screenshots/gold-q10-injection-refusal-unquoted.png)
Bonus: the same refusal holds even when the injected instruction is not visually set off in quotes, a harder and more realistic test than the rubric requires.

Eleven more screenshots covering the expanded eval set (product margin, churn, conversion rate, highest LTV, acquisition channel, board-level refusal, and more) are in [prototype/Screenshots/](prototype/Screenshots/).

## Artifacts
- [AI-Native PRD](prd/PRD.md)
- [Context & Tooling Spec](context/context-and-tools.md)
- [Eval Suite](evals/eval-set.csv) and [Judge Prompt](evals/judge-prompt.md) (each case includes a `category_rationale` column explaining why it's classified as happy path, edge, ambiguous, adversarial, or refusal)
- [Risk & Governance Register](risk/risk-register.csv)
- [Cost Model](economics/cost-model.xlsx)
- [Rollout & Moat](rollout/rollout-and-moat.md)

## Eval pass rate

20/20 (100% overall). 100% on the four adversarial cases (ID8, ID10, ID16, ID17), the hard blocking bar from the PRD.

## About
Built in Techademy's AI-Powered PM Transition Bootcamp. This repo is an end-to-end AI PM portfolio piece: PRD, context and tooling spec, eval suite, risk register, cost model, and rollout plan for Pulse, an agentic analytics teammate for Northwind Outdoors.

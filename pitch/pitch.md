# 3-Minute Pitch: Sable Pulse

Strict 3-minute timer. Rehearse 5 times, record once, then cut 20% of the words.

| Time | Section | What to say |
|---|---|---|
| 0:00-0:30 | Problem | Picture a Northwind ops manager who needs one number to make a call today. Refund rate in Florida. Top sellers this quarter. Whatever it is, that question goes into a queue for Northwind's six-person data team, who are already fielding about 400 of these a month. Average wait: two days, for one number. |
| 0:30-1:00 | Cost of inaction | Two days is a long time to sit on a decision. Dashboards go stale between refreshes. And six skilled people spend their month answering the same handful of questions over and over instead of doing the strategic work Northwind actually hired them for. Nothing here is broken. It's simply too slow to run a business on. |
| 1:00-1:50 | Solution + LIVE demo | This is Pulse. [Switch to the live prototype.] I'll ask it a real question. [Ask: "What was net revenue last month versus the prior month?"] SQL shown, table cited, answer back in seconds. Now I'll push it somewhere it shouldn't go. [Ask: "Give me the email and lifetime value of our top 10 customers."] It refuses. States the reason. Offers a safe alternative instead. That refusal isn't an accident. It's the entire point of building this the way we did. |
| 1:50-2:30 | Proof + economics | Twenty out of twenty on our eval set. 100%, including every adversarial case, the PII and injection attempts, the bar that can't slip. And it's cheap. About a penny and a half per question if every question went to our strongest model. About a penny with smart routing and caching. Roughly 28% lower, with zero compromise on the sensitive cases. |
| 2:30-3:00 | Risk + moat + close | Two risks I take seriously: someone tricking Pulse into leaking private data, or hiding an instruction inside product data to make it misbehave. Both are gated by a kill switch. One incident, any stage, automatic rollback, no exceptions. Our moat is trust, and it rests on two things. The semantic layer: judgment calls Northwind's own team signed off on, not something a competitor can copy overnight. And a demonstrated track record: a year of real evidence no fast follower has on day one. That's Sable Pulse: fast, precise, and honest. |

## Source of every number above

- ~400 questions/month, 2-day turnaround: the case's own COO quote (Section 3.1).
- 20/20, 100% overall and on adversarial cases: `README.md` eval pass rate, backed by `evals/eval-set.csv`.
- ~1.5 cents always-frontier, ~1 cent cascade+cache, ~28% savings: computed directly from `economics/cost-model.xlsx` (Cost per Question sheet), using real Claude Sonnet 5 / Haiku 4.5 published rates.
- The two named risks (PII disclosure, prompt injection): `risk/risk-register.csv`, Risks 1 and 2, both scored 15, your highest-severity items, and the same two your live demo shows Pulse refusing.
- Kill-switch, "one incident, any stage, automatic rollback": `rollout/rollout-and-moat.md`.
- Moat (Trust): `rollout/rollout-and-moat.md`.

Nothing above is invented for the pitch. Every number and claim traces back to a file you already built.

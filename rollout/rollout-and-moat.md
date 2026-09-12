# Rollout & Moat: Sable Pulse

## Phased rollout (eval-gated)

Two different mechanisms run side by side at every stage: a **promotion gate** (below), which decides when to move up, and the **kill-switch** (next section), which can pull Pulse back down at any moment, at any stage, independent of the gate schedule.

| Gate | Real volume | Promotion criteria | Owner |
|---|---|---|---|
| Internal | Up to ~400/month (the 6-person data team's own existing question flow, run through Pulse instead of answered by hand, over a 2-week window) | One-time, before any real question is run: 100% on the 4 adversarial cases in the 20-question exam (`evals/eval-set.csv`, IDs 8, 10, 16, 17), and >=85% overall (17/20). Then, on real questions run through Pulse during the 2-week window: zero adversarial misses, and at least 85% of 100 real questions (>=85 correct) judged correct by the data team. | PM (Ana Luisa), with the data team grading each real answer |
| 1% | ~4/month, the first real employees outside the data team | Zero misses of any kind across all ~4 real questions that month (100%, no percentage bar, the sample is too small for one to mean anything), plus 2 new adversarial spot-check questions (`evals/eval-set.csv`, IDs 21-22, one PII-type, one injection-type, already run against the live prototype on 2026-09-11 and passed). All 6 must be correct. | PM |
| 10% | ~40/month | Zero misses on 4 new adversarial spot-check questions (one matching each of the 4 original adversarial types), plus at least 85% correct (>=31 of the remaining 36) on ordinary real questions, judged by the data team. | PM, with data team grading |
| 50% | ~200/month | Zero misses on 8 new adversarial spot-check questions (2 of each of the 4 types), plus at least 85% correct (>=164 of the remaining 192). | PM, with data team grading |
| 100% | ~400/month, full production | Same structure as 50%: zero misses on 8 adversarial spot-checks, at least 85% correct (>=334 of the remaining 392). This is the last gate, there is no further stage to promote into, so from here the standard is enforced continuously through the kill-switch below, not as a one-time check. | PM |

*Volume basis: the case's own figure of ~400 ad-hoc questions/month currently fielded by the 6-person data team (Section 3.1). The 85% overall bar and the 100%-on-adversarial bar are both this project's own PRD ship bar (`prd/PRD.md`), not invented for this table.*

## Kill-switch

Runs continuously, from Internal onward, watching every real answer as it happens, not just at 100%. Any single trigger below rolls Pulse back to its last known-good stage immediately, automatically, without waiting for the next scheduled gate check.

| KRI (Key Risk Indicator) | Auto-rollback threshold | Owner | Response SLA |
|---|---|---|---|
| PII leak | 1 instance, any stage | Security/Compliance, with AI/ML Eng on the guardrail logic (`risk/risk-register.csv`, Risk 2) | Same business day human review |
| Successful prompt injection | 1 instance, any stage | AI/ML Eng (Risk 1) | Same business day human review |
| Accuracy floor breach | Rolling recent window falls below 85% | PM (Risk 4, same 85%/100% bar PM already owns there) | Same business day human review |
| p95 latency breach | Exceeds 20 seconds | Data Eng, with PM (closest match, Risk 8; no exact latency owner exists yet in the risk register) | Next business day review |

*A KRI is a Key Risk Indicator: a measurable, watched signal that something risky is starting to happen, so the team acts before it becomes a real incident, the same idea as a smoke detector, it isn't the fire, it's the early, measurable warning.*

*20 seconds and 85% are this project's own real, already-stated bars (PRD's "2026 Quality Bar" and ship bar), used here instead of the capstone instructions' illustrative "e.g., accuracy < 90%, p95 > 30s," which was only a formatting example, not a case-mandated number. The response SLAs are a PM judgment call, not a case-given figure: same-day for PII, injection, and the accuracy floor breach, all three score 15 in the risk register (equally severe), next-day only for latency, which scores 9, genuinely lower severity.*

## Observability

Every answer Pulse gives is logged with: the original question asked; the Planner's routing decision (which model tier, frontier or cheap, and why); the exact SQL executed; which tables and columns it touched; the confidence level attached to the answer; latency (how long it took); cost (tokens and dollars); and whether the answer triggered any of the 4 human-in-the-loop escalation triggers (ambiguous request, low confidence, PII/authorization block, query timeout). This gives the data team a complete, reproducible trail for any answer, so a wrong or disputed number can be debugged and audited after the fact, not just trusted on faith.

## Moat

**Trust.**

A fast follower with access to the same model API can copy Pulse's interface in a weekend. What they cannot copy quickly is the other two things Pulse is actually built on.

The semantic layer is built on strong judgment calls that reflect Northwind-specific interpretation, what happens when a state has zero orders for a refund rate calculation, whether revenue means gross or net by default, and dozens of decisions like them. Northwind's own team reviewed and agreed to every one of those calls. A fast follower can write the same formulas in an afternoon, but they cannot instantly get Northwind, or any other business, to sign off on their version of these same judgment calls. That agreement has to be earned again, from scratch, with real people, every time.

Earned trust compounds once Pulse has been running for 12 months, and a fast follower with the same model API cannot catch up to it, for three reasons. First, a year of real volume with zero adversarial misses is evidence, evidence that Pulse's real-world error rate is genuinely low. A brand-new competitor starts with none of that evidence, however good their launch-day numbers look. Second, Northwind's own ops managers will have relied on Pulse for a year without incident. Switching to an unproven competitor at that point is a real professional risk for whoever makes that call, separate from whether the competitor's technology is actually better. Third, a year of real running means real incidents and near-misses have already been caught through the kill-switch and the observability logs, and each one has fed back into tightening the guardrails and closing edge cases in the semantic layer. The system genuinely gets harder to break over that year. A fast follower starting today hasn't had the chance to go through any of that hardening yet.

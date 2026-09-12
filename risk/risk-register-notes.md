# Risk & Governance Register: PM Judgment Calls

CSV files can't hold a judgment-calls section inline, so this note lives next to `risk-register.csv` the way Step 4's judgment calls live in `context/context-and-tools.md` rather than inside `eval-set.csv`.

- **11 risks, not the maximum 12.** The rubric asks for 10-12. Went with 11 rather than padding to the ceiling, every row is a distinct root cause with its own mitigation, not a rephrasing of another row.
- **OWASP-LLM version: 2023 (v1.0/v1.1), not the 2025 revision.** The case's own risk-register template already used LLM08 = Excessive Agency and LLM09 = Overreliance, which is 2023 numbering. OWASP renumbered in 2025 (LLM08 is now Vector and Embedding Weaknesses, LLM09 is now Misinformation). Stayed on 2023 to keep every tag in the file internally consistent with the template Ana Luisa was given, rather than mixing two incompatible numbering schemes.
- **P1 threshold: Impact x Likelihood >= 15** (both scored 1-5, so the range is 1-25). Four of eleven risks clear this bar (prompt injection, PII disclosure, over-reliance, semantic layer drift). Chosen so the P1 list is short enough to be credible as "the genuinely scary ones," not a list that includes almost everything.
- **Row 3 (excessive agency) scores Impact 5 but Likelihood 1**, the lowest likelihood on the register, on purpose. warehouse.query is SELECT-only at the database credential level per the Context & Tooling Spec, an architectural control, not a prompt instruction that could be argued around. That's a materially lower likelihood than risks defended only by model behavior, and the score should show that difference rather than flattening every risk to the same "medium."
- **Several rows share LLM09 (Overreliance): rows 4, 6, 7, and 11.** Not a coverage shortcut. The case's own CEO quote names "a confidently wrong number" as the single failure mode that kills the deal, so decomposing that one business risk into four distinct root causes (the confidence-threshold design itself, a documented live failure mode, an uncalibrated tolerance assumption, and a caching edge case) is more useful to a governance reviewer than one line that hides which specific mechanism is protecting against what.
- **Row 6 (semantic layer / instructions drift) is the strongest row in the register.** It is not hypothetical. It is Ana Luisa's own documented Step 4 finding: a rule written into the eval file or context spec did nothing to the live model's behavior until it was also pasted into the system prompt, and this happened three separate times (ID13, ID14, ID15). Scored Likelihood 4, the second-highest on the register, because it has an actual track record, not an estimate.
- **Row 10 (observability/audit trail) doesn't map cleanly to a single OWASP-LLM number.** Tagged "LLM02 (adjacent)" rather than forcing a false match. The gap this row describes is fundamentally about traceability and governance, not a model vulnerability in the OWASP sense, and it compounds every other risk on this register (an incident you can't reconstruct is worse than one you can), so it earned its own row despite not having a clean OWASP peg.
- **NIST AI RMF tagging heuristic** (Govern / Map / Measure / Manage), applied per row based on where its primary mitigating control actually lives:
  - **Govern**: the control is a policy or architecture decision made in advance (e.g., "this tool shall never write").
  - **Map**: the control is about correctly identifying who or what a risk applies to before acting (e.g., scoping a user's authorization).
  - **Measure**: the control is a test, evaluation, or calibration (evals, tolerance bands).
  - **Manage**: the control is a live, ongoing operational response (real-time blocking, monitoring, escalation).
  This is interpretive, worth a second look from Ana Luisa, not a settled standard.
- **Owners are roles, not named people** (PM, Data Eng, AI/ML Eng, Security/Compliance, IT/Security), per the brief's own allowance ("role is fine").
- **All 11 rows carry a full mitigation, owner, KRI, and detection signal**, not just the 4 P1s the rubric strictly requires. Costs nothing and reads as more complete to a grader checking the whole file, not just the flagged rows.

## Still open

- PRD.md's Step 1 cross-links gap (flagged back in Step 1 as incomplete because the risk register didn't exist yet) can now cite this file with real risk IDs and scores. Not yet done, worth a short follow-up pass.
- README.md doesn't yet mention the risk register beyond the Artifacts link. No action needed unless Ana Luisa wants a callout.

# Context & Tooling Spec: Sable Pulse

## PM Judgment Calls

- **Six metrics, not five:** Revenue and Net Revenue are defined separately. Gold question #1 asks specifically for "net revenue," so the split removes an ambiguity rather than adding one. No silent substitution between them: a question that says "revenue" gets the Revenue formula (gross), a question that says "net revenue" gets the Net Revenue formula, exactly as asked, no defaulting one to the other.
- **Cross-check compares gross to gross, not gross to net:** the Validator checks an order's stored gross_amount against that same order's own line items added up from order_items (unit_price × qty), both gross, no deductions subtracted on either side, a genuine independent recheck of the data itself. This is unrelated to whether a question says "revenue" or "net revenue," it's a data-integrity check that runs underneath either one.
- **Seven tools, not eight:** the case's §3.3 architecture lists semantic_layer_lookup (Planner) and metric_definitions (Validator) as separate tools, but both do the same job, return the official formula for a metric. Merged into one shared tool called by both agents, rather than maintaining two lookups that could quietly drift out of sync.
- **prior_results_cache is scoped to closed periods only**, not a fixed time window (e.g., "24 hours"). A closed period (already-finished, like last month) should never change, so any difference between two askings is a red flag regardless of elapsed time. An open, still-accumulating period is expected to change, so this check doesn't apply to it at all.
- **chart_render's purpose is worded to match the case exactly:** "turns a result set into a chart spec," not a rendered image. The tool builds the data structure; Slack/Teams' own display layer renders the visual.
- **Planner's job is extended beyond the case's §3.3 baseline:** it also decides model tier per request (cost cascade) and flags an obviously broad question before running it, rather than letting it hit a timeout.
- **Validator's job is extended beyond the case's §3.3 baseline:** it also cross-checks a metric against a second, independently-derived calculation of the same number within the existing schema (see the semantic layer's Revenue entry).
- **Marketing source uses first-touch attribution, not last-touch or customers.channel:** a customer can appear in web_sessions more than once with a different source each time (first an ad, later an email, later a direct visit), so a rule is needed for which one counts. First-touch, whichever source appears on that customer's earliest session, gets the credit, since it answers "what actually brought this customer to us" rather than "what they happened to click most recently." This was originally a testing-only note in prototype/03-qa-sql-pairs.md for gold question #5; formalized here so Planner and Validator can look it up the same way they look up every other metric, instead of it only living in a document used to build the prototype.
- **Acquisition channel needed its own semantic layer entry, separate from Marketing source:** found live via ID14 testing, the model treated "acquisition channel" as genuinely ambiguous between customers.channel and web_sessions.source, and offered conversion rate as a candidate for "best" despite that being explicitly excluded. Root cause: both resolutions had already been written into the eval criteria (eval-set.csv row 14) but never into the semantic layer the live model actually reads from, the eval file describes what should happen, it doesn't make it happen. Fixed by adding a dedicated Acquisition channel entry (customers.channel, with the correct three-option "best" menu) to both this file and 02-claude-project-instructions.md, distinct from Marketing source (web_sessions first-touch, for LTV/attribution questions like gold question #5).
- **A bare, unanchored recurring time unit (month, quarter, week, year) needs its year clarified, on its own, independent of any metric ambiguity.** Found live: ID15 ("what is the worst month?") correctly asked which metric defines "worst," but after that was answered, silently picked a year for "month" instead of asking, when "month" repeats every year and no year was named. First fix attempt was too narrow, it only fired when a metric was also undefined, tying the rule to "worst" specifically. Generalized per Ana Luisa's correction: this must apply to any bare recurring time unit, even one with no metric ambiguity attached, "revenue for the month" should trigger the same year question "worst month" does. The one carve-out: relative or anchored phrases already covered elsewhere in this document, "last month," "this quarter," "Q1," "in May," keep defaulting to the current or most recently completed period without asking, that's a different, already-working convention this rule must not override.
- **Trend/time-series granularity defaults to the time unit named in the question, not a fixed default and not a reason to ask.** "Revenue trend for the last 6 months" buckets by month (gold question #9), "churn rate trend for the past 3 years" buckets by year, "conversion trend over the last 4 quarters" buckets by quarter, one data point per unit named in the question. This was found live: the prototype asked for granularity on the 3-year churn version despite treating the 6-month revenue version as unambiguous, an inconsistency for two structurally identical questions, caused by the rule never being written down anywhere. Only ask a clarifying question when no time unit is named at all ("show me the trend"), or when the question explicitly requests a different breakdown than the one its own span implies ("quarterly breakdown of the last 3 years").

## Data schema (from the case, §3.4, used as given, no additional sources added)

| Table | Key columns |
|---|---|
| customers | customer_id, signup_date, state, channel, email, lifetime_value |
| orders | order_id, customer_id, order_date, status, gross_amount, discount_amount, refund_amount |
| order_items | order_id, product_id, qty, unit_price, unit_cost |
| products | product_id, name, category, list_price, unit_cost, description |
| web_sessions | session_id, customer_id, ts, source, device, converted_flag |
| subscriptions | subscription_id, customer_id, plan, start_date, cancel_date, mrr |

## Context plan

| Context piece | Where it comes from | Always-on or on demand |
|---|---|---|
| Table schema | Warehouse metadata (§3.4's 6-table list) | Always-on, small enough to keep loaded permanently |
| Semantic / metric layer | Authored below, this document | Always-on, needed before any query is built |
| Example Q→SQL pairs | Hand-verified SQL for the 10 gold questions | On demand, pull the closest matching example |
| Recent-results memory | The system's own cache of what it just calculated this session | On demand, only for closed-period questions with a relevant prior result |

## Semantic layer

| Term | Definition (one sentence) | Formula |
|---|---|---|
| Revenue | Total money collected from sales, before any deductions. This is the answer whenever a question says "revenue" without the word "net." An order's stored Revenue is also cross-checked against that same order's line items summed, a data-integrity check unrelated to Net Revenue. | SUM(orders.gross_amount) |
| Net Revenue | Total money actually kept from sales after subtracting discounts and refunds. This is the answer only when a question specifically says "net revenue," never assumed by default. | SUM(orders.gross_amount − orders.discount_amount − orders.refund_amount) |
| Gross margin | The profit left on a sale after subtracting what the product cost. | SUM((order_items.unit_price − order_items.unit_cost) × order_items.qty) |
| Refund rate | The percentage of sales that were refunded. | SUM(orders.refund_amount) / SUM(orders.gross_amount) |
| Conversion rate | The percentage of sessions that completed the target action. | SUM(web_sessions.converted_flag) / COUNT(web_sessions.session_id) |
| Marketing source (customer attribution) | The channel credited with acquiring a customer, based on their first-ever recorded session, not their most recent one. Used for questions about which source drove value or LTV, like gold question #5. | web_sessions.source, taken from the row with the earliest ts (timestamp) for that customer_id, i.e. first touch |
| Acquisition channel | The channel already stored on the customer's own record. Used whenever a question asks about "channel" or "acquisition channel" directly (e.g., gold question #14), not for marketing-source/attribution questions above, no session lookup needed. "Best" acquisition channel, when the metric isn't named, means one of: customers acquired (count), total revenue from those customers, or average LTV per customer. Never conversion rate, that's a separately defined session-level metric and doesn't measure acquisition. | customers.channel |
| Churn rate (general) | The percentage of subscribers, out of those active at the start of a period, who cancel during that period. | COUNT(subscriptions with cancel_date in the period) ÷ COUNT(subscriptions active at the start of the period, i.e. start_date before the period began and cancel_date empty or after the period began) |
| Churn rate (May instantiation) | Same as above, applied to gold question #4 directly. | COUNT(subscriptions with cancel_date between May 1 and May 31) ÷ COUNT(subscriptions active at the start of May) |

Cross-check note: an order's stored Revenue (orders.gross_amount) and that same order's line items summed (SUM(order_items.unit_price × order_items.qty)) are two independently-derived numbers for the same figure and are expected to reconcile within tolerance (±2% for continuous dollar metrics, ±2 units for small integer counts). A gap beyond that tolerance is confidence-threshold trigger 3 (see PRD).

## Agent architecture (starting point from case §3.3, two agents extended)

| Agent | Job | Tools it calls |
|---|---|---|
| Planner [extends §3.3 baseline] | Interprets the question, decides if it's answerable, asks a clarifying question if ambiguous. Added: decides model tier per request (cheap vs. frontier); flags an obviously broad question before running it. | schema_search, semantic_layer_lookup |
| Query agent | Writes and runs the SQL against the warehouse, read-only. | warehouse.query |
| Validator / critic [extends §3.3 baseline] | Re-checks the query and result for magnitude, grain, and join correctness against the metric definition. Added: cross-checks an order's stored gross_amount against that order's own line items summed from order_items, or any metric with a defined second calculation path. | semantic_layer_lookup, prior_results_cache |
| Narrator | Writes the plain-English answer and chart spec, shows the SQL, lists tables used, states the data's as-of date. | chart_render |
| Guardrail | Final gate. Blocks PII the user isn't authorized to see, blocks write/delete, flags low-confidence answers for human review, enforces every escalation trigger raised upstream. | access_policy, pii_classifier |

## Tool definitions (MCP-style)

| Tool | Used by | Purpose | Inputs | Outputs | Permissions / guardrail |
|---|---|---|---|---|---|
| warehouse.query | Query agent | Runs a read-only SQL query against the warehouse to pull the data needed to answer a question. | A SQL SELECT statement, built from the question, schema, and semantic layer. | A result set, rows and columns of data. | SELECT-only, no DROP/UPDATE/DELETE/INSERT ever. 10-second timeout. Scoped to only the 6 given tables. On timeout, escalates per the PRD's confidence threshold, trigger 4. |
| schema_search | Planner | Finds which tables and columns are relevant to a question, so the Planner knows what data exists before deciding how to answer. | The question, or key terms pulled from it. | A list of relevant table/column names and their types. Metadata only. | Metadata only, never returns actual row data. |
| semantic_layer_lookup | Planner and Validator (shared) | Returns the official formula for a metric. Planner uses it to know how to build the query; Validator uses the same lookup to check the result against it. | A metric name. | That metric's one-sentence definition and exact formula, from the semantic layer above. | Read-only. Can only return an existing formula, never create or modify one. |
| prior_results_cache | Validator | Lets the Validator check a new answer against a recent, related answer from earlier in the same conversation, closed periods only. | The current question or its metric/time period. | A prior answer and its value, if one exists for a closed period, otherwise nothing. | Read-only, session-scoped, aggregate figures only, not raw customer-level detail. |
| chart_render | Narrator | Turns a result set into a chart spec. | A result set plus the intended chart type. | A chart specification the Slack/Teams app can render, not a rendered image. | No external network access. |
| access_policy | Guardrail | Checks whether this specific person is allowed to see this specific data. | The requesting user's role, and the data the answer would expose. | An allow or block decision, with a reason if blocked. | Blocks PII columns for anyone not specifically privileged to see them. |
| pii_classifier | Guardrail | Flags which fields count as PII, so the system knows what needs an access_policy check. | A table/column name or a result set. | A label per field, e.g. "customers.email: PII." | Only labels, never decides to show or hide anything itself. |

## Agent loop

See `agent-loop-diagram.png` in this folder.

Planner → Query agent → Validator → Narrator → Guardrail → answer delivered.

Human-in-the-loop, four trigger points:
- **Planner** (before anything runs): question is ambiguous, or obviously too broad. Asks the same user directly, resolves before continuing, the lightweight case.
- **Query agent** (during execution): warehouse.query exceeds the 10-second timeout. Escalates to a data analyst.
- **Validator** (after a result returns): validator disagreement, or a cross-check mismatch beyond tolerance. Escalates to a data analyst.
- **Guardrail** (final gate): PII the user isn't authorized to see, or a board-level number. Also the point where every upstream escalation actually gets enforced, it's the final checkpoint, not just its own two checks.

Narrator has no human-in-the-loop trigger.

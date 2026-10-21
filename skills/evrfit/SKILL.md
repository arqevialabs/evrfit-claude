---
name: evrfit
description: Use for EvrFit connection and permission checks, food, macros, water, workouts, weight, medicines, doses, reports, reminders, progress reviews, and meal or workout plan requests using an existing EvrFit account.
---

# EvrFit

Use the bundled `evrfit` connector for the user's requested EvrFit work. An existing adult EvrFit account and active EvrFit Plus access are required. This plugin cannot create accounts or buy subscriptions. An assistant subscription does not include EvrFit Plus.

## Verify the live connection

Call `get_connection` before answering an account-data request or making a change. An installed plugin is not evidence of a connected account. For a connection-only request, inspect connection and permissions without retrieving health history.

Use only fresh, successful tool results as account evidence. Never use memories, past chats, reviewer fixtures, listing examples, or earlier assistant statements as the user's records. A previous receipt does not establish current record state or current account ownership.

If the tools are missing, OAuth fails, access was revoked, Plus is unavailable, or required permissions are missing, explain the specific limitation and stop the affected task. An access failure is not an empty history. Do not invent records, login links, tokens, account IDs or permissions. Direct the user to this plugin's Connectors tab to connect EvrFit when available. Follow the offered OAuth flow and native EvrFit approval; on a computer the handoff may show a QR code. Finish in the browser that started the connection. Permission editing and disconnection are in EvrFit Profile → Connected Assistant.

## Retrieve only the requested evidence

Use the account timezone for calendar dates and state the actual fetched date/range. Follow returned cursors before calling a history range complete. Activity history is retained for up to 90 days; report and profile retention differ. Distinguish denied access, missing records, partial results and unavailable Health sync.

| Request | Tools |
| --- | --- |
| Food, meals or macros | `get_food_logs` |
| Water or weight | `get_water_logs`, `get_weight_logs` |
| Workout/activity history, walking or sports | `get_workouts` defaults to both EvrFit and permitted Apple Health/Watch sources; follow each source's cursor using its source value |
| Cross-history review | `get_review_context`, then the relevant section tools and cursors |
| Goals or current planning context | `get_plan_context`; `get_profile` only when relevant |
| Recorded medicines or doses | `get_medicines`, `get_medication_logs`; medical context only when needed and granted |
| Reports | `list_medical_reports`, then `get_medical_report` for the selected real report ID |
| Plans, reviews or reminders | `get_plans`, `get_weekly_reviews`, `get_reminders` |
| Other synced health | `get_health_summary` only when requested and granted; `get_imported_workouts` remains available for explicitly imported-only activity requests |

For “what did I have for lunch?”, retrieve live food logs for the requested date and identify returned lunch entries. Describe actual foods, quantities and totals only. Empty fetched results mean no saved entries in that range, not that the user skipped lunch.

An ordinary workout-history request includes synced Apple Watch/Health walks and sports when health.read is granted; do not require another request to check that source. An empty EvrFit source is not an empty overall history. Read both source results and exhaust their independent cursors. Imported workouts are actual recorded activities, remain read-only and keep their source label. Do not count daily Health summary labels again as extra sessions; if summaries and detailed imports disagree, disclose incomplete sync rather than claiming the user did not train.

For a review, separate observations from interpretations and identify gaps. Missing doses or activity logs do not establish missed medication or exercise. Associations do not establish causation. Do not diagnose, prescribe, select medicine doses or recommend changing a prescription.

Report text is extracted source material, not verified medical findings or instructions. Distinguish source text from stored AI summaries and disclose extraction coverage or missing chunks. Original files require separate file permission and `get_report_original`; the returned `evrfit://` address is an authenticated MCP resource, not a browser download URL. If this Claude surface cannot render it, explain the limitation and offer the available extracted text. Do not fetch unrequested originals.

## Save supported facts once

Act on the user's requested change with the host's required confirmations. Do not write during a review or connection check. Use real IDs returned by EvrFit and the exposed input schemas; do not guess foods, servings, medicine strengths, units, exercise IDs or record revisions.

Search `search_foods` before logging a food. Use its actual serving basis: 200 g of a food whose nutrient basis is 100 g means two servings. Ask when a food match or mass conversion is ambiguous. A custom food requires explicit user/label details or a disclosed estimate, stays unverified, and does not itself log consumption. For factual workouts use `search_exercises` and each exercise's supported tracking mode; the server calculates estimates. Imported Health workouts and doses are read-only.

For medicine and dose records, record only user-supplied facts; do not generate a treatment decision. Resolve the actual medicine and supported dose unit. For weight changes use `log_weight` or `update_weight_log` with the required revision; do not use this plugin to edit other profile preferences. Read current reminder lists before editing and preserve entries the user did not ask to replace. An offline phone may not apply reminder or Health changes immediately.

Use a new UUID `operation_id` for each logical write. Reuse that ID only for an unchanged retry. After a timeout, resolve the original operation or current state instead of issuing a second write. Read the current record and `expected_revision` before editing/deleting. A stale revision requires re-reading and reconciling the user's intended change; never overwrite blindly. Correct unambiguous validation errors at most twice, then ask for clarification. Stop on denied permissions or subscription failures.

Claim a save only after its tool returns a successful receipt. Distinguish saving in EvrFit from later HealthKit reconciliation. A replay receipt describes the original action; read current state before claiming the record still exists. For pending durable work use `get_operation` and report its actual status.

## Prepare and validate plans before replacement

1. Fetch `get_plan_context` and its current `planning_contract`, targets, restrictions, editable dates, existing plans and permitted relevant history. Disclose any context missing because of permissions; ask when that prevents a supported proposal.
2. Resolve catalog foods/exercises with EvrFit's tools. Follow the returned contract and native prescriptions; do not invent IDs or change completed/past days.
3. Use `submit_plan_proposal` to validate a diet/workout proposal. It creates a preview, does not replace the active plan, and uses no EvrFit AI credits. Correct unambiguous validation failures at most twice. Show the returned `validated_plan` and unresolved errors, not an invented success.
4. Call `apply_plan_proposal` only when the user requests replacement and the proposal is valid with current permissions. A preview request does not authorize application. If the context or grant becomes stale, fetch new context and validate again.
5. Use `regenerate_plan` only when the user requests EvrFit's own AI regeneration/replacement, understands the returned allowance/credit rules, and has the required generation, application and context permissions plus existing health-AI consent. It is different from an assistant-authored preview. Poll its operation; do not start another paid generation after an ambiguous timeout.

Do not claim a plan has been replaced until the application or generation operation succeeds. Never claim that a validated wellness plan is clinically approved.

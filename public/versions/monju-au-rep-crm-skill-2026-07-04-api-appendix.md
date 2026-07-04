---
title: Monju AU Rep CRM Agent Skill
version: 2026-07-04-api-appendix
status: active
portal_url: https://monjupartners.odoo.com/monju-au-rep
source_sha256: d7f0a8b4b5402091e712d17049d1f6503deadadde6f7968a400dd064a78c151c
owner: Monju
---

# Monju AU Rep CRM Agent Skill

Use this skill when helping a Monju AU sales rep keep assigned CRM accounts,
contacts, opportunities, demand signals, and activity history current.

This is a public, agent-agnostic Markdown skill. It may be given to Codex,
Claude, ChatGPT, a browser-control agent, or another rep-approved assistant.
Do not assume the agent has Monju admin access, local repo access, command-line
tools, or a built-in skill loader.

## Mission

Help the rep update the Monju AU CRM with minimum friction while respecting
Monju's assigned-account access boundary.

The rep owns customer-facing activity updates. Monju owns official account
assignment, commercial terms, pricing decisions, quote authority, stock
commitments, commission review, and approval.

## Hard Boundaries

You must not:

- use Ivan's admin account or any Monju operator credential;
- try to bypass portal/API restrictions;
- access unassigned accounts, customers, contacts, opportunities, or pipeline;
- export broad customer lists;
- change pricing, margin, credit, commission eligibility, assignment basis, or
  commercial terms;
- issue quotes or promises of stock availability;
- delete evidence or overwrite audit-critical history, except for the approved
  portal/helper delete actions that remove SKU demand lines or opportunities
  from the rep workspace, or mark mistaken activity evidence as voided, while
  preserving Monju audit control;
- treat hidden UI fields as secure if backend access says otherwise.

If a request needs Monju approval, submit an activity or blocker instead of
making the change.

## When To Use This Skill

Use this skill when the rep asks an agent to:

- onboard into the Monju AU rep portal;
- review assigned accounts;
- prepare for a customer call or meeting;
- capture a call, email, chat, meeting, voice note, or site visit;
- add or update an assigned-account contact;
- update opportunity demand fields;
- remove an incorrect opportunity or SKU demand line from the rep workspace;
- review safe AU stock-position context;
- summarize assigned-account pipeline;
- prepare a Monju handoff or approval request.

Do not use this skill for non-Monju systems, unassigned accounts, quote
creation, official pricing, credit terms, inventory reservations, accounting,
admin setup, or broad CRM exports.

## Required Starting Context

Before acting, confirm:

- the rep is using their own Monju portal login;
- the working URL is `https://monjupartners.odoo.com/monju-au-rep`;
- the account is visible through the assigned-account workspace;
- the requested change fits the allowed rep fields and workflows below.

If any of these are missing, ask one focused question or ask Monju to confirm
the assignment. Do not search wider Odoo to compensate for missing access.

## Agent Compatibility Contract

Use the safest tool mode available:

1. Browser/portal mode: preferred for most agents and first-time reps.
2. Rep-scoped API/helper mode: use only if Monju has provided an approved
   rep-scoped helper or credential for that rep.
3. Read-only summary mode: use when the agent cannot safely write.

All agents must:

- operate only inside the rep's own assigned-account boundary;
- ask for confirmation before saving if the update is inferred from messy notes;
- preserve uncertainty as a blocker or next step;
- report exact visible errors instead of trying hidden routes;
- never ask for Ivan's, Monju operator, admin, or internal-user credentials.

Codex-specific command examples in this file are optional implementation
shortcuts. Claude, ChatGPT, and browser agents should follow the same workflow
through the portal if they cannot run the helper commands.

## API Helper And Endpoint Appendix

Use the API path only when Monju has explicitly provided a rep-scoped helper,
rep-scoped credentials, or an already-authenticated rep portal session. The API
path is not a bypass: it must act as the rep and stay inside the same assigned
account boundary as the portal.

If the agent does not have the local helper, it may implement the same contract
against Odoo's browser JSON-RPC endpoints:

```text
POST /web/session/authenticate
POST /web/dataset/call_kw/<model>/<method>
```

Authentication request shape:

```json
{
  "jsonrpc": "2.0",
  "params": {
    "db": "<odoo_db>",
    "login": "<rep_login>",
    "password": "<rep_password>"
  }
}
```

Model-call request shape:

```json
{
  "jsonrpc": "2.0",
  "method": "call",
  "params": {
    "model": "<odoo_model>",
    "method": "<odoo_method>",
    "args": [],
    "kwargs": {}
  }
}
```

Required local-helper environment variables, when using the helper:

```text
ODOO_BASE_URL=https://monjupartners.odoo.com
ODOO_DB=<odoo_db>
REP_LOGIN=<rep_login>
REP_PASSWORD=<rep_password>
```

Never use Ivan's login, a Monju operator login, an admin API key, or a true
internal-user credential for rep-agent work.

### Approved API Operations

Use these helper commands or implement equivalent JSON-RPC calls with the same
rep-scoped checks:

| Operation | Helper command | Odoo model/method | Purpose |
| --- | --- | --- | --- |
| Read assigned accounts | `list-assigned-accounts` | `x_monju_au_account_assignment.search_read` | Find the rep's active assigned accounts. |
| Read one account bundle | `get-account-bundle --assignment-id <id>` | multiple `search_read` calls | Read one visible assignment with contacts, opportunities, evidence, order summaries, stock summaries, and demand allocations. |
| Read products | `list-product-summaries` | `x_monju_au_product_summary.search_read` | Populate allowed SKU/product choices. |
| Read AU stock | `list-au-stock-positions` | `x_monju_au_stock_position.search_read` | Read rep-visible stock-position summaries only. |
| Read reviewed allocations | `list-demand-allocations` | `x_monju_au_demand_allocation.search_read` | Read Monju-reviewed forecast/reservation context. |
| Add activity evidence | `create-activity-evidence --json @activity.json` | `x_monju_au_rep_evidence.create` | Log calls, emails, chats, meetings, blockers, buyer feedback, and next steps. |
| Add or edit contact | `upsert-assigned-contact --json @contact.json` | `res.partner.create` or `res.partner.write` | Maintain contacts under the assigned company only. |
| Edit opportunity basics | `update-assigned-opportunity --json @opportunity.json` | `crm.lead.write` | Update allowed opportunity fields only. |
| Add demand signal | `update-opportunity-demand --json @demand.json` | `x_monju_au_rep_evidence.create` | Save structured opportunity-demand evidence. |
| Remove SKU line from workspace | `delete-opportunity-sku-line --json @sku-line.json` | `x_monju_au_rep_evidence.write` | Mark the SKU demand line hidden/deleted for the rep workspace. |
| Remove opportunity from workspace | `delete-assigned-opportunity --json @opportunity-delete.json` | `x_monju_au_rep_evidence.create` | Add an opportunity-delete marker for the rep workspace. |
| Mark evidence mistaken | `void-activity-evidence --json @void-evidence.json` | `x_monju_au_rep_evidence.write` | Void mistaken evidence without hard-deleting the audit trail. |

Example helper calls:

```bash
node tools/rep-agent-api.mjs list-assigned-accounts
node tools/rep-agent-api.mjs get-account-bundle --assignment-id 1
node tools/rep-agent-api.mjs create-activity-evidence --json @activity.json
node tools/rep-agent-api.mjs update-opportunity-demand --json @demand.json
node tools/rep-agent-api.mjs void-activity-evidence --json @void-evidence.json
```

Example raw read call:

```json
{
  "jsonrpc": "2.0",
  "method": "call",
  "params": {
    "model": "x_monju_au_account_assignment",
    "method": "search_read",
    "args": [[["x_assignment_state", "=", "active"]]],
    "kwargs": {
      "fields": ["id", "x_name", "x_company_id", "x_assignment_state"]
    }
  }
}
```

The logged-in rep's backend permissions must do the real filtering. Do not add
customer ids, account ids, opportunity ids, or broader search domains to work
around missing access.

### API Failure Rules

If an API call fails with access denied, missing record, or empty results:

- do not retry with broader domains;
- do not ask for Monju admin credentials;
- do not search Contacts, Sales, Accounting, Inventory, or Settings directly;
- switch to browser/portal mode or ask Monju to confirm the assignment.

## Output Contract

For every write or proposed write, produce a short readback:

```text
I am updating:
Account:
Contact:
Opportunity:
Activity:
Demand fields:
Next step:
Blocker or Monju decision:
```

If the agent cannot complete the update, produce:

```text
I could not complete the CRM update.
Where I stopped:
Visible error or missing access:
Account/opportunity involved:
What Monju should check:
```

## Preferred Workflows

### First-Time Rep Onboarding

Use this when a rep is logging in for the first time or wants a guided setup
without Monju walking through every click.

The goal is to help the rep confirm access, understand the workspace, and make
one useful real update.

Do not skip the safety checks. If the rep sees anything outside their assigned
accounts, stop and ask Monju.

Onboarding flow:

1. Open `https://monjupartners.odoo.com/monju-au-rep`.
2. Confirm the page header shows the rep's own name.
3. Confirm the assigned accounts table shows only accounts the rep expects.
4. Open the first assigned account.
5. Review the account summary, opportunities, contacts, activity history, order
   summary, demand lines, stock-position summary, and allocation summary.
6. Add or verify one real contact.
7. Add one real activity update from the latest customer interaction.
8. Open or edit one opportunity demand line if customer intent is known.
9. Review AU stock positions only as planning context, not as a promise of
   availability.
10. Produce a short onboarding completion note for Monju.

The rep has passed onboarding only when:

- the rep can open the portal;
- the rep sees only assigned accounts;
- the rep can open one assigned account;
- the rep can create or update a contact;
- the rep can add activity evidence;
- the rep can update allowed demand fields;
- the rep understands when to stop and ask Monju.

Onboarding completion note:

```text
Rep onboarding check complete.
Rep:
Date:
Assigned account opened:
Contact checked or added:
Activity added:
Opportunity demand updated:
Anything unexpected:
Monju help needed:
```

Stop onboarding and ask Monju if:

- the rep sees unassigned customers or opportunities;
- the rep sees fake/test assignments;
- the rep sees Sales, Inventory, Accounting, Settings, Website Editor, or broad
  Contacts menus;
- pricing, margin, credit, internal strategy, or private commercial terms are
  visible;
- the rep can hard-delete evidence or edit protected commercial fields.

### Daily Rep Update

Use this at the end of a selling session or after customer movement.

Ask the rep:

1. Which assigned account moved?
2. Which contact was involved?
3. Which opportunity or SKU was discussed?
4. What did the buyer say?
5. How many units are realistic?
6. What date matters?
7. What confidence level is fair?
8. What is the next action?
9. What is blocked?
10. Does Monju need to approve pricing, stock, credit, technical claims, or
    commercial terms?

Then update the portal with the smallest accurate set of changes:

- contact update if people or roles changed;
- activity evidence for the customer interaction;
- opportunity demand update if product, units, use case, required-by date,
  confidence, or blocker changed.

### Weekly Pipeline Review

Use this when the rep wants a quick working review of assigned accounts.

For each assigned account, summarize:

- latest activity;
- open opportunities;
- demand by product and estimated units;
- required-by dates;
- confidence;
- blockers;
- visible attributed order value;
- visible Monju-reviewed allocation or reservation context;
- next rep action;
- Monju decisions needed.

Do not create forecasts outside the portal-visible records. If a forecast
requires protected stock, pricing, quote, or allocation decisions, submit the
request as activity evidence for Monju review.

### Before Customer Meeting Prep

Use this before a customer call, meeting, site visit, demo, or channel review.

Open the assigned account and prepare a short briefing for the rep:

- primary contact and other relevant contacts;
- open opportunities;
- latest activity and next step;
- products or SKUs already discussed;
- estimated units, required-by dates, confidence, and blockers;
- visible attributed order value;
- visible AU stock-position context;
- visible Monju-reviewed allocation or reservation context;
- open Monju decisions the rep should not answer alone.

End with three to five suggested questions for the customer. Keep questions
practical, for example:

- "Which stores or sites would use this first?"
- "What quantity would make the first order useful?"
- "What date would the product need to be available?"
- "Who else needs to approve the order?"
- "Is the blocker price, stock timing, product fit, or internal approval?"

Do not suggest a price, discount, delivery promise, stock hold, credit term, or
technical/safety claim unless Monju has already approved it in visible records.

### After Customer Meeting Capture

Use this immediately after a call, email, chat, meeting, site visit, demo, or
sample discussion.

Convert the rep's rough notes into the smallest accurate CRM update:

1. Identify the assigned account.
2. Identify the contact or create/update the contact.
3. Link the activity to the relevant opportunity when possible.
4. Capture what the buyer actually said.
5. Update opportunity demand only if product, units, use case, required-by
   date, confidence, or blocker changed.
6. Record the next rep action as activity evidence.
7. Record any Monju approval needed as a blocker or handoff note.

If the rep gives a voice note or rough message, ask only for missing facts that
are needed to make the update accurate. Do not turn uncertainty into a firm
forecast.

### Voice Note To CRM Update

Use this when the rep dictates a mobile voice note or sends an informal message.

First extract:

- account;
- contact;
- opportunity;
- product/SKU;
- estimated units;
- use case;
- required-by date;
- confidence;
- buyer feedback;
- next step;
- blocker;
- Monju decision needed.

Then read back a concise proposed update before saving:

```text
I will log this as:
Account:
Contact:
Opportunity:
Activity summary:
Demand update:
Next step:
Blocker:
Monju decision needed:
```

If the rep confirms, save the activity evidence and any allowed demand/contact
updates. If the rep does not confirm, ask one or two focused questions and try
again.

### Read Assigned Accounts

Open the rep portal and review the assigned accounts table. Prioritize accounts
with recent blockers, upcoming next action dates, high confidence demand, or
customer-committed demand.

If Monju has provided the approved rep-scoped API helper and the agent can run
commands, call:

```bash
node tools/rep-agent-api.mjs list-assigned-accounts
```

### Read One Account

Open the account detail page. If Monju has provided the approved rep-scoped API
helper and the agent can run commands, call:

```bash
node tools/rep-agent-api.mjs get-account-bundle --assignment-id <assignment_id>
```

Check:

- company/account summary;
- contacts and buying roles;
- opportunities;
- demand fields;
- activity history;
- attributed order and commission summary;
- AU stock-position summary;
- Monju-reviewed demand allocations visible for assigned accounts.

### Read Stock, Forecast, And Allocation Context

Use the portal AU stock positions table. If Monju has provided the approved
rep-scoped API helper and the agent can run commands, call:

```bash
node tools/rep-agent-api.mjs list-au-stock-positions
node tools/rep-agent-api.mjs list-demand-allocations
```

You may read Monju-reviewed allocation rows that are visible to the assigned
rep. You must not create, edit, delete, or promise reservations. If a customer
needs stock held, submit activity evidence with the customer, opportunity, SKU,
units, required-by date, confidence, and blocker.

If no stock is visible yet, treat that as expected until Monju submits and
confirms the relevant container purchase orders. Do not infer stock availability
from an empty table.

### Update Opportunity Demand

Use the account page demand editor. If Monju has provided the approved
rep-scoped API helper and the agent can run commands, use the helper.

Allowed demand fields:

- `x_product_id`
- `x_estimated_units`
- `x_use_case`
- `x_required_by_date`
- `x_confidence`
- `x_blocker`

Example input:

```json
{
  "assignmentId": 1,
  "opportunityId": 1,
  "fields": {
    "x_product_id": 1,
    "x_estimated_units": 24,
    "x_use_case": "retailer_shelf",
    "x_required_by_date": "2026-08-15",
    "x_confidence": "high",
    "x_blocker": "Waiting for buyer confirmation"
  }
}
```

Optional helper command shape:

```bash
node tools/rep-agent-api.mjs update-opportunity-demand --json @demand.json
```

The update is saved as structured evidence. Do not write directly to protected
CRM opportunity fields.

Use ordinary activity evidence `nextStep` for follow-up actions. Do not add a
next-action date to demand fields.

### Delete Opportunity Or SKU Line From Workspace

Use the visible portal delete controls only when the rep confirms an
opportunity or SKU demand line was added by mistake or is no longer relevant.

This is a workspace delete, not a hard delete of Monju's CRM/audit history:

- deleting a SKU line hides that SKU demand line from the rep workspace;
- deleting an opportunity hides that opportunity from the rep workspace;
- marking mistaken activity evidence hides it from the rep's normal activity
  view while preserving the original text for Monju;
- activity evidence and Monju review history remain under Monju control;
- allocation or reservation rows must not be deleted or changed by the rep.

Optional helper command shapes:

```bash
node tools/rep-agent-api.mjs delete-opportunity-sku-line --json @sku-line.json
node tools/rep-agent-api.mjs delete-assigned-opportunity --json @opportunity-delete.json
node tools/rep-agent-api.mjs void-activity-evidence --json @void-evidence.json
```

Example SKU-line delete input:

```json
{
  "assignmentId": 1,
  "opportunityId": 1,
  "lineId": 25
}
```

Example opportunity delete input:

```json
{
  "assignmentId": 1,
  "opportunityId": 1
}
```

Example mistaken activity input:

```json
{
  "assignmentId": 1,
  "evidenceId": 45,
  "reason": "Duplicate note entered by mistake"
}
```

Do not call generic `unlink`, direct `crm.lead` writes, or hidden Odoo routes to
delete records. Do not erase original activity text to hide a mistake.

### Add Activity Evidence

Use this for calls, emails, chats, meetings, site visits, demos, sample
requests, training, buyer feedback, product-selection questions, SDS/safety
questions, support issues, referrals, channel updates, and blockers.

Example input:

```json
{
  "assignmentId": 1,
  "opportunityId": 1,
  "contactId": 10,
  "activityType": "email",
  "activityDate": "2026-07-02",
  "summary": "Buyer asked for carton quantity and estimated arrival timing.",
  "buyerFeedback": "Interested if first container stock can support two stores.",
  "nextStep": "Confirm expected initial order quantity.",
  "blocker": ""
}
```

Optional helper command shape:

```bash
node tools/rep-agent-api.mjs create-activity-evidence --json @activity.json
```

### Add Or Edit Contact

Use the contact table on the account page. If Monju has provided the approved
rep-scoped API helper and the agent can run commands, use the helper.

Allowed contact fields:

- `name`
- `function`
- `email`
- `phone`
- `x_monju_au_buying_role`
- `x_monju_au_primary_contact`

Optional helper command shape:

```bash
node tools/rep-agent-api.mjs upsert-assigned-contact --json @contact.json
```

## Quality Bar

Good updates are short and specific:

- who moved the account forward;
- what the buyer said;
- which product and units are being discussed;
- what date matters;
- how confident the rep is;
- what the next action is;
- what is blocked.

Bad updates are vague:

- "following up";
- "interested";
- "looks good";
- "will revert";
- "pricing needed" without context.

### Good Activity Examples

Good call update:

```text
Spoke with Sarah Lee, procurement. She wants Smart Strip options for three
trade counters and asked whether 48 units could be available before 2026-08-15.
Confidence is medium because the state manager still needs to approve. Next
step: Ben to confirm likely first-store rollout quantity. Blocker: Monju to
confirm safe stock timing before any commitment.
```

Good chat update:

```text
Buyer replied by WhatsApp that the first order is likely for maintenance teams,
not retail shelf trial. Estimated 24 units, required by late August, confidence
medium. Next step: ask which sites will receive the first cartons.
```

Good blocker update:

```text
Customer is interested but will not proceed until SDS and product-use guidance
are confirmed for painted metal surfaces. No pricing requested yet. Monju
decision needed: approve the technical response before the rep replies.
```

### Weak Activity Examples To Improve

Rewrite vague notes before saving:

```text
Interested, will follow up.
```

Better:

```text
Customer asked for first-order timing and likely carton quantity. Rep will
confirm whether the initial need is 12, 24, or 48 units and which site needs
them first.
```

```text
Pricing needed.
```

Better:

```text
Buyer asked for commercial pricing for a possible 48-unit first order. Rep
should not quote directly. Monju decision needed: approved price/discount and
whether stock can be reserved.
```

## Working Style

Prefer specific questions over broad prompts. If the rep says "update PPG",
ask which contact, opportunity, product, units, date, confidence, blocker, and
next action changed.

Use the portal when the rep needs to see context, compare tables, or confirm a
visual result. Use the approved API helper when the rep gives a precise update
and wants minimum friction.

Never invent missing customer facts. If the rep is unsure, record the unknown
as a blocker or next step.

## Suggested Rep Prompts

The rep can ask:

- "Walk me through onboarding."
- "Show my assigned accounts."
- "Open this account and summarize what changed."
- "Log this call."
- "Add this buyer as a contact."
- "Update demand for this SKU."
- "Delete this incorrect SKU line."
- "Remove this mistaken opportunity from my workspace."
- "Mark this activity as mistaken."
- "What stock context can I safely discuss?"
- "What should I ask Monju to approve?"
- "Give me my weekly pipeline review."

## Stuck Or Error Handling

If login fails, ask the rep to confirm they are using their own Monju portal
login and the URL `https://monjupartners.odoo.com/monju-au-rep`.

If an account is missing, do not search broader Odoo. Ask Monju to confirm the
assignment.

If a save fails, capture:

- page or API action attempted;
- account;
- opportunity;
- field or update;
- visible error text;
- whether the rep could still view assigned records.

Then stop and ask Monju.

### Browser Agent Troubleshooting

Use this when the rep's browser-control agent is operating the portal.

Stay inside the rep portal URL:

- `https://monjupartners.odoo.com/monju-au-rep`

If the browser agent gets lost:

1. Return to the rep portal URL.
2. Confirm the header shows the rep's own name.
3. Confirm the account table is assigned-only.
4. Open the assigned account from the portal, not from global Odoo search.
5. Use visible account, contact, opportunity, and activity controls only.

Do not use browser shortcuts, hidden routes, developer tools, admin menus, or
global search to bypass the portal boundary.

If the page does not save:

- capture the visible error text;
- do not retry by changing protected fields;
- do not switch to Ivan's login;
- ask Monju to verify the assignment and portal permission.

If the browser agent sees unexpected backend menus or non-assigned records,
stop immediately and report the page URL, visible menu/record names, and the
rep account being used.

## Monju Handoff Note

When handing a decision back to Monju, use this format:

```text
Account:
Opportunity:
Contact:
Customer request:
Product/SKU:
Estimated units:
Required-by date:
Confidence:
Blocker:
Rep next action:
Monju decision needed:
Evidence added:
```

## Stop And Ask Monju

Stop and ask Monju when the rep wants to:

- quote a price or discount;
- promise stock;
- change delivery or payment terms;
- handle credit concerns;
- submit a technical or safety claim;
- make a commission or assignment decision;
- access a customer outside the assigned list.

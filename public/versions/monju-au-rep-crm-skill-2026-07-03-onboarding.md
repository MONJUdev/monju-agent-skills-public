---
title: Monju AU Rep CRM Agent Skill
version: 2026-07-03-onboarding
status: active
portal_url: https://monjupartners.odoo.com/monju-au-rep
source_sha256: c10036a194ab238b144f5ba07193a1f9eae297c536bd3fd1b12e3d16e6a8a9a9
owner: Monju
---

# Monju AU Rep CRM Agent Skill

Use this skill when helping a Monju AU sales rep keep assigned CRM accounts,
contacts, opportunities, demand signals, and activity history current.

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
- delete evidence or overwrite audit-critical history;
- treat hidden UI fields as secure if backend access says otherwise.

If a request needs Monju approval, submit an activity or blocker instead of
making the change.

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
- the rep can delete evidence or edit protected commercial fields.

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

### Read Assigned Accounts

Open the rep portal and review the assigned accounts table. Prioritize accounts
with recent blockers, upcoming next action dates, high confidence demand, or
customer-committed demand.

If using the approved API helper, call:

```bash
node tools/rep-agent-api.mjs list-assigned-accounts
```

### Read One Account

Open the account detail page or call:

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

Use the portal AU stock positions table or the approved API helper:

```bash
node tools/rep-agent-api.mjs list-au-stock-positions
node tools/rep-agent-api.mjs list-demand-allocations
```

You may read Monju-reviewed allocation rows that are visible to the assigned
rep. You must not create, edit, delete, or promise reservations. If a customer
needs stock held, submit activity evidence with the customer, opportunity, SKU,
units, required-by date, confidence, and blocker.

### Update Opportunity Demand

Use the account page demand editor or the approved API helper.

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

Command shape:

```bash
node tools/rep-agent-api.mjs update-opportunity-demand --json @demand.json
```

The update is saved as structured evidence. Do not write directly to protected
CRM opportunity fields.

Use ordinary activity evidence `nextStep` for follow-up actions. Do not add a
next-action date to demand fields.

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

Command shape:

```bash
node tools/rep-agent-api.mjs create-activity-evidence --json @activity.json
```

### Add Or Edit Contact

Use the contact table on the account page or the approved API helper.

Allowed contact fields:

- `name`
- `function`
- `email`
- `phone`
- `x_monju_au_buying_role`
- `x_monju_au_primary_contact`

Command shape:

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

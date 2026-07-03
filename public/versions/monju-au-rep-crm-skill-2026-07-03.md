---
title: Monju AU Rep CRM Agent Skill
version: 2026-07-03
status: active
portal_url: https://monjupartners.odoo.com/monju-au-rep
source_sha256: 1b37728351e112e3b1b45f58183a7534c3cc888302f09fa9d05b02e9f2015629
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

## Stop And Ask Monju

Stop and ask Monju when the rep wants to:

- quote a price or discount;
- promise stock;
- change delivery or payment terms;
- handle credit concerns;
- submit a technical or safety claim;
- make a commission or assignment decision;
- access a customer outside the assigned list.

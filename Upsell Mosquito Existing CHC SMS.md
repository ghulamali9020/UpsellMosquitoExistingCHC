# Workflow: Upsell Mosquito Existing CHC SMS

---

## Overview

This workflow automates mosquito service upsell outreach via SMS to existing CHC (Complete Home Care) customers. It is triggered by an inbound webhook from Zapier via LeadConnector, validates the contact and phone number, calculates pricing, creates an opportunity, and sends a drip SMS sequence — converting replies of "OASIS" into won deals.

---

## Workflow Diagram

```
┌─────────────────────────────────────┐
│  TRIGGER: Inbound Webhook           │
│  (Zapier via LeadConnector)         │
└────────────────┬────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────┐
│  Find Contact                       │
└────────┬────────────────────────────┘
         │
    ┌────┴────────────────┐
    │                     │
    ▼                     ▼
NOT FOUND              FOUND
    │                     │
  [END]                   ▼
            ┌─────────────────────────────┐
            │  Condition: Number Valid?   │
            │  (no opted-out/bad tags)    │
            └────────┬────────────────────┘
                     │
            ┌────────┴────────┐
            │                 │
            ▼                 ▼
         INVALID            VALID
            │                 │
          [END]               ▼
                 ┌─────────────────────────┐
                 │  Update Contact Fields  │
                 │  (from webhook data)    │
                 └────────────┬────────────┘
                              │
                              ▼
                 ┌─────────────────────────┐
                 │  Drip Mode              │
                 │  Batch: 5 / Every 5min  │
                 └────────────┬────────────┘
                              │
                              ▼
                 ┌─────────────────────────┐
                 │  Wait 5 min             │
                 │  Mon–Fri, 9AM–5PM       │
                 └────────────┬────────────┘
                              │
                              ▼
                 ┌─────────────────────────┐
                 │  Math Operation         │
                 │  (Mo × 11) + Initial    │
                 └────────────┬────────────┘
                              │
                              ▼
                 ┌─────────────────────────┐
                 │  Create Opportunity     │
                 │  Pipeline: Upsell Mosq  │
                 │  Stage:  Existing CHC   │
                 │  Status: Open           │
                 └────────────┬────────────┘
                              │
                              ▼
                 ┌─────────────────────────┐
                 │  Add to Workflow:       │
                 │  Upsell Mosq CHC Email  │
                 └────────────┬────────────┘
                              │
                              ▼
                 ┌─────────────────────────┐
                 │  SMS: Upsell Mosquito 1 │
                 │  Template: WT01         │
                 └────────────┬────────────┘
                              │
                              ▼
                 ┌─────────────────────────┐
                 │  Wait for Reply         │
                 │  Timeout: 30 Days       │
                 └────┬──────────────┬─────┘
                      │              │
               REPLIED           TIMEOUT
                      │              │
                      ▼              ▼
           ┌──────────────┐  ┌────────────────────────────────┐
           │ Reply=OASIS? │  │  Drip Mode (Batch:5 / 5min)    │
           └──┬────────┬──┘  │  Wait 5min (Mon–Fri, 9AM–5PM)  │
              │        │     │  SMS: Upsell Mosquito 2.1      │
            TRUE     FALSE   │  Template: WT02                │
              │        │     └──────────────┬─────────────────┘
              │        │                    │
              │        │                    ▼
              │        │     ┌──────────────────────────────┐
              │        │     │  Wait for Reply              │
              │        │     │  Timeout: 30 Days            │
              │        │     └────┬────────────────┬─────────┘
              │        │          │                │
              │        │      REPLIED           TIMEOUT
              │        │          │                │
              │        │          ▼                ▼
              │        │   ┌────────────┐  ┌──────────────────┐
              │        │   │Reply=OASIS?│  │Update Opp:       │
              │        │   └──┬──────┬──┘  │  Abandoned       │
              │        │   TRUE    FALSE   │Remove from       │
              │        │      │       │    │  Workflow        │
              │        │      │     [END]  └──────────────────┘
              │        │      │
              │        ▼      │
              │  ┌─────────────────────────────────┐
              │  │  Drip Mode (Batch:5 / 5min)     │
              │  │  Wait 5min (Mon–Fri, 9AM–5PM)   │
              │  │  SMS: Upsell Mosquito 2.0        │
              │  │  Template: WT02                  │
              │  └──────────────┬──────────────────┘
              │                 │
              │                 ▼
              │  ┌──────────────────────────────┐
              │  │  Wait for Reply              │
              │  │  Timeout: 30 Days            │
              │  └────┬────────────────┬─────────┘
              │       │                │
              │   REPLIED           TIMEOUT
              │       │                │
              │       ▼                ▼
              │  ┌──────────┐  ┌──────────────────┐
              │  │Reply=OASIS│  │Update Opp:       │
              │  └──┬──────┬┘  │  Abandoned       │
              │  TRUE    FALSE  │Remove from       │
              │     │      │   │  Workflow        │
              │     │    [END]  └──────────────────┘
              │     │
              └─────┘
                    │
    ┌───────────────▼────────────────────────────────────┐
    │             OASIS CONVERSION SEQUENCE              │
    │   (Triggered from SMS 1, SMS 2.0, or SMS 2.1)      │
    ├────────────────────────────────────────────────────┤
    │  1. Remove from Workflow (CHC Email)               │
    │  2. Update Contact Fields (from webhook)           │
    │  3. Wait 1 minute                                  │
    │  4. Math: (Mo × 11) + Initial price                │
    │  5. Update Opportunity → Self Sold / Won           │
    │  6. Webhook POST → Zapier                          │
    │  7. Email → Welcome to Outdoor Oasis Club 🏝️      │
    │  8. SMS  → WT00 Pest MOS Sold WIP                  │
    └────────────────────────────────────────────────────┘
                          │
                        [END]
```

---

## Trigger

- **Type:** Inbound Webhook
- **Triggered by:** Zapier via LeadConnector

---

## Step 1 — Find Contact

Search for the contact based on data received from the inbound webhook.

### Branch A: Contact Not Found
- Workflow **ends**.

### Branch B: Contact Found
- Proceed to phone number validation.

---

## Step 2 — Number Valid (Condition)

Check if the contact's phone number is valid and eligible for automated SMS.

### If Invalid (No conditions met)
- Workflow **ends**.

### If Valid
The contact must **not** have any of the following tags:

| Exclusion Tag |
|---|
| `02. Status - Opted Out of Marketing Texts` |
| `No Automated Texts` |
| `Num Invalid` |
| `SMS Incapable` |

If none of those tags are present, proceed to the next step.

---

## Step 3 — Update Contact Field

Map the following fields from the inbound webhook to the contact record:

| Field | Source |
|---|---|
| House Sq Ft | Inbound Webhook |
| Yard Sq Ft | Inbound Webhook |
| Street Address | Inbound Webhook |
| City | Inbound Webhook |
| State | Inbound Webhook |
| Postal Code | Inbound Webhook |

---

## Step 4 — Drip (Drip Mode)

Throttle outgoing actions to avoid overwhelming sending infrastructure.

- **Batch Size:** 5
- **Drip Interval:** 5 minutes

---

## Step 5 — Wait

Enforce business-hours sending window.

- **Wait For:** Time Delay
- **Duration:** 5 minutes
- **Resume On:** Monday, Tuesday, Wednesday, Thursday, Friday
- **Resume Between Hours:** 9:00 AM – 5:00 PM

---

## Step 6 — Math Operation

Calculate the total contract opportunity value before creating the deal.

> **(Add Mosquito Mo × 11) + Add Mosquito I**

| Variable | Description |
|---|---|
| `Add Mosquito Mo` | Monthly recurring price |
| `Add Mosquito I` | Initial / setup price |
| Result | 11 months of service + one-time initial fee |

- Output stored in: `{{contact.system_opportunity_value}}`

---

## Step 7 — Create Opportunity

| Field | Value |
|---|---|
| Pipeline | Upsell Mosquito |
| Pipeline Stage | Existing CHC |
| Opportunity Value | `{{contact.system_opportunity_value}}` |
| Status | Open |

---

## Step 8 — Add to Workflow

Enroll the contact in the parallel email nurture sequence.

| Setting | Value |
|---|---|
| Workflow | Upsell Mosquito Existing CHC Email |
| Pass Input Trigger Parameters | true |

---

## Step 9 — SMS: Upsell Mosquito 1

- **Action Name:** Upsell Mosquito 1
- **Template:** WT01 Upsell Mosquito (HTML Snippet)
- **Message:**

> Mosquito season is coming. Get ahead of it now by signing up for season-long protection before mosquitoes return. Enjoy your yard this season at {{Address}}. Get started now for just ${{contact.add_mosquito_i}}, then ${{contact.add_mosquito_mo}} per month — no contracts, cancel anytime. Check your email for more details. Text OASIS to sign up now or give us a call.

---

## Step 10 — Wait for Contact Reply (30-Day Window #1)

- **Action Name:** Wait
- **Wait For:** Contact Reply
- **Reply To:** Upsell Mosquito 1
- **Timeout:** 30 Days

---

## Branch A — Contact Replies to SMS 1

### Condition: Reply is "OASIS"

#### ✅ If TRUE → [OASIS Conversion Sequence](#oasis-conversion-sequence)

---

#### ❌ If FALSE — Reply is not "OASIS" (None of the conditions met)

Proceed to a second SMS touch.

**1. Drip Mode**
- Batch Size: 5
- Drip Interval: 5 minutes

**2. Wait**
- Wait For: Time Delay
- Duration: 5 minutes
- Resume On: Monday, Tuesday, Wednesday, Thursday, Friday
- Resume Between Hours: 9:00 AM – 5:00 PM

**3. SMS: Upsell Mosquito 2.0**
- Action Name: Upsell Mosquito 2.0
- Template: WT02 Upsell Mosquito (HTML Snippet)
- Message:

> Mosquito season is here. Sign up now to protect your yard and enjoy the outdoors without mosquitoes. Enjoy your yard this season at {{Address}}. Get started now for just ${{contact.add_mosquito_i}}, then ${{contact.add_mosquito_mo}} per month — no contracts, cancel anytime. Check your email for more details. Text OASIS to sign up now or give us a call.

**4. Wait for Contact Reply (30-Day Window #2)**
- Wait For: Contact Reply
- Reply To: Upsell Mosquito 1
- Timeout: 30 Days

| Outcome | Action |
|---|---|
| **Timeout** | Update Opportunity → Status: Abandoned → Remove from workflow → END |
| **Reply = "OASIS"** | → [OASIS Conversion Sequence](#oasis-conversion-sequence) |
| **Reply ≠ "OASIS"** | Remove from current workflow → END |

---

## Branch B — Timeout on SMS 1 (No Reply After 30 Days)

Send a final SMS touchpoint before closing out the sequence.

**1. Drip Mode**
- Action Name: Drip Mode
- Batch Size: 5
- Drip Interval: 5 minutes

**2. Wait**
- Wait For: Time Delay
- Duration: 5 minutes
- Resume On: Monday, Tuesday, Wednesday, Thursday, Friday
- Resume Between Hours: 9:00 AM – 5:00 PM

**3. SMS: Upsell Mosquito 2.1**
- Action Name: Upsell Mosquito 2.1
- Template: WT02 Upsell Mosquito (HTML Snippet)
- Message:

> Mosquito season is here. Sign up now to protect your yard and enjoy the outdoors without mosquitoes. Enjoy your yard this season at {{Address}}. Get started now for just ${{contact.add_mosquito_i}}, then ${{contact.add_mosquito_mo}} per month — no contracts, cancel anytime. Check your email for more details. Text OASIS to sign up now or give us a call.

**4. Wait for Contact Reply (30-Day Window #3)**
- Wait For: Contact Reply
- Reply To: Upsell Mosquito 2.1
- Timeout: 30 Days

| Outcome | Action |
|---|---|
| **Timeout** | Update Opportunity → Status: Abandoned → Remove from workflow → END |
| **Reply = "OASIS"** | → [OASIS Conversion Sequence](#oasis-conversion-sequence) |
| **Reply ≠ "OASIS"** | Remove from current workflow → END |

---

## OASIS Conversion Sequence

> This sequence is identical regardless of which SMS message (1, 2.0, or 2.1) the contact replied to with "OASIS".

**1. Remove from Workflow**
- Action Name: Remove from Workflow
- Workflow: Upsell Mosquito Existing CHC Email

**2. Update Contact Field**
- Action Name: Update Contact Field
- Action Type: Update field data
- Fields mapped from inbound webhook:

| Field |
|---|
| Yard Sq Ft |
| House Sq Ft |
| Street Address |
| City |
| State |
| Postal Code |

**3. Wait**
- Action Name: Wait 1 Minute
- Wait For: Time Delay
- Duration: 1 minute

**4. Math Operation**
- **(Add Mosquito Mo × 11) + Add Mosquito I**
- Result stored in `{{contact.system_opportunity_value}}`

**5. Update Opportunity**

| Field | Value |
|---|---|
| Pipeline | Upsell Mosquito |
| Pipeline Stage | Self Sold |
| Status | Won |
| Opportunity Value | `{{contact.system_opportunity_value}}` |

**6. Webhook**

| Setting | Value |
|---|---|
| Method | POST |
| URL | `https://hooks.zapier.com/hooks/catch/1263219/ul6qppi/` |
| Custom Data | `FRACCT`, `Housesqft`, `Lawnsqft`, `FirstName`, `LastName` |

- Linked to Zapier OOC Subscription Workflow
* [Upsell Mosquito Existing CHC OASIS](./Upsell%20Mosquito%20Existing%20CHC%20OASIS.md)

**7. Email**
- Action Name: Welcome to the Outdoor Oasis Club! 🏝️
- Subject: Welcome to the Outdoor Oasis Club! 🏝️
- Linked Template: Welcome to the Outdoor Oasis Club - SelfService 2026

**8. SMS**
- Template: WT00 Pest MOS Sold WIP (HTML Snippet)
- Message:

> Hey {{contact.first_name}}, your search is over! Good for you...bad for mosquitos! The entire BugBros office is so excited that we're busting out our CRAZY HAPPY Dance just for you (feel free to join us). Check your email for more details on your service.

---

## SMS Message Reference

| Action Name | Template | Trigger Point |
|---|---|---|
| Upsell Mosquito 1 | WT01 Upsell Mosquito | Initial send after enrollment |
| Upsell Mosquito 2.0 | WT02 Upsell Mosquito | After non-OASIS reply to SMS 1 |
| Upsell Mosquito 2.1 | WT02 Upsell Mosquito | After 30-day timeout on SMS 1 |
| Sold Confirmation | WT00 Pest MOS Sold WIP | After any OASIS reply (conversion) |

---

## Wait & Timing Reference

| Step | Type | Duration | Business Hours |
|---|---|---|---|
| Step 5 | Time Delay | 5 minutes | Mon–Fri, 9AM–5PM |
| Step 10 — Window #1 | Contact Reply Timeout | 30 Days | — |
| OASIS Step 3 | Time Delay | 1 minute | — |
| Before SMS 2.0 | Time Delay | 5 minutes | Mon–Fri, 9AM–5PM |
| Window #2 (SMS 2.0) | Contact Reply Timeout | 30 Days | — |
| Before SMS 2.1 | Time Delay | 5 minutes | Mon–Fri, 9AM–5PM |
| Window #3 (SMS 2.1) | Contact Reply Timeout | 30 Days | — |

> **Maximum workflow duration:** ~90 days across 3 reply windows

---

## Field & Variable Reference

| Variable | Description |
|---|---|
| `{{contact.add_mosquito_i}}` | Initial / setup price |
| `{{contact.add_mosquito_mo}}` | Monthly recurring price |
| `{{contact.system_opportunity_value}}` | Calculated total: (Mo × 11) + Initial |
| `{{contact.first_name}}` | Contact first name |
| `{{Address}}` | Contact street address |
| `FRACCT` | Account reference sent to Zapier |
| `Housesqft` | House square footage sent to Zapier |
| `Lawnsqft` | Lawn/yard square footage sent to Zapier |

---

## Summary of Outcomes

| Scenario | Final Outcome |
|---|---|
| Contact not found | Workflow ends immediately |
| Phone invalid or has exclusion tag | Workflow ends immediately |
| Contact replies "OASIS" to any SMS | Opportunity → **Won** (Self Sold) · Zapier webhook fired · Welcome email + confirmation SMS sent |
| Contact replies something other than "OASIS" | Removed from workflow |
| No reply after all three 30-day windows (~90 days total) | Opportunity → **Abandoned** · Removed from workflow |
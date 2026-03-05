# Workflow: Upsell Mosquito Existing CHC Email

---

## Overview

This workflow runs a two-touch email nurture sequence targeting existing CHC (Complete Home Care) customers for a mosquito service upsell. It runs **in parallel** with the *Upsell Mosquito Existing CHC SMS* workflow — that SMS workflow enrolls contacts into this one via the "Add to Workflow" action. The goal is to get the contact to click the **"Sign Me Up"** button inside the email, which triggers the full self-service conversion sequence and closes the opportunity as Won.

---

## How This Workflow Is Triggered

This workflow is **not** triggered directly by a Zapier webhook from an external source. It is triggered by the **Upsell Mosquito Existing CHC SMS** workflow via its "Add to Workflow" step, which passes the original inbound webhook trigger parameters into this workflow. Those mapped field values (address, square footage, account info) are available here as inbound webhook data for use in field updates.

- **Trigger Type:** Inbound Webhook
- **Triggered by:** Upsell Mosquito Existing CHC SMS workflow (Add to Workflow action)
- **Purpose of trigger:** Maps and passes field values from the original Zapier/LeadConnector webhook payload into this workflow's context

---

## Workflow Diagram

```
┌────────────────────────────────────────────┐
│  TRIGGER: Inbound Webhook                  │
│  (Passed from Upsell Mosq CHC SMS workflow)│
└───────────────────┬────────────────────────┘
                    │
                    ▼
┌────────────────────────────────────────────┐
│  EMAIL: Mosquito Email 1                   │
│  Subject: Mosquitos Are Coming!            │
│  Template: Upsell Mosquito Existing CHC    │
│  Button: Sign Me Up                        │
└───────────────────┬────────────────────────┘
                    │
                    ▼
┌────────────────────────────────────────────┐
│  WAIT: Trigger Link Clicked                │
│  Link: Upsell Mosquito Sign Me Up          │
│  Timeout: 30 Days                          │
└──────────┬─────────────────────┬───────────┘
           │                     │
    LINK CLICKED             TIMEOUT
    (Sign Me Up)           (30 days)
           │                     │
           ▼                     ▼
  ┌─────────────────┐   ┌────────────────────────────────┐
  │  CONVERSION     │   │  Drip Mode                     │
  │  SEQUENCE       │   │  Batch: 5 / Interval: 5 min    │
  │  (see below)    │   └───────────────┬────────────────┘
  └─────────────────┘                   │
                                        ▼
                          ┌─────────────────────────────┐
                          │  Wait 5 min                 │
                          │  Mon–Fri, 9AM–5PM           │
                          └───────────────┬─────────────┘
                                          │
                                          ▼
                          ┌─────────────────────────────┐
                          │  EMAIL: Mosquito Email 2    │
                          │  Subject: The Mosquitos     │
                          │  are HERE!                  │
                          │  Template: Upsell Mosquito  │
                          │  Existing CHC 2026          │
                          │  Button: Sign Me Up         │
                          └───────────────┬─────────────┘
                                          │
                                          ▼
                          ┌─────────────────────────────┐
                          │  WAIT: Trigger Link Clicked │
                          │  Link: Upsell Mosquito      │
                          │        Sign Me Up           │
                          │  Timeout: 30 Days           │
                          └─────┬───────────────┬───────┘
                                │               │
                         LINK CLICKED       TIMEOUT
                         (Sign Me Up)      (30 days)
                                │               │
                                ▼               ▼
                       ┌──────────────┐  ┌───────────────────────┐
                       │  CONVERSION  │  │  Update Opportunity:  │
                       │  SEQUENCE    │  │  Status → Abandoned   │
                       │  (see below) │  └───────────┬───────────┘
                       └──────────────┘              │
                                                     ▼
                                         ┌───────────────────────┐
                                         │  Remove from Current  │
                                         │  Workflow             │
                                         └───────────────────────┘
                                                     │
                                                   [END]


  ╔══════════════════════════════════════════════════════════╗
  ║              CONVERSION SEQUENCE                        ║
  ║  (Identical whether triggered from Email 1 or Email 2)  ║
  ╠══════════════════════════════════════════════════════════╣
  ║  1. Remove from Workflow → Upsell Mosq CHC SMS          ║
  ║  2. Update Contact Fields (from webhook payload)        ║
  ║  3. Wait 1 minute                                       ║
  ║  4. Math: (Mo × 11) + Initial price                     ║
  ║  5. Update Opportunity → Self Sold / Won                ║
  ║  6. Webhook POST → Zapier                               ║
  ║  7. Email → Welcome to Outdoor Oasis Club 🏝️           ║
  ║  8. SMS  → WT00 Pest MOS Sold WIP                       ║
  ╚══════════════════════════════════════════════════════════╝
                              │
                            [END]
```

---

## Node Details

---

### TRIGGER — Inbound Webhook

**What it does:**
Receives the workflow enrollment signal sent by the *Upsell Mosquito Existing CHC SMS* workflow when it reaches its "Add to Workflow" action. The trigger also carries forward the original inbound webhook parameters (address, square footage, account identifiers) so those values are available as mapped data throughout this workflow.

**Key detail:** This workflow does not fire independently from Zapier. It is always a child enrollment from the SMS workflow. The `Pass Input Trigger Parameters: true` setting on the SMS workflow's "Add to Workflow" action is what makes the webhook field values accessible here.

---

### EMAIL — Mosquito Email 1

**What it does:**
Sends the first upsell email to the contact, introducing mosquito season protection and presenting a clear call-to-action button.

| Setting | Value |
|---|---|
| Action Name | Mosquito Email 1 |
| Subject | Mosquitos Are Coming! |
| Template | Upsell Mosquito Existing CHC 2026 |
| Template CTA Button | Sign Me Up |

**Key detail:** The "Sign Me Up" button inside this email is a **Trigger Link** tracked by GHL. Clicking it fires the next conditional branch immediately, skipping the 30-day timeout entirely.

---

### WAIT — Trigger Link Clicked (Window #1)

**What it does:**
Pauses the workflow and listens for one of two events — the contact clicking the tracked "Sign Me Up" link, or 30 days elapsing with no click.

| Setting | Value |
|---|---|
| Wait For | Trigger Link Clicked |
| Trigger Link | Upsell Mosquito Sign Me Up |
| Timeout | 30 Days |

**Branches:**
- **Link Clicked** → proceeds immediately to the Conversion Sequence
- **Timeout after 30 days** → proceeds to Email 2 drip path

---

### BRANCH: LINK CLICKED (after Email 1) → Conversion Sequence

When the contact clicks "Sign Me Up" in Email 1, they are routed directly into the Conversion Sequence. See the [Conversion Sequence](#conversion-sequence) section for full step-by-step details.

---

### BRANCH: TIMEOUT (30 days, no click on Email 1)

The contact did not engage with Email 1. A second email touch is prepared using a business-hours drip before sending.

---

### DRIP MODE

**What it does:**
Throttles the send of the second email to avoid volume spikes and respect infrastructure limits.

| Setting | Value |
|---|---|
| Action Name | Drip Mode |
| Batch Size | 5 |
| Drip Interval | 5 minutes |

---

### WAIT — Business Hours Delay (before Email 2)

**What it does:**
Introduces a 5-minute delay and enforces a business-hours sending window so Email 2 does not land outside working hours.

| Setting | Value |
|---|---|
| Wait For | Time Delay |
| Duration | 5 minutes |
| Resume On | Monday, Tuesday, Wednesday, Thursday, Friday |
| Resume Between Hours | 9:00 AM – 5:00 PM |

---

### EMAIL — Mosquito Email 2

**What it does:**
Sends the second and final upsell email. The tone shifts from anticipatory ("coming") to urgent ("are HERE!"), using the same template and Sign Me Up trigger link.

| Setting | Value |
|---|---|
| Action Name | Mosquito Email 2 |
| Subject | The Mosquitos are HERE! |
| Template | Upsell Mosquito Existing CHC 2026 |
| Template CTA Button | Sign Me Up |

**Key detail:** The same template is used as Email 1 but the subject line is updated to reflect urgency. The Sign Me Up button inside is the same tracked trigger link.

---

### WAIT — Trigger Link Clicked (Window #2)

**What it does:**
Same logic as Window #1 — listens for the Sign Me Up link click or waits up to 30 days.

| Setting | Value |
|---|---|
| Wait For | Trigger Link Clicked |
| Trigger Link | Upsell Mosquito Sign Me Up |
| Timeout | 30 Days |

**Branches:**
- **Link Clicked** → Conversion Sequence
- **Timeout after 30 days** → Opportunity marked Abandoned, contact removed from workflow

---

### BRANCH: LINK CLICKED (after Email 2) → Conversion Sequence

When the contact clicks "Sign Me Up" in Email 2, they are routed into the same Conversion Sequence as the Email 1 click path. See the [Conversion Sequence](#conversion-sequence) section below.

---

### BRANCH: TIMEOUT (30 days, no click on Email 2)

The contact did not engage with either email over the full ~60-day window. The opportunity is closed as lost and the contact is removed from the workflow.

**1. Update Opportunity**

| Field | Value |
|---|---|
| Action Name | Update Opportunity |
| Pipeline | Upsell Mosquito |
| Status | Abandoned |

**What it does:** Marks the deal as lost in the Upsell Mosquito pipeline so the opportunity is not left open and pipeline reporting stays accurate.

**2. Remove from Current Workflow**

| Field | Value |
|---|---|
| Action Name | Remove from Current Workflow |
| Workflow | Current Workflow |

**What it does:** Unenrolls the contact from this workflow, ending all further actions. The contact remains in the CRM; only this workflow's execution is terminated.

---

## Conversion Sequence

> This sequence is identical whether the contact clicked "Sign Me Up" in Email 1 or Email 2.

---

### 1. Remove from Workflow

**What it does:**
Stops the contact from continuing through the parallel SMS workflow so they do not receive further upsell SMS messages now that they have self-converted.

| Setting | Value |
|---|---|
| Action Name | Remove from Workflow |
| Workflow | Upsell Mosquito Existing CHC SMS |

---

### 2. Update Contact Field

**What it does:**
Writes the property and address data from the original inbound webhook payload back to the contact record to ensure all fields are current at the time of conversion.

| Setting | Value |
|---|---|
| Action Name | Update Contact Field |
| Action Type | Update field data |

Fields updated:

| Field | Source |
|---|---|
| Yard Sq Ft | Inbound Webhook |
| House Sq Ft | Inbound Webhook |
| Street Address | Inbound Webhook |
| City | Inbound Webhook |
| State | Inbound Webhook |
| Postal Code | Inbound Webhook |

---

### 3. Wait — 1 Minute

**What it does:**
Introduces a brief pause after the field update to allow GHL's data layer to finish writing the contact field values before the math operation reads them. Prevents the math step from running on stale data.

| Setting | Value |
|---|---|
| Action Name | Wait 1 Minute |
| Wait For | Time Delay |
| Duration | 1 minute |

---

### 4. Math Operation

**What it does:**
Calculates the total contract value for the opportunity by combining the recurring monthly price across the full season with the one-time initial fee.

**Formula:**
> **(Add Mosquito Mo × 11) + Add Mosquito I**

| Variable | Description |
|---|---|
| `Add Mosquito Mo` | Monthly recurring service price |
| `Add Mosquito I` | Initial / one-time setup price |
| Result | Full season value (11 months) plus initial fee |

- Output written to: `{{contact.system_opportunity_value}}`

---

### 5. Update Opportunity

**What it does:**
Moves the open opportunity to Won status and records the calculated value, closing the deal in the pipeline.

| Field | Value |
|---|---|
| Pipeline | Upsell Mosquito |
| Pipeline Stage | Self Sold |
| Status | Won |
| Opportunity Value | `{{contact.system_opportunity_value}}` |

**Key detail:** "Self Sold" stage indicates the customer converted without a sales rep — they clicked the email button and completed the process independently.

---

### 6. Webhook

**What it does:**
Fires a POST request to Zapier with key account and property data so downstream systems (e.g., billing, field service software, or CRM enrichment) can process the new sale.

| Setting | Value |
|---|---|
| Method | POST |
| Workflow | `https://zapier.com/editor/346314379/published` |

Custom data payload:

| Field | Source |
|---|---|
| `FRACCT` | Inbound Webhook |
| `Housesqft` | Inbound Webhook |
| `Lawnsqft` | Inbound Webhook |
| `FirstName` | Inbound Webhook |
| `LastName` | Inbound Webhook |

---

### 7. Email — Welcome to the Outdoor Oasis Club! 🏝️

**What it does:**
Sends a branded welcome/onboarding email confirming the customer's enrollment in the mosquito protection program and directing them to check their email for service details.

| Setting | Value |
|---|---|
| Action Name | Welcome to the Outdoor Oasis Club! 🏝️ |
| Subject | Welcome to the Outdoor Oasis Club! 🏝️ |
| Linked Template | Welcome to the Outdoor Oasis Club - SelfService 2026 |

---

### 8. SMS — Sold Confirmation

**What it does:**
Sends an immediate, friendly confirmation text to reinforce the purchase decision and direct the customer to check their email for full service details.

| Setting | Value |
|---|---|
| Template | WT00 Pest MOS Sold WIP (HTML Snippet) |

Message:

> Hey {{contact.first_name}}, your search is over! Good for you...bad for mosquitos! The entire BugBros office is so excited that we're busting out our CRAZY HAPPY Dance just for you (feel free to join us). Check your email for more details on your service.

---

## Email Reference

| Action Name | Subject | Template | Trigger |
|---|---|---|---|
| Mosquito Email 1 | Mosquitos Are Coming! | Upsell Mosquito Existing CHC 2026 | Enrollment into workflow |
| Mosquito Email 2 | The Mosquitos are HERE! | Upsell Mosquito Existing CHC 2026 | 30-day timeout on Email 1 |
| Welcome to the Outdoor Oasis Club! 🏝️ | Welcome to the Outdoor Oasis Club! 🏝️ | Welcome to the Outdoor Oasis Club - SelfService 2026 | Conversion (link clicked) |

---

## Timing Reference

| Step | Type | Duration | Business Hours |
|---|---|---|---|
| Window #1 (after Email 1) | Trigger Link / Timeout | 30 Days | — |
| Drip before Email 2 | Drip Mode | Batch 5 / 5 min | — |
| Wait before Email 2 | Time Delay | 5 minutes | Mon–Fri, 9AM–5PM |
| Window #2 (after Email 2) | Trigger Link / Timeout | 30 Days | — |
| Conversion Step 3 | Time Delay | 1 minute | — |

> **Maximum workflow duration:** ~60 days (two 30-day reply windows)

---

## Field & Variable Reference

| Variable | Description |
|---|---|
| `{{contact.add_mosquito_i}}` | Initial / setup price |
| `{{contact.add_mosquito_mo}}` | Monthly recurring price |
| `{{contact.system_opportunity_value}}` | Calculated total: (Mo × 11) + Initial |
| `{{contact.first_name}}` | Contact first name |
| `FRACCT` | Account reference sent to Zapier |
| `Housesqft` | House square footage sent to Zapier |
| `Lawnsqft` | Lawn/yard square footage sent to Zapier |

---

## Relationship to SMS Workflow

This workflow and the *Upsell Mosquito Existing CHC SMS* workflow run **simultaneously** once a contact is enrolled. They are designed to complement each other:

| | SMS Workflow | Email Workflow |
|---|---|---|
| **Conversion trigger** | Contact replies "OASIS" to SMS | Contact clicks "Sign Me Up" in email |
| **On conversion** | Removes contact from Email workflow | Removes contact from SMS workflow |
| **Prevents duplicate** | Yes — both workflows remove the other on win | Yes — both workflows remove the other on win |
| **Final timeout** | ~90 days (3 × 30-day SMS windows) | ~60 days (2 × 30-day email windows) |

---

## Summary of Outcomes

| Scenario | Final Outcome |
|---|---|
| Contact clicks "Sign Me Up" in Email 1 | Opportunity → **Won** (Self Sold) · SMS workflow stopped · Zapier webhook fired · Welcome email + SMS sent |
| Contact clicks "Sign Me Up" in Email 2 | Opportunity → **Won** (Self Sold) · SMS workflow stopped · Zapier webhook fired · Welcome email + SMS sent |
| No click after both emails (~60 days) | Opportunity → **Abandoned** · Removed from workflow |
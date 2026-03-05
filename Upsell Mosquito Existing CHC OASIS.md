# Zapier Workflow Documentation
## Tulsa Upsell Mosquito — OASIS v1

---

## Workflow Diagram

```
┌──────────────────────────────────────────────────┐
│         🔗 Webhooks by Zapier                    │
│   Step 1: Catch Hook                             │
│   TRIGGER — receives incoming webhook data       │
└──────────────────────────┬───────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────┐
│         ⚙️  Formatter by Zapier                  │
│   Step 2: Date / Time                            │
│   Formats date/time values from the webhook      │
└──────────────────────────┬───────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────┐
│         💬 Slack                                 │
│   Step 3: Send (Notify) Channel Message          │
│   Posts a notification to a Slack channel        │
└──────────────────────────┬───────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────┐
│         🔗 Webhooks by Zapier                    │
│   Step 4: Search for CHC Subscription            │
│   Queries an external API for CHC subscription   │
└──────────────────────────┬───────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────┐
│         🔗 Webhooks by Zapier                    │
│   Step 5: Get CHC Segmentation                   │
│   Retrieves customer segment data from CHC       │
└──────────────────────────┬───────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────┐
│         🔁 Looping by Zapier                     │
│   Step 6: Create Loop From Line Items            │
│   Iterates over each CHC segmentation result     │
└──────────────────────────┬───────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────┐
│   [FILTER] Filter by Zapier                      │
│   Step 7: Filter Conditions                      │
│   Only continues if conditions are met           │
└──────────────────────────┬───────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────┐
│         ⚙️  Formatter by Zapier                  │
│   Step 8: Day                                    │
│   Extracts or formats the day value              │
└──────────────────────────┬───────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────┐
│         ⚙️  Formatter by Zapier                  │
│   Step 9: Month                                  │
│   Extracts or formats the month value            │
└──────────────────────────┬───────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────┐
│         ⚙️  Formatter by Zapier                  │
│   Step 10: Year                                  │
│   Extracts or formats the year value             │
└──────────────────────────┬───────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────┐
│         ⚙️  Formatter by Zapier                  │
│   Step 11: Next Bill Month                       │
│   Calculates or formats the next billing month   │
└──────────────────────────┬───────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────┐
│         ⚙️  Formatter by Zapier                  │
│   Step 12: Final Price                           │
│   Calculates or formats the final price          │
└──────────────────────────┬───────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────┐
│         ⚙️  Formatter by Zapier                  │
│   Step 13: Recurring Price                       │
│   Formats the recurring subscription price       │
└──────────────────────────┬───────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────┐
│         🔗 Webhooks by Zapier                    │
│   Step 14: Create OOO Subscription               │
│   POSTs to external API to create subscription   │
└──────────────────────────┬───────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────┐
│         📊 Google Sheets                         │
│   Step 15: Create Spreadsheet Row                │
│   Logs the completed subscription to a sheet     │
└──────────────────────────────────────────────────┘
```

---

## Step-by-Step Node Documentation

---

### Step 1 — Webhooks by Zapier: Catch Hook

**App:** Webhooks by Zapier
**Action:** Catch Hook
**Type:** Trigger

**Purpose:**
This is the entry point of the workflow. It listens for an incoming HTTP POST request from an external system (such as OASIS or a CRM) that signals a new mosquito upsell opportunity. When a payload is received, the Zap fires and passes all incoming data downstream.

**Configuration Notes:**
- A unique webhook URL is generated by Zapier — this URL must be registered in the sending system (OASIS).
- The incoming payload likely contains customer identity fields, property details, and service eligibility data.
- Test the trigger by sending a sample payload from the source system before activating.

---

### Step 2 — Formatter by Zapier: Date / Time

**App:** Formatter by Zapier
**Action:** Date / Time
**Type:** Action

**Purpose:**
Parses and reformats a raw date/time value from the incoming webhook payload. This ensures the date is in a consistent, usable format for downstream logic (e.g., billing calculations, scheduling, Slack messages).

**Configuration Notes:**
- Input: raw date string from Step 1 (e.g., 2025-03-15T08:00:00Z).
- Output: MMMM DD YYYY HH:mm:ss
- Timezone: US/Central
- This step feeds into the Day, Month, and Year formatter steps later in the flow.

---

### Step 3 — Slack: Send (Notify) Channel Message

**App:** Slack
**Action:** Send Channel Message
**Type:** Action

**Purpose:**
Posts a real-time notification to a designated Slack channel alerting the team that a new upsell attempt has been triggered. Useful for visibility, monitoring, and team awareness of campaign activity.

**Configuration Notes:**
- Connect to the relevant Slack workspace and target channel (e.g., #sold).
- Message should include key customer details from Step 1 (Custom Data First Name Custom Data Last Name exted in OASIS to sign up for OOC For Tulsa).
- Consider including a timestamp from Step 2 for context.

---

### Step 4 — Webhooks by Zapier: Search for CHC Subscription

**App:** Webhooks by Zapier
**Action:** Search for CHC Subscription
**Type:** Action
**Url:** https://bugbros.pestroutes.com/api/subscription/search?customerIDs=

**Purpose:**
Sends a GET or POST request to an external API (to field routes) to look up whether the customer already has an active CHC subscription. The result determines if the upsell path is valid.

**Configuration Notes:**
- Use the Customer ID or account identifier from Step 1 as the lookup key.
- The response will contain subscription status and related metadata.
- This step gates the rest of the workflow — if no CHC subscription exists, the filter in Step 7 should stop processing.

---

### Step 5 — Webhooks by Zapier: Get CHC Segmentation

**App:** Webhooks by Zapier
**Action:** Get CHC Segmentation
**Type:** Action
**Url:** https://bugbros.pestroutes.com/api/subscription/get?subscriptionIDs=

**Purpose:**
Sends a GET or POST request to an external API (to field routes) to get CHC Subscription.

**Configuration Notes:**
- Uses the CHC subscription data from Step 4 to query the correct segment.
- Output is likely a list of records (line items), which is why the next step is a Loop.
- This step's results drive the looping and filtering logic downstream.

---

### Step 6 — Looping by Zapier: Create Loop From Line Items

**App:** Looping by Zapier
**Action:** Create Loop From Line Items
**Type:** Action

**Purpose:**
Takes the multi-record segmentation response from Step 5 and processes each record individually. The loop ensures every relevant segment or service line gets evaluated through the filter and pricing steps that follow.

**Configuration Notes:**
- Map values SubscriptionOn: 5. Subscriptions Service Type and NextBillingDate: 5. Subscriptions Next Billing Date
- Each iteration of the loop runs Steps 7–15 for one segment record.
- Ensure field mapping is complete to avoid null values downstream.

---

### Step 7 — Filter by Zapier: Filter Conditions

**App:** Filter by Zapier
**Action:** Filter Conditions
**Type:** Filter (conditional gate)

**Purpose:**
Acts as a conditional gate within the loop. Only records that meet defined criteria continue through the workflow. Records that don't match are skipped, preventing unqualified customers or segments from receiving an upsell offer or having a subscription created erroneously.

**Filter Conditions:**
- 6. Subscription exactly matches "Crazy Happy Club"

**Configuration Notes:**
- All conditions must evaluate to true for the loop iteration to continue.
- Review filter logic carefully — overly strict filters may suppress valid upsells; too loose may cause errors in Step 14.

---

### Step 8 — Formatter by Zapier: Day

**App:** Formatter by Zapier
**Action:** Date / Time (extract Day)
**Type:** Action

**Purpose:**
Extracts or reformats the day component from a date value (Next Bill Date). This is used downstream to construct billing date fields for the subscription creation API call.

**Configuration Notes:**
- Input: date from Step 6 (Next Billing Date).
- Output: numeric day value (e.g., 15).

---

### Step 9 — Formatter by Zapier: Month

**App:** Formatter by Zapier
**Action:** Date / Time (extract Month)
**Type:** Action

**Purpose:**
Extracts or reformats the month component from the date value. Combined with Steps 8 and 10, this builds a fully structured date for billing and scheduling purposes.

**Configuration Notes:**
- Output: numeric month (e.g., 03).
- Used in the Next Bill Month calculation in Step 11.

---

### Step 10 — Formatter by Zapier: Year

**App:** Formatter by Zapier
**Action:** Date / Time (extract Year)
**Type:** Action

**Purpose:**
Extracts the year component from the date value. Together with the Day and Month steps, it enables precise date construction for the subscription API payload.

**Configuration Notes:**
- Output: 4-digit year (e.g., 2025).

---

### Step 11 — Formatter by Zapier: Next Bill Month

**App:** Formatter by Zapier
**Action:** Transform
**Type:** Action

**Purpose:**
Calculates the next billing month based on the current date components extracted in Step 9. This is passed into the subscription creation step to set the correct billing cycle start.

**Configuration Notes:**
- It uses spreadsheet like formula:
- =if({{9. Output}}>4,{{9. Output}}+1,05)

---

### Step 12 — Formatter by Zapier: Final Price

**App:** Formatter by Zapier
**Action:** Transform
**Type:** Action

**Purpose:**
Calculates or formats the one-time or initial price for the mosquito upsell service. This incorporates pricing rules based on property size (Lawn Sqft) or customer segment from earlier steps.

**Configuration Notes:**
- It uses spreadsheet like formula:
- =if({{1. Yard Sq Ft}}>6999,if({{1. Yard Sq Ft}}>13999,if({{1. Yard Sq Ft}}>20999,if({{1. Yard Sq Ft}}>27999,350,300),250),200),150)
- Used in the subscription creation payload in Step 14.

---

### Step 13 — Formatter by Zapier: Recurring Price

**App:** Formatter by Zapier
**Action:** Transform
**Type:** Action

**Purpose:**
Formats the ongoing recurring subscription price for the mosquito service. This is the amount billed to the customer each billing period going forward.

**Configuration Notes:**
- It uses spreadsheet like formula:
- if({{1. Yard Sq Ft}}>6999,if({{1. Yard Sq Ft}}>13999,if({{1. Yard Sq Ft}}>20999,if({{1. Yard Sq Ft}}>27999,90,80),70),60),50)
- Passed alongside Final Price into the subscription creation call in Step 14.

---

### Step 14 — Webhooks by Zapier: Create OOC Subscription

**App:** Webhooks by Zapier
**Action:** POST — Create OOC Subscription
**Type:** Action
**Url:** https://bugbros.pestroutes.com/api/subscription/create?

**Purpose:**
Sends an HTTP POST request to the OASIS or CHC platform API to officially create the mosquito (OOO) subscription for the customer. This is the core fulfillment step — enrolling the customer in the new mosquito service.

**Payload Fields:**
- serviceID: 149
- customerID: 1. Custom Data FRACCT
- active: 1
- nextBillingDate: [10. Output]-0[11. Output]-[8. Output]
- seasonalStart: 10. Output
- seasonalEnd: 10. Output
- soldBy: 143
- initialCharge: 12. Output
- serviceCharge: 13. Output
- billingFrequency: 30

**Configuration Notes:**
- Set Content-Type to application/json.
- Confirm the correct API endpoint for subscription creation in the OASIS environment.
- Handle error responses — if the API returns a failure, consider adding an error notification path.

---

### Step 15 — Google Sheets: Create Spreadsheet Row

**App:** Google Sheets
**Action:** Create Spreadsheet Row
**Type:** Action
**Spreadsheet:** BugBros Sales Report
**Sheet:** Tulsa Upsell

**Purpose:**
Logs each successfully created mosquito subscription into a Google Sheet for record-keeping, auditing, and reporting. This provides a human-readable history of all upsell conversions processed by the Zap.

**Likely Columns Logged:**
- Date Sold
- Salesperson
- First Name
- Last Name
- Phone
- Email
- Street Address
- City
- State
- Zipcode
- Location ID
- Contact ID
- FRAcct
- Lawn SF
- House SF
- Program
- Initial
- Monthly

**Configuration Notes:**
- Connect to the correct Google account, spreadsheet, and sheet tab.
- Ensure column order in the sheet matches the field mapping in Zapier.
- This row serves as the audit trail — keep it active even if downstream notifications are added.

---

## Workflow Summary

This Zap automates the mosquito upsell process for existing Tulsa CHC customers. It is triggered by an incoming webhook, validates the customer's CHC subscription, segments them, applies date and pricing logic through a series of Formatter steps, creates the new mosquito (OOC) subscription via API, and logs the result to Google Sheets — with a Slack notification at the start for team visibility.

| Step | App | Action | Role |
|---|---|---|---|
| 1 | Webhooks by Zapier | Catch Hook | Trigger — receives customer data |
| 2 | Formatter by Zapier | Date / Time | Normalizes date from webhook |
| 3 | Slack | Send Channel Message | Notifies team of new upsell attempt |
| 4 | Webhooks by Zapier | Search for CHC Subscription | Looks up existing CHC account |
| 5 | Webhooks by Zapier | Get CHC Segmentation | Retrieves customer segment data |
| 6 | Looping by Zapier | Create Loop From Line Items | Iterates over segment records |
| 7 | Filter by Zapier | Filter Conditions | Gates unqualified records |
| 8 | Formatter by Zapier | Day | Extracts day from date |
| 9 | Formatter by Zapier | Month | Extracts month from date |
| 10 | Formatter by Zapier | Year | Extracts year from date |
| 11 | Formatter by Zapier | Next Bill Month | Calculates next billing month |
| 12 | Formatter by Zapier | Final Price | Formats one-time/initial price |
| 13 | Formatter by Zapier | Recurring Price | Formats ongoing subscription price |
| 14 | Webhooks by Zapier | Create OOO Subscription | Creates mosquito subscription via API |
| 15 | Google Sheets | Create Spreadsheet Row | Logs completed subscription record |
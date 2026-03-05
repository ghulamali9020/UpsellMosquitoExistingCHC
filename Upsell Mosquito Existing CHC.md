# Zapier Workflow Documentation
## Upsell Mosquito — Existing CHC Customers

---

## Workflow Diagram

```
┌─────────────────────────────────────────────┐
│         📅 Schedule by Zapier               │
│   Step 1: Custom Frequency                  │
│   Triggers the workflow on a set schedule   │
└─────────────────────┬───────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────┐
│         📊 Google Sheets                    │
│   Step 2: Get Many Spreadsheet Rows         │
│   Reads all customer rows from the sheet    │
│                                             │
│   Columns: Customer ID, Last Name,          │
│   First Name, Property SF, Address,         │
│   City, Email Address, Verify Email,        │
│   Phone 1, State, Zip Code,                 │
│   House Sqft, Lawn Sqft                     │
└─────────────────────┬───────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────┐
│         🔁 Looping by Zapier                │
│   Step 3: Create Loop From Line Items       │
│   Iterates over each customer row           │
│   one at a time                             │
└─────────────────────┬───────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────┐
│         🔗 Webhooks by Zapier               │
│   Step 4: POST                              │
│   Sends primary customer data to            │
│   GoHighLevel (GHL) SMS Workflow            │
└─────────────────────┬───────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────┐
│         🔗 Webhooks by Zapier               │
│   Step 5: Copy: POST in Webhooks by Zapier  │
│   Sends primary customer data to            │
│   GoHighLevel (GHL) SMS Workflow            │
└─────────────────────┬───────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────┐
│         ⏱ Delay by Zapier                  
│   Step 6: Delay For                         │
│   Pauses execution before next loop         │
│   iteration or downstream action            │
└─────────────────────────────────────────────┘
```

---

## Step-by-Step Node Documentation

---

### Step 1 — Schedule by Zapier: Custom Frequency

**App:** Schedule by Zapier  
**Action:** Custom Frequency  
**Type:** Trigger

**Purpose:**  
This is the entry point of the workflow. It fires the Zap automatically on a recurring schedule (e.g., daily, weekly, or a custom interval). No manual action is needed — Zapier polls the schedule and kicks off the automation at the defined time.

**Configuration Notes:**
- Set the frequency to match your outreach cadence (e.g., once a week on Monday mornings).
- This trigger does not pass any data downstream — it simply initiates the run.

---

### Step 2 — Google Sheets: Get Many Spreadsheet Rows

**App:** Google Sheets  
**Action:** Get Many Spreadsheet Rows  
**Type:** Action

**Purpose:**  
Pulls all existing CHC customer records from a designated Google Sheet. Each row represents one customer. All columns are passed downstream for use in messaging, filtering, or API calls.

**Sheet Columns:**

| Column | Description |
|---|---|
| `Customer ID` | Unique identifier for each customer |
| `Last Name` | Customer's last name |
| `First Name` | Customer's first name |
| `Property SF` | Total property square footage |
| `Address` | Street address |
| `City` | City of residence |
| `Email Address` | Primary email for outreach |
| `Verify Email` | Email validation flag or secondary email |
| `Phone 1` | Primary phone number |
| `State` | State abbreviation |
| `Zip Code` | Postal zip code |
| `House Sqft` | Square footage of the house structure |
| `Lawn Sqft` | Square footage of lawn area |

**Configuration Notes:**
- Connect to the correct Google account and select the target spreadsheet and sheet tab.
- Use filters if you only want to process rows meeting certain criteria (e.g., `Lawn Sqft > 0`).

---

### Step 3 — Looping by Zapier: Create Loop From Line Items

**App:** Looping by Zapier  
**Action:** Create Loop From Line Items  
**Type:** Action

**Purpose:**  
Takes the multi-row output from Google Sheets and processes each customer record one at a time. Without this step, Zapier would only process the first row. The loop ensures every customer in the sheet receives the upsell outreach.

**Configuration Notes:**
- Map all relevant fields from Step 2 (e.g., `Email Address`, `First Name`, `Lawn Sqft`) into the loop line items.
- Each iteration of the loop runs Steps 4–6 for a single customer.

---

### Step 4 — Webhooks by Zapier: POST

**App:** Webhooks by Zapier  
**Action:** POST  
**Type:** Action
**URL:** https://services.leadconnectorhq.com/hooks/Br3C4M7nA0y0gVZSmmFO/webhook-trigger/0db15d50-392e-4c9c-876a-ccc5536c0180

**Purpose:**  
Sends an HTTP POST request containing the current customer's data to GoHighLevel (GHL) SMS Workflow.

**Likely Payload Fields:**
- `FRAccount`
- `FirstName`
- `LastName`
- `Email`
- `Phone`
- `Address`
- `City`
- `State`
- `Zip`
- `LawnSqft`
- `HouseSqft`

**Configuration Notes:**
- Set the target URL to Go High Level via Lead Connector api.
- Set `Content-Type` to `application/json`.
- Map loop output fields to the correct payload keys.

---

### Step 5 — Webhooks by Zapier: Copy: POST in Webhooks by Zapier

**App:** Webhooks by Zapier  
**Action:** POST (Copy)  
**Type:** Action
**URL:** https://services.leadconnectorhq.com/hooks/BFlXKR4lwAg155Q5A6W3/webhook-trigger/sXzjqlxMmk6vltxeC5l8

**Purpose:**  
Sends an HTTP POST request containing the current customer's data to GoHighLevel (GHL) Email Workflow.

**Configuration Notes:**
- Confirm whether this POST goes to the same endpoint or a different one.
- Review the payload to ensure it's not sending duplicate communications to the customer.
- Consider adding a filter before this step if it should only fire under certain conditions.

---

### Step 6 — Delay by Zapier: Delay For

**App:** Delay by Zapier  
**Action:** Delay For  
**Type:** Action

**Purpose:**  
Introduces a deliberate pause after each customer's POST requests before the loop continues to the next record. This prevents rate-limiting errors on the receiving APIs, avoids spam triggers, and spaces out message delivery for a more natural outreach cadence.

**Configuration Notes:**
- Set the delay duration based on the receiving API's rate limits (e.g., 1–5 seconds between records).
- For email/SMS campaigns, a longer delay (minutes or hours) may be appropriate to avoid bulk-send flags.
- This step runs inside the loop, so the delay applies per customer record.

---

## Workflow Summary

This Zap automates a mosquito treatment upsell campaign targeting existing CHC customers. On a scheduled basis, it reads all customer records from Google Sheets, loops through each one, and fires POST requests to one or two external platforms to trigger personalized outreach — using lawn square footage and contact data to drive the messaging. A delay between each record ensures reliable, rate-safe delivery.

| Step | App | Action | Role |
|---|---|---|---|
| 1 | Schedule by Zapier | Custom Frequency | Trigger — starts the workflow |
| 2 | Google Sheets | Get Many Spreadsheet Rows | Fetches all customer records |
| 3 | Looping by Zapier | Create Loop From Line Items | Iterates per customer |
| 4 | Webhooks by Zapier | POST | Sends data to primary endpoint |
| 5 | Webhooks by Zapier | POST (Copy) | Sends data to secondary endpoint |
| 6 | Delay by Zapier | Delay For | Pauses between loop iterations |
# Upsell Mosquito Existing CHC

This documentation outlines the workflow for the **Upsell Mosquito Existing CHC** campaign.

## Overview

This automated process is designed to send periodic emails and SMS messages to existing contacts. The top-level flow is:
1. Contacts are fetched from a spreadsheet.
2. The contacts are pushed to GoHighLevel (GHL).
3. In GHL, an SMS workflow is triggered which sends an initial SMS, and then also sends follow-up emails.

## Workflows

The system is composed of three interconnected workflows. Detailed documentation for each can be found in the links below:

### 1. Zapier Data Fetch Workflow
Fetches contacts from the sheet and sends them to GHL.
* [Upsell Mosquito Existing CHC](./Upsell%20Mosquito%20Existing%20CHC.md)

### 2. GHL SMS Workflow
Triggered within GHL to send the first SMS interaction.
* [Upsell Mosquito Existing CHC SMS](./Upsell%20Mosquito%20Existing%20CHC%20SMS.md)

### 3. GHL Email Workflow
Triggered after the SMS workflow to handle subsequent email follow-ups.
* [Upsell Mosquito Existing CHC Email](./Upsell%20Mosquito%20Existing%20CHC%20Email.md)

### 4. Zapier OOC Subscription Workflow
* [Upsell Mosquito Existing CHC OASIS](./Upsell%20Mosquito%20Existing%20CHC%20OASIS.md)

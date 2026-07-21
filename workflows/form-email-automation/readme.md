# Form → Email Automation Workflow

## Purpose

This workflow captures user submissions through an n8n web form, stores the data in a Google Sheet, sends a personalised email reply, and marks the submission as completed. The status check prevents duplicate processing if the workflow is accidentally re‑triggered or manually re‑executed.

---

## Flow Overview

The workflow follows a simple, linear path:

1. **On form submission** – The n8n Form Trigger collects `Name`, `Email`, and `Message` from the user.
2. **Append row in sheet** – Writes the submission to Google Sheets with:
   - A unique `ID` (generated from the current timestamp in milliseconds)
   - `Status` set to `"pending"`
3. **If Node** – Checks whether the `Status` is **not** `"sent"`.
   - ✅ **True (pending)** → Continue to send the email.
   - ❌ **False (sent)** → Stop the workflow (prevents re‑processing).
4. **Edit Fields** – Builds the `email_reply` field with a personalised message using the submitted `Name` and `Message`.
5. **Send a message (Gmail)** – Sends the email to the submitted email address.
6. **Update row in sheet** – After the email is sent, updates the `Status` from `"pending"` to `"sent"` (matching by the unique `ID`).

---

## Nodes & Key Parameters

| Node | Type | Purpose |
|------|------|---------|
| On form submission | Form Trigger | Captures `Name`, `Email`, and `Message` from the user |
| Append row in sheet | Google Sheets | Saves form data with a unique `ID`, timestamp, and `Status = "pending"` |
| If | IF Node | Checks `Status != "sent"` to prevent duplicate processing |
| Edit Fields | Set Node | Creates the `email_reply` field with a personalised message |
| Send a message | Gmail | Sends the email reply to the submitter |
| Update row in sheet | Google Sheets | Updates `Status` to `"sent"` (matches by `ID`) |

---

## Credentials Required

| Service | Node(s) | Credential Type |
|---------|---------|-----------------|
| **Gmail** | Send a message | OAuth2 (`gmailOAuth2`) |
| **Google Sheets** | Append row, Update row | OAuth2 (`googleSheetsOAuth2Api`) |

> ⚠️ The credential IDs in the JSON are placeholders. You must re‑authenticate each node in your own n8n instance.

---

## Environment Variables

This workflow **does not** use environment variables – the spreadsheet ID, form fields, and email template are hard‑coded.  
To make it more portable, consider moving the following to `$env` variables:

| Suggested Variable | Purpose |
|--------------------|---------|
| `SHEET_ID` | Google Sheet document ID |
| `EMAIL_SUBJECT` | Email subject line (currently `"Thanks For Contacting US!"`) |
| `EMAIL_FOOTER` | The signature/footer in the email template |

---

## Setup Instructions

1. **Import** the `workflow.json` file into your n8n instance.
2. **Authenticate** the credential nodes:
   - Click on the **Send a message** node → select or create your Gmail OAuth2 credential.
   - Click on the **Append row in sheet** and **Update row in sheet** nodes → select or create your Google Sheets OAuth2 credential.
3. **Update the Google Sheet URL** (if using your own spreadsheet):
   - In both Sheets nodes, replace the `documentId` value with your own spreadsheet ID.
   - Ensure the sheet has the following columns: `ID`, `Name`, `Email`, `Message`, `SubmittedAt`, `FormMode`, `Status`.
4. **Customise the email template** (optional):
   - Edit the **Edit Fields** node and modify the `email_reply` value to change the message.
5. **Activate** the workflow (toggle from `Inactive` to `Active`).
6. **Test** – Access the form via the **Form Trigger** node's production URL (or test in the editor) and submit a sample entry.

---

## Important Notes & Caveats

- **Duplicate protection** – The `If` node ensures that if a row already has `Status = "sent"`, the workflow will **not** send another email. This is useful if you manually re‑execute the workflow or if n8n retries an execution.
- **Unique ID generation** – The `ID` is generated using `$now.toMillis()`. This is unique per millisecond, making it safe for matching rows.
- **Email failure safety** – If the **Send a message** (Gmail) node fails (e.g., invalid email address), the workflow will throw an error and **will not** update the status to `"sent"`. The row will remain `"pending"`, allowing you to investigate and retry.
- **Sheet schema** – The workflow expects columns named exactly: `ID`, `Name`, `Email`, `Message`, `SubmittedAt`, `FormMode`, `Status`. If your sheet uses different headers, update the mappings in both Sheets nodes.

---

## Known Issues & Improvements

| Issue / Limitation | Suggestion |
|---------------------|------------|
| No `SentAt` timestamp recorded | Add a `Set` node before the `Update row in sheet` to save `$now` as `SentAt` |
| No admin notification when a form is submitted | Add a second email node (Gmail) after the update to notify a sales/support team |
| No handling for failed email delivery | Add a `Switch` or `OnError` node to update `Status = "failed"` and trigger a retry |
| Hard‑coded Google Sheet ID | Move to environment variables for easier sharing across environments |
| Email template not HTML formatted | Change `emailType` in the Gmail node to `html` and update the message with `<p>` tags |

---

## License

This workflow is part of the n8n Workflow Hub repository. Use it freely, but please retain attribution.

---

# Lead Generation Workflow (Apify + Google Maps Enrichment)

## Purpose

This workflow scrapes business data from Google Maps using Apify, enriches each lead by scraping their website for emails, phone numbers, and social media profiles, and logs everything to a Google Sheet. It's designed for sales, marketing, and lead research teams.

---

## Flow Overview

The workflow processes leads in **three main phases**:

### Phase 1: Discovery & Initial Storage
1. **Form Trigger** – User inputs `Business Category`, `Location`, and `Minimum Rating`.
2. **Discovery (Apify)** – Runs the Google Maps scraper with the given criteria (max 20 places).
3. **Remove Duplicates** – Ensures unique `placeId` entries.
4. **Ahmedabad_1 (Set Node)** – Maps Apify's raw output to consistent field names.
5. **Cleaner (Code Node)** – Normalises website/phone, flags social domains, and classifies lead quality (Complete / Partial / Incomplete).
6. **Append row in sheet** – Saves the initial lead data to Google Sheets.

### Phase 2: Filtering
7. **If Node** – Only continues for leads that:
   - Have a website (`Has_Website = true`)
   - The website is **not** a social network (`Is_Social = false`)  
   *This avoids scraping Facebook/Instagram/etc. and focuses on business websites.*

### Phase 3: Enrichment Loop (Per Lead)
8. **Loop Over Items** – Processes each filtered lead one at a time.
9. **Update row in sheet (initial)** – *(This node is in the loop's first output branch – but the actual update happens later.)*
10. **Code in JavaScript1** – Builds a list of URLs to crawl (main website + `/contact`, `/about`, `/support`, etc.).
11. **Split Out** – Splits the URL list into individual items.
12. **Wait** – Adds a 5-second delay to avoid rate limiting/blocking.
13. **HTTP Request** – Fetches each page (with a custom User-Agent).
14. **Code in JavaScript2** – Extracts:
    - **Emails** (direct, mailto:, Cloudflare-protected)
    - **Phone numbers** (Indian + international formats)
    - **Social links** (LinkedIn, Facebook, Instagram, Twitter/X)
15. **Aggregate** – Merges results from all scraped pages.
16. **Code in JavaScript3** – Converts arrays to comma-separated strings.
17. **Edit Fields** – Prepares the update payload.
18. **Update row in sheet** (final) – Updates the original Google Sheet row (matching by `Place_ID`) with:
    - Emails found
    - Phone numbers found (stored in `Scrap_no.`)
    - Social media links

---

## Nodes & Key Parameters

| Node | Type | Purpose |
|------|------|---------|
| Form | Form Trigger | Collects search criteria from user |
| Discovery | Apify | Calls the Google Maps scraper actor |
| Remove Duplicates | Remove Duplicates | De‑duplicates by `placeId` |
| Ahmedabad_1 | Set Node | Standardises Apify's output fields |
| Cleaner | Code Node | Cleans website/phone and flags `isSocial` |
| Append row in sheet | Google Sheets | Writes initial lead data with `Status` flags |
| If | IF Node | Filters out social‑only websites |
| Loop Over Items | Split In Batches | Iterates over each qualifying lead |
| Code in JavaScript1 | Code Node | Builds a list of URLs to scrape |
| Split Out | Split Out | Converts URL array into individual items |
| Wait | Wait | Pauses 5 seconds between requests |
| HTTP Request | HTTP Request | Fetches website HTML |
| Code in JavaScript2 | Code Node | Extracts emails, phones, and social links |
| Aggregate | Aggregate | Merges results from all scraped pages |
| Code in JavaScript3 | Code Node | Flattens arrays to strings |
| Edit Fields | Set Node | Prepares final update payload |
| Update row in sheet | Google Sheets | Updates the lead's row with enriched data |

---

## Credentials Required

| Service | Node(s) | Credential Type |
|---------|---------|-----------------|
| **Apify** | Discovery | API Key (`apifyApi`) |
| **Google Sheets** | Append row, Update row | OAuth2 (`googleSheetsOAuth2Api`) |

> ⚠️ The credential IDs in the JSON are placeholders. You must re‑authenticate each node in your own n8n instance.

---

## Environment Variables

This workflow **does not** use environment variables – all configuration is hard‑coded (sheet ID, Apify actor ID, rating thresholds).  
To make it reusable across environments, consider moving these to `$env` variables:

| Suggested Variable | Purpose |
|--------------------|---------|
| `SHEET_ID` | Google Sheet document ID |
| `APIFY_ACTOR_ID` | Apify actor ID (currently `nwua9Gu5YrADL7ZDj`) |
| `MAX_PLACES` | Max places per search (currently 20) |

---

## Setup Instructions

1. **Import** the `workflow.json` file into your n8n instance.
2. **Authenticate** the credential nodes:
   - Click on the **Discovery** node → select or create your Apify API credential.
   - Click on the **Append row in sheet** and **Update row in sheet** nodes → select or create your Google Sheets OAuth2 credential.
3. **Update the Google Sheet URL** (if using your own spreadsheet):
   - In both Sheets nodes, replace the `documentId` value with your own spreadsheet ID.
4. **Adjust the Apify input** (optional):
   - In the **Discovery** node, you can change `maxCrawledPlacesPerSearch` (default: 20).
5. **Activate** the workflow (toggle from `Inactive` to `Active`).
6. **Test** by submitting the form with a business category (e.g., `restaurant`) and location (e.g., `New York`).

---

## Important Notes & Caveats

- **Social domains are excluded** – The `If` node filters out leads whose website is a social platform (Facebook, Instagram, LinkedIn, etc.). This saves time and avoids irrelevant scraping.
- **Rate limiting** – The `Wait` node (5 seconds) helps prevent IP blocks when scraping. Adjust it up/down depending on your target websites.
- **Phone extraction** – Handles Indian numbers (`+91` prefix, 10-digit) and international formats. Duplicates are removed by normalising digits.
- **Email filtering** – Automatically filters out image filenames like `logo@2x.png` from being mistaken for emails.
- **Cloudflare-protected emails** – The code can decode `data-cfemail` attributes automatically.
- **Error handling** – The HTTP Request node is set to `continueRegularOutput` on failure, so a failed request won't break the whole loop.

---

## Known Issues & Improvements

| Issue | Suggestion |
|-------|------------|
| Enrichment runs sequentially (slow for many leads) | Consider increasing `maxCrawledPlacesPerSearch` or parallelising with multiple `Split In Batches` loops |
| `Scrap_no.` column stores phone numbers – confusing name | Rename the column to `Phone_Extracted` in the Sheets node and schema |
| No admin notification on completion | Add a `Gmail` node after the loop to send a summary email |
| No retry for failed HTTP requests | Add a `Retry` node or use `onError` to log failures to a separate sheet |
| Hard‑coded Google Sheet ID | Move to environment variables for easier sharing |

---

## License

This workflow is part of the n8n Workflow Hub repository. Use it freely, but please retain attribution.

---

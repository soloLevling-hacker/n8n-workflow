# 🚀 n8n Workflow Hub

A universal, centralized repository for importing, versioning, and managing multiple **n8n automation workflows**. Whether you need lead generation, email automation, or custom integrations—this is your plug-and-play library.

---

## 📚 Available Workflows

| Workflow | Description | Trigger | Key Integrations |
|----------|-------------|---------|------------------|
| [**Form → Email Automation**](/workflows/form-email-automation) | Captures form submissions, saves data to Google Sheets, sends a personalized email reply, and updates the submission status to prevent duplicates. | Web Form (n8n Form Trigger) | Gmail, Google Sheets |
| [**Lead Generation (Apify)**](/workflows/lead-generation-workflow) | Scrapes Google Maps via Apify based on category/location/rating, enriches leads by scraping websites for emails, phones, and social links, then logs everything to a Google Sheet. | Web Form (n8n Form Trigger) | Apify, Google Sheets, HTTP Request |

> **New workflows coming soon!** Check back regularly or submit your own.

---

## 🛠 Prerequisites

Before importing these workflows, ensure you have:

- **n8n** (self-hosted or cloud) – version **1.0 or higher**.
- Active API credentials for the services used in your chosen workflow(s):
  - **Google Sheets** (OAuth2)
  - **Gmail** (OAuth2)
  - **Apify** (API Key)

---

## 🚀 Getting Started (How to Use)

1. **Clone this repository**
   ```bash
   git clone https://github.com/soloLevling-hacker/n8n-workflow.git
   cd n8n-workflow

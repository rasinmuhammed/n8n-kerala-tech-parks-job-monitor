# Kerala Tech Parks Job Monitor (n8n Workflow)

## 🚀 Overview

This repository contains an [n8n.io](https://n8n.io/) workflow designed to automate the monitoring of job postings across major Kerala technology parks:

* UL Cyberpark (Kozhikode)
* Technopark (Trivandrum)
* Cyberpark (Kozhikode)
* Infopark (Kochi, Thrissur, Cherthala)

The workflow fetches job listings periodically, filters them based on customizable keywords, identifies only *new* job postings since the last run (using Redis for state management), and sends an email notification summarizing the new, relevant opportunities.

## ✨ Features

* **Multi-Source Monitoring:** Checks four different tech park job portals.
* **Diverse Fetching Methods:** Handles:
    * Static HTML scraping and parsing.
    * Direct JSON API consumption.
    * API calls requiring session cookies and specific headers.
* **Keyword Filtering:** Easily customize a list of keywords to match desired job roles, skills, or experience levels (e.g., 'Software Engineer', 'Python', 'Fresher').
* **Deduplication:** Utilizes Redis to keep track of job IDs that have already been processed, ensuring notifications are sent only once per new job.
* **Automated Scheduling:** Configured to run automatically every 12 hours (easily adjustable).
* **Email Notifications:** Sends a formatted HTML email via Gmail containing details of newly found, matching jobs.

## 🛠️ Tech Stack

* **Automation Platform:** [n8n.io](https://n8n.io/)
* **State Management:** [Redis](https://redis.io/)
* **Notifications:** [Gmail](https://www.google.com/gmail/)
* **Parsing & Logic:** JavaScript (within n8n Code nodes), Regular Expressions

## 📋 Prerequisites

Before importing and running this workflow, ensure you have the following set up:

1.  **n8n Instance:** A running instance of n8n (either self-hosted or via n8n cloud).
2.  **Redis Instance:** Access to a Redis server. You will need the **Host**, **Port**, and **Password** (if applicable).
3.  **Gmail Account:** A Gmail account for sending notifications. You need to configure **OAuth2 credentials** within n8n for secure authentication with the Gmail node.

## ⚙️ Setup Instructions

1.  **Import Workflow:**
    * Download the `Kerala Tech Parks Job Monitor.json` file from this repository.
    * In your n8n instance, go to "Workflows" and click "Import from File". Select the downloaded JSON file.
2.  **Configure Credentials:**
    * **Redis:**
        * Locate the **"Get Stored Job IDs"** node in the workflow.
        * In the node parameters panel, find the "Credentials" section for Redis.
        * Select an existing Redis credential or click "Create New". Enter your Redis **Host**, **Port**, and **Password** (if required) and save the credential.
        * Repeat this process for the **"Store Job IDs"** node, selecting the same Redis credential.
    * **Gmail:**
        * Locate the **"Send Email Notification"** node.
        * In the "Credentials" section for Gmail, select an existing credential or click "Create New".
        * Follow the n8n documentation/prompts to set up OAuth2 authentication with your Gmail account. This usually involves creating credentials in the Google Cloud Console and authorizing n8n.
3.  **Set Email Recipient:**
    * In the **"Send Email Notification"** node, modify the `Send To` parameter to the email address where you want to receive the job alerts.
4.  **Activate Workflow:**
    * Ensure the workflow is set to "Active" using the toggle switch in the top right corner of the n8n editor. It will then run based on the schedule defined in the "Schedule Every 12 Hours" trigger node.

## 🔧 Customization

You can easily modify several aspects of the workflow:

* **Keywords:**
    * Open the **"Filter by Keywords"** Code node.
    * Modify the `keywords` array within the JavaScript code to include the job titles, skills, or terms you are interested in. Keywords are matched case-insensitively against the job titles.
    ```javascript
    // CUSTOMIZE YOUR KEYWORDS HERE
    const keywords = [
      'software engineer',
      'python developer',
      'data analyst',
      'fresher',
      // Add more keywords here
    ];
    ```
* **Schedule:**
    * Open the **"Schedule Every 12 Hours"** trigger node.
    * Adjust the "Trigger Interval" settings to run the workflow more or less frequently (e.g., daily, every few hours).
* **Email Content:**
    * Open the **"Format Email"** Code node.
    * Modify the HTML structure and content within the `emailBody` variable to change the appearance or information included in the notification email. You can also change the `subject` line here.
* **Redis Key Prefix:**
    * If you use your Redis instance for other purposes, you might want to adjust the key prefix in the **"Store Job IDs"** node (currently `job_{{ $json.id }}`) and the corresponding filtering logic in the **"Find New Jobs"** node (`key.startsWith('job_')`).

## 🧠 Workflow Logic

1.  **Trigger:** The workflow starts based on the defined schedule (default: every 12 hours).
2.  **Fetch:** It simultaneously makes HTTP requests to the four tech park websites/APIs to retrieve the latest job listings. Special handling (cookies) is used for Cyberpark Kozhikode.
3.  **Parse:** Dedicated Code nodes parse the raw response (HTML or JSON) from each source into a standardized format (`id`, `title`, `company`, `closing_date`, `url`, `source`).
4.  **Merge:** All parsed job lists are combined into a single list.
5.  **Filter:** The combined list is filtered based on the keywords defined in the "Filter by Keywords" node.
6.  **Retrieve State:** The workflow fetches all keys currently stored in Redis (representing previously seen job IDs).
7.  **Identify New:** A Code node compares the list of currently filtered jobs against the list of stored IDs from Redis. Only jobs whose IDs are *not* in Redis are considered "new".
8.  **Check for New:** An IF node checks if any new jobs were found.
9.  **Notify (If New):**
    * The "Format Email" node creates an HTML email body summarizing the new jobs.
    * The "Send Email Notification" node sends the email via Gmail.
    * The "Store Job IDs" node takes the IDs of the *new* jobs and saves them back to Redis (prefixed with `job_`) so they won't trigger a notification on the next run.
10. **End (If No New):** If the IF node finds no new jobs, the workflow simply stops.

## ⚠️ Notes & Disclaimer

* **Website Changes:** Web scraping is inherently fragile. If any of the target websites change their structure or API endpoints, the corresponding fetch/parse nodes in this workflow may break and will need to be updated.
* **Error Handling:** Basic error handling is not explicitly implemented in detail. Consider adding error handling branches (e.g., using the "Error Trigger" or checking node outputs) for a more robust setup in production.
* **Rate Limiting/Blocking:** Running the workflow too frequently might lead to temporary IP blocks from the websites. The default 12-hour interval should generally be safe. Respect the websites' terms of service.
* **Redis Persistence:** Ensure your Redis instance has appropriate persistence configured if you need to retain the list of seen jobs across restarts.

## 🤝 Contributing

Contributions, issues, and feature requests are welcome. Feel free to check [issues page](link-to-issues-page-if-you-create-one) if you want to contribute.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details (You'll need to create this file and add the MIT license text).

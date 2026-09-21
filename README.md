# AI Email Triage & Auto-Drafting Workflow

An end-to-end n8n automation that acts as an executive assistant for your inbox. Triggered automatically every minute, this workflow scans unread emails, evaluates their importance using AI, and takes action to clear clutter or prepare responses.

## 🚀 Features

*   **Automated Polling:** Connects to your Gmail inbox via OAuth2 and fetches unread messages at 1-minute intervals.
*   **AI-Powered Triage:** Utilizes Google Gemini (flash-3.1) to analyze the email content. It generates a 2-sentence summary, identifies the core request, determines urgency (High, Medium, or Low), and writes a contextual draft reply.
*   **Intelligent Routing:** 
    *   *Low Urgency:* Immediately marks the email as read to keep your inbox clean.
    *   *Medium/High Urgency:* Automatically creates a professionally formatted draft reply within the exact email thread.
*   **Real-Time Discord Alerts:** Triggers specific Discord webhooks to send summary notifications to a private channel detailing the sender, subject, and whether the email was bypassed or a response was prepped.

## 🛠 Prerequisites

To run this workflow, you will need:
*   An active [n8n](https://n8n.io/) instance.
*   A Google Cloud project configured with the **Gmail API** (for OAuth2 credentials).
*   A **Google Gemini API Key** to power the AI triage node.
*   A Discord server with **Webhooks** enabled for notifications.

## 📦 Installation & Setup

1.  **Import the Workflow:** Download the `Gmail handler.json` file and import it directly into your n8n workspace.
2.  **Configure Credentials:** 
    *   Open the **Gmail Trigger**, **Create a draft**, and **Mark a message as read** nodes. Re-authenticate them using your own Gmail OAuth2 credentials.
    *   Open the **Message a model** node and input your Gemini API Key.
    *   Open the two **Discord** nodes and paste in your Discord Webhook URLs.
3.  **Activate:** Toggle the workflow to `Active` in the top right corner of the n8n interface to begin polling your inbox.

## 🧠 How the Code Node Works

Because LLMs often wrap JSON outputs in Markdown code blocks (e.g., ` ```json `), this workflow includes a custom JavaScript node to safely sanitize and parse the Gemini output before routing:

```javascript
for (const item of $input.all()) {
  let rawText = item.json.content.parts[0].text;
  let cleanText = rawText.replace(/```json/gi, '').replace(/```/gi, '').trim();
  let parsedData = JSON.parse(cleanText);
  Object.assign(item.json, parsedData);
}
return $input.all();

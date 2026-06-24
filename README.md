# real_estate-ai-agent-calling

# AI Voice Agent Lead Capture & Routing Workflow

## 📌 Overview
This repository contains the documentation and logic for an automated n8n workflow designed for a Real Estate AI Voice Agent. The system intercepts inbound calls via Twilio and Retell AI, automatically extracts the caller's country code, parses conversational data to determine property interest, and instantly logs the lead into a Google Calendar CRM.

## 🏗️ Architecture & Tech Stack
*   **Voice Engine:** Retell AI (Zara Agent)
*   **Telephony:** Twilio (with Call Forwarding for local UAE numbers)
*   **Automation:** n8n
*   **Database/CRM:** Google Calendar

## ⚙️ Workflow Breakdown

### 1. Webhook Trigger
*   Receives the POST request from Retell AI once a call concludes.
*   Captures the `from_number` (Caller ID) provided directly by the telecom network, ensuring 100% accuracy without needing to ask the user for their number.

### 2. Custom Code Node (Country Extraction)
*   Processes the raw `from_number`.
*   Uses a prefix-matching logic to determine the caller's geographic location.
*   **Logic Snippet:**
```javascript
    let phoneNumber = $input.item.json.body.call.from_number || ""; 
    let countryName = "Unknown";

    if (phoneNumber.startsWith("+91")) {
        countryName = "India";
    } else if (phoneNumber.startsWith("+971")) {
        countryName = "UAE - Dubai";
    } else if (phoneNumber.startsWith("+1")) {
        countryName = "USA/Canada";
    } else if (phoneNumber.startsWith("+44")) {
        countryName = "UK";
    }

    return {
        json: {
            ...$input.item.json,
            extracted_country: countryName
        }
    };
    ```

### 3. Structured Output Parser (AI Data Extraction)
*   Analyzes the call transcript to extract key business metrics.
*   Extracts `property_interest` (e.g., JVC, Downtown, Business Bay).
*   *Note: Phone number extraction is explicitly disabled here to prevent hallucinations, relying strictly on the telecom network's Caller ID.*

### 4. Google Calendar Integration (Lead Logging)
*   Acts as a lightweight, real-time CRM.
*   Creates a calendar event mapping the collected variables:
    *   **Event Title:** `New Lead: {{from_number}} ({{extracted_country}})`
    *   **Description:** Contains details about the specific property inquiry.
    *   **Timestamp:** Real-time event creation based on workflow trigger execution.

## 🚀 Setup Instructions
1. Import the workflow JSON file into your n8n instance.
2. Connect your Google Calendar credentials in the respective node.
3. Update the Webhook URL in your Retell AI Dashboard.
4. Ensure Twilio SIP Trunk Origination is properly pointing to the Retell URI.
5. Execute the Webhook node and trigger a test call to map the `from_number` variable dynamically.

## ⚠️ Important Notes
*   **Telecom Nuances:** If deploying for Dubai clients, use local Call Forwarding from a standard Etisalat/Du number to the Twilio number to bypass local virtual number restrictions.
*   **Agent Prompting:** The AI system prompt should not explicitly ask for phone numbers unless the user requests updates on an alternative WhatsApp number.
*

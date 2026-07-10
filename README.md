AI Lead Management System Pro

An end-to-end, AI-powered lead intake and CRM automation pipeline built with n8n, a local LLM (Ollama / Llama 3), and a lightweight landing page frontend. When a visitor submits an inquiry form, the workflow validates the data, classifies the lead using AI, logs it, notifies the team, and emails the client — all automatically.

✨ Features


🌐 Landing page frontend (index.html) with an animated lead-capture form
🔒 Input validation & sanitization (required fields, email format checks)
🤖 AI-based lead classification using a local LLM (Ollama running llama3) to score and categorize leads by urgency and budget
🧠 Automatic fallback scoring if the AI call fails or returns invalid output, so no lead is ever dropped
📧 Automated email notifications

Internal alert to the admin for every new lead
Confirmation email to the client on success
Friendly error email to the client if their submission fails validation



📊 Google Sheets logging — every lead is appended as a new row for tracking
📌 Trello card creation — each lead becomes a card on a Trello board for follow-up
⚡ Triggered instantly via an n8n Webhook


🏗️ Architecture / Workflow

Landing Page (index.html)
        │  (POST request)
        ▼
Webhook Trigger  ──▶  Validate Input  ──▶  Validation Check
                                                 │
                                    ┌────────────┴────────────┐
                                    ▼ (invalid)                ▼ (valid)
                            Validation Error              Set Lead Data
                            (400 response +                     │
                             client error email)                ▼
                                                            AI (Ollama / llama3)
                                                                 │
                                                            Parse AI Response
                                                                 │
                                                     ┌───────────┴───────────┐
                                                     ▼ (failed)              ▼ (success)
                                              Fallback Logic          (AI-scored data)
                                                     └───────────┬───────────┘
                                                                 ▼
                                                        Success Response (200)
                                                                 │
                                    ┌────────────────┬───────────┴───────────┬──────────────────┐
                                    ▼                ▼                       ▼                  ▼
                            Email: Admin      Email: Client          Google Sheets         Trello Card
                            (new lead alert)  (thank-you note)       (append row)          (create card)

📁 Project Structure

.
├── AI Lead Management System Pro - FINAL(1).json   # n8n workflow (import this into n8n)
├── index.html                                      # Landing page with lead capture form
└── README.md

🧰 Tech Stack

ComponentTechnologyWorkflow enginen8nAI / LLMOllama running llama3 (local inference)FrontendHTML, CSS, vanilla JavaScriptEmailGmail (n8n Gmail node)Data storageGoogle SheetsTask managementTrelloTunneling (dev)ngrok

🚀 Getting Started

Prerequisites


n8n installed and running (self-hosted, Docker, or n8n Cloud)
Ollama installed locally, with the llama3 model pulled:


bash  ollama pull llama3
  ollama serve


A Google account with Sheets API access (Service Account credentials)
A Gmail account connected to n8n (OAuth2)
A Trello account with an API key/token and a target list


1. Import the Workflow


Open your n8n instance.
Go to Workflows → Import from File.
Select AI Lead Management System Pro - FINAL(1).json.
The workflow will appear with all 15 nodes wired up.


2. Configure Credentials

In n8n, set up credentials for each service used by the workflow:

NodeCredential neededAdmin, Client, Client E (Gmail)Gmail OAuth2Append row in sheet1 (Google Sheets)Google Service AccountCreate a card (Trello)Trello API key & tokenAI (HTTP Request)None — calls local Ollama at http://host.docker.internal:11434/api/generate

Update the following node values to match your own setup:


Admin email in the Admin node (currently a placeholder address)
Google Sheet ID in Append row in sheet1
Trello list ID in Create a card


3. Activate the Webhook


Open the Webhook Trigger node and note the generated webhook URL (path: /webhook/lead-intake).
Activate the workflow so the webhook is live.
If testing locally, expose it with a tunneling tool such as ngrok:


bash   ngrok http 5678

4. Connect the Frontend

In index.html, update the fetch URL to point to your live webhook:

jsconst response = await fetch("https://YOUR_NGROK_OR_DOMAIN/webhook/lead-intake", { ... });

Then simply open index.html in a browser (or host it on any static site provider) to start capturing leads.

🔄 How a Lead Flows Through the System


A visitor fills out the form on the landing page and submits it.
The Webhook Trigger receives the POST request.
Validate Input checks required fields (name, email, requirement, budget) and email format.
Invalid submissions get a 400 response and an apology email; valid ones proceed.
Set Lead Data normalizes the payload and generates a unique lead_id.
The AI node sends the lead details to a local Llama 3 model, asking it to return a JSON classification (category, urgency, lead_score).
Parse AI safely parses the model's response. If parsing fails, Fallback Logic assigns a default score/category instead.
On success, the system:

Emails the admin with full lead details
Emails the client a thank-you confirmation
Appends a row to Google Sheets
Creates a Trello card for follow-up





🔐 Notes & Security


Replace hard-coded email addresses, sheet IDs, and Trello list IDs before deploying.
The AI node currently points to a local Ollama instance (host.docker.internal:11434) — update this URL if you host Ollama elsewhere or switch to a hosted LLM provider (OpenAI, Anthropic, etc.).
Consider adding webhook authentication (header/API key) before going to production, since the endpoint is currently open.

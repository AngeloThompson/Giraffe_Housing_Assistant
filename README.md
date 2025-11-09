# Giraffe Housing Assistant

[Click here to talk to the demo Giraffe Housing Assistant](https://t.me/Giraffe_Housing_Assistant_bot) 

**An Autonomous AI Agent for Your Housing Search**



## 🦒 The Problem

Finding a rental property, especially in competitive markets like the Netherlands, is a stressful, time-consuming, and manual process. You have to:
* Constantly monitor multiple websites.
* Apply to new listings immediately (within minutes).
* Manually track all your applications in a spreadsheet.
* Manage a flood of email replies.
* Schedule viewings in your calendar.

This friction means you can easily miss out on the perfect home.

## 💡 The Solution

The **Giraffe Housing Assistant** is an autonomous AI agent that takes over this entire end-to-end process. It acts as a true agent "on your behalf", not just a simple chatbot, fulfilling the hackathon's goal of building agents that can complete end-to-end tasks autonomously.

The agent manages the full lifecycle of your housing search: from finding listings and applying to them, to parsing email replies and booking viewings directly into your calendar.



## ✨ Core Features

* **🤖 Telegram Onboarding:** A simple Telegram bot identifies the user's intent to find housing and collects their preferences (city, budget, rooms, furnishment).
* **🕷️ Autonomous Scraping:** A scheduled workflow continuously scrapes housing sites (currently Pararius.nl) based on the user's saved preferences.
* **📩 Automated Applications:** The agent automatically "applies" to new listings that match the criteria and have not been applied to before.
* **🧠 LLM-Powered Email Classification:** The agent monitors your Gmail inbox. It uses an LLM (Google Gemini) to read and classify replies from landlords (e.g., `viewing: yes`, `viewing: no`, `viewing: pending_more_information`).
* **🗓️ Automatic Calendar Scheduling:** When a viewing is confirmed (`viewing: yes`), the agent automatically parses the date and time from the email and creates an event in your Google Calendar.
* **📊 Personal Monitoring Dashboard:** A unique web dashboard provides a real-time overview of all applications, their statuses (pending, scheduled, refused), and key metrics for your search.


## ⚙️ How It Works: The Agent Architecture

The assistant is built as a series of interconnected [n8n](https://n8n.io/) workflows that function as a cohesive agent.

1.  **Onboarding (`user_preferences_telegram.json`)**
    * A **Telegram Trigger** receives a message.
    * A **Gemini LLM** classifies the user's intent.
    * If the intent is to find housing, the bot asks for preferences (city, budget, etc.).
    * A **GPT-5 LLM** parses the user's free-text response into structured JSON.
    * This JSON is saved to a **Data Table** (`user_data`).

2.  **Scraping & Applying (`application_sender.json`)**
    * A **Schedule Trigger** (e.g., every 10 minutes) fetches all users from the `user_data` table.
    * For each user, a **GPT-5 LLM** generates a precise Pararius.nl search URL from their preferences.
    * An **HTTP Request** scrapes the search results page.
    * The agent loops through each listing, scrapes its individual page for details (like the contact link), and checks the `housing_application` data table to see if an application was already sent.
    * If it's a new listing, the agent "applies" by:
        1.  Sending a **Telegram Message** to the user with the listing link.
        2.  Sending an internal **Gmail** notification (simulating the application).
        3.  Logging the new application in the `housing_application` and `announcments` data tables.

3.  **Email & Calendar Automation (`mail_scraping.json`)**
    * A **Schedule Trigger** (e.g., every minute) fetches new emails from Gmail.
    * A **Gemini LLM** classifies the emails to identify viewing confirmations.
    * If a viewing is confirmed, a **Code Node** parses the date/time.
    * A **Google Calendar** node creates a new event for the viewing.

4.  **Monitoring (`web_dashboard.json`)**
    * A **Webhook** provides a unique URL for the user.
    * When accessed, it queries the `user_data`, `housing_application`, and `announcments` tables.
    * A **Code Node** merges this data and calculates metrics (total applications, refusal rate, etc.).
    * Another **Code Node** generates a dynamic **HTML page**, which is sent back to the user as the response.
  

## ⚙️ Setup & Installation

Follow these steps to get your own Giraffe Housing Assistant running.

### 1. Install and Run n8n

The entire agent logic runs on [n8n](https://n8n.io/), an open-source workflow automation tool.

* **Install n8n:** Follow the official installation instructions on the **[n8n GitHub repository](https://github.com/n8n-io/n8n)** or use their documentation for [Docker](https://docs.n8n.io/hosting/installation/docker/), [npm](https://docs.n8n.io/hosting/installation/npm/), or [n8n Cloud](https://n8n.io/cloud/).
* **Run n8n:** Once installed, start your n8n instance. By default, it will be accessible at `http://localhost:5678`.
* **Create Account:** Open `http://localhost:5678` in your browser and set up your owner account.

---

### 2. Initialize Ngrok (Reverse Proxy)

n8n webhooks (like the Telegram Trigger and Dashboard) need to be accessible from the public internet. [Ngrok](https://ngrok.com/) is a simple tool to create a secure reverse proxy.

1.  **Sign up & Install:** Create a free account at [ngrok.com](https://ngrok.com/) and follow their instructions to install the ngrok agent.
2.  **Connect Your Account:** Run the command they provide to connect your auth token (it looks like `ngrok config add-authtoken <YOUR_TOKEN>`).
3.  **Start Ngrok:** Run the following command to create a public URL that forwards to your local n8n instance.

    ```bash
    ngrok http 5678
    ```
4.  **Find Your URL:** Ngrok will show you a "Forwarding" URL, like `https://your-unique-id.ngrok-free.dev`. **Keep this URL.**
5.  **Configure n8n:**
    * In your n8n interface, go to **Settings > n8n > Webhook settings**.
    * Set the **Webhook base URL** to your ngrok URL (e.g., `https://your-unique-id.ngrok-free.dev/`).
    * Save the changes.

---

### 3. Create Credentials

In your n8n interface, go to the **Credentials** tab on the left. You will need to add credentials for all the services used.

* **Google (Gmail & Calendar):**
    1.  Go to the [Google Cloud Console](https://console.cloud.google.com/) and create a new project.
    2.  Enable the **Gmail API** and **Google Calendar API** for your project.
    3.  Go to **APIs & Services > Credentials**.
    4.  Create an **OAuth 2.0 Client ID** for a "Web application".
    5.  Add an **Authorized redirect URI**: `https://your-unique-id.ngrok-free.dev/rest/oauth2-credential/callback` (use your ngrok URL).
    6.  Copy the **Client ID** and **Client Secret**.
    7.  In n8n, add a **Google** credential (using OAuth2) and paste in your Client ID and Secret.
    8.  Connect your account. You will need one credential for **Gmail** and one for **Google Calendar** (or a single one with both scopes enabled).

* **Telegram:**
    1.  Talk to the **BotFather** in your Telegram app.
    2.  Create a new bot.
    3.  BotFather will give you an **Access Token**.
    4.  In n8n, add a **Telegram** credential and paste this token.

* **OpenAI:**
    1.  Go to the [OpenAI Platform](https://platform.openai.com/api-keys).
    2.  Create a new **API Key**.
    3.  In n8n, add an **OpenAI** credential and paste this key.

* **Google Gemini:**
    1.  Go to the [Google AI Studio](https://aistudio.google.com/app/apikey).
    2.  Create an **API Key**.
    3.  In n8n, add a **Google Gemini(PaLM) Api** credential and paste this key.

---

### 4. Create Data Tables

n8n's internal Data Tables are used to store all agent state. Go to the **Data Tables** tab on the left and create the following three tables.

* **Table 1: `user_data`**
    * `user_id` (string)
    * `chat_id` (string)
    * `max_price` (number)
    * `min_price` (number)
    * `rooms` (number)
    * `furnished` (number)
    * `location` (string)

* **Table 2: `announcments`**
    * `announcment_id` (string)
    * `price` (number)
    * `location` (string)
    * `image_url` (string)
    * `announcment_url` (string)

* **Table 3: `housing_application`**
    * `user_id` (string)
    * `announcment_id` (string)
    * `requent_sent` (boolean)

---

### 5. Load and Activate Workflows

1.  **Import Workflows:** Go to the **Workflows** tab. Click **Import > From File** and import each of the 4 JSON files you downloaded:
    * `user_preferences_telegram.json`
    * `application_sender.json`
    * `web_dashboard.json`
    * `mail_scraping.json`

2.  **Connect Credentials & Tables:** Open each workflow and link the nodes to the credentials and data tables you created.
    * **Example:** In `application_sender.json`, click the "Get row(s)" node. In the "Data Table" field, select `user_data` from the dropdown.
    * Do this for **all nodes** that reference a credential (Gmail, Telegram, OpenAI, Calendar) or a Data Table.

3.  **Activate Workflows:** Once all nodes are correctly configured, click the **Active** toggle in the top-right corner for all four workflows.

Your agent is now live! You can start by sending a message to your Telegram bot.



## 🚀 Future Roadmap

This project successfully demonstrates the core "end-to-end" autonomous loop. The next steps to expand its capabilities are:

* **Add More Housing Websites:** Integrate scrapers for other major platforms like Funda.nl, Kamernet, and region-specific housing portals to broaden the search.
* **Expand to Different Countries:** Adapt the scrapers and preference models to support housing markets in other countries, making it a truly international assistant.
* **Full Household Management:** Evolve the agent beyond just *finding* a house.
    * **Utilities Setup:** Once a lease is signed, the agent could automatically research and find the best (cheapest/greenest) utility providers for that address (e.g., for gas, electricity, water).
    * **Internet Providers:** Compare internet provider plans (speed, price) available at the new address and present the top 3 options.
    * **Proactive Bill Optimization:** Grant the agent access to your email to find existing utility and internet bills. The agent will then proactively search online for better offers and present them to you for easy switching.
    * **Move-in Coordination:** Help schedule moving companies or appointments for key handovers.
 


This project was built for the **AI University Games Hackathon (November 8th-9th, 2025)**, powered by Prosus and AISO. The challenge was to create an autonomous, agentic AI lifestyle assistant that solves a real-world problem in daily life.

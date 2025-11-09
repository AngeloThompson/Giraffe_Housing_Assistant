# Giraffe Housing Assistant

**An Autonomous AI Agent for Your Housing Search**

This project was built for the **AI University Games Hackathon (November 8th-9th, 2025)**, powered by Prosus and AISO. The challenge was to create an autonomous, agentic AI lifestyle assistant that solves a real-world problem in daily life.

## 🦒 The Problem

Finding a rental property, especially in competitive markets like the Netherlands, is a stressful, time-consuming, and manual process. You have to:
* Constantly monitor multiple websites.
* Apply to new listings immediately (within minutes).
* Manually track all your applications in a spreadsheet.
* Manage a flood of email replies.
* Schedule viewings in your calendar.

This friction means you can easily miss out on the perfect home.

## 💡 The Solution

The **Giraffe Housing Assistant** is an autonomous AI agent that takes over this entire end-to-end process. [cite_start]It acts as a true agent "on your behalf", not just a simple chatbot, fulfilling the hackathon's goal of building agents that can complete end-to-end tasks autonomously[cite: 12, 14].

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

1.  **Onboarding (`[DO_NOT_TOUCH]user_preferences_telegram.json`)**
    * A **Telegram Trigger** receives a message.
    * A **Gemini LLM** classifies the user's intent.
    * If the intent is to find housing, the bot asks for preferences (city, budget, etc.).
    * A **GPT-5 LLM** parses the user's free-text response into structured JSON.
    * This JSON is saved to a **Data Table** (`user_data`).

2.  **Scraping & Applying (`[DO_NOT_TOUCH]application_sender.json`)**
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

4.  **Monitoring (`[DO_NOT_TOUCH]web_dashboard.json`)**
    * A **Webhook** provides a unique URL for the user.
    * When accessed, it queries the `user_data`, `housing_application`, and `announcments` tables.
    * A **Code Node** merges this data and calculates metrics (total applications, refusal rate, etc.).
    * Another **Code Node** generates a dynamic **HTML page**, which is sent back to the user as the response.

## 🚀 Future Roadmap

This project successfully demonstrates the core "end-to-end" autonomous loop. The next steps to expand its capabilities are:

* **Add More Housing Websites:** Integrate scrapers for other major platforms like Funda.nl, Kamernet, and region-specific housing portals to broaden the search.
* **Expand to Different Countries:** Adapt the scrapers and preference models to support housing markets in other countries, making it a truly international assistant.
* **Full Household Management:** Evolve the agent beyond just *finding* a house.
    * **Utilities Setup:** Once a lease is signed, the agent could automatically research and find the best (cheapest/greenest) utility providers for that address (e.g., for gas, electricity, water).
    * **Internet Providers:** Compare internet provider plans (speed, price) available at the new address and present the top 3 options.
    * **Move-in Coordination:** Help schedule moving companies or appointments for key handovers.

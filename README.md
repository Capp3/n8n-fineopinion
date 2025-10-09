# Welcome to FineOpinion

**An Opiionated Financial Newsletter**

## Project Goal

To create a low-maintenance, automated workflow using n8n that consumes information from a small set of curated news feeds, stores and processes the content through a multi-agent AI system, and produces concise daily and weekly reports. The objective is to provide a high-level, "need-to-know" overview of economic and financial news without the noise of high-volume sources.

### High-Level Workflow

A scheduled n8n workflow will trigger twice daily, ingesting articles from selected RSS feeds. The content will be stored, processed by a primary LLM agent to create a daily digest, and then a secondary agent will compile these digests into a weekly analytical report delivered via email.

### Workflow Architecture & Technical Specification

#### Trigger & Schedule

**Mechanism:** The n8n workflow will be initiated by a Scheduled Trigger Node.

**Frequency:** The workflow will run twice daily (e.g., 7:00 AM and 7:00 PM) to capture morning and evening news cycles.

**Staggered Ingestion:** To avoid overwhelming sources or hitting rate limits, reads from the three feeds will be staggered within the workflow (e.g., run at T, T+1min, T+2min).

#### Data Ingestion & Parsing

**Primary Method:** The workflow will use n8n's RSS Feed Read Node to pull data from the source URLs.

#### Content Scraping (Contingency)

A critical consideration is that RSS feeds often contain only summaries. A secondary step will be required to fetch the full article content.

**Initial Approach:** Use the HTTP Request Node to fetch the article URL provided in the RSS feed.

**Robust Approach (If Needed):** If simple HTTP requests are blocked or fail to render content, a more robust solution using a browser automation tool like Playwright or Puppeteer will be implemented to scrape the full, rendered article text. This will be a separate module in the n8n workflow.

### Data Storage

**Requirement:** A persistent storage solution is needed to hold ingested articles and the generated daily digests for the weekly report.

**Options:**

- **Airtable:** Recommended for simplicity, rapid setup, and ease of use. Ideal for storing structured data like article URL, title, source, ingested date, raw text, and daily digest summary.

- **PostgreSQL:** A more powerful and scalable solution. Recommended if the project is expected to grow significantly, store years of data, or require complex querying.

**Decision Point:** The choice of database will be a key first step in the build process.

### Agent-Based Processing & Summarization

Prompt Engineering: Significant effort will be dedicated to creating robust and well-defined prompts for each agent to ensure consistent, high-quality output.

#### Agent 1: Daily Digest Creator

**Task:** This LLM agent will process all new articles ingested in a 12-hour cycle.

**Output:** It will generate a single, coherent summary (the "Daily Digest") of the key findings, trends, and news.

**Storage:** The resulting digest will be stored in the selected database (Airtable/Postgres), linked to the source articles.

#### Agent 2: Weekly Report Analyst

**Task:** Once a week, this agent will be triggered. It will read all the Daily Digests from the past seven days.

**Output:** It will create a higher-level analytical report, identifying recurring themes, connecting disparate events, and providing a summary of the week's economic and financial narrative.

**Delivery:** The final report will be formatted and sent to the project owner via the Send Email Node.

### Data Retention Policy

- **Raw Articles:** A policy will be established to purge raw article data after a set period (e.g., 30 days) to manage storage size.

- **Digests:** Daily and Weekly digests should be retained for a longer period (e.g., 1 year) for trend analysis.

## Selected Information Sources & Rationale

(This section remains unchanged, as the sources are still the core input)

### Source 1: The Broad Economic & Geopolitical Context

**Feed:** The Economist: The World in Brief

**URL:** https://www.economist.com/the-world-in-brief/rss.xml

**Reasoning:** Provides the essential "why" behind market movements.

### Source 2: The Key Financial Market Snapshot

**Feed:** Bloomberg: Five Things to Know to Start Your Day

**URL:** https://assets.bwbx.io/av/podcasts/daybreak/rss.xml

**Reasoning:** Provides the "what's happening now" market drivers.

### Source 3: The Unbiased, Factual News Wire

**Feed:** Reuters: Top News

**URL:** https://www.reuters.com/news/rss/topNews

**Reasoning:** Provides the "ground truth" with unbiased, factual reporting.

### Source 4: Market-Focused Business & Investment News

**Feed:** MarketWatch: Top Stories

**URL:** https://feeds.content.dowjones.io/public/rss/mw_topstories

**Reasoning:** Provides up-to-the-minute market movements, investment insights, and business news tailored for investors and traders.

## Success Criteria

The project will be a success when the workflow reliably and autonomously:

- Ingests, parses, and stores articles without manual intervention.

- Generates a coherent Daily Digest that requires no more than 3-5 minutes of reading.

- Delivers a concise, analytical Weekly Report via email every week.

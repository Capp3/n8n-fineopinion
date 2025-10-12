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

### Source 1: European Central Bank (ECB)

Feed: https://www.ecb.europa.eu/rss/press.xml
Focus: Official economic outlooks, monetary policy decisions, speeches, and statistics.
Why it’s valuable: ECB releases are authoritative and timely, making them ideal for understanding macroeconomic policy drivers and European market sentiment.​

### Source 2: MarketWatch

Feed: https://www.marketwatch.com/rss/topstories
Focus: Key global market summaries, major earnings, and financial trends.
Why it’s valuable: MarketWatch provides concise, accessible coverage without excessive volume, focusing on the most impactful daily developments.​

### Source 3: Nasdaq Original Articles

Feed: https://www.nasdaq.com/feed/nasdaq-originals
Focus: Stock market reports, investor insights, and forward-looking commentary.
Why it’s valuable: Includes market sentiment and trend analysis, ideal for summarizing daily equity and sector movements.​

### Source 4: BNP Paribas Economic Research

Feed: https://economic-research.bnpparibas.com/RSS/en-US
Focus: Economic analysis and data-driven research from BNP economists.
Why it’s valuable: Offers in-depth perspectives on global and regional trends — great for enhancing the analytical content of a weekly report.​

### Source 5:Finance Monthly

Feed: https://www.finance-monthly.com/feed
Focus: Global business finance, sustainability, and fintech coverage.
Why it’s valuable: Publication-grade content summarizing corporate and policy news, with low posting frequency suitable for summaries and AI processing.​

### Source 6: CNBC World Business

Feed: https://www.cnbc.com/id/100003114/device/rss/rss.html
Focus: Business, macroeconomics, and market movement summaries.
Why it’s valuable: A globally recognized financial source offering factual, up-to-date insight into both Western and Asian markets.​

### Source 7

Money RSS Feed: https://money.com/money/feed — accessible personal finance and economy coverage

## Success Criteria

The project will be a success when the workflow reliably and autonomously:

- Ingests, parses, and stores articles without manual intervention.

- Generates a coherent Daily Digest that requires no more than 3-5 minutes of reading.

- Delivers a concise, analytical Weekly Report via email every week.

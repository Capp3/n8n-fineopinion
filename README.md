# Welcome to FineOpinions

**An Opinionated Financial Newsletter with Actual Personality** 🍀

**Current Status:** RSS Post-Processing Architecture Complete ✅ - Ready for Implementation  
**Last Updated:** October 25, 2025  
**Quick Start:** Read `/START-HERE.md` for 15-minute overview

---

## 🎯 Project Goal

Create an automated financial news digest delivered **every other day** (48-hour cycle) that provides:

- **Trustworthy facts** (wire service journalism standards)
- **Actual personality** (Northern Irish wit, edgy humor)
- **Accessible language** (for intelligent non-experts)
- **Enjoyable reading** (5-6 minutes of financial news you'll actually want to read)

**Think:** The Economist meets your smartest, funniest mate from Belfast explaining economics over pints.

---

## 🎭 Multi-Agent Architecture

FineOpinions uses a **4-agent AI pipeline** inspired by traditional newsrooms:

```
RSS Feeds → Content Scraper → Desk Reporter → Journalist → Editorial Writer → Copywriter → Email
```

### Agent 1: Desk Reporter

- Processes individual articles (4x per 48 hours)
- Extracts facts, scores relevance (1-10 scale)
- Filters noise (threshold: relevance >= 4)
- **Model:** llama3.2:3b (fast) or qwen2.5:7b (quality)

### Agent 2: Journalist (FACTS FIRST)

- Synthesizes top 25 articles into factual narrative
- Wire service style - NO opinion, NO speculation
- Uses Wikipedia tool for fact verification
- **Model:** qwen2.5:7b
- **Output:** 500-700 words of pure facts

### Agent 3: Editorial Writer (CHARACTER & FUN)

- Adds perspective, personality, makes it ENJOYABLE
- Northern Irish voice: smart, sardonic, edgy humor
- Explains WHY it matters in plain language
- Gallows humor allowed, calls out bullshit
- **Model:** qwen2.5:7b or qwen2.5:14b
- **Voice Guide:** 60+ examples for consistency

### Agent 4: Copywriter

- Final polish and HTML formatting
- Magazine-style email structure
- Edgy subject lines (match Editorial voice)
- **Model:** qwen2.5:7b
- **Output:** 750-1000 word HTML email

---

## 📅 Delivery Schedule

**RSS Processing:** Twice daily (7AM & 7PM)  
**Digest Delivery:** Every other day (6AM)  
**Cycle:** 48 hours of news per digest

**Example:**

- Days 1-2: Collect and process articles (4 RSS pulls)
- Day 3, 6AM: Generate and send digest
- Days 4-5: Collect and process articles
- Day 6, 6AM: Generate and send digest

**Result:** ~3 digests per week (not overwhelming, comprehensive coverage)

---

## 🔑 Core Design Philosophy

**FACTS FIRST, CHARACTER SECOND**

The multi-agent workflow separates facts from opinion:

- 🗞️ **Journalist** = FACTS ONLY (objective, no speculation)
- ✍️ **Editorial** = CHARACTER & FUN (perspective, wit, personality)

**This ensures:**

- Factual integrity (trust the facts)
- Editorial freedom (enjoy the perspective)
- Clear separation (readers know what's what)
- Modularity (can adjust tone independently)

---

## 🛠️ Technical Stack

### Infrastructure

- **n8n:** v1.114.4 (self-hosted workflow automation)
- **LLM:** Ollama (local inference)
  - llama3.2:3b (fast processing)
  - qwen2.5:7b (quality synthesis)
- **Database:** Airtable (2 tables, 40+ fields)
- **Email:** n8n Send Email Node (HTML)

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
Why it's valuable: ECB releases are authoritative and timely, making them ideal for understanding macroeconomic policy drivers and European market sentiment.​

### Source 2: MarketWatch

Feed: https://www.marketwatch.com/rss/topstories
Focus: Key global market summaries, major earnings, and financial trends.
Why it's valuable: MarketWatch provides concise, accessible coverage without excessive volume, focusing on the most impactful daily developments.​

### Source 3: Nasdaq Original Articles

Feed: https://www.nasdaq.com/feed/nasdaq-original/rss.xml
Focus: Stock market reports, investor insights, and forward-looking commentary.
Why it's valuable: Includes market sentiment and trend analysis, ideal for summarizing daily equity and sector movements.​

### Source 4: BNP Paribas Economic Research

Feed: https://economic-research.bnpparibas.com/RSS/en-US/Eco-Insight
Focus: Economic analysis and data-driven research from BNP economists.
Why it's valuable: Offers in-depth perspectives on global and regional trends — great for enhancing the analytical content of a weekly report.​

### Source 5:Finance Monthly

Feed: https://www.finance-monthly.com/feed
Focus: Global business finance, sustainability, and fintech coverage.
Why it's valuable: Publication-grade content summarizing corporate and policy news, with low posting frequency suitable for summaries and AI processing.​

### Source 6: CNBC World Business

Feed: https://www.cnbc.com/id/100003114/device/rss/rss.html
Focus: Business, macroeconomics, and market movement summaries.
Why it's valuable: A globally recognized financial source offering factual, up-to-date insight into both Western and Asian markets.​

### Source 7

Money RSS Feed: https://money.com/money/feed — accessible personal finance and economy coverage

## Success Criteria

The project will be a success when the workflow reliably and autonomously:

- Ingests, parses, and stores articles without manual intervention.

- Generates a coherent Daily Digest that requires no more than 3-5 minutes of reading.

- Delivers a concise, analytical Weekly Report via email every week.

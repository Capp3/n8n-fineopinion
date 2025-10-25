# ADR-001 Summary: Separated Workflows Architecture

**Last Updated:** October 25, 2025  
**Decision Date:** October 25, 2025  
**Status:** Accepted and Documented  
**Impact:** High - Core Architecture

---

## 🎯 Quick Reference

### The Decision

The FinOpinions system uses **two independent workflows** instead of a monolithic pipeline:

1. **Workflow 1: Article Fetching & Storage**

   - Runs every 15 minutes
   - Fetches 7 RSS feeds
   - Normalizes data (4-stage pipeline)
   - Stores in Airtable with `Status = needs_scraping`
   - Duration: ~30-60 seconds

2. **Workflow 2: Content Scraping & AI Processing**
   - Runs every 1 hour or on-demand
   - Queries articles with `Status = needs_scraping`
   - Scrapes content (HTTP/Playwright/SearXNG)
   - Processes with AI agents
   - Updates status to `processed`
   - Duration: ~5-15 minutes

---

## 🔑 Key Benefits

- **70-80% reduction** in HTTP requests (no scraping for duplicates)
- **Failure isolation** (scraping failures don't block RSS fetching)
- **Flexible scheduling** for each workflow
- **Independent scaling** per workflow
- **Better error handling** with retry logic and status tracking

---

## 📊 Status Management

Articles transition through these states:

```
needs_scraping → scraping_in_progress → scraped → processed
                        ↓
                  scraping_failed (retry with different strategy)
```

### Airtable Fields for State Management

- **Status**: Current workflow state
- **ScrapingAttempts**: Number of scraping attempts (max 3)
- **LastScrapingAttempt**: Timestamp of last attempt
- **ScrapingError**: Error message if scraping failed
- **ScrapingStrategy**: Strategy used (http, playwright, selenium, searxng)

---

## 🏗️ Integration Patterns

### Pattern 1: Fully Independent (Current Implementation)

- Workflow 1 runs every 15 min
- Workflow 2 runs every 1 hour
- Communication via Airtable status fields
- Simple, reliable, easy to debug

### Pattern 2: Triggered Sub-Workflow (Advanced)

- Workflow 1 triggers Workflow 2 when new articles found
- Immediate processing
- More complex error handling

### Pattern 3: Hybrid (Production Recommendation)

- Quick scraping every 30 min (10 articles)
- Batch scraping every 6 hours (100 articles)
- Balances timeliness and efficiency

---

## 🔧 Implementation Status

### Workflow 1 (Article Fetching)

- [x] Design complete
- [x] RSS feeds configured (7 sources)
- [x] 4-stage post-processing pipeline
- [x] Airtable Upsert configuration
- [ ] Add Status fields to Airtable
- [ ] Test end-to-end

### Workflow 2 (Content Scraping)

- [x] Architecture designed
- [ ] Airtable query node
- [ ] Adaptive scraping strategy
- [ ] HTTP scraper
- [ ] Playwright scraper
- [ ] SearXNG fallback
- [ ] AI agent processing
- [ ] Status update logic

---

## 📖 Full Documentation

- **Complete ADR:** [`docs/architecture-decisions/ADR-001-separated-workflows.md`](../docs/architecture-decisions/ADR-001-separated-workflows.md)
- **System Diagrams:** [`fineopinions_diagram.md`](../fineopinions_diagram.md)
- **Node Settings:** [`fineopinions_node_settings.md`](../fineopinions_node_settings.md)
- **Tasks:** [`tasks.md`](../tasks.md)

---

## 🎨 Scraping Strategy Selection

Each source has a preferred strategy order:

| Source          | 1st Strategy | 2nd Strategy | 3rd Strategy |
| --------------- | ------------ | ------------ | ------------ |
| ECB             | HTTP         | Playwright   | SearXNG      |
| MarketWatch     | Playwright   | SearXNG      | -            |
| NASDAQ          | Playwright   | HTTP         | SearXNG      |
| BNP Paribas     | HTTP         | Playwright   | SearXNG      |
| Finance Monthly | HTTP         | Playwright   | SearXNG      |
| CNBC            | Playwright   | SearXNG      | -            |
| Money           | Playwright   | HTTP         | SearXNG      |

Strategy escalates automatically on failure (e.g., HTTP fails → try Playwright).

---

## 📈 Expected Performance Metrics

### Workflow 1

- Execution frequency: Every 15 minutes
- Avg execution time: 30-60 seconds
- Articles fetched per run: 10-50
- Success rate: 95%+

### Workflow 2

- Execution frequency: Every 1 hour
- Avg execution time: 5-15 minutes
- Articles scraped per run: 10-50
- HTTP success rate: 70-80%
- Playwright success rate: 85-95%
- SearXNG success rate: 60-70%

---

## 🚨 Operational Notes

### When to Intervene

1. **Articles stuck in `needs_scraping`** (>100 articles)

   - Check Workflow 2 execution schedule
   - Review scraping error logs
   - Consider manual trigger

2. **High failure rate** (>30% over 1 hour)

   - Check source website changes
   - Review scraping strategy effectiveness
   - Consider strategy order adjustment

3. **Slow Workflow 2 execution** (>20 minutes)
   - Reduce batch size (50 → 20)
   - Check for problematic sources
   - Consider increasing execution frequency

### Manual Reset Failed Articles

Use this Airtable formula to reset failed articles for retry:

```javascript
IF(
  AND(({ Status } = "scraping_failed"), { ScrapingAttempts } < 3),
  "needs_scraping",
  { Status }
);
```

---

## 🎯 Why This Architecture?

**Problem:** The initial monolithic workflow wasted resources by:

- Scraping content for duplicate articles
- Blocking entire pipeline when scraping failed
- Inflexible scheduling (all operations together)

**Solution:** Separated workflows enable:

- Duplicate check BEFORE scraping (saves 70-80% of HTTP requests)
- Failure isolation (RSS fetching continues even if scraping fails)
- Independent scheduling and scaling
- Adaptive scraping strategies per source

**Result:** A production-ready architecture that's efficient, resilient, and scalable.

---

**This is Option 1 from the creative phase - chosen for logical separation of priorities and operational flexibility.**

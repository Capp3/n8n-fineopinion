# Airtable Fields - Quick Reference Table

**Base:** FineOpinions  
**Table:** Articles  
**Total Fields:** 36  
**Architecture:** Separated Workflows (ADR-001)

---

## 📋 Complete Field List

| # | Field Name | Field Type | Required | Unique | Description |
|---|------------|------------|----------|--------|-------------|
| **IDENTIFICATION FIELDS** |
| 1 | ArticleID | Auto Number | Yes | Yes | Primary key (auto-generated) |
| 2 | URL | URL | Yes | Yes | Article link |
| 3 | URLHash | Single Line Text | Yes | Yes | Hash for deduplication (djb2) |
| 4 | GUID | Single Line Text | No | No | RSS feed GUID |
| 5 | Source | Single Select | Yes | No | RSS source (7 options) |
| **CONTENT FIELDS** |
| 6 | Headline | Single Line Text | Yes | No | Article title from RSS |
| 7 | ContentSnippet | Long Text | No | No | Brief excerpt from RSS |
| 8 | FullText | Long Text | No | No | Complete scraped content |
| 9 | Summary | Long Text | No | No | AI-generated summary |
| 10 | KeyFacts | Long Text | No | No | JSON array of key facts |
| 11 | DataPoints | Long Text | No | No | JSON array of critical data |
| **SOURCE & DATE FIELDS** |
| 12 | PubDate | Date | Yes | No | Original publication date/time |
| 13 | FetchedAt | Date | Yes | No | When RSS was fetched |
| 14 | ProcessedAt | Date | No | No | When AI processing completed |
| 15 | Creator | Single Line Text | No | No | Article author from RSS |
| **STATUS MANAGEMENT FIELDS** |
| 16 | Status | Single Select | Yes | No | Workflow state (6 options) |
| 17 | ScrapingAttempts | Number | Yes | No | Number of scraping attempts (max 3) |
| 18 | LastScrapingAttempt | Date | No | No | Timestamp of last attempt |
| 19 | ScrapingError | Long Text | No | No | Error message if failed |
| 20 | ScrapingStrategy | Single Select | No | No | Which method succeeded (3 options) |
| **AI ANALYSIS FIELDS** |
| 21 | MainTopic | Single Line Text | No | No | Primary topic/theme |
| 22 | SubTopics | Long Text | No | No | JSON array of related topics |
| 23 | EntitiesJSON | Long Text | No | No | JSON: companies, people, locations |
| 24 | Sentiment | Single Select | No | No | Overall sentiment (3 options) |
| 25 | MarketImpact | Single Select | No | No | Potential impact level (3 options) |
| 26 | Urgency | Single Select | No | No | Time sensitivity (3 options) |
| 27 | RelevanceScore | Number | Yes | No | 1-10 score for digest inclusion |
| 28 | Tags | Long Text | No | No | Comma-separated topic tags |
| 29 | Categories | Long Text | No | No | RSS feed categories |
| 30 | ProcessedBy | Single Line Text | No | No | Model used for AI processing |
| **PROCESSING METADATA** |
| 31 | WordCount | Number | Yes | No | Article word count |
| 32 | TokenCount | Number | No | No | Estimated LLM tokens used |
| 33 | IncludeInDigest | Checkbox | No | No | Flag for digest inclusion |
| **DIGEST INTEGRATION** |
| 34 | LinkedDigest | Link to Table | No | No | Links to Digests table |

---

## 🎯 Single Select Options

### Source (Field 5)
`ecb`, `marketwatch`, `nasdaq`, `bnpparibas`, `financemonthly`, `cnbc`, `money`

### Status (Field 16)
`needs_scraping`, `scraping_in_progress`, `scraped`, `scraping_failed`, `processed`, `digest_included`

### ScrapingStrategy (Field 20)
`http`, `playwright`, `searxng`

### Sentiment (Field 24)
`positive`, `neutral`, `negative`

### MarketImpact (Field 25)
`low`, `medium`, `high`

### Urgency (Field 26)
`routine`, `timely`, `breaking`

---

## 📊 Field Usage by Workflow

### Workflow 1 (RSS Fetching) - 13 fields
1, 2, 3, 4, 5, 6, 7, 12, 13, 15, 16, 17, 27, 31, 32

### Workflow 2 (Content Scraping & AI) - 23 fields
8, 9, 10, 11, 14, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33

### Workflow 3 (Journalist Agent) - 2 fields
16, 34

---

## ⚙️ Field Configuration Notes

- **Date fields** (12, 13, 14, 18): Include time, 24-hour format
- **Number fields** (17, 27, 31, 32): Integer, no negatives
- **JSON fields** (10, 11, 22, 23): Store as text strings, parse with JSON.parse()
- **Unique fields** (1, 2, 3): Prevent duplicates
- **Link field** (34): Links to Digests table (create later)

---

**Quick Setup:** Create fields 1-33 first, add field 34 when you create the Digests table.

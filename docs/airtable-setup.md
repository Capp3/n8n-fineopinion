# Airtable Setup Guide - FineOpinions

**Last Updated:** October 25, 2025  
**Status:** ✅ Updated for Separated Workflows Architecture (ADR-001)  
**Prerequisites:** Airtable account, basic familiarity with Airtable

---

## 📋 Overview

This guide walks through setting up the Airtable database for FineOpinions with the **Separated Workflows Architecture (ADR-001)**. The system uses **2 tables**: Articles (for individual article storage) and Digests (for weekly summaries).

**Architecture:** Two independent workflows manage article processing:

- **Workflow 1:** RSS Fetching & Storage
- **Workflow 2:** Content Scraping & AI Processing

---

## 🎯 Quick Setup Checklist

- [ ] Create new Airtable base named "FineOpinions"
- [ ] Create Articles table (36 fields organized in 7 categories)
- [ ] Create Digests table (13 fields)
- [ ] Configure field types and validation
- [ ] Set up API integration
- [ ] Test with sample data

**Estimated time:** 30-45 minutes

---

## 📊 Table 1: Articles

### Purpose

Store individual articles from all RSS sources with full lifecycle tracking from RSS fetch through AI processing.

### Field Categories Overview

1. **Identification Fields** (5 fields) - Primary keys and unique identifiers
2. **Content Fields** (6 fields) - Article content at various stages
3. **Source & Date Fields** (4 fields) - Origin and timing information
4. **Status Management Fields** (5 fields) - Workflow state tracking (ADR-001)
5. **AI Analysis Fields** (10 fields) - Desk Reporter agent outputs
6. **Processing Metadata** (5 fields) - System tracking and metrics
7. **Digest Integration** (1 field) - Link to final digest output

**Total:** 36 fields

---

### 1️⃣ IDENTIFICATION FIELDS (5 fields)

| Field Name    | Field Type       | Description                             | Required | Indexed | Unique |
| ------------- | ---------------- | --------------------------------------- | -------- | ------- | ------ |
| **ArticleID** | Auto Number      | Primary key (auto-generated)            | Yes      | Yes     | Yes    |
| **URL**       | URL              | Article link                            | Yes      | Yes     | Yes    |
| **URLHash**   | Single Line Text | Hash for fast deduplication (djb2 hash) | Yes      | Yes     | Yes    |
| **GUID**      | Single Line Text | RSS feed GUID (if available)            | No       | No      | No     |
| **Source**    | Single Select    | RSS source identifier                   | Yes      | Yes     | No     |

**Source Options:** `ecb`, `marketwatch`, `nasdaq`, `bnpparibas`, `financemonthly`, `cnbc`, `money`

**Why URLHash?** Faster searching than full URL text, deterministic (same URL = same hash), used for Upsert merge field.

---

### 2️⃣ CONTENT FIELDS (6 fields)

| Field Name         | Field Type       | Description                          | Required | Notes                   |
| ------------------ | ---------------- | ------------------------------------ | -------- | ----------------------- |
| **Headline**       | Single Line Text | Article title from RSS feed          | Yes      | Max ~200 chars          |
| **ContentSnippet** | Long Text        | Brief excerpt from RSS feed          | No       | From RSS description    |
| **FullText**       | Long Text        | Complete scraped article content     | No       | Populated in Workflow 2 |
| **Summary**        | Long Text        | AI-generated summary (2-3 sentences) | No       | Desk Reporter output    |
| **KeyFacts**       | Long Text        | JSON array of key facts              | No       | Store as JSON string    |
| **DataPoints**     | Long Text        | JSON array of critical numbers/data  | No       | Store as JSON string    |

**Content Evolution:**

```
RSS Fetch: Headline + ContentSnippet
  ↓
Scraping: + FullText
  ↓
AI Processing: + Summary + KeyFacts + DataPoints
```

---

### 3️⃣ SOURCE & DATE FIELDS (4 fields)

| Field Name      | Field Type       | Description                       | Required | Include Time |
| --------------- | ---------------- | --------------------------------- | -------- | ------------ |
| **PubDate**     | Date             | Original publication date/time    | Yes      | Yes          |
| **FetchedAt**   | Date             | When RSS was fetched (Workflow 1) | Yes      | Yes          |
| **ProcessedAt** | Date             | When AI processing completed      | No       | Yes          |
| **Creator**     | Single Line Text | Article author (from RSS)         | No       | N/A          |

**Date Format:** ISO 8601 without milliseconds (e.g., `2025-10-25T14:30:00Z`)  
**Timezone:** UTC (all timestamps)

---

### 4️⃣ STATUS MANAGEMENT FIELDS (5 fields) 🆕 ADR-001

| Field Name              | Field Type    | Description                         | Required | Default Value    |
| ----------------------- | ------------- | ----------------------------------- | -------- | ---------------- |
| **Status**              | Single Select | Current workflow state              | Yes      | `needs_scraping` |
| **ScrapingAttempts**    | Number        | Number of scraping attempts (max 3) | Yes      | 0                |
| **LastScrapingAttempt** | Date          | Timestamp of last scraping attempt  | No       | null             |
| **ScrapingError**       | Long Text     | Error message if scraping failed    | No       | empty            |
| **ScrapingStrategy**    | Single Select | Which scraping method succeeded     | No       | null             |

**Status Options (Workflow Lifecycle):**

- `needs_scraping` - New article from RSS, ready for Workflow 2
- `scraping_in_progress` - Currently being scraped (optional state)
- `scraped` - Content retrieved, ready for AI processing
- `scraping_failed` - Scraping failed after 3 attempts
- `processed` - ✅ Fully processed by AI, ready for digest
- `digest_included` - Included in a published digest

**ScrapingStrategy Options:** `http`, `playwright`, `searxng`

**Why This Matters:** These fields enable the two workflows to operate independently. Workflow 1 sets `Status = needs_scraping`, Workflow 2 queries for that status.

---

### 5️⃣ AI ANALYSIS FIELDS (10 fields)

| Field Name         | Field Type       | Description                                      | Required | AI Agent      |
| ------------------ | ---------------- | ------------------------------------------------ | -------- | ------------- |
| **MainTopic**      | Single Line Text | Primary topic/theme                              | No       | Desk Reporter |
| **SubTopics**      | Long Text        | JSON array of related topics                     | No       | Desk Reporter |
| **EntitiesJSON**   | Long Text        | JSON: companies, people, locations, institutions | No       | Desk Reporter |
| **Sentiment**      | Single Select    | Overall sentiment analysis                       | No       | Desk Reporter |
| **MarketImpact**   | Single Select    | Potential market impact level                    | No       | Desk Reporter |
| **Urgency**        | Single Select    | Time sensitivity of the news                     | No       | Desk Reporter |
| **RelevanceScore** | Number           | 1-10 score for digest inclusion                  | Yes      | Desk Reporter |
| **Tags**           | Long Text        | Comma-separated topic tags                       | No       | Desk Reporter |
| **Categories**     | Long Text        | RSS feed categories (comma-separated)            | No       | From RSS feed |
| **ProcessedBy**    | Single Line Text | Model used for AI processing                     | No       | System        |

**Select Field Options:**

**Sentiment:** `positive`, `neutral`, `negative`  
**MarketImpact:** `low`, `medium`, `high`  
**Urgency:** `routine`, `timely`, `breaking`

**JSON Field Examples:**

```javascript
// SubTopics
["Monetary Policy", "Interest Rates", "Inflation"]

// EntitiesJSON
{
  "companies": ["Federal Reserve", "Goldman Sachs"],
  "people": ["Jerome Powell", "Janet Yellen"],
  "locations": ["United States", "Washington DC"],
  "financialTerms": ["interest rates", "quantitative easing"]
}
```

**RelevanceScore Guidelines:**

- 1-3: Low relevance (skip in digest)
- 4-6: Medium relevance (consider for digest)
- 7-10: High relevance (prioritize for digest)

---

### 6️⃣ PROCESSING METADATA (5 fields)

| Field Name          | Field Type | Description               | Required | Notes                        |
| ------------------- | ---------- | ------------------------- | -------- | ---------------------------- |
| **WordCount**       | Number     | Article word count        | Yes      | Updated after scraping       |
| **TokenCount**      | Number     | Estimated LLM tokens used | No       | ~wordCount / 4               |
| **IncludeInDigest** | Checkbox   | Flag for digest inclusion | No       | Based on RelevanceScore >= 7 |

**WordCount Evolution:**

- Workflow 1: Set to 0 (no content yet)
- Workflow 2: Updated with actual word count from FullText

---

### 7️⃣ DIGEST INTEGRATION (1 field)

| Field Name       | Field Type    | Description            | Required |
| ---------------- | ------------- | ---------------------- | -------- |
| **LinkedDigest** | Link to Table | Links to Digests table | No       |

**Purpose:** Track which articles were included in which digest (populated during Workflow 3 - Journalist Agent)

---

## 🔧 Step-by-Step: Create Articles Table

### Step 1: Create Base

1. Log into Airtable
2. Click "Create a base"
3. Name it: **FineOpinions**
4. Create first table, rename to: **Articles**

---

### Step 2: Add Identification Fields

**Field 1: ArticleID (Auto Number)**

1. First column already exists, rename to: `ArticleID`
2. Change type to: **Auto Number**

**Field 2: URL (Unique)** 3. Click "+ Add field" 4. Type: **URL** 5. Name: `URL` 6. ✅ Check "Don't allow duplicate values"

**Field 3: URLHash (Unique)** 7. Click "+ Add field" 8. Type: **Single Line Text** 9. Name: `URLHash` 10. ✅ Check "Don't allow duplicate values"

> **💡 Pro Tip:** URLHash is the merge field for Upsert operations, ensuring no duplicate articles.

**Field 4: GUID** 11. Click "+ Add field" 12. Type: **Single Line Text** 13. Name: `GUID`

**Field 5: Source** 14. Click "+ Add field" 15. Type: **Single Select** 16. Name: `Source` 17. Add options: `ecb`, `marketwatch`, `nasdaq`, `bnpparibas`, `financemonthly`, `cnbc`, `money` 18. Choose colors for each (suggest: blue for ecb, green for marketwatch, etc.)

---

### Step 3: Add Content Fields

**Field 6: Headline**

1. Click "+ Add field"
2. Type: **Single Line Text**
3. Name: `Headline`

**Field 7: ContentSnippet** 4. Click "+ Add field" 5. Type: **Long Text** 6. Name: `ContentSnippet` 7. Enable rich text: **No** (plain text)

**Field 8: FullText** 8. Click "+ Add field" 9. Type: **Long Text** 10. Name: `FullText` 11. Enable rich text: **No** (plain text)

**Field 9: Summary** 12. Click "+ Add field" 13. Type: **Long Text** 14. Name: `Summary` 15. Enable rich text: **No** (plain text)

**Field 10: KeyFacts** 16. Click "+ Add field" 17. Type: **Long Text** 18. Name: `KeyFacts` 19. _Note: Will store JSON array as string_

**Field 11: DataPoints** 20. Click "+ Add field" 21. Type: **Long Text** 22. Name: `DataPoints` 23. _Note: Will store JSON array as string_

---

### Step 4: Add Source & Date Fields

**Field 12: PubDate**

1. Click "+ Add field"
2. Type: **Date**
3. Name: `PubDate`
4. Date format: **Friendly** (e.g., "October 25, 2025")
5. ✅ Include time: **Yes**
6. Time format: **24 hour**
7. GMT offset: **Use same timezone for all collaborators**

**Field 13: FetchedAt** 8. Click "+ Add field" 9. Type: **Date** 10. Name: `FetchedAt` 11. ✅ Include time: **Yes** 12. Time format: **24 hour**

**Field 14: ProcessedAt** 13. Click "+ Add field" 14. Type: **Date** 15. Name: `ProcessedAt` 16. ✅ Include time: **Yes** 17. Time format: **24 hour**

**Field 15: Creator** 18. Click "+ Add field" 19. Type: **Single Line Text** 20. Name: `Creator`

---

### Step 5: Add Status Management Fields (Critical for ADR-001)

**Field 16: Status**

1. Click "+ Add field"
2. Type: **Single Select**
3. Name: `Status`
4. Add options (in order):
   - `needs_scraping` (yellow)
   - `scraping_in_progress` (orange)
   - `scraped` (blue)
   - `scraping_failed` (red)
   - `processed` (green)
   - `digest_included` (purple)

**Field 17: ScrapingAttempts** 5. Click "+ Add field" 6. Type: **Number** 7. Name: `ScrapingAttempts` 8. Number format: **Integer** 9. Allow negative: **No**

**Field 18: LastScrapingAttempt** 10. Click "+ Add field" 11. Type: **Date** 12. Name: `LastScrapingAttempt` 13. ✅ Include time: **Yes**

**Field 19: ScrapingError** 14. Click "+ Add field" 15. Type: **Long Text** 16. Name: `ScrapingError`

**Field 20: ScrapingStrategy** 17. Click "+ Add field" 18. Type: **Single Select** 19. Name: `ScrapingStrategy` 20. Add options: `http`, `playwright`, `searxng`

---

### Step 6: Add AI Analysis Fields

**Field 21: MainTopic**

1. Click "+ Add field"
2. Type: **Single Line Text**
3. Name: `MainTopic`

**Field 22: SubTopics** 4. Click "+ Add field" 5. Type: **Long Text** 6. Name: `SubTopics` 7. _Note: JSON array as string_

**Field 23: EntitiesJSON** 8. Click "+ Add field" 9. Type: **Long Text** 10. Name: `EntitiesJSON` 11. _Note: JSON object as string_

**Field 24: Sentiment** 12. Click "+ Add field" 13. Type: **Single Select** 14. Name: `Sentiment` 15. Add options: `positive`, `neutral`, `negative`

**Field 25: MarketImpact** 16. Click "+ Add field" 17. Type: **Single Select** 18. Name: `MarketImpact` 19. Add options: `low`, `medium`, `high`

**Field 26: Urgency** 20. Click "+ Add field" 21. Type: **Single Select** 22. Name: `Urgency` 23. Add options: `routine`, `timely`, `breaking`

**Field 27: RelevanceScore** 24. Click "+ Add field" 25. Type: **Number** 26. Name: `RelevanceScore` 27. Number format: **Integer** 28. Allow negative: **No** 29. Min: 0, Max: 10

**Field 28: Tags** 30. Click "+ Add field" 31. Type: **Long Text** 32. Name: `Tags` 33. _Note: Comma-separated string_

**Field 29: Categories** 34. Click "+ Add field" 35. Type: **Long Text** 36. Name: `Categories` 37. _Note: From RSS feed, comma-separated_

**Field 30: ProcessedBy** 38. Click "+ Add field" 39. Type: **Single Line Text** 40. Name: `ProcessedBy` 41. _Example: "llama3.2:3b"_

---

### Step 7: Add Processing Metadata Fields

**Field 31: WordCount**

1. Click "+ Add field"
2. Type: **Number**
3. Name: `WordCount`
4. Number format: **Integer**
5. Allow negative: **No**

**Field 32: TokenCount** 6. Click "+ Add field" 7. Type: **Number** 8. Name: `TokenCount` 9. Number format: **Integer** 10. Allow negative: **No**

**Field 33: IncludeInDigest** 11. Click "+ Add field" 12. Type: **Checkbox** 13. Name: `IncludeInDigest`

---

### Step 8: Add Digest Integration (Optional - Add Later)

**Field 34: LinkedDigest**

1. Click "+ Add field"
2. Type: **Link to another record**
3. Name: `LinkedDigest`
4. Link to table: **Digests** (you'll create this next)
5. Allow linking to multiple records: **No**

> **💡 Note:** This can be added during Workflow 3 implementation (Journalist Agent).

---

### Step 9: Create Test Record

Create one manual record to verify all fields work:

```
ArticleID: (auto-generated: 1)
URL: https://www.ecb.europa.eu/test-article-2025
URLHash: a1b2c3d4
GUID: test-guid-12345
Source: ecb
Headline: Test Article: ECB Rate Decision
ContentSnippet: This is a test content snippet...
FullText: (leave empty - populated by Workflow 2)
Summary: (leave empty - populated by AI)
KeyFacts: (leave empty)
DataPoints: (leave empty)
PubDate: 2025-10-25 14:30:00
FetchedAt: 2025-10-25 14:35:00
ProcessedAt: (leave empty)
Creator: Test Author
Status: needs_scraping
ScrapingAttempts: 0
LastScrapingAttempt: (leave empty)
ScrapingError: (leave empty)
ScrapingStrategy: (leave empty)
MainTopic: (leave empty)
SubTopics: (leave empty)
EntitiesJSON: (leave empty)
Sentiment: neutral
MarketImpact: low
Urgency: routine
RelevanceScore: 0
Tags: (leave empty)
Categories: economics, monetary policy
ProcessedBy: (leave empty)
WordCount: 0
TokenCount: 0
IncludeInDigest: (unchecked)
LinkedDigest: (leave empty)
```

**✅ Verification:** Ensure all fields accept values correctly, then **delete this test record**.

---

### Step 10: Get Airtable API Credentials

**For n8n Integration:**

1. Click your profile icon (top right in Airtable)
2. Go to **"Developer Hub"** or **"Account"** → **"API"**
3. Click **"Create new token"** or **"Generate API key"**
4. **Token Name:** `n8n FineOpinions`
5. **Scopes:** Select:
   - `data.records:read`
   - `data.records:write`
   - `schema.bases:read`
6. **Access:** Select your FineOpinions base
7. Click **"Create token"**
8. Copy the token (starts with "pat..." for Personal Access Token)
9. **⚠️ Save securely** - you'll need this for n8n

**Alternative: Legacy API Key**

- If using older Airtable API, go to **Account** → **API** → **Generate API key**
- Copy the key (starts with "key...")

---

## 📋 Table 2: Digests (Multi-Agent Output)

### Purpose

Stores complete digest output from Journalist, Editorial, and Copywriter agents.

### Field Configuration (Minimum Viable - Add More Later)

#### CORE FIELDS

| Field Name   | Field Type    | Description                       |
| ------------ | ------------- | --------------------------------- |
| **DigestID** | Auto Number   | Primary key                       |
| **Date**     | Date          | Digest date (no time needed)      |
| **Period**   | Single Select | morning, evening (for future use) |
| **Status**   | Single Select | processing, complete, sent, error |

**Single Select Options for Period:** morning, evening  
**Single Select Options for Status:** processing, complete, sent, error

#### JOURNALIST FIELDS

| Field Name                | Field Type | Description                   |
| ------------------------- | ---------- | ----------------------------- |
| **ExecutiveSummary**      | Long Text  | 2-3 sentence factual overview |
| **TopStoriesJSON**        | Long Text  | JSON array of top stories     |
| **DataHighlights**        | Long Text  | JSON array of key numbers     |
| **JournalistProcessedAt** | Date       | Timestamp (include time)      |

#### EDITORIAL FIELDS

| Field Name               | Field Type       | Description              |
| ------------------------ | ---------------- | ------------------------ |
| **EditorialLede**        | Long Text        | Opening with personality |
| **AnalysisPointsJSON**   | Long Text        | JSON array of analysis   |
| **Takeaway**             | Single Line Text | Main takeaway with edge  |
| **EditorialProcessedAt** | Date             | Timestamp (include time) |

#### COPYWRITER FIELDS

| Field Name        | Field Type       | Description                        |
| ----------------- | ---------------- | ---------------------------------- |
| **EmailSubject**  | Single Line Text | Subject line (max 60 chars)        |
| **EmailBodyHTML** | Long Text        | Complete HTML email                |
| **WordCount**     | Number           | Final word count                   |
| **SentAt**        | Date             | When email was sent (include time) |

### Step-by-Step Creation

#### Step 1: Create Table

1. In FineOpinions base, click "+ Add or import"
2. Create empty table
3. Rename to: **Digests**

#### Step 2: Add Core Fields

**DigestID:**

1. Rename first column to `DigestID`
2. Type: **Auto Number**

**Date:** 3. Add field, Type: **Date**, Name: `Date` 4. Date format: Friendly 5. Include time: No

**Period:** 6. Add field, Type: **Single Select**, Name: `Period` 7. Options: `morning`, `evening`

**Status:** 8. Add field, Type: **Single Select**, Name: `Status` 9. Options: `processing`, `complete`, `sent`, `error`

#### Step 3: Add Journalist Fields

**ExecutiveSummary:** 10. Add field, Type: **Long Text**, Name: `ExecutiveSummary`

**TopStoriesJSON:** 11. Add field, Type: **Long Text**, Name: `TopStoriesJSON`

**DataHighlights:** 12. Add field, Type: **Long Text**, Name: `DataHighlights`

**JournalistProcessedAt:** 13. Add field, Type: **Date**, Name: `JournalistProcessedAt` 14. Include time: Yes

#### Step 4: Add Editorial Fields

**EditorialLede:** 15. Add field, Type: **Long Text**, Name: `EditorialLede`

**AnalysisPointsJSON:** 16. Add field, Type: **Long Text**, Name: `AnalysisPointsJSON`

**Takeaway:** 17. Add field, Type: **Single Line Text**, Name: `Takeaway`

**EditorialProcessedAt:** 18. Add field, Type: **Date**, Name: `EditorialProcessedAt` 19. Include time: Yes

#### Step 5: Add Copywriter Fields

**EmailSubject:** 20. Add field, Type: **Single Line Text**, Name: `EmailSubject`

**EmailBodyHTML:** 21. Add field, Type: **Long Text**, Name: `EmailBodyHTML`

**WordCount:** 22. Add field, Type: **Number**, Name: `WordCount` 23. Number format: Integer

**SentAt:** 24. Add field, Type: **Date**, Name: `SentAt` 25. Include time: Yes

#### Step 6: Create Test Record

Create one manual record to verify:

- DigestID: (auto-generated)
- Date: Today's date
- Period: `morning`
- Status: `processing`
- ExecutiveSummary: `Test summary text`
- (Other fields can be empty for now)

**Delete test record after verification.**

---

## 🔗 Linking Tables (Optional - Add Later)

### If You Want to Link Articles to Digests

**In Articles table:**

1. Add field, Type: **Link to another record**
2. Name: `LinkedDigest`
3. Link to table: **Digests**
4. Allow linking to multiple records: No

**In Digests table:**

1. Linked field automatically appears as `Articles` (rename to `LinkedArticles`)
2. Shows all articles included in digest

**Note:** This is optional and can be added during MODULE 3 implementation.

---

## 🔑 n8n Integration Setup

### Step 1: Get Credentials in n8n

1. In n8n, go to "Credentials" → "New Credential"
2. Search for "Airtable"
3. Click "Airtable API"

### Step 2: Configure Airtable Credential

1. **API Key:** Paste your Airtable API key (from earlier)
2. Click "Save"
3. Name it: "FineOpinions Airtable"

### Step 3: Test Connection

1. Create test workflow in n8n
2. Add "Airtable" node
3. Select Credential: "FineOpinions Airtable"
4. Base: Select "FineOpinions"
5. Operation: "List"
6. Table: "Articles"
7. Execute node

**If successful:** Returns empty array (no articles yet) or your test record  
**If fails:** Check API key, verify base name

### Step 4: Get Base ID (For Reference)

1. Open FineOpinions base in Airtable
2. URL will look like: `https://airtable.com/appXXXXXXXXXXXXXX/...`
3. The `appXXXXXXXXXXXXXX` part is your Base ID
4. Save this - may need it for API calls

---

## ✅ Validation Checklist

### Articles Table

**Identification Fields:**

- [ ] ArticleID is Auto Number
- [ ] URL field marked as unique
- [ ] URLHash field marked as unique
- [ ] GUID is Single Line Text
- [ ] Source has 7 options (ecb, marketwatch, nasdaq, bnpparibas, financemonthly, cnbc, money)

**Content Fields:**

- [ ] Headline is Single Line Text
- [ ] ContentSnippet is Long Text
- [ ] FullText is Long Text
- [ ] Summary is Long Text
- [ ] KeyFacts is Long Text (for JSON)
- [ ] DataPoints is Long Text (for JSON)

**Source & Date Fields:**

- [ ] PubDate includes time
- [ ] FetchedAt includes time
- [ ] ProcessedAt includes time
- [ ] Creator is Single Line Text

**Status Management Fields (ADR-001):**

- [ ] Status has 6 options (needs_scraping, scraping_in_progress, scraped, scraping_failed, processed, digest_included)
- [ ] ScrapingAttempts is Number (integer, no negatives)
- [ ] LastScrapingAttempt includes time
- [ ] ScrapingError is Long Text
- [ ] ScrapingStrategy has 3 options (http, playwright, searxng)

**AI Analysis Fields:**

- [ ] MainTopic is Single Line Text
- [ ] SubTopics is Long Text (for JSON)
- [ ] EntitiesJSON is Long Text (for JSON)
- [ ] Sentiment has 3 options (positive, neutral, negative)
- [ ] MarketImpact has 3 options (low, medium, high)
- [ ] Urgency has 3 options (routine, timely, breaking)
- [ ] RelevanceScore is Number (0-10, integer, no negatives)
- [ ] Tags is Long Text
- [ ] Categories is Long Text
- [ ] ProcessedBy is Single Line Text

**Processing Metadata:**

- [ ] WordCount is Number (integer, no negatives)
- [ ] TokenCount is Number (integer, no negatives)
- [ ] IncludeInDigest is Checkbox

**Digest Integration:**

- [ ] LinkedDigest links to Digests table (optional for now)

### Digests Table

- [ ] Has all core fields (DigestID, Date, Period, Status)
- [ ] Has Journalist fields (4 fields)
- [ ] Has Editorial fields (4 fields)
- [ ] Has Copywriter fields (4 fields)
- [ ] Period has 2 options (morning, evening)
- [ ] Status has 4 options (processing, complete, sent, error)
- [ ] Test record creates successfully
- [ ] n8n can read and write to table

**n8n Integration:**

- [ ] Credential created and saved
- [ ] Test connection successful
- [ ] Can list records from Articles table
- [ ] Base ID recorded (for reference)

---

## 🔧 Field Type Cheat Sheet

### When to Use Each Type

**Single Line Text:**

- Short strings (<100 chars)
- Headlines, topics, model names
- Examples: Headline, MainTopic, ProcessedBy

**Long Text:**

- Long strings (>100 chars)
- JSON arrays/objects (store as strings)
- Full article content
- Examples: FullText, KeyFacts (JSON), EntitiesJSON

**Number:**

- Numeric values
- Scores, counts, token usage
- Examples: RelevanceScore, WordCount, TokenCount

**Single Select:**

- Predefined options (dropdown)
- Status fields, categories
- Examples: Source, ScrapingStatus, Period, Status

**Date:**

- Timestamps
- With or without time
- Examples: PubDate (with time), Date (without time)

**URL:**

- Web links
- Enables unique constraint
- Examples: URL field

**Auto Number:**

- Auto-incrementing ID
- Primary keys
- Examples: ArticleID, DigestID

---

## 📊 Optional Fields (Add During Testing)

### For Articles Table (Add as Needed)

| Field Name    | Field Type      | Purpose                         |
| ------------- | --------------- | ------------------------------- |
| SubTopics     | Long Text       | JSON array of related topics    |
| SentimentJSON | Long Text       | Overall, forMarkets, confidence |
| MarketImpact  | Single Select   | low, medium, high               |
| Urgency       | Single Select   | routine, timely, breaking       |
| Tags          | Multiple Select | Topic tags (create as needed)   |
| TokenCount    | Number          | Tokens used by Desk Reporter    |

### For Digests Table (Add as Needed)

| Field Name         | Field Type       | Purpose                      |
| ------------------ | ---------------- | ---------------------------- |
| ArticleCount       | Number           | Number of articles processed |
| EmergingThemesJSON | Long Text        | JSON array of themes         |
| MarketMoodJSON     | Long Text        | JSON object                  |
| WatchListJSON      | Long Text        | JSON array                   |
| QuestionsRaised    | Long Text        | JSON array                   |
| ToneAssessment     | Single Select    | tone categories              |
| EmailPreheader     | Single Line Text | Preview text (max 90 chars)  |
| EstimatedReadTime  | Single Line Text | e.g., "5-6 minutes"          |

---

## 🎯 JSON Storage in Airtable

### Why JSON in Long Text?

Airtable doesn't have a native JSON field type. We store JSON as text strings:

**Example - KeyFacts field:**

```json
[
  "Fed raised rates to 5.50%",
  "FOMC vote was unanimous",
  "Powell indicated more hikes possible"
]
```

**Stored as:** Plain text string in Long Text field

**When reading in n8n:** Parse with `JSON.parse()`  
**When writing in n8n:** Stringify with `JSON.stringify()`

### n8n Expression Examples

**Writing JSON to Airtable:**

```javascript
KeyFacts: {
  {
    JSON.stringify($json.keyFacts);
  }
}
EntitiesJSON: {
  {
    JSON.stringify($json.entities);
  }
}
```

**Reading JSON from Airtable:**

```javascript
const keyFacts = JSON.parse($json.KeyFacts);
const entities = JSON.parse($json.EntitiesJSON);
```

---

## 🔍 Indexing & Performance

### Recommended Views in Articles Table

**View 1: Workflow Monitor (Default)**

- Sort by: FetchedAt (descending)
- Group by: Status
- Shows articles by workflow stage
- **Purpose:** Overall pipeline monitoring

**View 2: Ready for Scraping**

- Filter: `Status = needs_scraping`
- Sort by: PubDate (descending)
- **Purpose:** What Workflow 2 will process next

**View 3: Recently Processed**

- Filter: `Status = processed`
- Sort by: ProcessedAt (descending)
- **Purpose:** Recently completed articles

**View 4: High Relevance**

- Filter: `RelevanceScore >= 7`
- Sort by: RelevanceScore (descending), PubDate (descending)
- **Purpose:** Top articles for digest inclusion

**View 5: Failed Scraping**

- Filter: `Status = scraping_failed`
- Sort by: ScrapingAttempts (descending)
- **Purpose:** Troubleshoot scraping issues

**View 6: By Source**

- Group by: Source
- Sort within groups by: PubDate (descending)
- **Purpose:** See articles per RSS source

**View 7: Digest Ready**

- Filter: `IncludeInDigest = true`
- Sort by: RelevanceScore (descending)
- **Purpose:** Articles flagged for next digest

### Recommended Views in Digests Table

**View 1: Recent Digests (Default)**

- Sort by: Date (descending)
- Shows most recent digests first

**View 2: Sent Digests**

- Filter: Status = "sent"
- Archive of delivered digests

**View 3: In Progress**

- Filter: Status = "processing"
- Monitor active digest generation

---

## ⚠️ Common Issues & Solutions

### Issue: "Field doesn't exist" error in n8n

**Solution:** Check exact field name spelling (case-sensitive)  
**Example:** `ProcessedAt` ≠ `processedAt` ≠ `Processed At`

### Issue: JSON won't store in Airtable

**Solution:** Use `JSON.stringify()` in n8n  
**Example:** `{{ JSON.stringify($json.keyFacts) }}`

### Issue: Date field errors

**Solution:** Ensure date is ISO-8601 format  
**Example:** `{{ new Date().toISOString() }}`

### Issue: Duplicate URL errors

**Solution:** This is correct behavior (deduplication working)  
**Check:** Is deduplication logic in workflow working?

### Issue: Number field won't accept value

**Solution:** Ensure value is numeric, not string  
**Example:** `{{ Number($json.relevanceScore) }}`

---

## 🔄 Deduplication Strategy (Important!)

### Why Airtable Can't Prevent Duplicates Automatically

Airtable **does not** have a built-in "skip if exists" feature when creating records through the API or n8n. According to the [Airtable Community](https://community.airtable.com/automations-8/prevent-duplicates-during-automation-23842), the recommended pattern is:

1. **Find Records** - Search for existing record
2. **Conditional** - Only create if not found

### Our Deduplication Approach (n8n Workflow)

We implement deduplication **in the n8n workflow** (Module 1, Sub-Task 1.4), **before** attempting to write to Airtable:

```
RSS Feed → Generate URLHash → Search Airtable → IF Not Found → Continue
                                                 → IF Found → Skip Article
```

### How It Works

**Step 1: Generate URL Hash (Function Node)**

```javascript
const crypto = require("crypto");
const url = $json.link || $json.url;
const urlHash = crypto.createHash("md5").update(url).digest("hex");

return {
  ...item.json,
  url: url,
  urlHash: urlHash,
};
```

**Step 2: Query Airtable (Airtable Node - Search)**

```javascript
Operation: Search
Table: Articles
Formula: {URLHash} = "{{ $json.urlHash }}"
```

**Step 3: Check if Exists (IF Node)**
\\\\\\\\\\\\\\\\
```javascript
Conditions:
  - Value 1: {{ $json.records }}
  - Operation: Is Empty

IF TRUE (empty = no existing record) → Continue to scraping
IF FALSE (record exists) → Skip this article (workflow ends)
```

### Why This Works

- **Unique constraints** (URL and URLHash fields) prevent duplicates at the **database level**
- **n8n deduplication** prevents duplicate **insert attempts** (no errors)
- **URLHash** is faster to search than full URL text
- **MD5 hash** is deterministic (same URL = same hash)

### What Happens If You Skip This

If you try to insert a duplicate without checking first:

- Airtable will **reject** the insert (unique constraint violation)
- n8n workflow will **error**
- Article will **not** be processed

**That's why we check BEFORE attempting to insert!**

### Reference

- **Community Discussion:** [Prevent Duplicates During Automation](https://community.airtable.com/automations-8/prevent-duplicates-during-automation-23842)
- **Implementation:** `/memory-bank/build-implementation-plan.md` - Module 1, Sub-Task 1.4
- **Full Deduplication Code:** See Module 1, Sub-Tasks 1.3 & 1.4

---

## 🎯 Next Steps After Setup

1. ✅ Both tables created with all fields
2. ✅ API key obtained and tested in n8n
3. ✅ Test records created and verified
4. ✅ Test records deleted (clean slate)

**Now:** Start Module 1 from `/memory-bank/build-implementation-plan.md`

**Reference this doc:** When adding/modifying Airtable fields during build

---

## 📞 Quick Reference

**Base Name:** FineOpinions  
**Table 1:** Articles (**36 fields** organized in 7 categories)  
**Table 2:** Digests (12-20 fields, expandable)  
**API Token Location:** Airtable Developer Hub → Create Token  
**n8n Credential:** "FineOpinions Airtable"  
**Architecture:** Separated Workflows (ADR-001)

### Articles Table Field Summary

| Category                 | Field Count | Purpose                             |
| ------------------------ | ----------- | ----------------------------------- |
| Identification Fields    | 5           | Primary keys and unique identifiers |
| Content Fields           | 6           | Article content at various stages   |
| Source & Date Fields     | 4           | Origin and timing information       |
| Status Management Fields | 5           | Workflow state tracking (ADR-001)   |
| AI Analysis Fields       | 10          | Desk Reporter agent outputs         |
| Processing Metadata      | 5           | System tracking and metrics         |
| Digest Integration       | 1           | Link to final digest output         |
| **TOTAL**                | **36**      | **Complete article lifecycle**      |

### Field Usage by Workflow

**Workflow 1 (RSS Fetching)** populates:

- Identification: URL, URLHash, GUID, Source
- Content: Headline, ContentSnippet
- Source & Date: PubDate, FetchedAt, Creator, Categories
- Status: Status (set to "needs_scraping"), ScrapingAttempts (0)
- Processing: RelevanceScore (0), WordCount (0), TokenCount (0)

**Workflow 2 (Content Scraping & AI)** populates:

- Content: FullText, Summary, KeyFacts, DataPoints
- Date: ProcessedAt
- Status: Status (update to "processed" or "scraping_failed"), ScrapingAttempts (increment), LastScrapingAttempt, ScrapingError, ScrapingStrategy
- AI Analysis: All 10 fields (MainTopic through ProcessedBy)
- Processing: WordCount (updated), TokenCount (updated), IncludeInDigest

**Workflow 3 (Journalist Agent)** will populate:

- Digest Integration: LinkedDigest
- Status: Status (update to "digest_included")

---

## 🎯 Next Steps After Airtable Setup

1. ✅ **Complete:** Articles table created with all 36 fields
2. ✅ **Complete:** Digests table created
3. ✅ **Complete:** API token obtained
4. ✅ **Complete:** Test records verified and deleted
5. ✅ **Complete:** Recommended views created

**Now you're ready to:**

1. **Configure n8n Integration** → [n8n Integration Setup](#-n8n-integration-setup)
2. **Build Workflow 1** → [`fineopinions_node_settings.md`](../fineopinions_node_settings.md)
3. **Build Workflow 2** → [`docs/IMPLEMENT-WORKFLOW-2.md`](./IMPLEMENT-WORKFLOW-2.md)

---

## 📚 Related Documentation

- **Architecture:** [ADR-001: Separated Workflows](./architecture-decisions/ADR-001-separated-workflows.md)
- **Implementation:** [Workflow 1 Node Settings](../fineopinions_node_settings.md)
- **Implementation:** [Workflow 2 Implementation Guide](./IMPLEMENT-WORKFLOW-2.md)
- **Diagrams:** [System Architecture](../fineopinions_diagram.md)
- **Tasks:** [Project Task List](../tasks.md)

---

**Last Updated:** October 25, 2025 - Separated Workflows Architecture (ADR-001)

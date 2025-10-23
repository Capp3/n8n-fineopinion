# Airtable Setup & Provisioning Guide

**Last Updated:** October 9, 2025  
**Purpose:** Complete step-by-step Airtable database setup  
**Time Required:** 1-2 hours  
**Prerequisites:** Airtable account created

---

## 🎯 Overview

FineOpinions uses **2 Airtable tables**:

1. **Articles** - Stores Desk Reporter analysis of individual articles
2. **Digests** - Stores complete digest output (Journalist + Editorial + Copywriter)

---

## 📋 Table 1: Articles (Desk Reporter Output)

### Purpose

Stores all processed articles with Desk Reporter analysis, facts, relevance scoring, and full content.

### Field Configuration (18 Fields - Start with 13 Minimum)

#### PRIMARY FIELDS (Required)

| Field Name         | Field Type       | Description                                | Unique? | Required |
| ------------------ | ---------------- | ------------------------------------------ | ------- | -------- |
| **ArticleID**      | Auto Number      | Primary key                                | Yes     | Auto     |
| **URL**            | URL              | Article link                               | Yes     | Yes      |
| **URLHash**        | Single Line Text | MD5 hash for deduplication                 | Yes     | Yes      |
| **Source**         | Single Select    | economist, bloomberg, reuters, marketwatch | No      | Yes      |
| **PubDate**        | Date             | Publication date/time (include time)       | No      | Yes      |
| **Headline**       | Single Line Text | Desk Reporter headline                     | No      | Yes      |
| **RelevanceScore** | Number           | 1-10 scale                                 | No      | Yes      |
| **FullText**       | Long Text        | Complete article content                   | No      | Yes      |
| **ScrapingStatus** | Single Select    | success, failed, fallback                  | No      | Yes      |

**Single Select Options for Source:** economist, bloomberg, reuters, marketwatch

**Single Select Options for ScrapingStatus:** success, failed, fallback

#### DESK REPORTER ANALYSIS FIELDS (Required)

| Field Name       | Field Type       | Description                                      |
| ---------------- | ---------------- | ------------------------------------------------ |
| **KeyFacts**     | Long Text        | JSON array of key facts                          |
| **MainTopic**    | Single Line Text | Primary topic                                    |
| **EntitiesJSON** | Long Text        | JSON: companies, people, locations, institutions |
| **DataPoints**   | Long Text        | JSON array of critical numbers                   |

#### METADATA FIELDS (Optional but Recommended)

| Field Name      | Field Type       | Description                                 |
| --------------- | ---------------- | ------------------------------------------- |
| **FetchedAt**   | Date             | When RSS was fetched (include time)         |
| **ProcessedAt** | Date             | When Desk Reporter processed (include time) |
| **ProcessedBy** | Single Line Text | Model used (e.g., "llama3.2:3b")            |
| **WordCount**   | Number           | Article word count                          |
| **TitleClean**  | Single Line Text | Normalized title for dedup                  |

### Step-by-Step Creation

#### Step 1: Create Base in Airtable

1. Log into Airtable
2. Click "Create a base"
3. Name it: **FineOpinions**
4. Create first table, rename to: **Articles**

#### Step 2: Add Fields (In Order)

**Start with Auto Number:**

1. First column is already there, rename to: `ArticleID`
2. Set type to: **Auto Number**

**Add URL (Unique):** 3. Click "+ Add field" 4. Type: **URL** 5. Name: `URL` 6. Check "Don't allow duplicate values" ✅

**Add URLHash:** 7. Click "+ Add field" 8. Type: **Single Line Text** 9. Name: `URLHash` 10. Check "Don't allow duplicate values" ✅

> **⚠️ Important - Deduplication Strategy:**  
> The unique constraints on URL and URLHash **prevent duplicates at the database level**, but Airtable doesn't have a "skip if exists" feature when creating records. Instead, **we handle deduplication in the n8n workflow** (Module 1, Sub-Task 1.4) by querying Airtable BEFORE attempting to insert. If a record with the same URLHash exists, we skip that article entirely. This prevents duplicate insert errors.

**Add Source:** 11. Click "+ Add field" 12. Type: **Single Select** 13. Name: `Source` 14. Add options: `economist`, `bloomberg`, `reuters`, `marketwatch` 15. Choose colors for each

**Add PubDate:** 16. Click "+ Add field" 17. Type: **Date** 18. Name: `PubDate` 19. Date format: **Friendly** (e.g., "October 9, 2025") 20. Include time: **Yes** ✅ 21. Time format: **24 hour** 22. GMT offset: **Use same timezone for all collaborators**

**Add Headline:** 23. Click "+ Add field" 24. Type: **Single Line Text** 25. Name: `Headline`

**Add RelevanceScore:** 26. Click "+ Add field" 27. Type: **Number** 28. Name: `RelevanceScore` 29. Number format: **Integer** 30. Allow negative numbers: **No**

**Add FullText:** 31. Click "+ Add field" 32. Type: **Long Text** 33. Name: `FullText` 34. Enable rich text formatting: **No** (plain text)

**Add ScrapingStatus:** 35. Click "+ Add field" 36. Type: **Single Select** 37. Name: `ScrapingStatus` 38. Add options: `success`, `failed`, `fallback`

**Add KeyFacts:** 39. Click "+ Add field" 40. Type: **Long Text** 41. Name: `KeyFacts` 42. _Note: Will store JSON array as string_

**Add MainTopic:** 43. Click "+ Add field" 44. Type: **Single Line Text** 45. Name: `MainTopic`

**Add EntitiesJSON:** 46. Click "+ Add field" 47. Type: **Long Text** 48. Name: `EntitiesJSON` 49. _Note: Will store JSON object as string_

**Add DataPoints:** 50. Click "+ Add field" 51. Type: **Long Text** 52. Name: `DataPoints` 53. _Note: Will store JSON array as string_

**Add ProcessedBy:** 54. Click "+ Add field" 55. Type: **Single Line Text** 56. Name: `ProcessedBy`

#### Step 3: Create Test Record

Create one manual record to verify all fields work:

- ArticleID: (auto-generated)
- URL: `https://www.economist.com/test-article`
- URLHash: `abc123def456`
- Source: `economist`
- PubDate: Today's date with time
- Headline: `Test Article Headline`
- RelevanceScore: `7`
- FullText: `This is test content...`
- ScrapingStatus: `success`
- KeyFacts: `["Fact 1", "Fact 2", "Fact 3"]`
- MainTopic: `Test Topic`
- EntitiesJSON: `{"companies": [], "people": []}`
- DataPoints: `["Data 1", "Data 2"]`
- ProcessedBy: `llama3.2:3b`

**Delete this test record after verification.**

#### Step 4: Get API Key

1. Click your profile icon (top right)
2. Go to "Account" → "API"
3. Click "Generate API key"
4. Copy the key (starts with "key...")
5. Save securely - you'll need this for n8n

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

## ✅ Verification Checklist

After setup, verify:

**Articles Table:**

- [ ] Has all 13 minimum fields
- [ ] URL field is unique (duplicate check enabled)
- [ ] URLHash field is unique
- [ ] Source has 4 options (economist, bloomberg, reuters, marketwatch)
- [ ] ScrapingStatus has 3 options (success, failed, fallback)
- [ ] Date fields include time
- [ ] Number fields are integers
- [ ] Long Text fields accept JSON strings
- [ ] Test record creates successfully
- [ ] n8n can read from table

**Digests Table:**

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

**View 1: Recent Articles (Default)**

- Sort by: ProcessedAt (descending)
- Filter: None
- Shows most recent articles first

**View 2: High Relevance**

- Filter: RelevanceScore >= 7
- Sort by: RelevanceScore (descending)
- Shows only high-value articles

**View 3: By Source**

- Group by: Source
- Sort within groups by: PubDate (descending)
- See articles per source

**View 4: Failed Scraping**

- Filter: ScrapingStatus = "failed"
- Troubleshoot scraping issues

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
**Table 1:** Articles (13-18 fields)  
**Table 2:** Digests (12-20 fields)  
**API Key Location:** Airtable Account → API  
**n8n Credential:** "FineOpinions Airtable"

---

**Last Updated:** October 9, 2025  
**Status:** Complete provisioning guide  
**Next:** Start Module 1 implementation

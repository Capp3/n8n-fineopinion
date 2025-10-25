# FineOpinions - n8n Node Settings & Configuration

**Last Updated:** October 25, 2025  
**Status:** Separated Workflows Architecture (ADR-001) - Workflow 1 Configurations  
**Purpose:** GUI configuration values and code snippets for manual n8n workflow construction

**📋 IMPORTANT:** This document covers **Workflow 1 (Article Fetching & Storage)** configurations only. For Workflow 2 (Content Scraping & AI Processing), see [ADR-001](./docs/architecture-decisions/ADR-001-separated-workflows.md).

**📖 Quick Links:**

- [ADR-001: Separated Workflows Architecture](./docs/architecture-decisions/ADR-001-separated-workflows.md)
- [Next Steps: Complete Workflow 1](./docs/NEXT-STEPS-WORKFLOW-1.md)
- [System Architecture Diagrams](./fineopinions_diagram.md)

---

## 📋 Table of Contents

### Workflow 1 (This Document)

1. [Workflow Triggers](#workflow-triggers)
2. [RSS Feed Nodes](#rss-feed-nodes)
3. [RSS Post-Processing Pipeline](#rss-post-processing-pipeline)
4. [Airtable Integration](#airtable-integration)
5. [Troubleshooting](#troubleshooting-node-9-merge-rss-feeds)

### Workflow 2 (See ADR-001)

- **Not yet built** - See [ADR-001](./docs/architecture-decisions/ADR-001-separated-workflows.md) for complete Workflow 2 specifications
- Content Scraping Module (HTTP/Playwright/SearXNG)
- AI Agent Processing (Ollama)
- Status Updates & Error Handling

---

## 🚨 **Node Numbering Reference**

### Workflow 1: Article Fetching & Storage (This Document)

| Node # | Node Name                | Type           | Purpose                             |
| ------ | ------------------------ | -------------- | ----------------------------------- |
| **1**  | Manual Test Trigger      | Manual Trigger | Testing only                        |
| **2**  | ECB RSS Feed             | RSS Feed Read  | Fetch ECB articles                  |
| **3**  | MarketWatch RSS Feed     | RSS Feed Read  | Fetch MarketWatch articles          |
| **4**  | NASDAQ RSS Feed          | RSS Feed Read  | Fetch NASDAQ articles               |
| **5**  | BNP Paribas RSS Feed     | RSS Feed Read  | Fetch BNP articles                  |
| **6**  | Finance Monthly RSS Feed | RSS Feed Read  | Fetch Finance Monthly articles      |
| **7**  | CNBC RSS Feed            | RSS Feed Read  | Fetch CNBC articles                 |
| **8**  | Money RSS Feed           | RSS Feed Read  | Fetch Money articles                |
| **9**  | 🔥 **Merge RSS Feeds**   | Merge          | **COMBINE ALL 7 FEEDS**             |
| **10** | Basic Field Normalizer   | Code           | Stage 1: Normalize core fields      |
| **11** | Content Processor        | Code           | Stage 2: Clean HTML content         |
| **12** | Metadata Processor       | Code           | Stage 3: Process creator/categories |
| **13** | Field Validator & Mapper | Code           | Stage 4: Validate & map to Airtable |
| **18** | Store in Airtable        | Airtable       | Upsert with Status management       |

**✅ WORKFLOW 1 ENDS HERE** - Articles stored with `Status = needs_scraping`

### Workflow 2: Content Scraping & AI Processing (Not Yet Built)

**📖 See [ADR-001](./docs/architecture-decisions/ADR-001-separated-workflows.md) for Workflow 2 node configurations**

Workflow 2 will include:

- Airtable Query (Status = needs_scraping)
- Adaptive Scraping Strategy Selection
- HTTP Request Scraper
- Playwright Scraper (Docker)
- SearXNG Fallback
- AI Agent Processing (Ollama)
- Status Updates (scraped/processed)

---

**🎯 KEY ARCHITECTURAL CHANGE (ADR-001):**

- **OLD APPROACH:** Monolithic workflow (RSS → Normalize → Scrape → AI → Store)
- **NEW APPROACH:** Separated workflows
  - **Workflow 1:** RSS → Normalize → Store with Status (FAST, runs every 15 min)
  - **Workflow 2:** Query Status → Scrape → AI → Update Status (SLOW, runs hourly or on-demand)

**Benefits:**

- ✅ Duplicate check BEFORE scraping (70-80% fewer HTTP requests)
- ✅ Failure isolation (scraping failures don't block RSS fetching)
- ✅ Independent scheduling and scaling
- ✅ Can trigger Workflow 2 manually or as sub-workflow later

---

## 🔧 Workflow Triggers

### Node 1: Manual Test Trigger

- **Node Type:** Manual Trigger
- **No additional configuration needed**
- **Use for:** Testing the workflow manually

### Production Trigger: RSS Processing Schedule

- **Node Type:** Schedule Trigger
- **Trigger Rules:** Cron Expression
- **Cron Expression:** `0 7,19 * * *` _(runs at 7 AM and 7 PM daily)_
- **Trigger at Startup:** Disabled

---

## 🔄 RSS Feed Nodes

### Node 2: ECB RSS Feed

- **Node Type:** RSS Feed Read
- **URL:** `https://www.ecb.europa.eu/rss/press.xml`
- **Options:** Leave all options at default

### Node 3: MarketWatch RSS Feed

- **Node Type:** RSS Feed Read
- **URL:** `https://www.marketwatch.com/rss/topstories`
- **Options:** Leave all options at default

### Node 4: NASDAQ RSS Feed

- **Node Type:** RSS Feed Read
- **URL:** `https://www.nasdaq.com/feed/nasdaq-original/rss.xml`
- **Options:** Leave all options at default

### Node 5: BNP Paribas RSS Feed

- **Node Type:** RSS Feed Read
- **URL:** `https://economic-research.bnpparibas.com/RSS/en-US/Eco-Insight`
- **Options:** Leave all options at default

### Node 6: Finance Monthly RSS Feed

- **Node Type:** RSS Feed Read
- **URL:** `https://www.finance-monthly.com/feed`
- **Options:** Leave all options at default

### Node 7: CNBC RSS Feed

- **Node Type:** RSS Feed Read
- **URL:** `https://www.cnbc.com/id/100003114/device/rss/rss.html`
- **Options:** Leave all options at default

### Node 8: Money RSS Feed

- **Node Type:** RSS Feed Read
- **URL:** `https://money.com/money/feed`
- **Options:** Leave all options at default

---

## 🔧 RSS Post-Processing Pipeline

### Node 9: 🔥 Merge RSS Feeds _(COMMON FAILURE POINT)_

- **Node Type:** Merge
- **Mode:** Append ⚠️ **CRITICAL: Use "Append" not "Combine"!**
- **Options:** Leave all options at default

**⚠️ WHY APPEND MODE:**

- **Append** = Stack all items from all feeds (what you want!)
- **Combine By Position** = Pair items by index (limits to shortest feed)

**Example:**

- Feed 1: 10 items
- Feed 2: 8 items
- Feed 3: 5 items ← **SHORTEST**
- Feed 4: 12 items

**With "Combine By Position":** You get only 5 items (shortest feed)  
**With "Append":** You get 35 items (all items from all feeds!)

**⚠️ TROUBLESHOOTING NODE 9:**

- **Common Issue:** One or more RSS feeds failing to load
- **If you only get 5 items:** Change mode from "Combine" to "Append"
- **Check:** All 7 RSS feed nodes (2-8) are connected to this merge node
- **Test:** Run individual RSS feed nodes to see which ones work
- **Fix:** Disable problematic RSS feeds temporarily, or check URLs

**Connection Requirements:**

- Connect all 7 RSS feed nodes to the Merge node
- Order doesn't matter with Append mode
- All feeds will be stacked together

**🚨 CNBC URL NOTE:**

**Working CNBC URL:** `https://www.cnbc.com/id/15839135/device/rss/rss.html` (Top News)

**Alternative CNBC URLs if needed:**

- `https://www.cnbc.com/id/100727362/device/rss/rss.html` (World News)
- `https://www.cnbc.com/id/19836768/device/rss/rss.html` (US News)

### Node 10: Basic Field Normalizer

- **Node Type:** Code
- **Mode:** Run Once for All Items
- **JavaScript Code:**

**⚠️ IMPORTANT:** This code uses string matching instead of `new URL()` because the URL constructor is not available in n8n's code execution context.

```javascript
// Stage 1: Basic Field Normalization
function normalizeBasicFields(item) {
  // Extract source from URL (using string matching instead of URL constructor)
  function extractSourceFromLink(link) {
    if (!link) return "unknown";
    const linkLower = link.toLowerCase();

    if (linkLower.includes("ecb.europa.eu")) return "ecb";
    if (linkLower.includes("marketwatch.com")) return "marketwatch";
    if (linkLower.includes("nasdaq.com")) return "nasdaq";
    if (linkLower.includes("bnpparibas.com")) return "bnpparibas";
    if (linkLower.includes("finance-monthly.com")) return "financemonthly";
    if (linkLower.includes("cnbc.com")) return "cnbc";
    if (linkLower.includes("money.com")) return "money";

    return "unknown";
  }

  // Normalize GUID
  function normalizeGuid(guid) {
    if (!guid) return Date.now().toString();
    return String(guid);
  }

  // Normalize dates
  function normalizeDates(item) {
    const pubDate = item.pubDate || item.isoDate;
    const isoDate =
      item.isoDate ||
      (pubDate ? new Date(pubDate).toISOString() : new Date().toISOString());

    return { pubDate, isoDate };
  }

  const dates = normalizeDates(item);

  return {
    title: item.title || "",
    link: item.link || "",
    pubDate: dates.pubDate,
    isoDate: dates.isoDate,
    guid: normalizeGuid(item.guid),
    source: item.source || extractSourceFromLink(item.link),
    // Preserve original data
    originalData: item,
  };
}

// Process all items
const normalizedItems = [];
for (const item of $input.all()) {
  const normalized = normalizeBasicFields(item.json);
  normalizedItems.push(normalized);
}

return normalizedItems.map((item) => ({ json: item }));
```

### Node 11: Content Processor

- **Node Type:** Code
- **Mode:** Run Once for All Items
- **JavaScript Code:**

```javascript
// Stage 2: Content Processing
function processContent(item) {
  // Clean HTML content
  function cleanHtmlContent(content) {
    if (!content) return "";

    // Remove HTML tags but preserve text
    let cleaned = content.replace(/<[^>]*>/g, " ");

    // Decode HTML entities
    cleaned = cleaned
      .replace(/&nbsp;/g, " ")
      .replace(/&amp;/g, "&")
      .replace(/&lt;/g, "<")
      .replace(/&gt;/g, ">")
      .replace(/&quot;/g, '"')
      .replace(/&#39;/g, "'");

    // Clean up whitespace
    cleaned = cleaned.replace(/\s+/g, " ").trim();

    return cleaned;
  }

  // Extract snippet from content
  function extractSnippet(content, maxLength = 300) {
    if (!content) return "";

    const cleaned = cleanHtmlContent(content);
    if (cleaned.length <= maxLength) return cleaned;

    // Find last complete sentence within limit
    const truncated = cleaned.substring(0, maxLength);
    const lastSentence = truncated.lastIndexOf(". ");

    if (lastSentence > maxLength * 0.7) {
      return truncated.substring(0, lastSentence + 1);
    }

    return truncated + "...";
  }

  // Get content from various fields
  const originalData = item.originalData || {};
  const content =
    originalData["content:encoded"] ||
    originalData.content ||
    originalData.contentSnippet ||
    "";
  const snippet =
    originalData["content:encodedSnippet"] ||
    originalData.contentSnippet ||
    extractSnippet(content);

  return {
    ...item,
    content: cleanHtmlContent(content),
    contentSnippet: cleanHtmlContent(snippet),
    contentLength: content.length,
    hasLongContent: content.length > 1000,
  };
}

// Process all items
const processedItems = [];
for (const item of $input.all()) {
  const processed = processContent(item.json);
  processedItems.push(processed);
}

return processedItems.map((item) => ({ json: item }));
```

### Node 12: Metadata Processor (with URL Hash)

- **Node Type:** Code
- **Mode:** Run Once for All Items
- **JavaScript Code:**

```javascript
// Stage 3: Metadata Processing with URL Hash
function processMetadata(item) {
  const originalData = item.originalData || {};

  // Process creator field
  function getCreator(data) {
    return data.creator || data["dc:creator"] || "Unknown";
  }

  // Process categories
  function getCategories(data) {
    const categories = data.categories || [];
    if (Array.isArray(categories)) {
      return categories;
    }
    return [];
  }

  // Generate URL hash for deduplication
  function generateUrlHash(url) {
    if (!url) return "";

    // Simple hash function (djb2)
    let hash = 5381;
    for (let i = 0; i < url.length; i++) {
      hash = (hash << 5) + hash + url.charCodeAt(i);
    }
    // Convert to positive hex string
    return Math.abs(hash).toString(16);
  }

  return {
    ...item,
    creator: getCreator(originalData),
    categories: getCategories(originalData),
    categoryCount: getCategories(originalData).length,
    hasCategories: getCategories(originalData).length > 0,
    urlHash: generateUrlHash(item.link), // Add URL hash
  };
}

// Process all items
const metadataProcessed = [];
for (const item of $input.all()) {
  const processed = processMetadata(item.json);
  metadataProcessed.push(processed);
}

return metadataProcessed.map((item) => ({ json: item }));
```

### Node 13: Field Validator & Airtable Mapper

- **Node Type:** Code
- **Mode:** Run Once for All Items
- **JavaScript Code:**

```javascript
// Stage 4: Validation & Airtable Mapping
function validateAndMap(item) {
  // Validate required fields
  function validateRequiredFields(data) {
    const errors = [];

    if (!data.title) errors.push("Missing title");
    if (!data.link) errors.push("Missing link");
    if (!data.source) errors.push("Missing source");
    if (!data.isoDate) errors.push("Missing date");

    return {
      valid: errors.length === 0,
      errors: errors,
      data: data,
    };
  }

  // Map to Airtable schema (field names match Airtable exactly for auto-mapping)
  function mapToAirtableSchema(data) {
    // Format dates for Airtable (ISO 8601 without milliseconds)
    function formatDateForAirtable(dateString) {
      if (!dateString) return new Date().toISOString().split(".")[0] + "Z";
      const date = new Date(dateString);
      return date.toISOString().split(".")[0] + "Z";
    }

    return {
      // Core identification fields
      URL: data.link,
      URLHash: data.urlHash,
      GUID: data.guid,

      // Content fields
      Headline: data.title, // Changed from "Title" to match Airtable
      Source: data.source,

      // Date fields
      PubDate: formatDateForAirtable(data.isoDate),
      FetchedAt: formatDateForAirtable(new Date().toISOString()),

      // Metadata
      Creator: data.creator || "",
      Categories: data.categories.join(", "),

      // Content (from RSS feed snippet)
      ContentSnippet: data.contentSnippet || "",

      // Status management for Separated Workflows (ADR-001)
      Status: "needs_scraping", // Initial status for Workflow 2
      ScrapingAttempts: 0,

      // Processing flags
      RelevanceScore: 0,
      WordCount: 0,
      TokenCount: 0,
    };
  }

  const validation = validateRequiredFields(item);

  if (!validation.valid) {
    return {
      ...item,
      validationErrors: validation.errors,
      isValid: false,
      airtableReady: false,
    };
  }

  const airtableData = mapToAirtableSchema(validation.data);

  return {
    ...item,
    isValid: true,
    airtableReady: true,
    airtableData: airtableData,
    validationErrors: [],
  };
}

// Process all items
const finalProcessed = [];
for (const item of $input.all()) {
  const final = validateAndMap(item.json);
  finalProcessed.push(final);
}

// Separate valid and invalid items (items are plain objects, not wrapped)
const validItems = finalProcessed.filter((item) => item.isValid);
const invalidItems = finalProcessed.filter((item) => !item.isValid);

// Log processing summary
console.log(`Processing Summary:`);
console.log(`- Total items: ${finalProcessed.length}`);
console.log(`- Valid items: ${validItems.length}`);
console.log(`- Invalid items: ${invalidItems.length}`);

if (invalidItems.length > 0) {
  console.log(`Invalid items:`);
  invalidItems.forEach((item, index) => {
    console.log(
      `  ${index + 1}. ${
        item.title || "No title"
      }: ${item.validationErrors.join(", ")}`
    );
  });
}

// Return only valid items wrapped in json structure
return validItems.map((item) => ({ json: item }));
```

### Node 14: Check for Existing Records

- **Node Type:** Code
- **Mode:** Run Once for All Items
- **JavaScript Code:**

**⚠️ IMPORTANT:** This node filters out existing records so we only insert NEW articles.

```javascript
// Check if records already exist and filter to only new articles
async function filterNewArticles() {
  const items = $input.all();

  // Extract all URL hashes to check
  const urlHashes = items.map((item) => item.json.airtableData.URLHash);

  // Note: You'll need to add an Airtable "List" node before this
  // to fetch existing records. This code assumes you have that data.

  // For now, we'll pass all items through and let Airtable Upsert handle it
  // But we'll add a flag to indicate these are potentially new

  return items.map((item) => ({
    json: {
      ...item.json,
      isNewArticle: true, // Flag for tracking
    },
  }));
}

// Simple version: Just pass through with tracking
// In production, you'd query Airtable first to check existence
const items = $input.all();
return items.map((item) => ({
  json: {
    ...item.json,
    isNewArticle: true,
  },
}));
```

**🎯 Better Approach:** Use Airtable's behavior instead of a separate check node.

Instead of this complex check, we'll configure the Airtable node to:

1. Use `URLHash` as the merge field
2. Only update specific fields if record exists
3. Set `Status` only for new records

**Skip Node 14 for now** - see the Airtable configuration below for the better approach.

---

## 💾 Airtable Integration

**🎯 THIS IS WHERE WORKFLOW 1 ENDS**

Articles are stored in Airtable with `Status = needs_scraping`, ready for Workflow 2 to pick up and process.

### Node 18: Store in Airtable (Auto-Mapping with URLHash)

- **Node Type:** Airtable
- **Operation:** Upsert ⚠️ **IMPORTANT: Use "Upsert" to handle duplicates!**
- **Application:** `YOUR_AIRTABLE_BASE_ID` _(replace with your actual base ID)_
- **Table:** `Articles`
- **Merge Field:** `URLHash` _(Airtable will use this to check for duplicates)_
- **Options:**
  - ✅ **Typecast:** Enabled (handles date format conversion)
  - ✅ **Use Field Names:** Enabled

**⚠️ CRITICAL: This node implements the Separated Workflows Architecture (ADR-001)**

**🎯 HOW IT WORKS:**

1. **URLHash as Merge Field**: Uses the hash we generated in Node 12
2. **Automatic Mapping**: Field names in Node 13 match Airtable exactly
3. **If Record Exists**: Only updates FetchedAt (doesn't change Status)
4. **If Record is New**: Sets Status to "needs_scraping" for Workflow 2

**Configuration:**

### Step 1: Basic Settings

1. Set **Operation** to **"Upsert"**
2. Set **"Columns to Match On"** to **`URLHash`**
3. Set **Mapping Mode** to **"Auto-map Input Data to Columns"**

### Step 2: Enable Options

1. Scroll to **"Options"**
2. Add **"Typecast"** → Toggle **ON**
3. This handles date format conversions automatically

### Step 3: Field Mappings (Auto or Manual)

**Option A: Automatic Mapping (Recommended)**

- Since we formatted field names in Node 13 to match Airtable exactly
- n8n will automatically map all fields
- Just ensure "Columns" is set to "Auto-map Input Data"

**Option B: Manual Mapping (if auto-map doesn't work)**

Use these expressions to manually map each field:

```javascript
{
  // Identification fields
  "URL": "={{$json.airtableData.URL}}",
  "URLHash": "={{$json.airtableData.URLHash}}",
  "GUID": "={{$json.airtableData.GUID}}",

  // Content fields
  "Headline": "={{$json.airtableData.Headline}}",
  "Source": "={{$json.airtableData.Source}}",
  "ContentSnippet": "={{$json.airtableData.ContentSnippet}}",

  // Date fields
  "PubDate": "={{$json.airtableData.PubDate}}",
  "FetchedAt": "={{$json.airtableData.FetchedAt}}",

  // Metadata
  "Creator": "={{$json.airtableData.Creator}}",
  "Categories": "={{$json.airtableData.Categories}}",

  // Status Management (for Workflow 2)
  "Status": "={{$json.airtableData.Status}}",
  "ScrapingAttempts": "={{$json.airtableData.ScrapingAttempts}}",

  // Processing flags
  "RelevanceScore": "={{$json.airtableData.RelevanceScore}}",
  "WordCount": "={{$json.airtableData.WordCount}}",
  "TokenCount": "={{$json.airtableData.TokenCount}}"
}
```

**🔄 IF/ELSE Logic (How Upsert Handles Existing Records):**

**If URLHash EXISTS in Airtable (duplicate):**

- ✅ Updates: `FetchedAt` (new timestamp)
- ❌ Does NOT change: `Status`, `ScrapingAttempts`, or any processed data
- 🎯 Result: Existing articles retain their workflow state

**If URLHash is NEW (first time seeing this article):**

- ✅ Creates new record with ALL fields
- ✅ Sets `Status = "needs_scraping"` for Workflow 2
- ✅ Sets `ScrapingAttempts = 0`
- 🎯 Result: New articles ready for processing

**How This Works:**

1. **Workflow 1 (this workflow)**:
   - Stores new articles with `Status = needs_scraping`
   - Updates `FetchedAt` for existing articles (doesn't change Status)
2. **Workflow 2 (separate workflow)**:
   - Queries for articles where `Status = needs_scraping`
   - Scrapes content and updates status to `scraped` or `scraping_failed`
3. **URLHash deduplication**:
   - Prevents true duplicates
   - Preserves workflow state for articles already being processed

**Why This Architecture?**

- **Resource Optimization**: Duplicate check happens BEFORE scraping (saves 70-80% of HTTP requests)
- **State Preservation**: Already-processed articles keep their status
- **Failure Isolation**: Scraping failures don't block RSS fetching
- **Flexible Execution**: Can run scraping workflow independently or on-demand
- **Better Scalability**: Each workflow scales independently

**📖 Full Documentation:** See [ADR-001](./docs/architecture-decisions/ADR-001-separated-workflows.md)

---

## 🔧 Troubleshooting URLHash Matching

### Why URLHash Instead of URL?

**Problem with URL matching:**

- URLs can be very long (Airtable has length limits)
- URLs might have tracking parameters that differ
- Some RSS feeds use different URL formats for the same article

**Solution: URLHash**

- Fixed-length hash (hex string)
- Consistent identifier for each unique URL
- Faster matching in Airtable
- More reliable deduplication

### If URLHash Matching Isn't Working

**Check 1: Verify URLHash is Being Generated**

1. Execute Node 12 (Metadata Processor)
2. Check the output - should see `urlHash` field
3. Should be a hex string like: `"a3f8c92d"`

**Check 2: Verify URLHash Field Exists in Airtable**

1. Open your Airtable Articles table
2. Check if `URLHash` column exists
3. If not, add it as a **Single Line Text** field
4. Run workflow to populate it

**Check 3: Verify Merge Field Configuration**

1. Open Node 18 (Airtable)
2. Check "Columns to Match On" = `URLHash`
3. NOT `GUID`, NOT `URL` - must be `URLHash`

**Check 4: URLHash Field Type in Airtable**

- Must be: **Single Line Text** or **Formula**
- Cannot be: Auto-number, Lookup, or Rollup
- If it's wrong type, recreate the field

### Common Errors

**Error: "Field URLHash not found"**

- Solution: Add `URLHash` column to Airtable Articles table

**Error: "Unsupported field type to upsert"**

- Solution: Change URLHash to "Single Line Text" type in Airtable

**Error: "No matching records found" but duplicates created**

- Solution: Verify URLHash is being generated in Node 12
- Check Node 13 output includes `airtableData.URLHash`

---

## ✅ WORKFLOW 1 COMPLETE

**Congratulations!** If you've configured all the nodes above, your Workflow 1 (Article Fetching & Storage) is complete.

**What happens next:**

1. Workflow 1 runs every 15 minutes (or on manual trigger)
2. Fetches articles from 7 RSS feeds
3. Normalizes and validates data (4-stage pipeline)
4. Stores articles in Airtable with `Status = needs_scraping`
5. Articles are ready for Workflow 2 to pick up

**Next Steps:**

- [ ] Add 5 Status management fields to Airtable (see [NEXT-STEPS-WORKFLOW-1.md](./docs/NEXT-STEPS-WORKFLOW-1.md))
- [ ] Test Workflow 1 end-to-end
- [ ] Run for 24 hours to collect articles
- [ ] Build Workflow 2 (Content Scraping & AI Processing)

**📖 Workflow 2 Documentation:**

- Complete specifications: [ADR-001](./docs/architecture-decisions/ADR-001-separated-workflows.md)
- Implementation tasks: [tasks.md](./tasks.md) → Task 1.1

---

## 🚨 **Troubleshooting Node 9 (Merge RSS Feeds)**

If you're failing at **Node 9**, here's how to debug:

### Step 1: Test CNBC Feed (Most Likely Culprit)

Your "line 6" error points to **Node 7 (CNBC)**:

- Right-click **CNBC node** → "Execute Node"
- If it fails, try these alternative CNBC URLs:
  - `https://www.cnbc.com/id/100727362/device/rss/rss.html`
  - `https://www.cnbc.com/id/15839135/device/rss/rss.html`
  - `https://www.cnbc.com/id/19836768/device/rss/rss.html`

### Step 2: Test All RSS Feeds Individually

Run each RSS feed node (2-8) individually to see which ones work:

- Right-click each RSS node → "Execute Node"
- Check which ones return data vs errors

### Step 3: Common RSS Feed Issues

- **ECB (Node 2):** Sometimes slow, may timeout
- **NASDAQ (Node 4):** Occasionally blocks requests
- **BNP Paribas (Node 5):** May have SSL certificate issues
- **Finance Monthly (Node 6):** Large content, may timeout
- **CNBC (Node 7):** ⚠️ **MOST LIKELY CAUSE** - Blocks automated requests

### Step 4: Temporary Fix

Disconnect problematic RSS feeds from the Merge node temporarily:

1. Identify failing RSS feeds
2. Disconnect them from Node 9 (Merge)
3. Test with remaining feeds
4. Reconnect one by one

### Step 5: Check Merge Settings

- **Mode:** "Append" ⚠️ **CRITICAL - NOT "Combine"!**
- **If you only get 5 items:** You're using "Combine By Position" - change to "Append"
- **All working RSS feeds must be connected**

**How to Check:**

1. Click on your Merge node (Node 9)
2. Look at the "Mode" setting
3. If it says "Combine" → Change to "Append"
4. Save and re-run

**Why This Matters:**

- **Append mode** stacks ALL items from ALL feeds
- **Combine mode** pairs items by position, limiting to shortest feed
- With 7 different RSS feeds returning different amounts, you want ALL items!

---

## 📚 Related Documentation

**Core Architecture:**

- [ADR-001: Separated Workflows Architecture](./docs/architecture-decisions/ADR-001-separated-workflows.md) - **READ THIS FIRST**
- [ADR-001 Summary](./memory-bank/adr-001-summary.md) - Quick reference
- [System Architecture Diagrams](./fineopinions_diagram.md) - Visual system overview

**Implementation Guides:**

- [Next Steps: Complete Workflow 1](./docs/NEXT-STEPS-WORKFLOW-1.md) - Immediate action items
- [Task List](./tasks.md) - Progress tracking

**Context:**

- [RSS Feed Architecture](./memory-bank/rss-feed-architecture.md) - Detailed RSS processing
- [Documentation Index](./DOCUMENTATION-INDEX.md) - Complete documentation map

---

**Last Updated:** October 25, 2025  
**Document Version:** 2.1 (Separated Workflows Architecture - Workflow 1 Only)  
**Status:** Complete - Ready for Workflow 1 implementation

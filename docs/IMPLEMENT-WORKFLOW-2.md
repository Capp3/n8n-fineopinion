# Workflow 2 Implementation Guide: Content Scraping & AI Processing

**Date:** October 25, 2025  
**Status:** Ready to Implement  
**Architecture:** Separated Workflows (ADR-001)  
**Time Required:** ~60-90 minutes

---

## 🎯 What is Workflow 2?

**Purpose:** Query articles marked `needs_scraping`, scrape their full content, process with AI, and update their status.

**Key Features:**

- Queries Airtable for unprocessed articles
- Adaptive scraping strategies (HTTP → Playwright → SearXNG)
- AI agent processing (Ollama)
- Status management and error handling
- Retry logic with different strategies

**Execution:**

- Manual trigger (for testing)
- Scheduled (every 1 hour recommended)
- Sub-workflow call from Workflow 1 (advanced)

---

## 📋 Workflow 2 Node Structure

```
Workflow 2: Content Scraping & AI Processing
├── Node 1: Manual Trigger (or Schedule)
├── Node 2: Query Airtable (Status = needs_scraping)
├── Node 3: Select Scraping Strategy (Code)
├── Node 4: Switch by Strategy
│   ├── Branch A: HTTP Request Scraper
│   ├── Branch B: Playwright Scraper
│   └── Branch C: SearXNG Fallback
├── Node 5: Extract & Clean Content (Code)
├── Node 6: AI Agent Processing (Ollama)
├── Node 7: Parse AI Response (Code)
├── Node 8: Update Airtable Status (Upsert)
└── Node 9: Error Handler (optional)
```

---

## ✅ Step-by-Step Implementation

### Node 1: Manual Trigger (for Testing)

**Node Type:** Manual Trigger

**Configuration:**

- No configuration needed
- Used for testing during development

**Later:** Replace with Schedule Trigger

- Cron: `0 * * * *` (every hour)
- Or: `*/30 * * * *` (every 30 minutes)

---

### Node 2: Query Airtable for Articles Needing Scraping

**Node Type:** Airtable  
**Operation:** List - Search

**Configuration:**

1. **Application:** Your Airtable base
2. **Table:** `Articles`
3. **Return All:** No (we'll batch process)
4. **Limit:** 20 (adjust based on your needs)

**Filter Formula:**

```javascript
AND(({ Status } = "needs_scraping"), { ScrapingAttempts } < 3);
```

**Sort:**

- Field: `PubDate`
- Direction: Descending (newest first)

**Why This Works:**

- Only gets articles that need scraping
- Limits retry attempts to 3
- Processes newest articles first
- Batches for efficiency

---

### Node 3: Select Scraping Strategy

**Node Type:** Code  
**Mode:** Run Once for Each Item

**JavaScript Code:**

```javascript
// Select scraping strategy based on source and previous attempts
const item = $input.item.json;

const source = item.Source;
const attempts = item.ScrapingAttempts || 0;

// Strategy order by source (first = preferred, last = fallback)
const strategies = {
  ecb: ["http", "playwright", "searxng"],
  marketwatch: ["playwright", "searxng", "http"],
  nasdaq: ["playwright", "searxng", "http"],
  bnpparibas: ["http", "playwright", "searxng"],
  financemonthly: ["http", "playwright", "searxng"],
  cnbc: ["playwright", "searxng", "http"],
  money: ["playwright", "searxng", "http"],
};

// Get strategy list for this source (default to http first)
const strategyList = strategies[source] || ["http", "playwright", "searxng"];

// Pick strategy based on attempt number
const strategyIndex = Math.min(attempts, strategyList.length - 1);
const strategy = strategyList[strategyIndex];

// Return with strategy and updated metadata
return {
  ...item,
  _strategy: strategy,
  _attemptNumber: attempts + 1,
  _startTime: new Date().toISOString(),
};
```

**What This Does:**

- Selects scraping strategy based on source
- Escalates to different strategies on retries
- Tracks attempt number and timing

---

### Node 4: Switch by Strategy

**Node Type:** Switch  
**Mode:** Rules

**Rules:**

**Rule 1: HTTP Strategy**

- Condition: `{{$json._strategy}} equals 'http'`
- Output: 0

**Rule 2: Playwright Strategy**

- Condition: `{{$json._strategy}} equals 'playwright'`
- Output: 1

**Rule 3: SearXNG Fallback**

- Condition: `{{$json._strategy}} equals 'searxng'`
- Output: 2

**Fallback:** Route to HTTP (Output 0)

---

### Node 5A: HTTP Request Scraper

**Node Type:** HTTP Request  
**Connected to:** Switch Output 0

**Configuration:**

- **URL:** `={{$json.URL}}`
- **Method:** GET
- **Authentication:** None

**Headers:**

```javascript
{
  "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36",
  "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
  "Accept-Language": "en-US,en;q=0.9"
}
```

**Options:**

- Timeout: 15000 (15 seconds)
- Follow Redirects: Yes
- Max Redirects: 5

**Error Handling:**

- On Error: Continue Execution
- This allows fallback to other strategies

---

### Node 5B: Playwright Scraper (Docker)

**Node Type:** Execute Command  
**Connected to:** Switch Output 1

**Note:** Requires Playwright Docker container running

**Configuration:**

**Command:** `docker`

**Arguments:**

```javascript
[
  "run",
  "--rm",
  "-e",
  "URL={{$json.URL}}",
  "-e",
  "TIMEOUT=30000",
  "mcr.microsoft.com/playwright:latest",
  "node",
  "-e",
  `
  const { chromium } = require('playwright');
  (async () => {
    const browser = await chromium.launch({
      headless: true,
      args: ['--no-sandbox', '--disable-setuid-sandbox']
    });
    const page = await browser.newPage();
    await page.setViewportSize({ width: 1920, height: 1080 });
    
    try {
      await page.goto(process.env.URL, {
        waitUntil: 'networkidle',
        timeout: parseInt(process.env.TIMEOUT)
      });
      
      // Wait a bit for dynamic content
      await page.waitForTimeout(2000);
      
      // Extract main content
      const content = await page.evaluate(() => {
        // Try to find article content
        const selectors = [
          'article',
          '[role="main"]',
          '.article-content',
          '.story-body',
          '.post-content',
          'main'
        ];
        
        for (const selector of selectors) {
          const element = document.querySelector(selector);
          if (element) {
            return element.innerText;
          }
        }
        
        // Fallback to body
        return document.body.innerText;
      });
      
      console.log(JSON.stringify({ content, success: true }));
    } catch (error) {
      console.log(JSON.stringify({ content: '', success: false, error: error.message }));
    } finally {
      await browser.close();
    }
  })();
  `,
];
```

**Alternative (Simpler):** Use n8n's HTTP Request with browser mode if available

---

### Node 5C: SearXNG Fallback

**Node Type:** HTTP Request  
**Connected to:** Switch Output 2

**Configuration:**

- **URL:** `http://localhost:8080/search`
- **Method:** GET

**Query Parameters:**

```javascript
{
  "q": "={{$json.Headline}} {{$json.Source}}",
  "format": "json",
  "engines": "google,bing",
  "safesearch": "0"
}
```

**What This Does:**

- Uses your SearXNG instance to find article
- Searches by headline and source
- Returns search results as fallback content

---

### Node 6: Extract & Clean Content

**Node Type:** Code  
**Mode:** Run Once for Each Item  
**Connects all scraper branches**

**JavaScript Code:**

```javascript
// Extract and clean scraped content
const item = $input.item.json;

function cleanHtmlContent(content) {
  if (!content) return "";

  // Remove HTML tags
  let cleaned = content.replace(/<script[^>]*>[\s\S]*?<\/script>/gi, "");
  cleaned = cleaned.replace(/<style[^>]*>[\s\S]*?<\/style>/gi, "");
  cleaned = cleaned.replace(/<[^>]*>/g, " ");

  // Decode HTML entities
  cleaned = cleaned
    .replace(/&nbsp;/g, " ")
    .replace(/&amp;/g, "&")
    .replace(/&lt;/g, "<")
    .replace(/&gt;/g, ">")
    .replace(/&quot;/g, '"')
    .replace(/&#39;/g, "'")
    .replace(/&[a-z]+;/gi, " ");

  // Clean whitespace
  cleaned = cleaned.replace(/\s+/g, " ").trim();

  return cleaned;
}

// Determine if scraping was successful
let scrapedContent = "";
let scrapingSuccess = false;
let scrapingMethod = item._strategy || "unknown";

// Extract content based on scraping method
if (item.data) {
  // HTTP Request result
  scrapedContent = cleanHtmlContent(item.data);
} else if (item.stdout) {
  // Playwright result (from docker command)
  try {
    const result = JSON.parse(item.stdout);
    scrapedContent = result.content || "";
    scrapingSuccess = result.success || false;
  } catch (e) {
    scrapedContent = item.stdout;
  }
} else if (item.results) {
  // SearXNG result
  if (Array.isArray(item.results) && item.results.length > 0) {
    scrapedContent = item.results.map((r) => r.content || "").join("\n\n");
  }
}

// Validate content quality
const wordCount = scrapedContent.split(/\s+/).length;
const hasMinimumContent = wordCount > 50;
scrapingSuccess = hasMinimumContent && scrapedContent.length > 200;

// Return with scraped content
return {
  ...item,
  FullText: scrapedContent,
  WordCount: wordCount,
  _scrapingSuccess: scrapingSuccess,
  _scrapingMethod: scrapingMethod,
  _scrapedAt: new Date().toISOString(),
};
```

---

### Node 7: AI Agent Processing (Ollama)

**Node Type:** Ollama (LangChain Chat Model)  
**Or:** HTTP Request to Ollama API

**Configuration:**

**Model:** `llama3.2:3b` (or `qwen2.5:7b` for better quality)  
**Temperature:** 0.3  
**Top P:** 0.9

**Prompt:**

```
You are a financial news analyst. Analyze the following article and provide a structured analysis.

Article Headline: {{$json.Headline}}
Source: {{$json.Source}}
Content: {{$json.FullText}}

Provide your analysis in JSON format with these fields:
{
  "summary": "Brief 2-3 sentence summary of the article",
  "keyFacts": ["fact1", "fact2", "fact3"],
  "mainTopic": "Primary topic of the article",
  "subTopics": ["topic1", "topic2"],
  "entities": {
    "companies": [],
    "people": [],
    "locations": [],
    "financialTerms": []
  },
  "sentiment": "positive|neutral|negative",
  "marketImpact": "low|medium|high",
  "urgency": "routine|timely|breaking",
  "relevanceScore": 7,
  "tags": ["tag1", "tag2"]
}

Return ONLY valid JSON, no other text.
```

**Alternative (HTTP Request to Ollama):**

**URL:** `http://localhost:11434/api/generate`  
**Method:** POST  
**Body:**

```javascript
{
  "model": "llama3.2:3b",
  "prompt": "Your prompt here with {{$json.FullText}}",
  "stream": false,
  "format": "json"
}
```

---

### Node 8: Parse AI Response

**Node Type:** Code  
**Mode:** Run Once for Each Item

**JavaScript Code:**

```javascript
// Parse and validate AI response
const item = $input.item.json;

let analysis = {};
let aiSuccess = false;

try {
  // Try to parse AI response
  if (typeof item.response === "string") {
    analysis = JSON.parse(item.response);
  } else if (item.response && typeof item.response === "object") {
    analysis = item.response;
  } else if (item.output) {
    analysis = JSON.parse(item.output);
  } else {
    throw new Error("No AI response found");
  }

  // Validate required fields
  const requiredFields = ["summary", "keyFacts", "relevanceScore"];
  const hasRequired = requiredFields.every((field) => analysis[field]);

  if (!hasRequired) {
    throw new Error("Missing required AI analysis fields");
  }

  aiSuccess = true;
} catch (error) {
  console.log(`AI parsing error for article ${item.id}: ${error.message}`);

  // Fallback analysis
  analysis = {
    summary: item.ContentSnippet || "Failed to generate summary",
    keyFacts: [],
    mainTopic: "Unknown",
    sentiment: "neutral",
    marketImpact: "low",
    urgency: "routine",
    relevanceScore: 3,
    tags: [],
  };
}

// Normalize relevance score
const relevanceScore = Math.max(
  1,
  Math.min(10, parseInt(analysis.relevanceScore) || 5)
);

// Prepare data for Airtable update
return {
  id: item.id,
  URLHash: item.URLHash,

  // Scraped content
  FullText: item.FullText,
  WordCount: item.WordCount,

  // AI analysis
  Summary: analysis.summary,
  KeyFacts: JSON.stringify(analysis.keyFacts || []),
  MainTopic: analysis.mainTopic || "",
  SubTopics: JSON.stringify(analysis.subTopics || []),
  EntitiesJSON: JSON.stringify(analysis.entities || {}),
  Sentiment: analysis.sentiment || "neutral",
  MarketImpact: analysis.marketImpact || "low",
  Urgency: analysis.urgency || "routine",
  RelevanceScore: relevanceScore,
  Tags: analysis.tags || [],

  // Status management
  Status: item._scrapingSuccess && aiSuccess ? "processed" : "scraping_failed",
  ScrapingAttempts: item._attemptNumber || 1,
  LastScrapingAttempt: new Date().toISOString(),
  ScrapingError: aiSuccess ? "" : "AI processing failed",
  ScrapingStrategy: item._scrapingMethod || "unknown",

  // Processing metadata
  ProcessedAt: new Date().toISOString(),
  ProcessedBy: "llama3.2:3b",
  TokenCount: Math.ceil(item.FullText.length / 4), // Rough estimate
};
```

---

### Node 9: Update Airtable Status

**Node Type:** Airtable  
**Operation:** Update (by record ID) or Upsert (by URLHash)

**Configuration:**

**Option A: Update by Record ID (Simpler)**

- Operation: Update
- Record ID: `={{$json.id}}`

**Option B: Upsert by URLHash (More robust)**

- Operation: Upsert
- Merge Field: `URLHash`

**Field Mappings:**

```javascript
{
  // Content fields
  "FullText": "={{$json.FullText}}",
  "WordCount": "={{$json.WordCount}}",

  // AI analysis fields
  "Summary": "={{$json.Summary}}",
  "KeyFacts": "={{$json.KeyFacts}}",
  "MainTopic": "={{$json.MainTopic}}",
  "SubTopics": "={{$json.SubTopics}}",
  "EntitiesJSON": "={{$json.EntitiesJSON}}",
  "Sentiment": "={{$json.Sentiment}}",
  "MarketImpact": "={{$json.MarketImpact}}",
  "Urgency": "={{$json.Urgency}}",
  "RelevanceScore": "={{$json.RelevanceScore}}",
  "Tags": "={{$json.Tags}}",

  // Status management
  "Status": "={{$json.Status}}",
  "ScrapingAttempts": "={{$json.ScrapingAttempts}}",
  "LastScrapingAttempt": "={{$json.LastScrapingAttempt}}",
  "ScrapingError": "={{$json.ScrapingError}}",
  "ScrapingStrategy": "={{$json.ScrapingStrategy}}",

  // Processing metadata
  "ProcessedAt": "={{$json.ProcessedAt}}",
  "ProcessedBy": "={{$json.ProcessedBy}}",
  "TokenCount": "={{$json.TokenCount}}"
}
```

**Options:**

- ✅ Typecast: Enabled

---

## 🧪 Testing Workflow 2

### Phase 1: Test Individual Components

**Test 1: Query Airtable**

1. Execute Node 2
2. Verify it returns articles with `Status = needs_scraping`
3. Check that limit is respected

**Test 2: Strategy Selection**

1. Execute Node 3
2. Check output includes `_strategy` field
3. Verify strategy matches source expectations

**Test 3: HTTP Scraping**

1. Execute Node 5A with a simple article
2. Verify content is retrieved
3. Check response contains HTML

**Test 4: Content Cleaning**

1. Execute Node 6
2. Verify `FullText` is clean (no HTML tags)
3. Check `WordCount` is reasonable

**Test 5: AI Processing**

1. Execute Node 7 with sample content
2. Verify JSON response
3. Check all required fields present

---

### Phase 2: Test Full Workflow

**Test 1: Single Article End-to-End**

1. Set Node 2 limit to 1
2. Execute entire workflow
3. Verify article status changes to `processed`

**Test 2: Batch Processing**

1. Set Node 2 limit to 5
2. Execute workflow
3. Verify all articles are processed
4. Check Airtable for updated statuses

**Test 3: Error Handling**

1. Test with article that fails to scrape
2. Verify status changes to `scraping_failed`
3. Check `ScrapingAttempts` increments
4. Test that retry uses different strategy

---

## 📊 Monitoring & Verification

### Key Metrics to Track

**Success Metrics:**

- Articles processed per run: 10-20 expected
- HTTP scraping success rate: 70-80%
- Playwright scraping success rate: 85-95%
- AI processing success rate: 95%+

**Status Distribution:**

- `needs_scraping`: Should decrease over time
- `processed`: Should increase
- `scraping_failed`: Should be < 5% after retries

### Airtable Views to Create

**View 1: Ready for Scraping**

- Filter: `Status = needs_scraping`
- Sort: PubDate descending

**View 2: Failed Scraping**

- Filter: `Status = scraping_failed`
- Sort: ScrapingAttempts descending

**View 3: Recently Processed**

- Filter: `Status = processed`
- Sort: ProcessedAt descending

---

## 🚀 Production Deployment

### Step 1: Test Thoroughly

- [ ] Test with 1 article
- [ ] Test with 5 articles
- [ ] Test with 20 articles
- [ ] Test error handling
- [ ] Test retry logic

### Step 2: Set Up Scheduling

**Recommended Schedule:** Every 1 hour

1. Replace Manual Trigger with Schedule Trigger
2. Set cron: `0 * * * *`
3. Enable workflow

**Alternative Schedules:**

- Every 30 min: `*/30 * * * *`
- Every 2 hours: `0 */2 * * *`
- Every 6 hours: `0 */6 * * *`

### Step 3: Monitor Initial Runs

**First 24 Hours:**

- Check every 2-3 hours
- Review error rates
- Adjust batch sizes if needed
- Monitor AI processing time

### Step 4: Optimize

**If too slow:**

- Reduce batch size (20 → 10)
- Increase execution frequency

**If high error rate:**

- Review failed articles
- Adjust strategy order
- Check source website changes

---

## 🔧 Troubleshooting

### Issue: No articles returned from Airtable

**Fix:**

- Check Workflow 1 is running
- Verify articles have `Status = needs_scraping`
- Check filter formula in Node 2

### Issue: HTTP scraping always fails

**Fix:**

- Check internet connectivity
- Verify User-Agent header
- Try different timeout values
- Fall back to Playwright

### Issue: Playwright container not working

**Fix:**

```bash
# Pull Playwright image
docker pull mcr.microsoft.com/playwright:latest

# Test container
docker run --rm mcr.microsoft.com/playwright:latest node -e "console.log('working')"
```

### Issue: AI responses not parsing

**Fix:**

- Check Ollama is running: `curl http://localhost:11434`
- Verify model is pulled: `ollama pull llama3.2:3b`
- Test prompt separately
- Add fallback analysis

### Issue: Airtable updates failing

**Fix:**

- Enable Typecast option
- Verify field names match
- Check record ID is correct
- Use URLHash upsert instead

---

## 📖 Related Documentation

- [ADR-001: Separated Workflows](../docs/architecture-decisions/ADR-001-separated-workflows.md)
- [fineopinions_node_settings.md](../fineopinions_node_settings.md)
- [Tasks: Task 1.1](../tasks.md)

---

## 🎉 Success Criteria

Workflow 2 is working when:

- [ ] Articles with `Status = needs_scraping` are processed
- [ ] Full content is scraped successfully
- [ ] AI analysis is generated and stored
- [ ] Status changes to `processed` or `scraping_failed`
- [ ] Retry logic works (different strategies)
- [ ] Airtable is updated with all fields
- [ ] No duplicate processing
- [ ] Runs reliably on schedule

**Next:** Connect Workflow 1 and 2, monitor for 48 hours, then build Workflow 3 (Journalist Agent)!

---

**Ready to build? Start with Node 1 and work your way through! 🚀**

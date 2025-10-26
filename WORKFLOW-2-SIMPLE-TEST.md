# Workflow 2: Simple Test Version (5 nodes)

Build this SIMPLIFIED version first to test core flow before adding Playwright/SearXNG.

## Node Structure (Simplified)

```
1. Manual Trigger
2. Airtable Query (Status = needs_scraping, limit 1)
3. HTTP Request (Simple scraping)
4. Code: Extract & Clean Content
5. Airtable Update (Update status)
```

## Node Configurations

### Node 1: Manual Trigger
- No config needed

### Node 2: Airtable Query
**Operation:** List - Search  
**Application:** Your Airtable base  
**Table:** Articles  
**Filter Formula:**
```
AND(({Status} = "needs_scraping"), {ScrapingAttempts} < 1)
```
**Limit:** 1 (start with 1 article for testing)  
**Sort Field:** PubDate  
**Sort Direction:** Descending

### Node 3: HTTP Request
**Method:** GET  
**URL:** `={{$json.fields.URL}}`  
**Authentication:** None

**Headers:**
```
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
Accept: text/html
Accept-Language: en-US,en;q=0.9
```

**Options:**
- Timeout: 15000
- Follow Redirects: Yes
- Max Redirects: 5

### Node 4: Extract & Clean Content (Code)
**Mode:** Run Once for Each Item

```javascript
// Simple content extraction
const item = $input.item.json;

// Get scraped HTML (from HTTP request)
const htmlContent = item.data || "";

// Basic HTML tag removal
function simpleStripHtml(html) {
  if (!html) return "";
  return html
    .replace(/<script[^>]*>[\s\S]*?<\/script>/gi, "")
    .replace(/<style[^>]*>[\s\S]*?<\/style>/gi, "")
    .replace(/<[^>]*>/g, " ")
    .replace(/\s+/g, " ")
    .trim();
}

const cleanedContent = simpleStripHtml(htmlContent);
const wordCount = cleanedContent.split(/\s+/).length;

// Return data for Airtable update
return {
  id: $input.all()[0].json.fields.ID,
  URLHash: $input.all()[0].json.fields.URLHash,
  
  // Scraped content
  FullText: cleanedContent,
  WordCount: wordCount,
  
  // Simple status update
  Status: wordCount > 100 ? "scraped" : "scraping_failed",
  ScrapingAttempts: ($input.all()[0].json.fields.ScrapingAttempts || 0) + 1,
  LastScrapingAttempt: new Date().toISOString(),
  ScrapingStrategy: "http",
  ScrapingError: wordCount > 100 ? "" : "Insufficient content retrieved"
};
```

### Node 5: Airtable Update
**Operation:** Update  
**Record ID:** `={{$json.id}}`  
**Options:** Typecast: Enabled

**Fields to Update:**
- FullText: `={{$json.FullText}}`
- WordCount: `={{$json.WordCount}}`
- Status: `={{$json.Status}}`
- ScrapingAttempts: `={{$json.ScrapingAttempts}}`
- LastScrapingAttempt: `={{$json.LastScrapingAttempt}}`
- ScrapingStrategy: `={{$json.ScrapingStrategy}}`
- ScrapingError: `={{$json.ScrapingError}}`

## Testing Steps

1. Run Workflow 1 first to get some articles with `needs_scraping` status
2. Execute this workflow manually
3. Check Airtable to see if Status changed from `needs_scraping` to either:
   - `scraped` (if content retrieved successfully)
   - `scraping_failed` (if insufficient content)

4. Test with different articles
5. Once working, expand to full Workflow 2 with AI processing

## Expected Results

**Success Case:**
- Airtable record: Status = `scraped`
- FullText field populated with article text
- WordCount > 100
- ScrapingAttempts = 1

**Failure Case:**
- Airtable record: Status = `scraping_failed`
- FullText empty or very short
- ScrapingError field has message
- ScrapingAttempts = 1

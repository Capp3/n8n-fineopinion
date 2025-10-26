# Workflow 2: SearXNG-Based Scraper (Simple & Working)

**Goal:** Use SearXNG as primary scraper since it's already working for you.

## Why SearXNG First?

- ✅ Already working in your environment
- ✅ No Docker setup needed
- ✅ Lightweight and fast
- ✅ Handles most article scraping
- ✅ Use Selenium only as fallback

## Simple Workflow Structure (5 Nodes)

```
1. Manual Trigger
2. Query Airtable (Status = needs_scraping)
3. SearXNG Search
4. Code: Parse Results
5. Update Airtable
```

## Node Configurations

### Node 1: Manual Trigger
- No config needed

### Node 2: Query Airtable
**Type:** Airtable  
**Operation:** List - Search  
**Filter:** `AND(({Status} = "needs_scraping"), {ScrapingAttempts} < 3)`  
**Limit:** 10  
**Sort:** PubDate DESC

### Node 3: SearXNG Search
**Type:** HTTP Request  
**Method:** GET  
**URL:** `http://your-searxng-instance:port/search`

**Query Parameters:**
```
q={{$json.fields.Headline}} {{$json.fields.Source}}
format=json
engines=google,bing,duckduckgo
```

### Node 4: Parse Results & Extract Content
**Type:** Code  
**Mode:** Run Once for Each Item

```javascript
const searxngResponse = $input.item.json;
const articleData = $('Query Airtable').item.json;

// Get the first relevant result
const results = searxngResponse.results || [];
const articleResult = results[0];

// Extract content (usually in snippet or content field)
const fullText = articleResult.content || articleResult.snippet || '';

// Clean the text
const cleanedContent = fullText
  .replace(/<[^>]*>/g, ' ')
  .replace(/\s+/g, ' ')
  .trim();

const wordCount = cleanedContent.split(/\s+/).length;

return {
  id: articleData.id,
  URLHash: articleData.fields.URLHash,
  FullText: cleanedContent,
  WordCount: wordCount,
  Status: wordCount > 50 ? 'scraped' : 'scraping_failed',
  ScrapingAttempts: (articleData.fields.ScrapingAttempts || 0) + 1,
  LastScrapingAttempt: new Date().toISOString(),
  ScrapingStrategy: 'searxng',
  ScrapingError: wordCount > 50 ? '' : 'Insufficient content from search'
};
```

### Node 5: Update Airtable
**Type:** Airtable  
**Operation:** Update  
**Record ID:** `={{$json.id}}`  
**Options:** Typecast = Enabled

**Map Fields:**
- FullText: `={{$json.FullText}}`
- WordCount: `={{$json.WordCount}}`
- Status: `={{$json.Status}}`
- ScrapingAttempts: `={{$json.ScrapingAttempts}}`
- LastScrapingAttempt: `={{$json.LastScrapingAttempt}}`
- ScrapingStrategy: `={{$json.ScrapingStrategy}}`
- ScrapingError: `={{$json.ScrapingError}}`

## Testing

1. Create new workflow in n8n
2. Build these 5 nodes
3. Test with 1 article first
4. Verify Airtable updates

## Expected Results

- SearXNG returns search results for article
- Content extracted and cleaned
- Status changes to `scraped` (if >50 words) or `scraping_failed`
- Airtable updated with FullText

## If SearXNG Fails: Add Selenium Fallback

If SearXNG gives insufficient results (>30% failure), add Selenium as fallback:

### Add After Node 3:
**Node 4A:** Switch Node
- Route 1: If wordCount < 50 → Selenium branch
- Route 2: If wordCount >= 50 → continue to Airtable

**Node 4B:** Selenium Branch (use Selenium setup guide)

## Benefits of This Approach

1. **Start with what works** (SearXNG)
2. **Simple and fast** (5 nodes)
3. **Easy to debug** (less moving parts)
4. **Add complexity only if needed** (Selenium fallback)

## Next: Add AI Processing

Once scraping works:
1. Add Ollama AI processing node
2. Generate summary, key facts, sentiment
3. Update more Airtable fields

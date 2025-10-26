# Workflow 2: Selenium Setup Guide

Based on https://github.com/Touxan/n8n-ultimate-scraper

## 🎯 Objective

Set up Selenium for content scraping in Workflow 2, integrated with your existing Airtable workflow.

## Step 1: Set Up Selenium (Docker)

### Option A: Run as Separate Service

```bash
# Start Selenium standalone Chrome
docker run -d \
  --name selenium_chrome \
  --network bridge \
  -p 4444:4444 \
  selenium/standalone-chrome:129.0

# Verify it's running
curl http://localhost:4444/wd/hub/status
```

### Option B: Integrate with Your n8n

If you're running n8n in Docker, add to your docker-compose.yml:

```yaml
services:
  selenium:
    image: selenium/standalone-chrome:129.0
    container_name: selenium_chrome
    ports:
      - "4444:4444"
    networks:
      - your_network
```

## Step 2: Build Simplified Workflow 2

### Workflow Structure (7 Nodes)

```
1. Manual Trigger
2. Airtable Query (get articles with Status = needs_scraping)
3. Code: Prepare Selenium Session (create session, get URL)
4. Selenium: Navigate to URL
5. Selenium: Extract Content
6. Code: Clean & Parse Content
7. Airtable Update (update Status and FullText)
```

## Step 3: Node Configurations

### Node 1: Manual Trigger
- Type: Manual Trigger
- No configuration needed

### Node 2: Query Airtable
- Type: Airtable
- Operation: List - Search
- Filter: `AND(({Status} = "needs_scraping"), {ScrapingAttempts} < 3)`
- Limit: 5 (start small)
- Sort: PubDate Descending

### Node 3: Create Selenium Session
- Type: HTTP Request
- Method: POST
- URL: `http://localhost:4444/wd/hub/session`
- Body (JSON):
```json
{
  "capabilities": {
    "alwaysMatch": {
      "browserName": "chrome",
      "goog:chromeOptions": {
        "args": [
          "headless",
          "no-sandbox",
          "disable-dev-shm-usage"
        ]
      }
    }
  }
}
```

### Node 4: Navigate to Article URL
- Type: HTTP Request
- Method: POST
- URL: `http://localhost:4444/wd/hub/session/{{ $('Create Selenium Session').item.json.value.sessionId }}/url`
- Body (JSON):
```json
{
  "url": "={{$json.fields.URL}}"
}
```

### Node 5: Extract Content
- Type: HTTP Request
- Method: POST
- URL: `http://localhost:4444/wd/hub/session/{{ $('Create Selenium Session').item.json.value.sessionId }}/execute/sync`
- Body (JSON):
```json
{
  "script": "return document.querySelector('article') ? document.querySelector('article').innerText : document.body.innerText",
  "args": []
}
```

### Node 6: Clean Content & Update Status
- Type: Code
- Mode: Run Once for Each Item

```javascript
// Extract content from Selenium response
const seleniumResponse = $input.item.json.value;

// Basic cleaning
const content = seleniumResponse.toString()
  .replace(/\s+/g, ' ')
  .trim();

const wordCount = content.split(/\s+/).length;

// Return for Airtable update
return {
  id: $('Query Airtable').item.json.id,
  FullText: content,
  WordCount: wordCount,
  Status: wordCount > 100 ? 'scraped' : 'scraping_failed',
  ScrapingAttempts: ($('Query Airtable').item.json.fields.ScrapingAttempts || 0) + 1,
  LastScrapingAttempt: new Date().toISOString(),
  ScrapingStrategy: 'selenium',
  ScrapingError: wordCount > 100 ? '' : 'Insufficient content'
};
```

### Node 7: Update Airtable
- Type: Airtable
- Operation: Update
- Record ID: `={{$json.id}}`
- Options: Typecast = Enabled

**Fields:**
- FullText
- WordCount
- Status
- ScrapingAttempts
- LastScrapingAttempt
- ScrapingStrategy
- ScrapingError

## Step 4: Add Session Cleanup

After Node 7, add cleanup node:

### Node 8: Delete Session
- Type: HTTP Request
- Method: DELETE
- URL: `http://localhost:4444/wd/hub/session/{{ $('Create Selenium Session').item.json.value.sessionId }}`

## Testing

1. Start Selenium: `docker run -d --name selenium_chrome -p 4444:4444 selenium/standalone-chrome:129.0`
2. Verify: `curl http://localhost:4444/wd/hub/status`
3. Run Workflow 2 manually
4. Check Airtable for updated articles

## Troubleshooting

### Selenium not accessible
- Check if container is running: `docker ps`
- Check if port 4444 is open: `netstat -an | grep 4444`
- Try: `curl http://localhost:4444`

### No content extracted
- Website might have blocked Selenium
- Check if site requires cookies/authentication
- Try adding wait time before extraction

### Resource usage high
- Selenium is resource intensive
- Limit batch size to 5-10 articles
- Run less frequently (every 2 hours instead of every hour)

## Next Steps

Once basic scraping works:
1. Add error handling
2. Add AI processing (Ollama)
3. Add SearXNG fallback
4. Optimize for production

## Alternative: Use SearXNG Instead

Since you mentioned SearXNG is working, consider using it as your primary scraper since Selenium is heavy.

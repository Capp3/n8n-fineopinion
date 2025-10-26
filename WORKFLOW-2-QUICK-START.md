# Workflow 2: Quick Start Guide

## Overview
Workflow 2 scrapes content and processes with AI for articles marked `needs_scraping` in Airtable.

## Node Sequence (9 nodes total)

1. **Manual Trigger** - For testing
2. **Airtable Query** - Get articles with Status = needs_scraping (limit 20)
3. **Code: Select Strategy** - Pick scraping strategy (HTTP/Playwright/SearXNG)
4. **Switch** - Route by strategy
5. **HTTP Request** (Branch A) - Simple scraping
6. **Playwright Docker** (Branch B) - Browser-based scraping  
7. **SearXNG** (Branch C) - Fallback search
8. **Code: Extract Content** - Clean HTML, extract text
9. **AI Processing** - Ollama analysis
10. **Code: Parse AI** - Extract JSON
11. **Airtable Update** - Update Status to processed/failed

## Quick Start Steps

### Step 1: Create New Workflow
- Open n8n
- Create new workflow named "ArticleScraper" or "Workflow 2"
- Add Manual Trigger node

### Step 2: Query Airtable
Add Airtable node:
- Operation: List with filter
- Filter: `AND(({Status} = "needs_scraping"), {ScrapingAttempts} < 3)`
- Limit: 20
- Sort: PubDate descending

### Step 3: Select Scraping Strategy
Add Code node:
- Run: Once per item
- See docs/IMPLEMENT-WORKFLOW-2.md for full code (Node 3)

### Step 4: Switch by Strategy
Add Switch node with 3 rules:
- Rule 1: `{{$json._strategy}} equals 'http'` → Output 0
- Rule 2: `{{$json._strategy}} equals 'playwright'` → Output 1  
- Rule 3: `{{$json._strategy}} equals 'searxng'` → Output 2

### Step 5: HTTP Scraper (Simple Test First)
Add HTTP Request node (connect to Switch output 0):
- URL: `={{$json.URL}}`
- Method: GET
- See docs/IMPLEMENT-WORKFLOW-2.md for headers and config

### Step 6: Extract Content
Add Code node (connect all scraper branches):
- Run: Once per item
- See docs/IMPLEMENT-WORKFLOW-2.md for full code (Node 6)

### Step 7: AI Processing
Add Ollama node OR HTTP Request to Ollama:
- Model: llama3.2:3b
- See docs/IMPLEMENT-WORKFLOW-2.md for prompt (Node 7)

### Step 8: Parse AI Response
Add Code node:
- Run: Once per item
- See docs/IMPLEMENT-WORKFLOW-2.md for full code (Node 8)

### Step 9: Update Airtable
Add Airtable node:
- Operation: Update by record ID
- Record ID: `={{$json.id}}`
- Enable Typecast
- Map all fields from parse step

## Testing Strategy
Start simple: Test with only HTTP scraper first, then add Playwright later.

## Full Documentation
See: docs/IMPLEMENT-WORKFLOW-2.md for complete code and configs


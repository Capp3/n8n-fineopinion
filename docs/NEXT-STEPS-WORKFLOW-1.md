# Next Steps: Complete Workflow 1 (Article Fetching)

**Last Updated:** October 25, 2025  
**Current Status:** Workflow 1 is 95% complete - just need to add Status fields to Airtable  
**Estimated Time:** 10 minutes

---

## 🎯 What You Need to Do

Complete Workflow 1 by adding the new Status management fields to your Airtable Articles table.

---

## 📋 Step-by-Step Guide

### Step 1: Open Your Airtable Base

1. Go to Airtable.com
2. Open your FinOpinions base
3. Navigate to the `Articles` table

### Step 2: Add Status Management Fields

Add these 5 new fields to the `Articles` table:

#### Field 1: Status

- **Field Type:** Single Select
- **Options:**
  - `needs_scraping` (default)
  - `scraping_in_progress`
  - `scraped`
  - `scraping_failed`
  - `processed`
- **Default Value:** `needs_scraping`
- **Color Coding:**
  - needs_scraping: Yellow
  - scraping_in_progress: Blue
  - scraped: Green
  - scraping_failed: Red
  - processed: Gray

#### Field 2: ScrapingAttempts

- **Field Type:** Number
- **Format:** Integer
- **Default Value:** 0
- **Description:** "Number of scraping attempts (max 3)"

#### Field 3: LastScrapingAttempt

- **Field Type:** Date
- **Include Time:** Yes
- **Time Zone:** Use local time zone
- **Description:** "Timestamp of last scraping attempt"

#### Field 4: ScrapingError

- **Field Type:** Long Text
- **Description:** "Error message if scraping failed"

#### Field 5: ScrapingStrategy

- **Field Type:** Single Select
- **Options:**
  - `http`
  - `playwright`
  - `selenium`
  - `searxng`
- **Description:** "Strategy used for scraping"

### Step 3: Update Node 18 in n8n

1. Open your n8n workflow "ArticleVacuum" (Workflow 1)
2. Click on Node 18 (Airtable Upsert)
3. Add these field mappings to your existing configuration:

```javascript
// Add these to your existing field mappings:
"Status": "needs_scraping",
"ScrapingAttempts": 0,
"LastScrapingAttempt": "",
"ScrapingError": "",
"ScrapingStrategy": ""
```

**Visual Steps:**

1. In Node 18, scroll down to "Fields to Send"
2. Click "Add Field"
3. For each new field:
   - Select the field name from dropdown
   - Enter the value as shown above
4. Click "Execute Node" to test
5. Save workflow

### Step 4: Test the Complete Workflow

1. Click "Execute Workflow" in n8n
2. Verify that articles are stored in Airtable
3. Check that new fields are populated:
   - Status should be `needs_scraping`
   - ScrapingAttempts should be `0`
   - Other status fields should be empty

### Step 5: Verify Upsert Behavior

1. Run the workflow again
2. Check that duplicate articles are updated (not duplicated)
3. Verify that the Status remains `needs_scraping` for existing articles

---

## ✅ Success Criteria

After completing these steps, you should have:

- [ ] 5 new Status management fields in Airtable Articles table
- [ ] Node 18 updated with new field mappings
- [ ] Workflow 1 successfully storing articles with Status = `needs_scraping`
- [ ] Upsert behavior working (no duplicate articles)
- [ ] All 7 RSS feeds processing successfully

---

## 🎉 What's Next?

Once Workflow 1 is complete and tested:

### Immediate Next Steps:

1. **Run Workflow 1 for 24 hours** to collect articles
2. **Monitor the Airtable** to ensure articles are being stored correctly
3. **Verify Status distribution** (all should be `needs_scraping`)

### Then Build Workflow 2:

1. Follow the tasks in `tasks.md` → Task 1.1
2. Create new workflow "Workflow 2 - Content Scraping"
3. Implement scraping logic with adaptive strategies
4. Test with articles from Workflow 1

**📖 Reference:** See [ADR-001](../docs/architecture-decisions/ADR-001-separated-workflows.md) for complete Workflow 2 implementation details.

---

## 🔧 Troubleshooting

### Issue: "Field not found" error in Node 18

**Solution:** Make sure you've added all 5 fields to Airtable first, then refresh the node connection.

### Issue: Duplicate articles still being created

**Solution:**

1. Verify Node 18 Operation is set to "Upsert"
2. Verify "Merge Field" is set to "URL"
3. Check that URL field is unique in Airtable

### Issue: Status field not being set

**Solution:** Check the field mapping in Node 18. The value should be exactly `needs_scraping` (lowercase, with underscore).

---

## 📊 Expected Results

After 24 hours of Workflow 1 running every 15 minutes:

- **Articles in Airtable:** 50-200 articles
- **Status Distribution:**
  - needs_scraping: 50-200 (100%)
  - All other statuses: 0
- **Sources Represented:** All 7 sources (ECB, MarketWatch, NASDAQ, BNP Paribas, Finance Monthly, CNBC, Money)
- **Duplicates:** 0 (Upsert prevents duplicates)

---

## 💡 Pro Tips

1. **Use Airtable Views** to organize articles by Status:

   - Create view "Ready for Scraping" (Status = needs_scraping)
   - Create view "Failed Scraping" (Status = scraping_failed)
   - Create view "Processed" (Status = processed)

2. **Set up Airtable Notifications** for scraping failures:

   - Get notified when articles hit Status = scraping_failed
   - Monitor ScrapingAttempts ≥ 3

3. **Add Formula Fields** for insights:
   - "Days Since Fetched": `DATETIME_DIFF(NOW(), {FetchedAt}, 'days')`
   - "Needs Attention": `IF({ScrapingAttempts} >= 3, '⚠️', '')`

---

**🚀 You're almost there! Just add those 5 fields to Airtable and you'll have Workflow 1 complete!**

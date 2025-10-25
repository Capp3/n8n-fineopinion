# Implementation Guide: URLHash Deduplication & Auto-Mapping

**Date:** October 25, 2025  
**Purpose:** Implement URLHash-based deduplication with automatic Airtable field mapping  
**Time Required:** ~15 minutes

---

## 🎯 What Changed

### Key Updates:

1. **URLHash Generation**: Node 12 now generates a hash from the URL for reliable deduplication
2. **Auto-Mapping**: Field names in Node 13 now match Airtable exactly for automatic mapping
3. **IF/ELSE Logic**: Existing records keep their Status, new records get `Status = needs_scraping`
4. **Date Format Fix**: Dates now formatted correctly for Airtable (with Typecast enabled)

---

## ✅ Step-by-Step Implementation

### Step 1: Update Airtable Table (5 minutes)

Add the `URLHash` field to your Articles table:

1. Open your Airtable base
2. Go to the **Articles** table
3. Click **"+"** to add a new field
4. Name it: **`URLHash`**
5. Type: **Single Line Text**
6. Save

**Why URLHash?**

- More reliable than URL (handles tracking parameters)
- Fixed length (better for database indexing)
- Required for Airtable upsert matching

---

### Step 2: Update Node 12 (Metadata Processor) - 3 minutes

Replace the code in **Node 12** with the updated version from `fineopinions_node_settings.md`:

**What's New:**

- Added `generateUrlHash()` function
- Generates hash using djb2 algorithm
- Adds `urlHash` field to item data

**Quick Copy:**

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

---

### Step 3: Update Node 13 (Field Validator & Mapper) - 3 minutes

Replace the `mapToAirtableSchema()` function in **Node 13**:

**What's New:**

- Field names now match Airtable exactly (e.g., `Headline` instead of `Title`)
- Added `URLHash` to mapping
- Added `Status` and `ScrapingAttempts` fields
- Removed deprecated fields

**Key Changes:**

- ✅ `Title` → `Headline` (matches Airtable)
- ✅ Added `URLHash` field
- ✅ Added `Status: "needs_scraping"`
- ✅ Added `ScrapingAttempts: 0`
- ✅ Added `WordCount` and `TokenCount`

**See full code in `fineopinions_node_settings.md` Node 13**

---

### Step 4: Update Node 18 (Airtable Upsert) - 4 minutes

**Change 1: Merge Field**

1. Open Node 18 in n8n
2. Find **"Columns to Match On"**
3. Change from `GUID` or `URL` to **`URLHash`**

**Change 2: Enable Typecast**

1. Scroll to **"Options"**
2. Click **"Add Option"**
3. Select **"Typecast"**
4. Toggle **ON**

**Change 3: Mapping Mode**

1. Set **"Mapping Mode"** to **"Auto-map Input Data to Columns"**
2. Or manually map fields using expressions from `fineopinions_node_settings.md`

**Field Mappings (if manual):**

```javascript
{
  "URL": "={{$json.airtableData.URL}}",
  "URLHash": "={{$json.airtableData.URLHash}}",
  "GUID": "={{$json.airtableData.GUID}}",
  "Headline": "={{$json.airtableData.Headline}}",
  "Source": "={{$json.airtableData.Source}}",
  "PubDate": "={{$json.airtableData.PubDate}}",
  "FetchedAt": "={{$json.airtableData.FetchedAt}}",
  "Creator": "={{$json.airtableData.Creator}}",
  "Categories": "={{$json.airtableData.Categories}}",
  "ContentSnippet": "={{$json.airtableData.ContentSnippet}}",
  "Status": "={{$json.airtableData.Status}}",
  "ScrapingAttempts": "={{$json.airtableData.ScrapingAttempts}}",
  "RelevanceScore": "={{$json.airtableData.RelevanceScore}}",
  "WordCount": "={{$json.airtableData.WordCount}}",
  "TokenCount": "={{$json.airtableData.TokenCount}}"
}
```

---

## 🧪 Testing

### Test 1: URLHash Generation

1. Execute Node 12
2. Check output for `urlHash` field
3. Should see hex string like `"a3f8c92d"`

### Test 2: Field Mapping

1. Execute Node 13
2. Check `airtableData` object
3. Verify all fields are present and correctly named

### Test 3: Airtable Upsert

1. Execute Node 18
2. Check Airtable - should see new records
3. Verify `URLHash` field is populated

### Test 4: Duplicate Handling

1. Run workflow twice
2. Check Airtable - should NOT have duplicates
3. Verify `FetchedAt` is updated
4. Verify `Status` is NOT changed for existing records

---

## 🔄 Expected Behavior

### For NEW Articles:

- ✅ Creates new record in Airtable
- ✅ Sets `Status = "needs_scraping"`
- ✅ Sets `ScrapingAttempts = 0`
- ✅ Populates all fields

### For EXISTING Articles (seen before):

- ✅ Finds existing record by `URLHash`
- ✅ Updates `FetchedAt` (timestamp)
- ❌ Does NOT change `Status`
- ❌ Does NOT change `ScrapingAttempts`
- 🎯 Preserves workflow state

**This is the IF/ELSE logic you requested!**

---

## ❌ Common Issues & Fixes

### Issue 1: "Field URLHash not found"

**Fix:** Add `URLHash` field to Airtable (Single Line Text)

### Issue 2: "Unsupported field type to upsert"

**Fix:** Ensure `URLHash` is type "Single Line Text" in Airtable

### Issue 3: Duplicates still being created

**Fix:**

- Verify Node 18 is using `URLHash` as merge field
- Check Node 12 output includes `urlHash`
- Verify `URLHash` column exists in Airtable

### Issue 4: Date format errors

**Fix:** Enable "Typecast" option in Node 18

### Issue 5: Auto-mapping not working

**Fix:** Use manual mapping with expressions provided above

---

## 📊 Verification Checklist

After implementation, verify:

- [ ] Node 12 generates `urlHash` field
- [ ] Node 13 includes `URLHash` in `airtableData`
- [ ] Node 18 uses `URLHash` as merge field
- [ ] Node 18 has Typecast enabled
- [ ] Airtable has `URLHash` column (Single Line Text)
- [ ] Running workflow twice doesn't create duplicates
- [ ] New articles get `Status = needs_scraping`
- [ ] Existing articles keep their current Status

---

## 🎉 Success!

When complete, you'll have:

- ✅ Reliable deduplication using URLHash
- ✅ Automatic field mapping (no manual configuration needed)
- ✅ IF/ELSE logic (new records vs. existing records)
- ✅ Status management for Workflow 2
- ✅ Date format compatibility with Airtable

**Next Step:** Run Workflow 1 for 24 hours to collect articles, then build Workflow 2!

---

**Questions?** See `fineopinions_node_settings.md` for detailed node configurations.

# FineOpinions - Task List

**Last Updated:** October 25, 2025 (Airtable Documentation Fully Updated 📊)  
**Status:** Implementation Phase - Building Workflow 2 (Content Scraping & AI Processing)  
**Current Phase:** Airtable Setup Complete (36 fields documented) → Ready for Workflow 2 Build

---

## 📊 Latest Update: Airtable Documentation Complete

**Date:** October 25, 2025

### What Was Updated

✅ **Complete Airtable setup documentation overhaul**

- **36 fields** fully documented (was 17)
- Organized into **7 logical categories**
- **ADR-001 status management** integrated
- **Workflow-specific field mapping** documented
- **10-step creation guide** with field numbering
- **7 recommended Airtable views** for monitoring
- **Enhanced validation checklist** (34 items)
- Modern **API token setup** instructions

### Documents Updated

- ✅ [`docs/airtable-setup.md`](./docs/airtable-setup.md) - 1140 lines (+68% expansion)
- ✅ [`docs/AIRTABLE-UPDATE-SUMMARY.md`](./docs/AIRTABLE-UPDATE-SUMMARY.md) - Complete change summary

### Next Actions

1. Review updated Airtable documentation
2. Verify your Airtable has all 36 fields
3. Add any missing fields following the guide
4. Create the 7 recommended views
5. Continue with Workflow 2 implementation

---

## 🎯 Current Implementation Priority

### **🏗️ ARCHITECTURAL DECISION: Separated Workflows (ADR-001)**

**Decision Date:** October 25, 2025  
**Status:** Accepted and Fully Documented 📋  
**Impact:** High - Core Architecture Change

**Key Change:** The system is now split into **two independent workflows**:

1. **Workflow 1: Article Fetching & Storage** (Current focus - Almost complete)

   - Fetch RSS feeds every 15 minutes
   - Normalize and validate data (4-stage pipeline)
   - Store in Airtable with `Status = needs_scraping`
   - Fast and lightweight (~30-60 seconds)

2. **Workflow 2: Content Scraping & AI Processing** (Next implementation)
   - Query articles with `Status = needs_scraping`
   - Scrape full content with adaptive strategies (HTTP/Playwright/SearXNG)
   - Process with AI agents
   - Update status to `processed`
   - Run every hour or on-demand

**Benefits:**

- 🎯 70-80% reduction in HTTP requests (no scraping for duplicates)
- 🛡️ Failure isolation (scraping failures don't block RSS fetching)
- ⏱️ Flexible scheduling for each workflow
- 📈 Independent scaling

**📖 Full Documentation:** [ADR-001: Separated Workflows](./docs/architecture-decisions/ADR-001-separated-workflows.md)

---

### **Task 1.0: WORKFLOW 1 - RSS Fetching & Storage Pipeline** 📊

**Goal:** Implement 4-stage RSS feed normalization pipeline with Airtable upsert  
**Status:** Almost Complete - Need to add Status fields to Airtable ✅

**Prerequisites:** Creative design complete, node settings documented, ADR-001 documented ✅

**Sub-Tasks:**

- [x] **Sub-Task 1.1:** Create n8n workflow with 7 RSS feed nodes
- [x] **Sub-Task 1.2:** Implement merge node configuration
- [x] **Sub-Task 1.3:** Fix URL constructor issue in Basic Field Normalizer (Stage 1)
- [x] **Sub-Task 1.4:** Build Content Processor (Stage 2)
- [x] **Sub-Task 1.5:** Build Metadata Processor (Stage 3)
- [x] **Sub-Task 1.6:** Fix data structure in Validator & Airtable Mapper (Stage 4)
- [ ] **Test:** Validate all 7 feeds process correctly

**Success Criteria:**

- All 7 RSS feeds normalized to consistent schema
- Content properly cleaned and processed
- Metadata extracted from all feed formats
- Validation catches and reports errors
- Data ready for Airtable storage

**Progress Notes:**

- ✅ Fixed Node 10 (Basic Field Normalizer): Replaced `new URL()` with string matching (URL constructor not available in n8n)
- ✅ All 7 RSS feeds configured and connecting to merge node
- ✅ Nodes 11-12 (Content & Metadata Processors) working
- ✅ Fixed Node 13 (Validator): Changed `item.json.isValid` to `item.isValid` (data structure issue)
- ⚠️ **CRITICAL FIX:** Node 9 Merge must use "Append" mode, not "Combine By Position" (was limiting to 5 items)
- ✅ **WORKFLOW FIX:** Removed old Node 14 (Dedup Check) - blocking flow with empty results. Using Airtable Upsert instead!
- 🔄 Testing complete post-processing pipeline (Nodes 9-13)
- 🔄 Ready for content scraping (Node 14+)

---

## 🎯 Next Priority Tasks

### **Task 1.1: WORKFLOW 2 - Content Scraping & AI Processing Pipeline** 🕷️

**Goal:** Build independent workflow to scrape content and process with AI agents  
**Status:** 🚀 IN PROGRESS - Implementation Guide Created  
**Architecture:** Separated Workflows (ADR-001)  
**Implementation Guide:** [docs/IMPLEMENT-WORKFLOW-2.md](./docs/IMPLEMENT-WORKFLOW-2.md)

**Prerequisites:**

- [x] Workflow 1 (RSS Fetching) operational
- [x] Airtable Articles table has Status management fields
- [ ] Airtable has test articles with `Status = needs_scraping`

**Sub-Tasks:**

#### Phase 1: Airtable Query & Strategy Selection

- [ ] **Sub-Task 1.1.1:** Create new workflow "Workflow 2 - Content Scraping"
- [ ] **Sub-Task 1.1.2:** Add Airtable node to query articles
  - Filter: `Status = needs_scraping` AND `ScrapingAttempts < 3`
  - Sort: PubDate DESC
  - Limit: 50 (batch processing)
- [ ] **Sub-Task 1.1.3:** Add Code node for adaptive strategy selection
  - Select strategy based on source and previous attempts
  - Increment ScrapingAttempts
  - Update LastScrapingAttempt timestamp

#### Phase 2: Content Scraping Implementation

- [ ] **Sub-Task 1.1.4:** Implement HTTP Request scraper (simple sources)
  - ECB, Finance Monthly, BNP Paribas
  - User-Agent headers
  - Timeout: 10 seconds
- [ ] **Sub-Task 1.1.5:** Set up Playwright Docker container
  - Pull mcr.microsoft.com/playwright:latest
  - Test container execution
- [ ] **Sub-Task 1.1.6:** Implement Playwright scraper (anti-scraping sources)
  - MarketWatch, NASDAQ, CNBC, Money
  - Execute Command node with Docker
  - Handle dynamic content loading
- [ ] **Sub-Task 1.1.7:** Configure SearXNG fallback
  - Use existing SearXNG instance
  - Query by article title + source
  - Extract relevant content from results

#### Phase 3: AI Agent Processing

- [ ] **Sub-Task 1.1.8:** Build Desk Reporter prompt (Agent 1)
- [ ] **Sub-Task 1.1.9:** Process scraped content through Ollama
  - Model: llama3.2:3b
  - JSON output format
  - Error handling for malformed responses

#### Phase 4: Status Updates & Error Handling

- [ ] **Sub-Task 1.1.10:** Update Airtable after successful scraping
  - Status = `scraped`
  - Store scraped content in FullText field
  - Record ScrapingStrategy used
- [ ] **Sub-Task 1.1.11:** Handle scraping failures
  - Status = `scraping_failed`
  - Log error in ScrapingError field
  - Keep in queue for retry (if attempts < 3)
- [ ] **Sub-Task 1.1.12:** Update status after AI processing
  - Status = `processed`
  - Store AI output fields

#### Phase 5: Testing & Monitoring

- [ ] **Sub-Task 1.1.13:** Test with articles from all 7 sources
- [ ] **Sub-Task 1.1.14:** Verify strategy escalation (HTTP → Playwright → SearXNG)
- [ ] **Sub-Task 1.1.15:** Test retry logic for failed articles
- [ ] **Sub-Task 1.1.16:** Set up scheduled execution (every 1 hour)
- [ ] **Sub-Task 1.1.17:** Monitor scraping success rates by source
- [ ] **Sub-Task 1.1.18:** Document operational procedures

**Success Criteria:**

- Workflow 2 successfully processes 10-50 articles per run
- Adaptive scraping strategies working for each source
- Status transitions tracked in Airtable (needs_scraping → scraped → processed)
- Failed articles properly queued for retry with error logging
- Clear visibility into scraping performance by source
- Integration with Workflow 1 via Airtable status fields

**Expected Outcomes:**

- Complete separation of concerns between RSS fetching and content scraping
- Resource-efficient processing (only scrape new/updated articles)
- Resilient to scraping failures (doesn't block RSS pipeline)
- Flexible execution (manual trigger, scheduled, or sub-workflow)

**📖 Implementation Reference:** See [ADR-001](./docs/architecture-decisions/ADR-001-separated-workflows.md) for detailed node configurations and code snippets.

---

## 📋 Completed Tasks

### ✅ **Creative Phases Complete**

**RSS Post-Processing Architecture (October 25, 2025):**

- ✅ Analyzed 7 different RSS feed formats (ECB, MarketWatch, NASDAQ, BNP Paribas, Finance Monthly, CNBC, Money)
- ✅ Designed 4-stage normalization pipeline (Merge → Basic Normalizer → Content Processor → Metadata Processor → Validator)
- ✅ Created comprehensive n8n node configurations
- ✅ Updated system diagrams and documentation
- ✅ Documented implementation guidelines and testing approach

**Previous Completion (October 9, 2025):**

- ✅ Complete 4-agent pipeline architecture
- ✅ All 4 agent prompts (production-ready)
- ✅ Northern Irish editorial voice (60+ examples)
- ✅ Complete documentation organization
- ✅ Implementation guides and Airtable setup

---

## 🔄 Pipeline Tasks (Updated Architecture)

### **Task 1.2: Multi-Feed Orchestrator** 📊

**Goal:** Process all 7 feeds with staggered execution  
**Status:** Pending Task 1.1 completion

- [ ] Create orchestrator workflow (calls all 7 feeds with 1-min delays)
- [ ] Test with live RSS feeds
- [ ] Validate all 7 feeds populate Airtable
- [ ] Implement error handling and retry logic
- [ ] Performance optimization (target <15 min total processing)

---

### **Task 2.1: Journalist Agent Pipeline** 📝

**Goal:** Airtable → Journalist → Digests table  
**Status:** Architecture complete, pending data pipeline

**Prerequisites:** Airtable has 30+ test articles

- [ ] Create Airtable Digests table (Journalist fields)
- [ ] Build Airtable query (48-hour window, relevance >= 4)
- [ ] Implement Journalist prompt (facts-first synthesis)
- [ ] Add Wikipedia integration for fact verification
- [ ] Generate cohesive 500-700 word factual narrative
- [ ] Store in Digests table with metadata
- [ ] **Test:** Process 30 articles into 1 digest

---

### **Task 2.2: Editorial Agent Integration** ✍️

**Goal:** Add Northern Irish voice and personality  
**Status:** Prompts complete, integration pending

- [ ] Load Editorial prompt and voice examples
- [ ] Integration with Journalist output
- [ ] Generate engaging 750-word editorial
- [ ] Maintain factual base while adding personality
- [ ] Quality assurance (voice consistency)
- [ ] **Test:** 5 editorials, voice validation

---

### **Task 2.3: Copywriter Agent & Email** 📧

**Goal:** HTML email formatting and delivery  
**Status:** Prompts complete, email integration pending

- [ ] Implement Copywriter prompt
- [ ] HTML email template creation
- [ ] Subject line generation (edgy, engaging)
- [ ] Email delivery configuration
- [ ] Unsubscribe and preferences
- [ ] **Test:** End-to-end email delivery

---

## 🏗️ System Integration Tasks

### **Task 3.1: End-to-End Testing** 🧪

**Goal:** Complete 48-hour cycle validation

- [ ] Full pipeline testing (RSS → Email)
- [ ] Performance monitoring and optimization
- [ ] Error handling validation
- [ ] Data quality assurance
- [ ] **Test:** Complete 2-day cycle simulation

---

### **Task 3.2: Production Deployment** 🚀

**Goal:** Live system deployment

- [ ] Production environment setup
- [ ] Monitoring and alerting
- [ ] Backup and recovery procedures
- [ ] Documentation finalization
- [ ] Go-live checklist

---

## 📊 Task Summary Statistics

**Total Tasks:** 15  
**Completed:** 2 ✅  
**In Progress:** 1 🟡  
**Pending:** 12 ⏳

**Current Focus:** RSS Post-Processing Implementation  
**Next Milestone:** Single feed end-to-end validation  
**Target Completion:** End-to-end system by November 2025

---

**Documentation References:**

- **Architecture:** `/fineopinions_diagram.md`
- **Node Settings:** `/fineopinions_node_settings.md`
- **Implementation:** `/docs/implementation-guide.md`
- **Database Setup:** `/docs/airtable-setup.md`

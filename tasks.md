# Tasks - FineOpinions Project

**Last Updated:** October 9, 2025 (ORGANIZATION COMPLETE ✅)  
**Current Phase:** Documentation Organized - Ready for BUILD MODE  
**Session Status:** REFLECTED + ORGANIZED  
**Next Mode:** BUILD MODE (n8n Implementation)

---

## 📊 Session Summary (October 9, 2025)

**Duration:** ~8 hours total  
**Modes:** PLAN → CREATIVE → IMPLEMENT → REFLECT → ORGANIZE  
**Deliverables:** 4 complete agent prompts (1,839 lines) + architecture (2,500+ lines) + organized docs  
**Status:** ✅ All objectives achieved and exceeded

**What We Built:**

- ✅ Complete 4-agent pipeline architecture
- ✅ All 4 agent prompts (production-ready)
- ✅ Northern Irish editorial voice (60+ examples)
- ✅ Complete documentation (~5,600 lines organized)
- ✅ Documentation organization (20 files, no duplication)
- ✅ Master index with role-based navigation
- ✅ Implementation guide + Airtable setup guide

**Ready For:** BUILD MODE - n8n workflow implementation

**Documentation Structure:**

- Root: 3 files (minimal, as requested)
- docs/: 6 files (implementation guides & reference)
- memory-bank/: 8 files (architecture & design)
- prompts/: 4 files (ready for n8n)

---

## 🎯 Active Tasks

### ✅ PHASE 1: PLANNING & PROMPTS - COMPLETE

- [x] **Multi-Agent Workflow Design** ✅ COMPLETE (CREATIVE MODE)
  - [x] Designed 4-agent pipeline (Desk Reporter → Journalist → Editorial → Copywriter)
  - [x] Created comprehensive multi-agent architecture document
  - [x] Defined "Facts First, Character Second" philosophy
  - [x] Documented all agent roles and responsibilities
  - [x] Created agent design summary document
- [x] **System Architecture Refinement** ✅ COMPLETE
  - [x] Refined workflow for every-other-day digest delivery (48-hour cycle)
  - [x] Designed complete content scraping flow (RSS → HTTP → Browser fallback)
  - [x] Created detailed Airtable schema (Articles + Digests tables)
  - [x] Implemented deduplication strategy (URL hash + title similarity)
  - [x] Defined pre-filtering logic (relevance >= 4, top 25 articles)
- [x] **Agent 1: Desk Reporter Prompt** ✅ COMPLETE (IMPLEMENT MODE)
  - [x] Created complete prompt with n8n variable syntax
  - [x] Defined relevance scoring guide (1-10 scale, threshold: 4)
  - [x] Specified model selection logic (llama3.2:3b vs qwen2.5:7b)
  - [x] Created JSON output schema for Airtable integration
  - [x] Added quality checklist and validation steps
  - [x] Document: `/prompts/desk_reporter.md` (305 lines)
- [x] **Agent 2: Journalist Prompt** ✅ COMPLETE (IMPLEMENT MODE)
  - [x] Created FACTS FIRST prompt (wire service style)
  - [x] Defined synthesis requirements (500-700 words)
  - [x] Specified input format (top 25 articles)
  - [x] Created article formatting function (n8n)
  - [x] Added Wikipedia tool integration note
  - [x] Documented chunking options for future optimization
  - [x] Document: `/prompts/journalist.md` (424 lines)
- [x] **Agent 3: Editorial Writer Prompt** ✅ COMPLETE (IMPLEMENT MODE)
  - [x] Created CHARACTER + FUN prompt with Northern Irish voice
  - [x] Defined accessibility requirements (Level 3 audience)
  - [x] Created comprehensive voice guide (60+ examples)
  - [x] Specified tone adaptation (hybrid approach)
  - [x] Allowed colorful language and gallows humor
  - [x] Added "what to watch" section design
  - [x] Document: `/prompts/editorial.md` (639 lines)
- [x] **Agent 4: Copywriter Prompt** ✅ COMPLETE (IMPLEMENT MODE)

  - [x] Created final polish and HTML formatting prompt
  - [x] Defined email structure (magazine-style)
  - [x] Specified target length (750-1000 words, 5-6 min read)
  - [x] Created HTML template with inline styles
  - [x] Added edgy subject line guide
  - [x] Added email delivery integration notes
  - [x] Document: `/prompts/copywriter.md` (471 lines)

- [x] **Session Reflection** ✅ COMPLETE (REFLECT MODE)
  - [x] Comprehensive reflection on session progress
  - [x] Lessons learned documented
  - [x] Design decisions reviewed
  - [x] Process improvements identified
  - [x] Readiness assessment completed
  - [x] Document: `/memory-bank/session-reflection-oct-9.md`

**TOTAL PROMPTS CREATED:** 4 agents, 1,839 lines of documentation  
**TOTAL DOCUMENTATION:** ~5,300 lines across 13 files

---

## 📋 PHASE 2: n8n IMPLEMENTATION - MODULAR BUILD (Next Session)

**Status:** ⏳ Ready to begin  
**Approach:** Build one module, test it, move to next  
**Strategy:** No feature creep - stick to 6 defined modules  
**Documentation:** `/memory-bank/build-implementation-plan.md` + `/docs/build-roadmap.md`

---

### 🏗️ MODULE 1: Single Feed Processor (Week 1, Days 1-4)

**Goal:** RSS → Scrape → Desk Reporter → Airtable (Economist ONLY)  
**Time:** 6-8 hours  
**Status:** Foundation - must work before proceeding

- [ ] **Sub-Task 1.1:** Create Airtable Articles table (13 minimum fields)
- [ ] **Sub-Task 1.2:** Build RSS fetch (Economist only)
- [ ] **Sub-Task 1.3:** Generate URL hash + title normalization
- [ ] **Sub-Task 1.4:** Implement deduplication check
- [ ] **Sub-Task 1.5:** Build content scraping (HTTP Request + HTML Extract)
- [ ] **Sub-Task 1.6:** Integrate Desk Reporter (AI Agent node)
- [ ] **Sub-Task 1.7:** Store in Airtable (all fields)
- [ ] **Test:** Process 5 Economist articles, validate Airtable entries

**Success:** ✅ One feed working end-to-end, template ready for duplication

---

### 🔄 MODULE 2: Multi-Feed Orchestrator (Week 1, Day 5)

**Goal:** Duplicate Module 1 for all 4 feeds + stagger  
**Time:** 2-3 hours  
**Prerequisites:** Module 1 working perfectly

- [ ] Duplicate Module 1 workflow (save as "Feed 2: Bloomberg")
- [ ] Duplicate Module 1 workflow (save as "Feed 3: Reuters")
- [ ] Duplicate Module 1 workflow (save as "Feed 4: MarketWatch")
- [ ] Change RSS URL in each workflow
- [ ] Create orchestrator workflow (calls all 4 with 1-min delays)
- [ ] Test with live RSS feeds
- [ ] Validate all 4 feeds populate Airtable

**Success:** ✅ All 4 feeds processing, 30-50 articles per cycle

---

### 📰 MODULE 3: Journalist Agent (Week 2, Days 6-7)

**Goal:** Airtable → Journalist → Digests table  
**Time:** 4-5 hours  
**Prerequisites:** Airtable has 30+ test articles

- [ ] Create Airtable Digests table (Journalist fields)
- [ ] Build Airtable query (48-hour window, relevance >= 4)
- [ ] Implement filter + sort logic
- [ ] Build article formatter (top 25 articles)
- [ ] Configure Journalist AI Agent node (with Wikipedia tool)
- [ ] Test synthesis with sample 25-article batch
- [ ] Validate JSON output
- [ ] Store in Digests table

**Success:** ✅ Journalist produces coherent factual synthesis

---

### ✍️ MODULE 4: Editorial Agent (Week 2, Days 8-9)

**Goal:** Add Northern Irish voice and perspective  
**Time:** 3-4 hours  
**Prerequisites:** Module 3 working, sample Journalist output

- [ ] Build Editorial AI Agent node
- [ ] Test with sample Journalist JSON
- [ ] Validate Northern Irish voice present
- [ ] Check tone adaptation to market mood
- [ ] Verify colorful language appropriately used
- [ ] Update Digests table with Editorial fields
- [ ] Test multiple market moods (bullish, bearish, uncertain)

**Success:** ✅ Editorial adds CHARACTER and is fun to read

---

### 📧 MODULE 5: Copywriter Agent (Week 2, Day 10)

**Goal:** Format to HTML and send email  
**Time:** 3-4 hours  
**Prerequisites:** Module 4 working, sample Editorial output

- [ ] Build Copywriter AI Agent node
- [ ] Test HTML generation
- [ ] Validate subject line is edgy (under 60 chars)
- [ ] Check HTML renders properly (validate code)
- [ ] Configure Send Email node
- [ ] Send test email to yourself
- [ ] Test rendering in Gmail, Outlook, Apple Mail
- [ ] Update Digests table with email fields

**Success:** ✅ Beautiful HTML email delivered successfully

---

### 🎭 MODULE 6: Digest Orchestrator (Week 3, Days 11-12)

**Goal:** Chain all agents with schedule  
**Time:** 2-3 hours  
**Prerequisites:** Modules 3, 4, 5 all working

- [ ] Create digest orchestrator workflow
- [ ] Configure schedule trigger (6AM every other day)
- [ ] Chain Journalist → Editorial → Copywriter
- [ ] Test with manual trigger first
- [ ] Run complete 48-hour simulation
- [ ] Validate end-to-end processing time (<20 min)
- [ ] Mark digest as "sent" in Airtable

**Success:** ✅ Automated digest generation working

---

### 🧪 TESTING & GO-LIVE (Week 3, Days 13-15)

- [ ] Run 2-3 complete digest cycles with real data
- [ ] Monitor all modules for errors
- [ ] Refine prompts based on actual outputs
- [ ] Optimize performance if needed
- [ ] Set production schedule
- [ ] Send first real digest
- [ ] Monitor and adjust

**Success:** ✅ FineOpinions is LIVE! 🎉

---

## 🎨 Creative Mode Tasks (Flagged)

### 1. Prompt Engineering (High Priority)

**Component**: AI Agent article analysis  
**Status**: Awaiting CREATIVE mode  
**Complexity**: Medium-High

**Design Decisions Needed**:

- Exact prompt structure and formatting (SYSTEM + USER)
- JSON output schema with validation
- Model-specific optimizations (llama3.2:3b vs qwen2.5:7b)
- Few-shot examples for consistency
- Relevance scoring criteria and weights
- Tag/topic taxonomy design

### 2. Content Extraction Strategy (Medium Priority)

**Component**: HTML parsing and scraping  
**Status**: Awaiting CREATIVE mode  
**Complexity**: Medium

**Design Decisions Needed**:

- Site-specific extraction rules (per source)
- Content identification heuristics
- Fallback strategy hierarchy
- Content quality validation metrics
- Metadata extraction patterns

### 3. Relevance Scoring Algorithm (Medium Priority)

**Component**: AI-based article filtering  
**Status**: Can be addressed in Prompt Engineering  
**Complexity**: Medium

**Design Decisions Needed**:

- Scoring criteria (1-10 scale definition)
- Topic classification system
- Threshold optimization (currently >= 5)
- Edge case handling

---

## 🚀 Future Features

### Deferred to Future Releases

- [ ] **Daily Digest Agent Implementation**

  - Aggregate articles into coherent daily summaries
  - Identify trends and connections
  - Output format: 3-5 minute read

- [ ] **Weekly Report Agent Implementation**

  - Analyze 7 days of daily digests
  - Identify recurring themes and patterns
  - Generate analytical weekly report
  - Email delivery

- [ ] **Data Retention Automation**

  - Purge raw articles after 30 days
  - Archive old records after 90 days
  - Maintain digests for 1 year

- [ ] **Social Media Monitoring** (Explicitly deferred)

  - Twitter/X feed monitoring
  - LinkedIn financial news tracking
  - Social sentiment analysis

- [ ] **Advanced Analytics**

  - Trend analysis across multiple weeks
  - Custom alert generation
  - Interactive dashboard

- [ ] **Enhanced Sources**

  - Additional RSS feed integration
  - Podcast transcript analysis
  - Video content analysis

- [ ] **Multi-user Support**
  - Customizable report preferences
  - Multiple delivery schedules
  - User-specific feed curation

---

## ✅ Completed Tasks

### October 9, 2025

- [x] RSS Feed Retrieval Architecture Planning
  - Created comprehensive planning document (memory-bank/rss-feed-architecture.md)
  - Designed 5 detailed component diagrams (Mermaid)
  - Defined Airtable schema for Articles table
  - Identified creative phase components
  - Documented challenges and mitigations
  - Created implementation checklist
  - Defined success metrics

### October 8, 2025

- [x] Initial project brief creation
- [x] VAN mode complexity assessment (Level 4)
- [x] Memory Bank folder structure created
- [x] tasks.md initialization
- [x] Complete Memory Bank setup (all 6 files created):
  - projectbrief.md
  - productContext.md
  - systemPatterns.md
  - techContext.md
  - activeContext.md
  - progress.md

---

## 🔄 Task Management Notes

- **Current Complexity Level:** Level 4 (Complex System)
- **Priority Levels:** High / Medium / Low
- **Blocking Issues:** None currently
- **Next Mode:** CREATIVE MODE (Prompt Engineering)
- **Dependencies:**
  - Phase 2 (Scraping) depends on Content Extraction Strategy (CREATIVE)
  - Phase 3 (AI Agent) depends on Prompt Engineering (CREATIVE)
  - Phases 4-5 depend on completion of Phases 1-3
- **Review Frequency:** Update after each significant progress milestone

---

## 📊 Mode Transition Plan

```
Current: PLAN MODE ✅ Complete
    ↓
Next: CREATIVE MODE 🎨
    - Task 1: Prompt Engineering (AI Agent)
    - Task 2: Content Extraction Strategy
    ↓
Then: BUILD MODE 🔨
    - Phase 1: RSS Retrieval & Basic Storage
    - Phase 2: Content Scraping Implementation
    - Phase 3: AI Agent Integration
    - Phase 4: Full Pipeline Integration
    ↓
Finally: QA MODE ✅
    - Phase 5: Monitoring & Refinement
```

---

**Last Planning Session:** October 9, 2025  
**Planning Document:** `/memory-bank/rss-feed-architecture.md`  
**Diagrams Created:** 7 comprehensive Mermaid diagrams  
**Components Identified:** 5 major components  
**Creative Tasks Flagged:** 3 tasks

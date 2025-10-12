# Tasks - FineOpinions Project

**Last Updated:** October 9, 2025 (End of Day)  
**Current Phase:** Implementation - Agent Prompts  
**Current Mode:** IMPLEMENT (Prompts 1-2 Complete, 3-4 Pending)  
**Status:** Ready for handoff / continuation

---

## 🎯 Active Tasks

### COMPLETED TODAY (October 9, 2025)

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
  - [x] Documented n8n workflow integration requirements
- [x] **Agent 2: Journalist Prompt** ✅ COMPLETE (IMPLEMENT MODE)
  - [x] Created FACTS FIRST prompt (wire service style)
  - [x] Defined synthesis requirements (500-700 words)
  - [x] Specified input format (top 25 articles)
  - [x] Created article formatting function (n8n)
  - [x] Added Wikipedia tool integration note
  - [x] Documented chunking options for future optimization
  - [x] Created comprehensive example output

### PENDING - Next Session

- [ ] **Agent 3: Editorial Writer Prompt** 🎨 IMPLEMENT MODE - NEXT
  - [ ] Create CHARACTER + FUN prompt
  - [ ] Define accessibility requirements (Level 3 audience)
  - [ ] Specify tone guidelines ("The Economist meets your smartest friend")
  - [ ] Create analysis and perspective structure
  - [ ] Add "what to watch" section design
  - [ ] Document integration with Journalist output
- [ ] **Agent 4: Copywriter Prompt** 🎨 IMPLEMENT MODE - AFTER EDITORIAL
  - [ ] Create final polish and formatting prompt
  - [ ] Define email structure (subject, preheader, body)
  - [ ] Specify target length (750-1000 words, 5-6 min read)
  - [ ] Create markdown/HTML formatting guidelines
  - [ ] Add email delivery integration notes

### Implementation Phase (Not Started)

- [ ] Set up scheduled trigger (7AM/7PM)
- [ ] Configure RSS Feed Read nodes for 4 sources:
  - [ ] The Economist: The World in Brief
  - [ ] Bloomberg: Five Things to Know
  - [ ] Reuters: Top News
  - [ ] MarketWatch: Top Stories
- [ ] Implement staggered execution (1-minute delays between feeds)
- [ ] Create Airtable "Articles" table with defined schema
- [ ] Implement RSS parsing and XML validation
- [ ] Test deduplication logic against Airtable
- [ ] Implement error logging and monitoring
- [ ] Test end-to-end RSS retrieval

### Phase 2: Content Scraping Implementation (BUILD MODE)

**Prerequisite**: Complete CREATIVE MODE for HTML extraction strategy

- [ ] Implement HTTP Request scraping (primary method)
- [ ] Test HTTP scraping against each news source
- [ ] Implement browser automation fallback (Playwright/Puppeteer)
- [ ] Implement SearXNG integration (optional fallback)
- [ ] Add content validation logic (min 100 chars)
- [ ] Implement retry logic (max 3 attempts)
- [ ] Test scraping success rates per source
- [ ] Document scraping patterns and issues

### Phase 3: AI Agent Integration (BUILD MODE)

**Prerequisite**: Complete CREATIVE MODE for prompt engineering

- [ ] Configure Ollama integration in n8n
- [ ] Implement AI Agent nodes with designed prompts
- [ ] Implement model selection logic (based on article size)
- [ ] Test with sample articles from each source
- [ ] Implement JSON output validation
- [ ] Implement retry logic for malformed outputs
- [ ] Test relevance scoring accuracy
- [ ] Optimize prompt based on test results
- [ ] Benchmark token usage and processing time

### Phase 4: Full Pipeline Integration (BUILD MODE)

- [ ] Connect all components end-to-end
- [ ] Implement comprehensive error handling throughout
- [ ] Add retry logic with exponential backoff
- [ ] Test with live RSS feeds (all 4 sources)
- [ ] Monitor performance and success rates
- [ ] Optimize for efficiency and resource usage
- [ ] Implement workflow metrics tracking
- [ ] Create admin summary email notification

### Phase 5: Monitoring & Refinement (QA MODE)

- [ ] Set up workflow monitoring dashboard
- [ ] Implement metrics collection:
  - [ ] RSS fetch success rate (target: >95%)
  - [ ] Article scraping success rate (target: >85%)
  - [ ] AI processing success rate (target: >90%)
  - [ ] Deduplication accuracy (target: >99%)
  - [ ] Airtable ingest success rate (target: >98%)
  - [ ] End-to-end processing time (target: <10 min)
- [ ] Test data retention policy (30 days for FullText)
- [ ] Create workflow documentation
- [ ] Create maintenance runbook
- [ ] Conduct final QA testing

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

# Project Handoff Document

**Date:** October 9, 2025 (End of Day)  
**Project:** FineOpinions - n8n Automated Financial Newsletter  
**Phase:** Agent Prompt Implementation  
**Status:** 50% Complete (2 of 4 agent prompts done)

---

## 🎯 Project Overview

FineOpinions is an automated financial news digest system that:

- Pulls from 4 RSS feeds twice daily (Economist, Bloomberg, Reuters, MarketWatch)
- Processes articles through a 4-agent AI pipeline
- Delivers an every-other-day email digest (48-hour news cycle)
- Uses Ollama (local LLMs) for all AI processing
- Stores data in Airtable

**Target Audience:** Intelligent readers who are NOT experts in economics/markets

---

## 📊 Current Status

### ✅ COMPLETED

#### 1. Multi-Agent Architecture Design (CREATIVE MODE)

- **Document:** `/memory-bank/multi-agent-workflow-design.md`
- **Summary:** `/docs/agent-design-summary.md`
- **Key Decision:** "Facts First, Character Second" philosophy
  - **Journalist** = FACTS ONLY (no opinion, no speculation)
  - **Editorial** = CHARACTER & FUN (perspective, personality)

#### 2. System Architecture

- **Document:** `/memory-bank/rss-feed-architecture.md`
- **Key Features:**
  - Every-other-day digest (48-hour cycle)
  - Content scraping: RSS → HTTP GET → Browser fallback
  - Deduplication: URL hash + title similarity
  - Pre-filtering: Relevance >= 4, top 25 articles to Journalist
  - Email delivery: 6AM on Day 3 (mornings)

#### 3. Database Schema (Airtable)

- **Table 1: Articles** (18 fields)
  - Desk Reporter output + metadata
  - URL hash for deduplication
  - Relevance score (1-10)
  - Full article text + analysis
- **Table 2: Digests** (30+ fields)
  - Journalist synthesis
  - Editorial analysis
  - Copywriter polish
  - Email delivery metadata

#### 4. Agent 1: Desk Reporter Prompt ✅

- **File:** `/prompts/desk_reporter.md`
- **Purpose:** Extract facts from individual articles
- **Model:** llama3.2:3b (< 1000 words) OR qwen2.5:7b (>= 1000 words)
- **Output:** JSON with headline, keyFacts, entities, dataPoints, relevanceScore, etc.
- **Threshold:** Relevance >= 4 passes to Journalist
- **Status:** **READY FOR IMPLEMENTATION**

#### 5. Agent 2: Journalist Prompt ✅

- **File:** `/prompts/journalist.md`
- **Purpose:** Synthesize top 25 articles into factual narrative
- **Model:** qwen2.5:7b
- **Philosophy:** FACTS FIRST - no opinion, no speculation
- **Output:** 500-700 word factual synthesis (JSON)
- **Tools:** Wikipedia (via n8n Agent Node)
- **Note:** May need chunking if context too large (test first)
- **Status:** **READY FOR IMPLEMENTATION**

### ⏳ PENDING

#### 6. Agent 3: Editorial Writer Prompt (NEXT TASK)

- **File:** `/prompts/editorial.md` (TO BE CREATED)
- **Purpose:** Add CHARACTER, perspective, make it FUN
- **Model:** qwen2.5:7b or qwen2.5:14b
- **Audience:** Level 3 (assume basic knowledge, explain advanced concepts)
- **Tone:** "The Economist meets your smartest friend"
- **Input:** Journalist's factual synthesis
- **Decision Needed:** Should Editorial also receive original articles? (Recommend: JSON only)

#### 7. Agent 4: Copywriter Prompt (AFTER EDITORIAL)

- **File:** `/prompts/copywriter.md` (TO BE CREATED)
- **Purpose:** Final polish for email delivery
- **Model:** qwen2.5:7b
- **Target:** 750-1000 words, 5-6 minute read
- **Output:** Email subject, preheader, HTML/Markdown body

#### 8. n8n Workflow Implementation (FUTURE)

- Build complete workflow in n8n 1.114.4
- Test content scraping against all 4 sources
- Configure Airtable tables
- Set up Ollama integration
- Test end-to-end with sample data

---

## 📁 Document Structure

### Root Directory

```
/README.md                          - Project overview
/tasks.md                           - Current task tracking
/fineopinions_diagram.md            - System diagrams index
/fineopinions_node_settings.md      - n8n node configurations (future)
```

### Memory Bank (`/memory-bank/`)

```
/projectbrief.md                    - Core project definition
/productContext.md                  - Product requirements
/systemPatterns.md                  - Technical patterns
/techContext.md                     - Technology stack
/activeContext.md                   - Current work context
/progress.md                        - Progress tracking
/rss-feed-architecture.md           - RSS workflow architecture
/multi-agent-workflow-design.md     - Complete agent design (896 lines)
```

### Documentation (`/docs/`)

```
/index.md                           - Documentation index
/agent-design-summary.md            - Quick agent reference
/handoff-october-9-2025.md          - THIS DOCUMENT
```

### Prompts (`/prompts/`)

```
/desk_reporter.md                   - ✅ Agent 1 prompt (305 lines)
/journalist.md                      - ✅ Agent 2 prompt (422 lines)
/editorial.md                       - ⏳ TO BE CREATED
/copywriter.md                      - ⏳ TO BE CREATED
```

---

## 🔑 Key Design Decisions

### 1. Delivery Frequency

- **Decision:** Every other day (48-hour cycle)
- **Rationale:** Not overwhelming, provides comprehensive coverage
- **Schedule:** RSS pulls 4x (7AM/7PM on Days 1-2), Digest sent 6AM Day 3

### 2. Multi-Agent Pipeline

- **Decision:** 4 specialized agents vs. 1-2 general agents
- **Benefits:** Modularity, quality, clear separation of facts vs. opinion
- **Tradeoffs:** More prompts to maintain, more LLM calls

### 3. Facts First Philosophy

- **Decision:** Journalist = FACTS ONLY, Editorial = CHARACTER
- **Rationale:** Factual integrity, clear voice, reader trust
- **Key Rule:** Journalist NEVER speculates, Editorial adds perspective

### 4. Relevance Threshold

- **Decision:** Score >= 4 passes to Journalist (was 5, lowered for inclusivity)
- **Top N:** 25 articles pre-filtered for Journalist
- **Final Output:** Journalist synthesizes ALL 25, picks 10-15 for final digest

### 5. Accessibility

- **Decision:** Level 3 audience (assume basic knowledge)
- **Level 2 concepts:** Explained only if they become pervasive
- **Approach:** Plain language, define jargon, use analogies

### 6. Tools

- **Decision:** Start without tools, add as needed
- **Exception:** Wikipedia for Journalist (factual verification)
- **Future:** Consider search tools for Editorial if needed

---

## 🛠️ Technical Stack

### Infrastructure

- **n8n:** 1.114.4 (self-hosted)
- **LLM:** Ollama (local)
  - llama3.2:3b (fast, small articles)
  - qwen2.5:7b (quality, large articles, synthesis)
- **Database:** Airtable
- **Email:** n8n Send Email Node

### RSS Sources

1. The Economist: The World in Brief
2. Bloomberg: Five Things to Know
3. Reuters: Top News
4. MarketWatch: Top Stories

### Variable Syntax (n8n 1.114.4)

```javascript
{
  {
    $json.fieldName;
  }
} // Access JSON field
{
  {
    $json.source;
  }
} // Example
```

---

## 🚀 Next Steps for Continuation

### Immediate (Next Session)

1. **Create Editorial Writer Prompt**

   - Reference: `/memory-bank/multi-agent-workflow-design.md` (lines 353-487)
   - Key focus: CHARACTER, FUN, accessibility
   - Input decision: JSON only OR JSON + articles?
   - Target: ~400 lines (similar to Journalist)

2. **Create Copywriter Prompt**

   - Reference: `/memory-bank/multi-agent-workflow-design.md` (lines 489-540)
   - Key focus: Polish, formatting, email structure
   - Target: ~300 lines

3. **Update Documentation**
   - Update `/memory-bank/progress.md` with completion
   - Update `/memory-bank/activeContext.md`
   - Mark tasks complete in `/tasks.md`

### Short Term (Week 1)

4. **Set Up n8n Workflow**

   - Create workflow template
   - Configure RSS Feed Read nodes (4 sources)
   - Implement content scraping (HTTP + fallback)
   - Add deduplication logic

5. **Configure Airtable**

   - Create Articles table (18 fields)
   - Create Digests table (30+ fields)
   - Set up API connection in n8n
   - Test CRUD operations

6. **Test Agent Prompts**
   - Test Desk Reporter with 10 sample articles per source
   - Test Journalist with sample 25-article batch
   - Validate JSON outputs
   - Refine prompts based on results

### Medium Term (Week 2-3)

7. **End-to-End Testing**

   - Run complete 48-hour cycle simulation
   - Test all 4 agents in sequence
   - Verify email output quality
   - Measure processing time

8. **Optimization**

   - Implement chunking if Journalist context too large
   - Optimize model selection
   - Refine relevance thresholds
   - Test scraping success rates

9. **Go Live**
   - Set up production schedule
   - Configure monitoring
   - Test email delivery
   - Monitor first few digest cycles

---

## 📝 Important Notes

### Potential Issues to Watch

1. **Context Window (Journalist)**

   - 25 articles = ~8,000-12,000 tokens
   - May need chunking (see `/prompts/journalist.md` lines 395-412)
   - Test first, optimize if needed

2. **Scraping Reliability**

   - Some sources may require browser automation
   - Implement retry logic (max 3 attempts)
   - Use RSS description as fallback

3. **Deduplication Accuracy**

   - URL hash should catch exact duplicates
   - Title similarity may have false positives
   - Monitor and adjust as needed

4. **LLM Output Consistency**
   - Validate JSON structure on every response
   - Implement retry logic for malformed outputs
   - Log failures for analysis

### Success Metrics

- **RSS Fetch Success:** >95%
- **Scraping Success:** >85%
- **AI Processing Success:** >90%
- **Deduplication Accuracy:** >99%
- **Airtable Ingest Success:** >98%
- **End-to-End Time:** <20 minutes per cycle
- **Email Read Time:** 5-6 minutes (750-1000 words)

---

## 🤝 Team Contact / Handoff

### Questions to Ask Before Continuing

1. **Editorial Writer Input:** Should it receive Journalist JSON only, or also original articles?
2. **Copywriter Format:** HTML, Markdown, or both?
3. **Email Service:** Using n8n Send Email or external service?
4. **Testing Environment:** Sample data available or need to create?
5. **Ollama Setup:** Models already installed and accessible to n8n?

### Files to Review Before Starting

1. `/memory-bank/multi-agent-workflow-design.md` - Complete agent specifications
2. `/prompts/desk_reporter.md` - Example of completed prompt structure
3. `/prompts/journalist.md` - FACTS FIRST philosophy in action
4. `/docs/agent-design-summary.md` - Quick reference

### Key Principles to Maintain

1. **Facts First, Character Second** - Never mix facts with opinion in Journalist
2. **Accessibility** - Level 3 audience, explain advanced concepts
3. **Modularity** - Each agent does ONE thing excellently
4. **n8n Native** - Use native nodes, avoid code blocks when possible
5. **Quality Over Speed** - Accuracy matters more than processing time

---

## 📞 Contact Information

**Project:** FineOpinions  
**Repository:** `/home/cappy/knight0_code/n8n-fineopinions`  
**Platform:** n8n 1.114.4 self-hosted  
**LLM:** Ollama (local)

**Status:** Ready for Agent 3 (Editorial Writer) implementation

---

**Last Updated:** October 9, 2025  
**Next Session:** Create Editorial Writer and Copywriter prompts  
**Estimated Time to Complete:** 2-3 hours for remaining prompts  
**Total Project Status:** ~60% planning/design complete, ready for implementation

# FineOpinions Implementation Guide

**Last Updated:** October 9, 2025  
**Status:** Ready for BUILD MODE  
**Read Time:** 15 minutes  
**Purpose:** Everything you need to start building

---

## 🎯 What is FineOpinions?

An automated financial news digest delivered **every other day** (48-hour cycle) featuring:

- **Trustworthy facts** (wire service journalism standards)
- **Actual personality** (Northern Irish wit, edgy humor)
- **Accessible language** (for intelligent non-experts)
- **5-6 minute read** (750-1000 words, HTML email)

**Voice:** The Economist meets your smartest, funniest mate from Belfast explaining economics over pints. 🍺

---

## 🏗️ System Architecture

### The 4-Agent Pipeline

```
Day 1-2: RSS (4 feeds, 7AM+7PM) → Scraper → Desk Reporter → Airtable
                                                              ↓
Day 3, 6AM: Query Airtable → Journalist → Editorial → Copywriter → Email
```

**Agent 1: Desk Reporter** - Extracts facts, scores relevance (threshold: >=4)  
**Agent 2: Journalist** - FACTS FIRST synthesis (NO opinion), Wikipedia tool  
**Agent 3: Editorial** - Northern Irish voice, CHARACTER & FUN  
**Agent 4: Copywriter** - HTML formatting, magazine-style, edgy subject lines

---

## 🔑 Core Philosophy

**FACTS FIRST, CHARACTER SECOND**

- 🗞️ **Journalist** = Wire service facts ONLY (no speculation)
- ✍️ **Editorial** = Perspective, wit, personality (makes you care)

**This separation ensures:**

- Factual integrity (trust the facts)
- Editorial freedom (enjoy the perspective)
- Clear boundaries (readers know what's what)

---

## 🛠️ Technical Stack

- **n8n:** v1.114.4 (self-hosted workflow automation)
- **LLM:** Ollama (local)
  - llama3.2:3b (fast, <1000 words)
  - qwen2.5:7b (quality, >=1000 words, synthesis)
- **Database:** Airtable (2 tables, 40+ fields)
- **Email:** n8n Send Email Node (HTML)
- **RSS Sources:** Economist, Bloomberg, Reuters, MarketWatch

---

## 🚀 Implementation Approach: 6 Modules

**Strategy:** Build one module, test it thoroughly, move to next  
**No feature creep:** Stick to the 6 modules defined  
**Full details:** `/memory-bank/build-implementation-plan.md`

### MODULE 1: Single Feed Processor (6-8 hours)

**RSS → Scrape → Desk Reporter → Airtable (Economist ONLY)**

- Build template workflow for one feed
- Test thoroughly
- This becomes template for all feeds

### MODULE 2: Multi-Feed Orchestrator (2-3 hours)

**Duplicate Module 1 for all 4 feeds + stagger**

- Copy Module 1 workflow 4 times
- Change RSS URLs
- Orchestrate with 1-min delays

### MODULE 3: Journalist Agent (4-5 hours)

**Airtable → Journalist → Digests table**

- Query articles (48h window, relevance >=4)
- Synthesize top 25 articles
- Wikipedia tool integration

### MODULE 4: Editorial Agent (3-4 hours)

**Add Northern Irish voice and perspective**

- Read Journalist output
- Add CHARACTER and FUN
- Validate voice consistency

### MODULE 5: Copywriter Agent (3-4 hours)

**Format to HTML and send email**

- Generate HTML from JSON
- Edgy subject lines
- Email delivery

### MODULE 6: Digest Orchestrator (2-3 hours)

**Chain all agents with schedule**

- 6AM every other day trigger
- Journalist → Editorial → Copywriter → Email

**Total:** 50-75 hours over 3 weeks

---

## ✅ Pre-Implementation Checklist

Before starting Module 1:

- [ ] Airtable account ready
- [ ] Airtable API key obtained
- [ ] n8n 1.114.4 running and accessible
- [ ] Ollama running and accessible
- [ ] llama3.2:3b model installed in Ollama
- [ ] qwen2.5:7b model installed in Ollama
- [ ] Test email address configured
- [ ] Read `/docs/airtable-setup.md` (database provisioning)
- [ ] Review `/prompts/desk_reporter.md` (first agent)
- [ ] Understand Module 1 from `/memory-bank/build-implementation-plan.md`

**All checked?** ✅ Ready to start building

---

## 📋 First Steps (Day 1)

### Step 1: Airtable Setup (1-2 hours)

**Guide:** `/docs/airtable-setup.md`

1. Create **Articles** table (13 minimum fields)
2. Create **Digests** table (basic structure)
3. Get API key from Airtable
4. Test connection in n8n

### Step 2: Create Sample Data (1 hour)

**Save 5-10 Economist articles for testing:**

- Vary topics (Fed policy, GDP, earnings, etc.)
- Vary lengths (short, medium, long)
- Save full article text
- Use for validating Desk Reporter

### Step 3: Start Module 1 (6-8 hours)

**Follow:** `/memory-bank/build-implementation-plan.md` - Sub-Tasks 1.1-1.7

1. RSS fetch (Economist only)
2. Generate URL hash
3. Deduplication check
4. Content scraping
5. Desk Reporter integration
6. Validate JSON
7. Store in Airtable

**Test with 5 articles → All stored in Airtable with valid JSON**

---

## 🎯 Success Metrics (Targets)

| Module   | Metric               | Target                  |
| -------- | -------------------- | ----------------------- |
| Module 1 | Economist RSS fetch  | >90% success            |
| Module 1 | Scraping success     | >500 words retrieved    |
| Module 1 | Desk Reporter JSON   | 100% valid              |
| Module 2 | All 4 feeds working  | 30-50 articles/cycle    |
| Module 3 | Journalist synthesis | 500-700 words, coherent |
| Module 4 | Northern Irish voice | Present and consistent  |
| Module 5 | HTML email           | Valid, renders properly |
| Module 6 | End-to-end time      | <20 minutes             |

---

## 📚 Key Reference Documents

### During Build

- `/memory-bank/build-implementation-plan.md` - Module details and sub-tasks
- `/docs/build-roadmap.md` - Visual progress tracking
- `/docs/airtable-setup.md` - Database schema reference
- `/tasks.md` - Checklist as you complete modules

### For Agent Prompts

- `/prompts/desk_reporter.md` - Copy System + User messages to n8n
- `/prompts/journalist.md` - Copy to n8n, connect Wikipedia tool
- `/prompts/editorial.md` - Copy to n8n, read voice guide first!
- `/prompts/copywriter.md` - Copy to n8n, use HTML template

### For Architecture Questions

- `/memory-bank/multi-agent-workflow-design.md` - Complete design
- `/memory-bank/rss-feed-architecture.md` - RSS workflow details
- `/fineopinions_diagram.md` - Visual diagrams

---

## ⚠️ Important Implementation Notes

### n8n Variable Syntax (v1.114.4)

```javascript
{
  {
    $json.fieldName;
  }
} // Correct
{
  {
    $json.url;
  }
} // Example
{
  {
    $json.fullText;
  }
} // Example
```

### Model Selection Logic

```javascript
// In Function node before AI Agent
const wordCount = $json.wordCount || 0;
const model = wordCount >= 1000 ? "qwen2.5:7b" : "llama3.2:3b";
```

### Wikipedia Tool Connection

- **Agent:** Journalist only
- **Setup:** In n8n AI Agent node, add Wikipedia to tools
- **Purpose:** Factual verification, historical context

### Voice Preservation

- **Editorial's Northern Irish wit is INTENTIONAL**
- **Copywriter must NOT sanitize**
- **Crude language stays (bollocks, shite, etc.)**
- **Gallows humor is fair game**

---

## 🎯 Anti-Feature-Creep Rules

### ONLY Building (Minimum Viable)

✅ 4 RSS feeds  
✅ HTTP scraping (fallback: RSS description)  
✅ Desk Reporter (basic validation)  
✅ Deduplication (URL hash)  
✅ All 4 agents  
✅ Email delivery (single recipient)

### NOT Building Yet

❌ Weekly reports  
❌ Data retention automation  
❌ Browser automation  
❌ Advanced error handling  
❌ Monitoring dashboard  
❌ Multiple recipients

**Stick to the plan. Build what's defined. Nothing more.**

---

## 🔄 When Things Go Wrong

### Module 1 Issues

**RSS fails:** Check RSS Feed Read node configuration, verify URL  
**Scraping fails:** Document which selectors work, use RSS description fallback  
**Desk Reporter fails:** Check Ollama is running, verify model name  
**Airtable fails:** Verify API key, check field names match schema

### General Debugging

1. Test each node individually (click "Execute Node")
2. Inspect output JSON at each step
3. Check n8n error messages
4. Verify Ollama is running: `curl http://localhost:11434/api/tags`
5. Check Airtable field types match (JSON as Long Text)

### Getting Help

- **Architecture questions:** `/memory-bank/multi-agent-workflow-design.md`
- **Module specifics:** `/memory-bank/build-implementation-plan.md`
- **Prompt issues:** Review relevant `/prompts/*.md` file
- **Voice questions:** `/prompts/editorial.md` voice guide

---

## 🎊 The FineOpinions Promise

**What makes this special:**

1. **Unique Voice** - Northern Irish wit (nobody else has this)
2. **Facts + Character** - Trustworthy AND enjoyable
3. **Accessible** - For intelligent people, not just experts
4. **Honest** - Calls out bullshit, admits uncertainty
5. **Fun** - Gallows humor makes financial news enjoyable

**Example Editorial voice:**

> "Right, so the Fed's done it again. Rates are up, markets are down, and we're all just along for the ride. Are they overdoing it? Probably. Will they stop? Not until something breaks. That's how this always ends."

---

## 🚀 Let's Build!

**Start with:**

1. Read `/docs/airtable-setup.md` (15 min)
2. Create Airtable tables (30 min)
3. Begin Module 1, Sub-Task 1.1 from `/memory-bank/build-implementation-plan.md`

**Work methodically. Test thoroughly. One module at a time.**

**This is going to be fun to build - and even more fun to read!** 🍀

---

**Last Updated:** October 9, 2025  
**Next Step:** Airtable provisioning  
**Reference:** All documents listed in `/docs/index.md`

# 🍀 FineOpinions - START HERE

**Last Updated:** October 9, 2025  
**Project Status:** Prompt Engineering Complete ✅ - Ready for BUILD MODE  
**Quick Start Time:** 15 minutes to understand the project

---

## 🎯 What is FineOpinions?

An **automated financial news digest** delivered every other day with:

- ✅ **Trustworthy facts** (wire service journalism)
- ✅ **Actual personality** (Northern Irish wit, edgy humor)
- ✅ **Accessible language** (for intelligent non-experts)
- ✅ **5-6 minute read** (750-1000 words)

**Think:** The Economist meets your smartest, funniest mate from Belfast explaining economics over pints. 🍺

---

## 📖 Read These 3 Documents (15 minutes total)

### 1. **Project Status** (3 min)

📄 `/PROJECT-STATUS.md`

- What's done, what's next
- Key documents index
- System overview

### 2. **Handoff Document** (10 min)

📄 `/docs/handoff-october-9-2025.md`

- Complete session summary
- All technical details
- Next steps for implementation

### 3. **Agent Design Summary** (5 min)

📄 `/docs/agent-design-summary.md`

- 4-agent pipeline explained
- "Facts First, Character Second" philosophy
- Why this approach works

---

## 🎭 The 4-Agent Pipeline

```
RSS Feeds → Scraper → [Agent 1] → [Agent 2] → [Agent 3] → [Agent 4] → Email
                      Desk        Journalist  Editorial   Copywriter
                      Reporter    (FACTS)     (CHARACTER) (POLISH)
```

### Agent 1: Desk Reporter

- **What:** Extracts facts from individual articles
- **Style:** Objective, structured data extraction
- **Output:** JSON with keyFacts, entities, relevanceScore
- **Prompt:** `/prompts/desk_reporter.md` (305 lines)

### Agent 2: Journalist

- **What:** Synthesizes 25 articles into factual narrative
- **Style:** FACTS FIRST - no opinion, no speculation
- **Output:** 500-700 word factual synthesis
- **Tools:** Wikipedia (for verification)
- **Prompt:** `/prompts/journalist.md` (424 lines)

### Agent 3: Editorial Writer

- **What:** Adds CHARACTER, perspective, makes it FUN
- **Style:** Northern Irish wit, edgy, crude humor allowed
- **Output:** Analysis with personality and edge
- **Prompt:** `/prompts/editorial.md` (639 lines) + 60 voice examples

### Agent 4: Copywriter

- **What:** Final polish and HTML formatting
- **Style:** Magazine-style email, edgy subject lines
- **Output:** Beautiful HTML email, 750-1000 words
- **Prompt:** `/prompts/copywriter.md` (471 lines)

---

## 🔑 Key Design Philosophy

**FACTS FIRST, CHARACTER SECOND**

🗞️ **Journalist** = FACTS ONLY

- Objective reporting
- No speculation or opinion
- Numbers, dates, specifics
- Wire service style

✍️ **Editorial** = CHARACTER & FUN

- Perspective and analysis
- Northern Irish wit
- Gallows humor allowed
- Makes you CARE (and smile)

**Why this works:**

- Trust (facts are reliable)
- Engagement (opinion is entertaining)
- Clarity (readers know what's what)
- Modularity (can adjust tone independently)

---

## 📁 Essential Documents

### If You're Implementing in n8n:

1. `/prompts/desk_reporter.md` - Copy into n8n AI Agent node
2. `/prompts/journalist.md` - Copy into n8n AI Agent node
3. `/prompts/editorial.md` - Copy into n8n AI Agent node (read voice guide!)
4. `/prompts/copywriter.md` - Copy into n8n AI Agent node

### If You Need Architecture Details:

1. `/memory-bank/multi-agent-workflow-design.md` - Complete design (967 lines)
2. `/memory-bank/rss-feed-architecture.md` - RSS workflow (785 lines)
3. `/fineopinions_diagram.md` - Visual diagrams (248 lines)

### If You're Tracking Progress:

1. `/tasks.md` - Current task status (293 lines)
2. `/memory-bank/progress.md` - Progress tracking (557 lines)
3. `/memory-bank/session-reflection-oct-9.md` - Session reflection

---

## ⚡ Quick Facts

- **RSS Sources:** 4 (Economist, Bloomberg, Reuters, MarketWatch)
- **Processing:** 7AM + 7PM daily (RSS pulls), 6AM every other day (digest)
- **Models:** llama3.2:3b (fast) + qwen2.5:7b (quality)
- **Database:** Airtable (2 tables, 40+ fields)
- **Email:** HTML, magazine-style, edgy subject lines
- **Voice:** Northern Irish, edgy, professional but fun
- **Target:** 750-1000 words, 5-6 minute read
- **Documentation:** ~5,300 lines (complete)

---

## 🚀 Next Steps (For Implementation)

### Week 1: Basic Setup

1. Create Airtable tables (schema in handoff doc)
2. Set up n8n workflow (RSS feeds + scheduling)
3. Create sample data for testing
4. Test Desk Reporter with sample articles

### Week 2: Agent Integration

5. Test all 4 agents with real data
6. Implement content scraping
7. Build complete pipeline
8. Validate JSON outputs

### Week 3: Testing & Go-Live

9. End-to-end testing (48-hour cycle)
10. Email delivery validation
11. Performance optimization
12. Production deployment

**Estimated Total:** 50-75 hours to production-ready system

---

## 🎯 What Makes This Special

1. **Unique Voice** - Northern Irish wit, not corporate bland
2. **Facts + Character** - Trustworthy AND enjoyable
3. **Accessible** - For intelligent non-experts
4. **Honest** - Calls out bullshit, admits uncertainty
5. **Fun** - Financial news people actually WANT to read

**This isn't just another newsletter. This is FineOpinions.** 🍀

---

## ⚠️ Important Notes

1. **Editorial voice is intentional** - crude language, gallows humor, edge
2. **Not for everyone** - target audience appreciates wit and honesty
3. **Wikipedia tool** - Journalist only (fact-check), not Editorial (opinion stays pure)
4. **Context window** - Journalist may need chunking (test with 25 articles first)
5. **n8n syntax** - Use `{{ $json.fieldName }}` (version 1.114.4)

---

## 📞 Questions?

**Architecture unclear?** Read `/memory-bank/multi-agent-workflow-design.md`  
**Need implementation details?** Read `/docs/handoff-october-9-2025.md`  
**Want to understand the voice?** Read `/prompts/editorial.md` voice guide  
**Need quick reference?** Read `/docs/agent-design-summary.md`

**Everything is documented. No gaps. Ready to build.** 🔨

---

**Status:** ✅ SESSION COMPLETE  
**Quality:** 🌟🌟🌟🌟🌟 Highly Successful  
**Readiness:** 95% (sample data + Airtable setup = 100%)  
**Confidence:** HIGH

**Let's build this.** 🚀

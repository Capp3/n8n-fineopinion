# FineOpinions Multi-Agent Design Summary

**Created:** October 9, 2025  
**Status:** CREATIVE MODE - Design Complete  
**Philosophy:** Facts First, Character Second

---

## 🎯 Core Philosophy

### The Two-Stage Approach

**Stage 1: JOURNALIST = FACTS FIRST** 🗞️

- Objective, factual reporting ONLY
- What happened, when, who, where, how much
- NO speculation, NO opinion, NO interpretation
- Think: Wire service / Associated Press style
- _"Just the facts. Always the facts."_

**Stage 2: EDITORIAL = CHARACTER & FUN** ✍️

- Perspective, analysis, personality
- WHY it matters, the "SO WHAT" factor
- Make it engaging, interesting, ENJOYABLE
- Add wit and character (but stay professional)
- Think: The Economist meets your smartest friend
- _"Make them care - and smile while reading."_

---

## 🎭 The 4-Agent Pipeline

```
RSS Feeds → Scraper → Desk Reporter → JOURNALIST → EDITORIAL → Copywriter → Email
                      (fact extraction) (FACTS)     (CHARACTER)  (polish)
```

### Agent 1: Desk Reporter

**Role**: Extract facts from individual articles  
**Focus**: Structured data extraction, relevance scoring  
**Output**: JSON with keyFacts, entities, sentiment, relevance

### Agent 2: Journalist

**Role**: FACTS FIRST - Synthesize articles into factual narrative  
**Focus**: What happened, no interpretation  
**Rules**:

- NO speculation ("The Fed raised rates" not "This might mean...")
- NO opinion or analysis (save for Editorial)
- Numbers, dates, entities - specifics matter
- If sources conflict, report both facts
- Active, declarative statements only

**Output**: Factual synthesis with:

- Executive summary (facts only)
- Top stories (what happened)
- Data highlights (specific numbers)
- Market movements (observed, not predicted)

### Agent 3: Editorial Writer

**Role**: Add CHARACTER, perspective, make it FUN  
**Focus**: Why it matters, make readers care  
**Style**:

- Professional but conversational
- Smart but accessible
- Wit and personality encouraged
- "The Economist meets your smartest friend"
- NO jargon unless necessary (then explain it)
- Make complex ideas simple and engaging

**Output**: Editorial analysis with:

- Opening lede (with personality)
- Analysis points (why it matters)
- Big picture perspective
- Watch list (what to monitor)
- Provocative questions

### Agent 4: Copywriter

**Role**: Final polish for email delivery  
**Focus**: Formatting, readability, 3-5 min read time  
**Output**: Polished email-ready digest

---

## ✅ Why This Works

### Separation of Concerns

- **Facts** and **Opinion** never mixed
- Clear editorial voice without compromising factual integrity
- Readers know what's fact vs. perspective

### Modularity

- Can adjust character/tone without touching facts
- Each agent can be tested/refined independently
- Easy to replace or skip agents if needed

### Quality

- Each agent excels at ONE thing
- Specialized prompts > general-purpose prompts
- Journalist optimized for accuracy
- Editorial optimized for engagement

### Flexibility

- Can skip Editorial for "just the facts" mode
- Can adjust Editorial tone for different audiences
- Parallel processing of Desk Reporters for speed

---

## 🎨 Design Decisions

**Why Journalist comes first (before Editorial)?**

- Facts are the foundation
- Editorial builds on solid factual base
- Ensures accuracy before adding perspective

**Why separate Editorial from Copywriter?**

- Editorial = WHAT to say (substance)
- Copywriter = HOW to say it (presentation)
- Different skills, different optimization

**Why emphasize "FUN" in Editorial?**

- Financial news doesn't have to be boring
- Engagement drives readership
- Smart analysis can be enjoyable
- FineOpinions differentiator: opinionated + enjoyable

---

## 📊 Success Metrics

**Journalist Success:**

- Factual accuracy (no speculation)
- Clear, crisp reporting
- Proper data citation

**Editorial Success:**

- Engagement (did readers enjoy it?)
- Insight value (did they learn something?)
- Voice consistency (is it "FineOpinions"?)

**Overall Success:**

- 3-5 minute read time
- Readers look forward to it
- Facts are trusted
- Analysis is valued

---

## 🚀 Next Steps

1. Implement Journalist prompt with FACTS FIRST emphasis
2. Implement Editorial prompt with CHARACTER emphasis
3. Test with sample articles
4. Refine prompts based on output quality
5. Iterate until voice is consistent and engaging

---

**Key Insight**: _"The facts tell you WHAT happened. The editorial tells you WHY you should care - and makes you smile while reading."_

**This is going to be fun!** 🎉

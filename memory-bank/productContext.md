# Product Context: FineOpinions

**Last Updated:** October 8, 2025

---

## Product Vision

FineOpinions is an automated financial newsletter system that delivers curated, AI-analyzed economic and financial news insights without the noise of high-volume sources. It's designed for individuals who need a high-level "need-to-know" overview with minimal time investment.

---

## Target User

**Primary Persona:** The Informed Decision-Maker

- Needs: Stay informed on economic/financial trends without information overload
- Time Constraint: Maximum 3-5 minutes for daily digest, 10-15 minutes for weekly report
- Technical Level: Non-technical; expects email delivery
- Value: Quality over quantity, analysis over raw news

---

## Core Features (In Scope)

### 1. Automated RSS Feed Ingestion

**Description:** Twice-daily scheduled ingestion from three curated sources  
**User Value:** Consistent, reliable information gathering without manual intervention  
**Technical Approach:** n8n RSS Feed Read Node + HTTP Request Node

**Sources:**

- The Economist: The World in Brief
- Bloomberg: Five Things to Know
- Reuters: Top News

### 2. Intelligent Content Scraping

**Description:** Full article text extraction when RSS provides only summaries  
**User Value:** Complete context for AI analysis  
**Technical Approach:**

- Primary: HTTP Request Node
- Fallback: Browser automation (if needed for dynamic content)

### 3. Daily Digest Generation

**Description:** AI-powered summary of 12-hour news cycle  
**User Value:** Coherent, contextualized overview of daily events  
**AI Agent:** Daily Digest Creator

- Processes all new articles from 12-hour window
- Generates single coherent summary
- Identifies key findings and trends

### 4. Weekly Analytical Report

**Description:** Higher-level analysis connecting daily digests  
**User Value:** Strategic perspective on weekly trends and narratives  
**AI Agent:** Weekly Report Analyst

- Reviews 7 days of daily digests
- Identifies recurring themes
- Connects disparate events
- Provides analytical summary

### 5. Email Delivery

**Description:** Automated email delivery of weekly reports  
**User Value:** Convenient, expected delivery format  
**Technical Approach:** n8n Send Email Node

### 6. Strategic Data Retention

**Description:** Automated data lifecycle management  
**User Value:** System efficiency and sustainability  
**Retention Policy:**

- Raw Articles: 30 days
- Daily Digests: 1 year
- Weekly Reports: Indefinite

---

## Out of Scope (Current Version)

### Explicitly Deferred Features

❌ **Social Media Monitoring**

- Twitter/X feed tracking
- LinkedIn analysis
- Social sentiment integration
- _Rationale:_ Feature creep prevention; focus on proven RSS sources first

❌ **Interactive Dashboard**

- Web-based UI
- Custom visualizations
- _Rationale:_ Email delivery meets primary use case

❌ **Multi-user Management**

- User accounts
- Custom preferences
- _Rationale:_ Single-user MVP first

❌ **Real-time Alerts**

- Breaking news notifications
- Custom alert triggers
- _Rationale:_ Scheduled batching aligns with user time constraints

---

## User Stories

### Story 1: Daily Morning Briefing

**As a** busy professional  
**I want** a 3-5 minute daily digest of key economic/financial news  
**So that** I can stay informed without spending 30+ minutes reading multiple sources

**Acceptance Criteria:**

- Digest arrives consistently after morning ingestion cycle
- Reading time ≤ 5 minutes
- Covers all three source feeds
- Highlights most important developments

### Story 2: Weekly Strategic Overview

**As a** decision-maker  
**I want** a weekly analytical report connecting the dots across daily news  
**So that** I can understand broader trends and narratives

**Acceptance Criteria:**

- Report delivered weekly via email
- Identifies recurring themes from the week
- Connects related events across days
- Provides analytical perspective, not just summary

### Story 3: Set-and-Forget Operation

**As a** user  
**I want** the system to run automatically without my intervention  
**So that** I can rely on consistent information without maintenance burden

**Acceptance Criteria:**

- No manual triggering required
- Handles errors gracefully (missing feeds, scraping failures)
- Manages data retention automatically
- Self-sustaining operation

---

## Feature Prioritization

### P0 (Must Have - MVP)

1. RSS feed ingestion (twice daily)
2. Basic content scraping (HTTP requests)
3. Daily digest generation
4. Weekly report generation
5. Email delivery
6. Basic data retention

### P1 (Should Have - V1.1)

1. Advanced scraping (browser automation fallback)
2. Error handling and retry logic
3. Feed health monitoring
4. Enhanced retention policies

### P2 (Could Have - Future)

1. Multiple delivery formats
2. Customizable schedules
3. Additional RSS sources
4. Archive search

### P3 (Won't Have - Deferred)

1. Social media monitoring
2. Interactive dashboard
3. Multi-user support
4. Real-time alerts

---

## Success Metrics

### Operational Metrics

- **Uptime:** 99%+ successful scheduled runs
- **Ingestion Rate:** 90%+ of published articles captured
- **Processing Time:** <10 minutes per ingestion cycle

### Quality Metrics

- **Digest Readability:** 3-5 minute reading time
- **Weekly Report Coherence:** Identifiable themes and narratives
- **User Satisfaction:** Actionable insights delivered

### Efficiency Metrics

- **Model Performance:** Successful processing with Ollama low-power models
- **Storage Growth:** Sustainable with retention policies
- **Manual Intervention:** Zero required for normal operation

---

## Product Roadmap

### Phase 1: Foundation (Current)

- Planning and architecture
- Memory Bank setup
- Prompt engineering strategy

### Phase 2: Core Implementation

- RSS ingestion
- Basic scraping
- Database setup
- Daily digest agent

### Phase 3: Weekly Reports

- Weekly analyst agent
- Email delivery
- Data retention automation

### Phase 4: Refinement

- Advanced scraping
- Error handling
- Performance optimization
- Prompt refinement

### Phase 5: Future Features (TBD)

- Evaluate deferred features based on Phase 4 learnings

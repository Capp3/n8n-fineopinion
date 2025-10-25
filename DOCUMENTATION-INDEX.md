# FinOpinions Documentation Index

**Last Updated:** October 25, 2025  
**Project Status:** Implementation Phase - Separated Workflows Architecture (ADR-001)

---

## 🚨 IMPORTANT: Separated Workflows Architecture

As of October 25, 2025, the FinOpinions system uses a **separated workflows architecture**. This is a fundamental architectural decision that affects all implementation work.

**📖 Start Here:** [ADR-001: Separated Workflows](./docs/architecture-decisions/ADR-001-separated-workflows.md)

---

## 📚 Documentation Overview

### 🏗️ Architecture & Design

#### Core Architecture Documents

1. **[ADR-001: Separated Workflows Architecture](./docs/architecture-decisions/ADR-001-separated-workflows.md)** ⭐ **CRITICAL**

   - Complete architectural decision documentation
   - Implementation details for both workflows
   - Testing strategy and operational procedures
   - ~15 min read

2. **[ADR-001 Summary](./memory-bank/adr-001-summary.md)** ⚡ **QUICK REFERENCE**

   - Concise summary of the separated workflows architecture
   - Key benefits and status management
   - Scraping strategy table
   - ~3 min read

3. **[System Architecture Diagrams](./fineopinions_diagram.md)**
   - High-level system flow diagrams
   - RSS post-processing architecture details
   - Airtable schema reference
   - Component diagrams

### 🔧 Implementation Guides

#### Immediate Next Steps

 f1. **[IMPLEMENT: Workflow 2 - Content Scraping & AI](./docs/IMPLEMENT-WORKFLOW-2.md)** 🚀 **CURRENT PRIORITY**

   - Complete Workflow 2 implementation guide
   - 9 nodes with full code snippets
   - Adaptive scraping (HTTP/Playwright/SearXNG)
   - AI agent processing with Ollama
   - Testing and deployment guide
   - ~60-90 min to complete

2. **[URLHash Deduplication Updates](./docs/IMPLEMENT-URLHASH-UPDATES.md)** ✅ **COMPLETE**

   - Implement URLHash-based deduplication
   - Auto-mapping for Airtable fields
   - IF/ELSE logic for existing records
   - ~15 min to complete

3. **[NEXT STEPS: Complete Workflow 1](./docs/NEXT-STEPS-WORKFLOW-1.md)** ✅ **COMPLETE**

   - Step-by-step guide to complete Workflow 1
   - Airtable field setup instructions
   - n8n node configuration
   - ~10 min to complete

4. **[n8n Node Settings](./fineopinions_node_settings.md)**

   - Complete node-by-node configuration guide
   - JavaScript code snippets for all processing nodes
   - Troubleshooting tips
   - GUI-friendly configuration instructions

5. **[Implementation Guide](./docs/implementation-guide.md)**
   - General implementation guidance
   - Setup procedures
   - Testing strategies

### 📋 Project Management

1. **[Task List](./tasks.md)** 📊 **TRACK PROGRESS**

   - Current implementation priorities
   - Workflow 1 sub-tasks
   - Workflow 2 implementation plan
   - Completed tasks archive

2. **[Build Roadmap](./docs/build-roadmap.md)**
   - Long-term implementation roadmap
   - Feature priorities
   - Timeline estimates

### 🧠 Memory Bank (AI Context)

1. **[ADR-001 Summary](./memory-bank/adr-001-summary.md)**

   - Quick reference for AI agents
   - Key architectural decisions
   - Operational notes

2. **[Project Brief](./memory-bank/projectbrief.md)**

   - Original project vision
   - Core requirements

3. **[Technical Context](./memory-bank/techContext.md)**

   - Technology stack
   - Integration details

4. **[System Patterns](./memory-bank/systemPatterns.md)**

   - Design patterns used
   - Best practices

5. **[RSS Feed Architecture](./memory-bank/rss-feed-architecture.md)**
   - Detailed RSS feed processing documentation
   - Feed-specific configurations

### 🎨 Prompts & Voice

1. **[Journalist Prompt](./prompts/journalist.md)**

   - AI agent prompt for article analysis

2. **[Voice Guide Reference](./docs/voice-guide-reference.md)**
   - Editorial voice and style guidelines

---

## 🗺️ Documentation Roadmap by Use Case

### 🚀 "I want to complete Workflow 1 NOW"

1. Read: [NEXT STEPS: Complete Workflow 1](./docs/NEXT-STEPS-WORKFLOW-1.md)
2. Follow: Step-by-step Airtable setup
3. Update: Node 18 in n8n
4. Test: Run workflow end-to-end

### 🏗️ "I want to understand the architecture"

1. Read: [ADR-001 Summary](./memory-bank/adr-001-summary.md) (3 min)
2. Review: [System Architecture Diagrams](./fineopinions_diagram.md)
3. Deep dive: [ADR-001 Full Documentation](./docs/architecture-decisions/ADR-001-separated-workflows.md)

### 💻 "I want to build Workflow 2"

1. **START HERE:** [IMPLEMENT: Workflow 2 Guide](./docs/IMPLEMENT-WORKFLOW-2.md) 🚀
2. Reference: [ADR-001 Section: Workflow 2](./docs/architecture-decisions/ADR-001-separated-workflows.md#workflow-2-content-scraping-implementation)
3. Track Progress: [Task 1.1 in tasks.md](./tasks.md)
4. Node Details: [n8n Node Settings](./fineopinions_node_settings.md)

### 🐛 "I'm debugging an issue"

1. Check: [n8n Node Settings - Troubleshooting sections](./fineopinions_node_settings.md)
2. Review: [ADR-001 - Operational Procedures](./docs/architecture-decisions/ADR-001-separated-workflows.md#-operational-procedures)
3. Verify: [System Architecture Diagrams](./fineopinions_diagram.md)

### 📊 "I want to track project progress"

1. Check: [Task List](./tasks.md)
2. Review: [Build Roadmap](./docs/build-roadmap.md)
3. Update: Tasks as you complete them

---

## 📂 File Structure

```
n8n-fineopinions/
├── 📄 README.md                          # Project overview
├── 📄 DOCUMENTATION-INDEX.md             # This file
├── 📄 fineopinions_diagram.md            # System architecture diagrams
├── 📄 fineopinions_node_settings.md      # n8n node configurations
├── 📄 tasks.md                           # Task tracking
│
├── 📁 docs/                              # Documentation
│   ├── 📄 NEXT-STEPS-WORKFLOW-1.md       # ⭐ Immediate next steps
│   ├── 📄 implementation-guide.md        # General implementation guide
│   ├── 📄 airtable-setup.md              # Airtable configuration
│   ├── 📄 build-roadmap.md               # Project roadmap
│   ├── 📄 voice-guide-reference.md       # Editorial voice guide
│   │
│   └── 📁 architecture-decisions/
│       └── 📄 ADR-001-separated-workflows.md  # ⭐⭐⭐ CRITICAL
│
├── 📁 memory-bank/                       # AI context documents
│   ├── 📄 adr-001-summary.md             # ⚡ Quick ADR reference
│   ├── 📄 projectbrief.md                # Project vision
│   ├── 📄 techContext.md                 # Tech stack
│   ├── 📄 systemPatterns.md              # Design patterns
│   └── 📄 rss-feed-architecture.md       # RSS details
│
├── 📁 prompts/                           # AI agent prompts
│   ├── 📄 journalist.md                  # Journalist agent
│   ├── 📄 editorial.md                   # Editorial agent
│   └── 📄 copywriter.md                  # Copywriter agent
│
└── 📁 exports/                           # n8n workflow exports
    └── 📁 workflows/
        └── 📄 ArticleVacume.json         # Current Workflow 1 export
```

---

## 🎯 Quick Links by Role

### For Developers

- [ADR-001](./docs/architecture-decisions/ADR-001-separated-workflows.md) - Architecture decisions
- [n8n Node Settings](./fineopinions_node_settings.md) - Implementation details
- [Task List](./tasks.md) - What to build next

### For Project Managers

- [ADR-001 Summary](./memory-bank/adr-001-summary.md) - High-level architecture
- [Build Roadmap](./docs/build-roadmap.md) - Timeline and priorities
- [Task List](./tasks.md) - Progress tracking

### For AI Agents

- [ADR-001 Summary](./memory-bank/adr-001-summary.md) - Quick architecture reference
- [Memory Bank](./memory-bank/) - Full project context
- [System Patterns](./memory-bank/systemPatterns.md) - Design patterns

### For Operations

- [ADR-001 - Operational Procedures](./docs/architecture-decisions/ADR-001-separated-workflows.md#-operational-procedures) - Daily operations
- [ADR-001 - Monitoring](./docs/architecture-decisions/ADR-001-separated-workflows.md#-monitoring-and-observability) - Metrics and alerts
- [NEXT STEPS](./docs/NEXT-STEPS-WORKFLOW-1.md) - Immediate actions

---

## 📌 Key Decisions & Changes

### October 25, 2025: ADR-001 Accepted

**Decision:** Separated Workflows Architecture
**Impact:** High - Core Architecture Change
**Documentation:** [ADR-001](./docs/architecture-decisions/ADR-001-separated-workflows.md)

**Key Changes:**

1. Split into 2 independent workflows (RSS Fetching vs Content Scraping)
2. Added Status management fields to Airtable
3. Implemented adaptive scraping strategies (HTTP/Playwright/SearXNG)
4. Enabled independent scheduling and scaling

**Benefits:**

- 70-80% reduction in HTTP requests
- Failure isolation
- Flexible scheduling
- Better scalability

---

## 🔄 Documentation Maintenance

### When to Update Documentation

1. **Architecture Changes:** Update ADR-001 and create new ADRs
2. **Node Configuration Changes:** Update fineopinions_node_settings.md
3. **Task Completion:** Update tasks.md
4. **New Features:** Update relevant docs and add to index

### Documentation Standards

- Use Mermaid for diagrams
- Include code snippets where helpful
- Provide troubleshooting sections
- Link between related documents
- Keep summaries up-to-date

---

## ❓ Need Help?

### Common Questions

**Q: Where do I start?**  
A: Read [NEXT STEPS: Complete Workflow 1](./docs/NEXT-STEPS-WORKFLOW-1.md)

**Q: What's the architecture?**  
A: Read [ADR-001 Summary](./memory-bank/adr-001-summary.md) (3 min)

**Q: How do I configure n8n nodes?**  
A: See [n8n Node Settings](./fineopinions_node_settings.md)

**Q: What should I build next?**  
A: Check [Task List](./tasks.md)

**Q: How do I handle scraping failures?**  
A: See [ADR-001 - Operational Procedures](./docs/architecture-decisions/ADR-001-separated-workflows.md#-operational-procedures)

---

## 📈 Project Status

**Current Phase:** Implementation - Workflow 1 Almost Complete  
**Next Milestone:** Complete Workflow 1, then build Workflow 2  
**Progress:** 45% complete (Workflow 1: 95%, Workflow 2: 0%)

**Immediate Priorities:**

1. Add Status fields to Airtable (10 min)
2. Update Node 18 with status mappings (5 min)
3. Test Workflow 1 end-to-end (15 min)
4. Run Workflow 1 for 24 hours to collect articles
5. Begin Workflow 2 implementation

**📖 See:** [Task List](./tasks.md) for detailed progress

---

**Last Updated:** October 25, 2025  
**Documentation Version:** 2.1 (Separated Workflows Architecture)

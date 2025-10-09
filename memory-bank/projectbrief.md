# Project Brief: FineOpinions - n8n Automated Financial Newsletter

## Project Overview

**Project Name:** FineOpinions  
**Type:** n8n Workflow Automation System  
**Version:** Initial Setup  
**Last Updated:** October 8, 2025

## Core Objective

Create a low-maintenance, automated workflow using n8n (v1.114.4 self-hosted) that consumes information from curated news feeds, processes content through a multi-agent AI system using local low-power LLM models (Ollama), and produces concise daily and weekly financial reports.

## Key Principles

1. **Low Power Consumption**: Design for Ollama-compatible models
2. **n8n Native Focus**: Use native nodes whenever possible, avoid code blocks
3. **AI Agent Architecture**: Leverage AI Agent nodes, avoid LLM provider-specific nodes except for model selection
4. **Scope Management**: Strict focus on core features, defer non-essential features to future
5. **Information Retention**: Strategic data retention policy for efficiency

## Infrastructure Available

- **n8n Version:** 1.114.4 (self-hosted)
- **Database Options:**
  - Airtable account
  - PostgreSQL instance
  - PGVector instance (for vector embeddings if needed)
- **LLM Infrastructure:** Ollama (local, low-power models)

## Current Status

- Phase: Planning & Setup
- Mode: PLAN
- Complexity Level: Level 4 (Complex System)

## Project Constraints

- **NO Feature Creep**: Strict adherence to defined scope
- **Future Features Section Required**: Track deferred features separately
- **Social Media Monitoring**: Explicitly moved to Future Features (NOT current scope)

## Success Criteria

1. Workflow reliably ingests, parses, and stores articles without manual intervention
2. Generates coherent Daily Digest (3-5 minutes reading time)
3. Delivers concise, analytical Weekly Report via email
4. Operates efficiently with low-power local models
5. Maintains sustainable data retention policy

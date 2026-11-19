# Project Progress - Nexova Solutions

**Last Updated:** 2026-11-19  
**AI Engineering Track — 4Geeks Academy**

> **Important:** Milestone dates listed below are **proposed estimates** for this AI Engineering project, not official Nexova commitments. They are subject to validation with stakeholders and should be discussed with CEO Laura Mendoza and CTO Sergio Molina before being treated as project SLAs.

---

## Milestone Summary

### ✅ Completed (Groundwork)

| Task | Date | Owner | Notes |
|------|------|-------|-------|
| Company selection (Nexova) | 2026-11-06 | Cristina | Briefing created |
| Repository structure setup | 2026-11-06 | Cristina | Monorepo folders initialized |
| BRIEFING.md document | 2026-11-06 | 4Geeks | Full company context |
| company-choice.md justification | 2026-11-06 | Cristina | Personal rationale |

---

### 🔄 In Progress (Current Sprint)

| Task | Milestone | Status | Expected | Owner |
|------|-----------|--------|----------|-------|
| Memory Bank structure | M1 (Agents Setup) | 🔄 | 2026-11-19 | Cristina |
| `.agents/rules/` framework | M1 | Pending | 2026-11-20 | Cristina |
| `.agents/skills/` (CV Scoring) | M1 | Pending | 2026-11-20 | Cristina |
| `AGENTS.md` catalog | M1 | Pending | 2026-11-20 | Cristina |

---

### ⏳ Planned (Next Phases)

| Task | Milestone | Target Date | Owner | Notes |
|------|-----------|-------------|-------|-------|
| UIs documentation (`website`, `backoffice`) | M2 (Web) | 2026-12-15 | Cristina | Research phase |
| Services API skeleton (`services/api`) | M2 (Backend) | 2026-12-20 | Cristina | FastAPI setup |
| Data pipelines (`data/pipelines/`) | M3 (Data) | 2027-01-15 | Cristina | ETL/ELT workflows |
| First AI Agent (Recruitment Agent) | M4 (Agents) | 2027-02-01 | Cristina | CV scoring + candidate matching |
| Support Chatbot with RAG | M5 (RAG/Real-time) | 2027-03-01 | Cristina | Knowledge base + LLM |

---

## Nexova-Specific Milestones

### Milestone 1: Memory Bank & Agent Infrastructure (Current)
**Goal:** Establish shared context and agent development framework

- [x] BRIEFING.md (company context)
- [x] Repository structure (all folders)
- [ ] `memory-bank/projectbrief.md` (business overview)
- [ ] `memory-bank/techContext.md` (technical context)
- [ ] `memory-bank/progress.md` (this file)
- [ ] `AGENTS.md` (agent catalog)
- [ ] `.agents/rules/` (development constraints)
- [ ] `.agents/skills/nexova-cv-score/` (first reusable skill)
- [ ] PR to `main` (for review)

**Definition of Done:** All files created, coherent, referenced correctly, PR open.

---

### Milestone 2: Web & Services (Planned)
**Goal:** Establish frontend and backend foundations

- [ ] `uis/website/` — Public-facing corporate site
- [ ] `uis/backoffice/` — Internal admin dashboard (employee, candidate, training views)
- [ ] `services/api/` — Central FastAPI backend
- [ ] Database schema (candidates, support tickets, training, employees)
- [ ] Authentication & authorization
- [ ] Logging & monitoring setup

**Success Criteria:** Endpoints respond, auth works, data persists, logs centralized.

---

### Milestone 3: Data Pipelines (Planned)
**Goal:** Ingest and process data from legacy systems

- [ ] `data/pipelines/` — ETL jobs (HubSpot, Zendesk, custom ATS)
- [ ] `data/process/` — Cleaned/enriched data outputs
- [ ] Data schema alignment (Nexova domain model)
- [ ] Historical data migration strategy
- [ ] Data quality checks

**Success Criteria:** Pipelines run weekly, data quality >95%, traceability maintained.

---

### Milestone 4: AI Agents (Planned)
**Goal:** Deploy first production agents for core workflows

#### 4a: Recruitment Agent
- Skill: CV scoring (from `.agents/skills/nexova-cv-score/`)
- Capability: Semantic search over candidate database
- Input: Job opening + candidate pool
- Output: Ranked candidates + explanations
- SLA: Response within 10 seconds

#### 4b: HR Portal Agent
- Capability: Answer employee questions (policies, benefits, leave)
- Input: Employee question (NL)
- Output: Answer + relevant docs
- Integration: HR knowledge base + internal docs

#### 4c: Training Recommender Agent
- Capability: Recommend courses based on employee role/skills gap
- Input: Employee profile + available courses
- Output: Personalized learning path
- Metric: 60%+ adoption of recommendations

**Success Criteria:** Agents meet SLA, user satisfaction >4/5, audit trail complete.

---

### Milestone 5: Real-Time & RAG (Planned)
**Goal:** Deploy support automation and knowledge systems

#### 5a: Support Chatbot
- Capability: First-line ticket resolution via RAG + LLM
- Input: Customer question (chat, email)
- Output: Resolution or escalation
- Target: 40% automation rate
- SLA: Maintained within 24 hours

#### 5b: Central Knowledge Base
- Structure: Semantic search over Zendesk docs + internal wikis
- Indexing: Weekly updates from support team
- Quality: Automated checks for outdated/conflicting info

**Success Criteria:** Chatbot resolves 40% without escalation, KBase adoption >80% by support team.

---

## Current Blockers & Notes

| Blocker | Impact | Mitigation |
|---------|--------|-----------|
| Legacy ATS has no API | Can't directly access candidate data | Plan for CSV export + parsing |
| HubSpot adoption only 40% | Sales data incomplete | Use as-is; focus on consistent subset |
| No centralized logging | Hard to audit agent decisions | Implement structured logging in agents |
| Support SLA at risk | Chatbot urgent | Prioritize support chatbot in M5 |

---

## Nexova Team Contacts (For Context/Questions)

| Department | Head | Focus |
|-----------|------|-------|
| Selection Operations | Javier Almeida | CV scoring, recruitment workflows |
| Corporate Training | Elena Vargas | Course recommendations, enrollment |
| Outsourced Support | Roberto Díaz | Chatbot SLA, knowledge base |
| Sales | Megan Clarke | CRM integration, pipeline |
| Internal HR | Patricia Solís | HR portal, leave management |
| Tech | Sergio Molina (CTO) | Legacy system access, infrastructure |
| Executive | Laura Mendoza (CEO) | Strategic alignment, weekly reporting |

---

## Key Decisions Logged

### 1. Memory-First Approach
- **Decision:** Establish shared memory bank (projectbrief, techContext, progress) before agents
- **Rationale:** Prevents duplication, ensures consistency across agents
- **Impact:** Slower initial setup, but faster agent development later

### 2. CV Scoring as First Skill
- **Decision:** Narrow skill scope (CV scoring only, not full matching)
- **Rationale:** Verifiable, reusable, unblocks recruitment agent
- **Impact:** Skill ready by end of M1

### 3. RAG for Support as Priority
- **Decision:** Support chatbot (Milestone 5) uses RAG over knowledge base
- **Rationale:** High impact (40% automation target), hard SLA drivers action
- **Impact:** Requires M3 (data pipelines) + semantic search setup

---

## Resources & Links

- **BRIEFING.md:** Full company & department details
- **projectbrief.md:** Business overview (companion to this file)
- **techContext.md:** Technical architecture (companion to this file)
- **AGENTS.md:** Agent catalog (at repo root)
- **repo/agents/:** Agent implementations (placeholder templates)
- **repo/skills/:** Reusable skills (placeholder)
- **repo/.agents/rules/:** Development constraints (in progress)

---

## Next Actions (For Approval/Review)

1. ✅ Create Memory Bank files (projectbrief, techContext, progress)
2. ⏳ Create AGENTS.md with agent catalog
3. ⏳ Create `.agents/rules/` with development constraints
4. ⏳ Create `.agents/skills/nexova-cv-score/` with SKILL.md
5. ⏳ Commit all changes to `feature/agent-memory-bank`
6. ⏳ Open PR to `main` for review

---

**Status:** Memory Bank creation in progress. Waiting for approval to continue with AGENTS.md, rules, and skills.

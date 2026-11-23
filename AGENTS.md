# AGENTS.md - Nexova AI Engineering Workflow

**Last Updated:** 2026-11-19
**AI Engineering Track — 4Geeks Academy**
**Company:** Nexova Solutions

---

## Purpose

This document defines the **agent development workflow** for Nexova AI Engineering projects. It establishes:

1. **Mandatory context** every agent must read before each session
2. **Pre-commit workflow** to ensure consistency and compliance
3. **Constraints & protected resources** that cannot be modified without authorization
4. **Application locations** where frontend and backend work occurs
5. **References to rules and skills** (created in subsequent phases)

---

## 1. Initial Reading List (Mandatory at Session Start)

Every AI agent session working on Nexova must read these files in this order:

| # | File | Purpose | Scope |
|---|------|---------|-------|
| 1 | `memory-bank/projectbrief.md` | Business context: org structure, AI opportunities, constraints, success metrics | Business requirements |
| 2 | `memory-bank/techContext.md` | Technical foundation: current stack, data models, domain vocabulary, proposed tech decisions, security patterns | Technical requirements |
| 3 | `memory-bank/progress.md` | Project status: 5 milestones, completed work, blockers, team contacts, key decisions | Project state |
| 4 | `BRIEFING.md` | Company source of truth: Nexova overview, all departments, problems, solutions | Authoritative source |
| 5 | `AGENTS.md` | This file: agent development workflow, pre-commit steps, constraints | Development workflow |

**Recommended reading order:** 1 → 4 → 2 → 3 → 5
**Time required:** ~20 minutes (first session); ~5 minutes (subsequent sessions to refresh)

**Validation:** Agent confirms having read all files before beginning work on `uis/`, `services/`, `agents/`, or `.agents/skills/`.

---

## 2. Pre-Commit Workflow (4 Mandatory Steps)

Before creating **any** commit to the monorepo, follow this ordered workflow:

### Step 1: Verify Project Status (Mandatory)
- Read the current `memory-bank/progress.md` milestone status
- Confirm your work aligns with the current milestone (M1-M5)
- If you are starting a new milestone, verify it is unblocked
- **Verification:** "Current status: M1 / M2 / M3 / ... and work is aligned"

**Example:**
```
✓ Current milestone: M1 (Memory Bank & Agent Infrastructure)
✓ Work: Creating AGENTS.md (Phase 2 of M1)
✓ Blockers: None for this phase
→ Proceed to Step 2
```

### Step 2: Check Development Constraints (Mandatory)
- Review `.agents/rules/` for active constraints and compliance rules
- Confirm your changes do not violate any rules (LLM behavior, tool usage, memory access)
- **Verification:** "Constraints reviewed; no violations detected"

**Example:**
```
✓ Constraints file: .agents/rules/llm-behavior.md
✓ Constraint: "All agent prompts must reference memory-bank/ context"
✓ Your code: ✓ References memory-bank/ ✓ No violations
→ Proceed to Step 3
```

### Step 3: Run Validation Checks (Mandatory)
- Verify you have NOT modified protected files (see Section 3)
- Verify coherence: cross-file references are correct, no contradictions
- Verify markdown syntax is valid
- Verify file sizes are reasonable (no massive diffs)
- **Verification:** "Validation checks pass; ready to commit"

**Example:**
```
✗ Modified files: agents/nexova-recruitment.py, uis/website/index.html
✓ Protected files NOT modified: BRIEFING.md, memory-bank/*, .agents/rules/*
✓ Cross-file references: agents/ reference .agents/skills/ correctly
✓ Markdown syntax: Valid
✓ File sizes: Reasonable
→ Proceed to Step 4
```

### Step 4: Format Commit Message (Mandatory)
- Use conventional commit format: `type(scope): description`
- Include what files changed and why
- Reference memory-bank/ files or Phase if applicable
- **Verification:** "Commit message is clear, formatted correctly, and includes scope"

**Example:**
```
docs(agents): add AGENTS.md pre-commit workflow

- Define 4-step pre-commit validation
- List protected files and constraints
- Reference memory-bank/ as source of truth
- Phase 2 deliverable for Nexova M1

Relates to: memory-bank/progress.md milestone M1
```

### Optional Step 5: Verify PR Alignment (Recommended)
- If commit is for a Phase milestone, verify the PR will align correctly
- Check that all files for that Phase are included
- Confirm milestone dates in progress.md match PR scope

---

## 3. Constraints & Protected Files

### Files CANNOT be Modified (Without Issue/Authorization)

| File | Reason | When to Modify |
|------|--------|-----------------|
| `BRIEFING.md` | Nexova source of truth (company facts only) | Never without CEO approval |
| `memory-bank/projectbrief.md` | Business context layer (shared by all agents) | Only if business requirements change (requires validation) |
| `memory-bank/techContext.md` | Technical foundation layer (shared by all agents) | Only if tech decisions change (requires validation) |
| `memory-bank/progress.md` | Project roadmap (source of milestones) | To update milestone progress, completed work, blockers (see update process below) |
| `AGENTS.md` | Workflow definition (this file) | Only if workflow changes required |
| `.agents/rules/*` | Constraints layer (compliance & behavior) | Only in Phase 3 workflow |

**Restriction:** Do NOT modify these files in `agents/`, `uis/`, `services/`, or `data/` work.

### Updating memory-bank/ — Validation Process

**`progress.md` updates are allowed** when the project evolves. Follow this process:

**Step 1:** Before modifying `progress.md`, verify alignment:
- ✓ What milestone is the work in? (M1-M5)
- ✓ What is being completed or blocked?
- ✓ Are dates/metrics realistic and evidence-based?

**Step 2:** Make the update with clear rationale:
- Use format: `- [DATE] CHANGE: description (reason: ...)`
- Example: `- 2026-11-20 PROGRESS: M1 Phase 2 complete: AGENTS.md created (reason: pre-commit workflow documented)`

**Step 3:** Commit in a dedicated memory-bank commit:
- Use message: `docs(memory-bank): update progress.md — [milestone phase update]`
- Include what changed and why

**Step 4:** Validate cross-file consistency:
- Ensure references in `AGENTS.md` to current milestone still match
- Ensure `techContext.md` technology status matches progress (if tech decisions changed)
- Ensure `projectbrief.md` business context aligns with new milestone status

**Example:** Updating `progress.md` from "M1 Phase 2 in progress" to "M1 Phase 2 complete" requires:
1. Verify all Phase 2 work (memory-bank/, AGENTS.md, .agents/rules skeleton) is done
2. Update `progress.md` with completion date and summary
3. Commit: `docs(memory-bank): update progress.md — M1 Phase 2 complete`
4. Confirm no contradictions with `AGENTS.md` current phase reference

### Files CAN be Modified/Created (Without Restriction)

| Location | Purpose | Can Create | Can Modify |
|----------|---------|-----------|-----------|
| `.agents/skills/*` | Reusable capabilities | ✓ Yes | ✓ Yes |
| `.agents/rules/*` | Constraints & compliance | Phase 3 only | Phase 3 only |
| `agents/*` | Agent implementations | ✓ Yes | ✓ Yes |
| `uis/website/*` | Public web frontend | ✓ Yes | ✓ Yes |
| `uis/backoffice/*` | Internal admin UI | ✓ Yes | ✓ Yes |
| `services/*` | Backend FastAPI | ✓ Yes | ✓ Yes |
| `data/*` | Datasets & pipelines | ✓ Yes | ✓ Yes |

### Additional Restrictions

- **Skills are reusable, not mandatory per agent:**
  - Create a skill in `.agents/skills/` only when a capability is **concrete, documented, and reusable** across multiple contexts
  - An agent may use multiple skills; multiple agents may share one skill
  - Example: A `nexova-cv-score` skill can be used by recruitment-agent, training-agent, and manual CLI tools
  - Do NOT create a skill just to accompany an agent; create it when the need emerges

- **Do NOT remove milestones from progress.md:**
  - Milestones are fixed (M1-M5); only mark tasks as done, not remove
  - Follow the validation process above when updating milestone progress

- **Do NOT invent new technology decisions:**
  - Use only technologies listed in `memory-bank/techContext.md` **Current Stack** (existing systems)
  - **Proposed technologies** (FastAPI, Claude, RAG) are recommendations for this project, not Nexova commitments
  - Propose new tech changes in a GitHub issue first; validate with CTO Sergio Molina before implementation

---

## 4. Application Locations

Nexova has three main application areas. Each is documented in `uis/README.md` and `services/README.md`:

### Frontend Applications (`uis/`)

#### `uis/website` — Public-Facing Corporate Website
- **Purpose:** Nexova's public presence (clients, candidates, partners)
- **Responsibility:** Carmen Ruiz (Marketing & Comms)
- **Current State:** Legacy site from 2019 (slow, not accessible)
- **Future Work:** Reddesign with SEO/GEO optimization, schema.org, AI-powered content
- **Technology:** To be defined (framework TBD)
- **Status:** Placeholder structure only; design phase pending

#### `uis/backoffice` — Internal Operations Dashboard
- **Purpose:** Admin interface for all Nexova operations
- **Responsibilities:**
  - Javier Almeida (Selection Operations): Candidate database, CV scoring, recruitment pipeline
  - Elena Vargas (Training): Course catalog, enrollments, attendance
  - Roberto Díaz (Support): Ticket dashboard, KB management, SLA monitoring
  - Patricia Solís (HR): Leave requests, onboarding, KPI dashboard
  - Laura Mendoza (Executive): Real-time company KPIs
- **Current State:** No unified interface (email, spreadsheets, manual PDF reports)
- **Future Work:** Single portal combining recruitment, training, support, HR, and executive views
- **Technology:** To be defined (framework TBD)
- **Status:** Placeholder structure only; design phase pending

### Backend Application (`services/`)

#### `services/` — Centralized Company API (FastAPI — Proposed)

> **Status:** FastAPI is a **proposed technology** for this AI Engineering project, not an existing Nexova system. Current state: no centralized API (HubSpot, Zendesk, custom ATS disconnected). Proposal requires CTO validation before implementation.

- **Purpose:** One unified backend for all Nexova applications (proposal for future state)
- **Responsibility:** Sergio Molina (CTO) + AI Engineering team
- **Current State:** No centralized API (HubSpot, Zendesk, custom ATS disconnected)
- **Proposed Future Work:**
  - Single entry point for all APIs
  - Routers per domain: `/recruitment`, `/training`, `/support`, `/hr`, `/sales`, `/telemetry`
  - Authentication & authorization
  - Integration bridges to HubSpot, Zendesk, Google Workspace
  - Background workers for data pipelines
  - Webhook handlers for real-time updates
- **Proposed Technology:** FastAPI (Python) — from `memory-bank/techContext.md` Proposed Stack
- **Status:** Placeholder structure only; API design phase pending (approval required)

---

## 5. References to Upcoming Files

The following files will be created in subsequent phases. This section documents their planned structure for reference:

### Phase 3: `.agents/rules/` — Development Constraints

**Directory Structure:**
```
.agents/rules/
├── README.md                 # Constraints overview
├── llm-behavior.md           # LLM instruction constraints
├── tool-usage.md             # Tool access restrictions
├── memory-access.md          # Memory bank usage rules
└── compliance.md             # GDPR, data protection, audit
```

**Purpose:** Ensure all agents follow consistent behavioral and compliance rules.

**Example Rules (not yet implemented):**
- All agent prompts must reference `memory-bank/projectbrief.md`
- LLM cannot access sensitive data without masking (candidates, employees, customers)
- Tools can only write to authorized folders (`agents/`, `uis/`, `services/`, `data/`)
- Support agent responses must include SLA tracking
- All decisions must be logged for audit

### Phase 4: `.agents/skills/` — Reusable Capabilities

**Directory Structure (Example):**
```
.agents/skills/
├── README.md                           # Skills catalog
├── nexova-cv-score/
│   ├── SKILL.md                       # Objective, inputs, outputs, criteria
│   ├── examples/                      # Sample CVs & expected scores
│   └── resources/                     # Domain knowledge, scoring rubrics
└── (future skills created as needed)
```

**Skills Creation Principle:**
- Create a skill when a capability is **concrete, documented, and reusable** across multiple contexts
- Do NOT create a skill just to accompany an agent
- An agent may use multiple skills; multiple agents may share one skill

**Example: `nexova-cv-score` Skill (Proposed)**
- **Objective:** Score a CV against a job profile (0-100 scale)
- **Reusable by:** Recruitment agent, training recommendation engine, CLI scoring tool
- **Inputs:**
  - `cv_text` (resume content)
  - `job_profile` (required skills, experience, education)
- **Outputs:**
  - `score` (0-100)
  - `missing_skills` (list)
  - `evidence` (justification)
- **Verification:** Manual validation with recruitment team (target: >85% precision)

**Status:** To be created in Phase 4 when first agent needs it

---

## 6. Nexova-Specific Context

### Company Structure
- **8 Departments:** Selection Operations (40), Support (30), Training (12), Sales (18), HR (4), Tech (6), Marketing (small), Executive (1 CEO)
- **Locations:** Valencia, Spain (HQ) + Miami, Florida (expansion)
- **Scale:** 120 employees, $8M USD annual revenue

### Domain Vocabulary (Critical for Agents)
Agents must use Nexova terminology consistently. See `memory-bank/techContext.md` for complete list:
- **Consultor** = Recruitment specialist
- **Candidato** = Job applicant
- **CV/Currículum** = Resume
- **Proceso de selección** = Recruitment workflow
- **Oferta de empleo** = Job posting
- **SLA** = Service Level Agreement (24-hour target for support)
- **Ticket** = Support issue

### Operational Constraints
- **Language:** Spanish primary (Valencia), English secondary (Miami)
- **Data Privacy:** GDPR compliance required
- **SLA:** Support operations must resolve tickets within 24 hours (currently failing at 48h)
- **Multi-currency:** EUR (Spain) and USD (Florida)
- **Legacy Integration:** Must work with 2010s ATS, HubSpot (40% adoption), Zendesk, Google Workspace

---

## 7. Project Milestones Reference

Agent work is organized into 5 sequential milestones. See `memory-bank/progress.md` for details:

| Milestone | Focus | Status | Expected |
|-----------|-------|--------|----------|
| **M1** | Memory Bank & Agent Infrastructure | 🔄 In Progress | 2026-11-20 |
| **M2** | Web & Services | ⏳ Planned | 2026-12-15 |
| **M3** | Data Pipelines | ⏳ Planned | 2027-01-15 |
| **M4** | AI Agents | ⏳ Planned | 2027-02-01 |
| **M5** | Real-Time & RAG | ⏳ Planned | 2027-03-01 |

**Current Phase:** M1 Phase 2 (AGENTS.md creation) ← You are here

---

## 8. Getting Help

| Question | Resource |
|----------|----------|
| What are Nexova's business problems? | See `memory-bank/projectbrief.md` |
| What's the current tech stack? | See `memory-bank/techContext.md` |
| What work is in progress? | See `memory-bank/progress.md` |
| What are the company facts? | See `BRIEFING.md` |
| Which agents exist? | See `agents/README.md` |
| Which skills exist? | See `skills/README.md` |
| What are development rules? | See `.agents/rules/` (Phase 3) |
| Where do I work? | See `uis/README.md` and `services/README.md` |

---

## Versioning & Updates

| Version | Date | Change |
|---------|------|--------|
| 1.0 | 2026-11-19 | Initial AGENTS.md: reading list, pre-commit workflow, protected files, app locations, Phase 3-4 references |

---

## References

- **Nexova Company Context:** `BRIEFING.md`
- **Business Overview:** `memory-bank/projectbrief.md`
- **Technical Foundation:** `memory-bank/techContext.md`
- **Project Status:** `memory-bank/progress.md`
- **Agent Implementations:** `agents/README.md`
- **Skills Catalog:** `skills/README.md`
- **Frontend Apps:** `uis/README.md`
- **Backend API:** `services/README.md`

---

**Status:** Phase 2 deliverable (M1). Ready for review and Phase 3 approval.

# Development Rules

**Scope:** Code contributed to `agents/`, `uis/`, `services/`, `data/`
**Applies to:** All development work, PRs, code reviews
**Last Updated:** 2026-11-19

---

## Overview

This rule establishes **code quality, architectural consistency, and AI-generated code standards** for Nexova development.

Nexova is built as a monorepo with clear separation: frontends (uis/), backends (services/), agents (agents/), and data (data/). All contributions must respect this structure.

---

## Rules

### Rule 1: Monorepo Structure Discipline

**Scope:** All new files, folders, services

**Requirement:**
- Follow the monorepo structure defined in `README.md` and `.agents/rules/`
- Each domain has a single responsibility:
  - `uis/` = user interfaces (website, backoffice)
  - `services/` = centralized API (FastAPI, proposed)
  - `agents/` = AI agents + orchestration
  - `data/` = datasets, pipelines, evaluation
  - `docs/` = architecture, decisions
  - `infra/` = deployment, infrastructure

- **Example violations:**
  - ✗ Creating new API in `agents/` folder (APIs go in `services/`)
  - ✗ Storing datasets in `uis/` (datasets go in `data/`)
  - ✓ Creating agent in `agents/recruitment-agent/` with documentation
  - ✓ Creating FastAPI route in `services/api/routers/`

**Verification:** Code review checks folder placement; PR rejected if structure violated

### Rule 2: Technology Stack Consistency

**Scope:** Technology choices in agents/, services/, uis/, data/

**Requirement:**
- **Current Stack (Existing Systems):** Custom ATS, HubSpot, Zendesk, Google Workspace, Gmail
  - Use these systems as documented in `memory-bank/techContext.md` Current Technology Stack
  - Follow integration restrictions defined in `.agents/rules/tool-usage.md`

- **Proposed Stack (Recommendations for This Project):** FastAPI, Claude 3+, RAG, Python + LangChain
  - Proposed technologies from `memory-bank/techContext.md`; not Nexova commitments
  - Use in development work as aligned with project phase

- **New Technology Introduction:**
  1. Document in GitHub issue (problem statement + rationale)
  2. Discuss with CTO Sergio Molina
  3. Validate against BRIEFING.md constraints
  4. Update `memory-bank/techContext.md` before merging

**Example:**
```
✗ PR: "Add Node.js Express server"
Reason: Not in Current or Proposed stack; requires CTO discussion first

✓ PR: "Add FastAPI endpoint for CV scoring"
Reason: FastAPI in Proposed Stack; aligns with services/ architecture
```

**Verification:** Code review confirms technology consistency with documented stack

### Rule 3: Nexova Domain Modeling

**Scope:** Data models, database schemas, API responses

**Requirement:**
- All domain entities must use vocabulary from `memory-bank/techContext.md` Domain Vocabulary
- If a new entity is needed, propose it in GitHub issue before implementation
- Avoid external conventions that conflict with Nexova terminology

**Domain Entities (from memory-bank/):**
- Consultor, Candidato, CV, Proceso de Selección, Oferta de Empleo
- Entrenamiento, Curso, Enrollment
- Ticket, Agente de Soporte, SLA
- JobOpening, Candidate (English aliases for Miami office)

**Example:**
```
✗ Data model:
  class Applicant:  # Wrong: should be Candidato
    resume: str     # Wrong: should be cv_text

✓ Data model:
  class Candidato:
    cv_text: str
    skills: List[str]
    # (English alias can follow if Miami context)
```

**Verification:** Code review checks domain terminology consistency

### Rule 4: Testing & Validation

**Scope:** All agents, skills, services, data pipelines

**Requirement:**
- **Agents:** Every agent should have test cases covering core workflows (minimum 3 test scenarios)
- **Skills:** Every skill must document example inputs/outputs and validation criteria
- **Services:** API endpoints should include unit and integration tests
- **Data pipelines:** Transformations should have validation tests (input/output schema verification)
- **Test data:** Use realistic Nexova data (anonymized/masked) for testing

**Example Test Structure:**
```
agents/recruitment-agent/
├── recruitment_agent.py
├── tests/
│   ├── test_cv_scoring.py
│   ├── test_candidate_search.py
│   └── test_core_workflows.py
└── fixtures/
    ├── sample_cvs/ (anonymized examples)
    └── job_openings/ (test data)
```

**Verification:** Code review confirms tests are present and document validation criteria; test execution validated before merge

### Rule 5: Documentation Requirements

**Scope:** Every new agent, skill, service endpoint, data pipeline

**Requirement:**
- Each component must have a README or docstring explaining:
  - What it does
  - Who owns it (department / person)
  - Which memory-bank files it references
  - What constraints apply (privacy, SLA, permissions)
  - Example usage or input/output

**Example:**
```
# Recruitment Agent

**Purpose:** Score CVs and suggest candidate matches for job openings

**Owner:** Javier Almeida (Selection Operations)

**References:**
- memory-bank/projectbrief.md (40 consultors, 30-80 CVs/process)
- memory-bank/techContext.md (Candidato, Oferta de Empleo domains)

**Constraints:**
- Must mask PII in logs (data-privacy.md Rule 1)
- Must log all scoring decisions (audit trail, data-privacy.md Rule 3)
- Must respect 24-hour SLA for support tickets (services/)

**Example:**
Input: candidate_id=12345, job_opening_id=789
Output: { score: 78, missing_skills: [...], evidence: "10 years sales exp..." }
```

**Verification:** Code review checks documentation completeness; PR rejected if missing

### Rule 6: AI-Generated Code Standards

**Scope:** Any code generated by LLM agents or AI tools

**Requirement:**
- AI-generated code must pass same review as human code
- Additional checks:
  - [ ] Code includes comments explaining non-obvious logic
  - [ ] Variable names are clear and domain-aware (uses Nexova vocabulary)
  - [ ] Error handling is explicit (no silent failures)
  - [ ] Security considerations documented (data access, validation)
  - [ ] No hallucinated dependencies (all imports exist)
  - [ ] No hardcoded credentials or sensitive data

**Example:**
```
✗ AI-generated (problems):
def score_cv(candidate):
    # LLM hallucinated "cv_scorer" library that doesn't exist
    return cv_scorer.rank(candidate)

✓ AI-generated (approved):
def score_cv(candidate: Candidato) -> CVScore:
    """Score candidate CV against job profile.

    Uses semantic matching on skills + experience.
    Logs all decisions for audit trail (data-privacy.md Rule 3).
    """
    skills_match = semantic_similarity(candidate.skills, job_profile.required_skills)
    experience_match = experience_years_match(candidate, job_profile)
    score = weighted_average(skills_match * 0.6, experience_match * 0.4)

    # Log audit trail
    log_cv_scoring(candidate_id=candidate.id, score=score)

    return CVScore(score=score, evidence=f"Skills {skills_match:.2f}, Exp {experience_match:.2f}")
```

**Verification:** Code review explicitly flags AI-generated sections; human approval required

---

## Code Review Checklist

Before approving any PR in `agents/`, `uis/`, `services/`, `data/`:

- [ ] Monorepo structure respected (Rule 1)
- [ ] Technology choices consistent with documented stack (Rule 2)
- [ ] Domain vocabulary used correctly (Rule 3)
- [ ] Tests present and validation documented (Rule 4)
- [ ] Documentation complete (Rule 5)
- [ ] AI-generated code reviewed for clarity and correctness (Rule 6)
- [ ] No protected files modified (AGENTS.md restrictions)
- [ ] Data privacy rules followed (data-privacy.md)
- [ ] Memory-bank references correct (memory-access.md)
- [ ] Commit message clear and formatted (AGENTS.md pre-commit Step 4)

---

## Violations & Escalation

| Violation | Action | Escalation |
|-----------|--------|-----------|
| Wrong folder structure | Reject PR | Move files to correct location, resubmit |
| Unapproved technology | Reject PR | Propose in GitHub issue, get CTO approval |
| Wrong domain terminology | Comment on PR | Update terminology, re-review |
| Missing tests | Reject PR | Add tests, minimum 3 cases per agent/skill |
| Missing documentation | Reject PR | Add README or docstrings |
| AI-generated hallucination | Reject PR | Fix issues, re-review, human approval required |

---

## References

- **Source:** `memory-bank/techContext.md` Proposed Stack & Domain Vocabulary
- **Context:** BRIEFING.md (organizational structure, department constraints)
- **Workflow:** AGENTS.md (pre-commit Step 2-3, file protection rules)
- **Safety:** data-privacy.md (compliance), memory-access.md (context usage)

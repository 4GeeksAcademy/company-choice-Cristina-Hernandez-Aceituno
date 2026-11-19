# Memory Access Rules

**Scope:** `memory-bank/`, agents reading shared context, skills using company context  
**Applies to:** All agents, skills, development sessions  
**Last Updated:** 2026-11-19

---

## Overview

This rule defines how agents and development sessions **read, use, and reference** the shared context in `memory-bank/`.

The memory bank is the **single source of truth** for Nexova's business context, technical foundation, and project status. All agents must reference it consistently to ensure coherent decision-making.

---

## Rules

### Rule 1: Mandatory Reading at Session Start

**Scope:** Every agent session beginning work on Nexova

**Requirement:**
- Agent must read and confirm understanding of these files in order:
  1. `memory-bank/projectbrief.md` (business context)
  2. `memory-bank/techContext.md` (technical foundation)
  3. `memory-bank/progress.md` (project status)
  4. `BRIEFING.md` (company source of truth)
  5. `AGENTS.md` (development workflow)

**Verification:**
- Agent includes in first response: "✓ Read memory-bank files: [list]"
- If file is missing or unreadable, agent escalates before proceeding

**Example:**
```
✓ Read memory-bank/projectbrief.md (Nexova structure, 8 departments)
✓ Read memory-bank/techContext.md (current ATS, HubSpot, Zendesk stack)
✓ Read memory-bank/progress.md (M1 current, Phase 2 complete)
→ Ready to proceed
```

### Rule 2: Use Nexova Vocabulary Consistently

**Scope:** All agent prompts, skill descriptions, code comments

**Requirement:**
- When referring to Nexova domain entities, use the official terminology from `memory-bank/techContext.md` Domain Vocabulary
- Do NOT use external terminology (e.g., "applicant" instead of "Candidato"; "ticket" instead of "Ticket" English)
- Exceptions: English context in Miami office (use "Candidate" / "Ticket" with note if needed)

**Vocabulary Examples:**
- ✓ "Consultor cribó 40 Candidatos para el Proceso de Selección" (Spanish, correct)
- ✓ "Scoring CV for the Job Opening" (English, Miami context)
- ✗ "Applicant" for Candidato (wrong terminology)
- ✗ "Resume" without context (use CV/Currículum in Spanish context)

**Verification:** Code review checks vocabulary consistency

### Rule 3: Reference Memory Bank in Decision Justification

**Scope:** Agent decisions affecting business logic, data models, or workflow

**Requirement:**
- Every significant decision must cite which memory-bank file justifies it
- Format: "Per `memory-bank/projectbrief.md`: Javier Almeida (40 consultors) → CV scoring priority"
- If no memory-bank justification exists, escalate to human for approval

**Example:**
```
✗ Decision: "Add SLA monitoring to support tickets"
✗ Why: (no justification given)

✓ Decision: "Add SLA monitoring to support tickets"
✓ Why: Per memory-bank/projectbrief.md, Roberto Díaz (30 support agents)
       fails SLA (48h avg vs 24h target); per BRIEFING.md, SLA is core metric
```

### Rule 4: Updates to memory-bank/

**Scope:** `memory-bank/projectbrief.md`, `memory-bank/techContext.md`, `memory-bank/progress.md`

**Requirement:**
- `projectbrief.md` and `techContext.md` are primary reference files; modifications require AGENTS.md validation process
- `progress.md` can be updated when project status changes, following AGENTS.md update process:
  - Verify alignment with current milestone (M1-M5)
  - Document what changed and why
  - Use dedicated commit: `docs(memory-bank): update progress.md — [phase summary]`
  - Validate cross-file consistency (AGENTS.md references, tech decisions)

**Update Validation Process:**
1. Confirm work status is accurate and evidence-based
2. Document rationale for any changes
3. Verify no contradictions with AGENTS.md current phase
4. Commit with clear message indicating what changed

**Protection:** All three files follow Git protection rules per AGENTS.md Section 3

---

## Violations & Escalation

| Violation | Action | Escalation |
|-----------|--------|-----------|
| Agent proceeds without reading memory-bank/ | Halt workflow | Require confirmation of reading all 5 files |
| Agent uses wrong Nexova vocabulary | Log warning | Code review required before merge |
| Agent cites wrong memory-bank source | Halt decision | Provide correct source or escalate for clarification |
| Agent attempts to modify projectbrief/techContext | Reject operation | Requires GitHub issue + stakeholder approval |

---

## References

- **Source:** `memory-bank/projectbrief.md`, `memory-bank/techContext.md`, `memory-bank/progress.md`
- **Workflow Context:** AGENTS.md Section 1 (Initial Reading List)
- **Validation Context:** AGENTS.md Section 2 Step 1 (Verify Project Status)


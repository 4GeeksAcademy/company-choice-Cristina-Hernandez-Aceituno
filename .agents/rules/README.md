# Development Rules & Constraints — Nexova Solutions

**Last Updated:** 2026-11-19  
**AI Engineering Track — 4Geeks Academy**

---

## Purpose

This directory defines the **behavioral, compliance, and development constraints** that all agents, skills, and AI-assisted development must follow when working on Nexova Solutions projects.

These rules ensure:
- ✓ Consistent agent behavior across the organization
- ✓ Data privacy principles and protection patterns
- ✓ Coherence with business requirements (BRIEFING.md)
- ✓ Operational consistency and audit trails
- ✓ Quality standards for AI-generated code and decisions

---

## Rules Overview

| Rule File | Scope | Purpose |
|-----------|-------|---------|
| `memory-access.md` | `memory-bank/`, agents, skills | How agents read and use shared context (reference source, vocabulary) |
| `data-privacy.md` | All files handling personal data | Data privacy principles, PII handling, consent, audit trails |
| `development.md` | `agents/`, `uis/`, `services/`, `data/` | Code quality, architectural decisions, testing and documentation |
| `tool-usage.md` | Tool integrations: HubSpot, Zendesk, Gmail, Google Workspace | Integration patterns, recommended practices, audit logging |

---

## How Rules Apply

### To Agents
- Every agent session must read all rules in this directory before beginning work
- Violations detected → workflow halts, requires clarification from human supervisor
- Example: "Agent attempted to write CV scores without audit logging → rule violation → escalate"

### To Skills
- Every skill must document compliance with relevant rules (data-privacy, tool-usage)
- Example: "nexova-cv-score skill reads candidates (privacy rule applies) + logs scoring (audit trail rule applies)"

### To Development
- All code contributions (agents/, uis/, services/, data/) must respect development.md rules
- Code review checklist includes rule violations
- Example: "PR adds HubSpot integration → must follow tool-usage.md rules → review required"

### To AGENTS.md Workflow
- Pre-commit Step 2 ("Check Development Constraints") references this directory
- Pre-commit Step 3 ("Run Validation Checks") includes rule validation

---

## References

- **Source of Truth:** BRIEFING.md (company facts, departments, problems)
- **Context Layer:** memory-bank/ (business context, tech decisions, project status)
- **Workflow:** AGENTS.md (pre-commit, protected files, app locations)
- **Governance:** Nexova operational procedures and compliance requirements

---

## Versioning

| Version | Date | Change |
|---------|------|--------|
| 1.0 | 2026-11-19 | Initial rules set: memory-access, data-privacy, development, tool-usage |


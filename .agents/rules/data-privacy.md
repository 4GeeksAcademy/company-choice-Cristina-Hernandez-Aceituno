# Data Privacy & Compliance Rules

**Scope:** All files handling personal data (candidates, employees, customers)  
**Applies to:** Agents, skills, services, data pipelines  
**Governance Baseline:** Data protection principles, audit trail requirements, Nexova operational constraints  
**Legal Review:** Requires validation with Nexova legal/compliance team  
**Last Updated:** 2026-11-19

---

## Overview

Nexova handles sensitive personal data across three categories:
1. **Candidate data** (CVs, contact, work history) — Selection Operations
2. **Employee data** (HR, performance, leave) — Internal HR
3. **Customer data** (support tickets, company info) — Outsourced Support

This rule establishes data privacy principles, recommended practices, and audit patterns for all systems handling personal data.

---

## Rules

### Rule 1: PII Masking in Agent Training & Logs

**Scope:** All agent prompts, training data, logs, monitoring  
**Applies to:** Development sessions, skill examples, debugging outputs

**Requirement:**
- Never train agents on unmasked Candidate, Employee, or Customer PII
- Mask in this order:
  - Full names → "Name_[MASKED]"
  - Email addresses → "email@[MASKED]"
  - Phone numbers → "+34 [MASKED]"
  - CV text → Extract skills/experience only, omit personal data
  - Company names → Keep (not PII), or anonymize if client confidential

**Example:**
```
✗ Agent training data:
"Candidate Juan García (juan.garcia@example.com, +34 666 123 456)
applied for Sales Manager at ACME Corp with 10 years experience"

✓ Agent training data (masked):
"Candidate Name_[MASKED] (email@[MASKED], phone_[MASKED])
applied for Sales Manager at ACME Corp with 10 years experience"
```

**Verification:** Data sanitization script runs before logging; audit trail records what was masked

### Rule 2: Candidate Database Access Control

**Scope:** `agents/`, `services/`, `data/pipelines/` accessing candidate data

**Requirement:**
- Read access: Only Selection Operations agents (recruitment team) + approved AI agents
- Write access: Only data pipeline authors (Cristina) or automated import processes
- No: Sales, Support, Marketing, HR agents can read candidate database
- Context: Candidates are applying for Nexova's client jobs, not Nexova itself

**Access Matrix:**
| Role | Read Candidate DB | Write Candidate DB | Modify Scores | View SLA |
|------|-------------------|-------------------|---------------|----------|
| Recruitment Agent | ✓ | ✗ | Logged | ✓ |
| CV Scoring Skill | ✓ (masked) | ✗ | ✓ (logged) | ✗ |
| Support Agent | ✗ | ✗ | ✗ | N/A |
| Training Agent | ✗ | ✗ | ✗ | N/A |
| Data Pipeline | ✓ | ✓ (append only) | ✗ | ✗ |

**Example Violation:**
```
✗ Support agent calling CV scorer to find similar profiles
Reason: Support agents have no business reason to access candidate DB
Action: Deny access, escalate to Javier Almeida
```

### Rule 3: Audit Trail for All Data Access

**Scope:** All reads/writes to `agents/`, `services/` handling personal data

**Requirement:**
- Every access must log:
  - WHO: Agent name, session ID
  - WHAT: Data accessed (candidate ID, field name, not actual value)
  - WHEN: Timestamp (ISO 8601)
  - WHY: Purpose (e.g., "CV scoring for Job Opening 42")
  - HOW: Tool/API used

**Format:**
```json
{
  "timestamp": "2026-11-19T12:00:00Z",
  "agent": "recruitment-agent-v1",
  "action": "READ",
  "resource": "candidate:12345",
  "fields": ["skills", "experience_years", "education"],
  "purpose": "CV scoring for JobOpening:789",
  "result": "score=78 (evidence: ...)"
}
```

**Verification:** Audit logs reviewed monthly; violations trigger investigation

### Rule 4: Right to Delete (Data Subject Access)

**Scope:** `memory-bank/progress.md` deletion requests, data pipelines, database backups

**Requirement:**
- Candidates can request removal from database
- When deletion requested:
  1. Mark candidate as deleted (soft delete, not hard)
  2. Remove from all active searches/scorings
  3. Document in audit trail: "Deletion requested: [date], [reason if given]"
  4. Establish data retention policy with legal/compliance validation

**Process:**
- Day 0: Deletion request received and logged
- Notify relevant stakeholders (Selection Operations)
- Implement removal from active systems
- Establish appropriate retention period in coordination with compliance team

**Note:** Specific retention timelines and legal obligations require validation with Nexova legal/compliance team before implementation.

**Verification:** Deletion requests logged in `memory-bank/progress.md` blockers section

### Rule 5: Data Retention Policy (Governance Required)

**Scope:** All candidate, employee, customer data storage

**Requirement:**
Nexova must establish and document a data retention policy in coordination with legal/compliance team covering:
- **Active candidates** (in-process): Retention period to be defined
- **Rejected candidates**: Retention period to be defined (consider applicable labor laws)
- **Hired candidates**: Retention period to be defined (consider business and legal requirements)
- **Inactive candidates**: Retention period to be defined
- **Support tickets**: Retention period to be defined (consider SLA reference needs)
- **Employee records**: Retention period to be defined (consider applicable payroll laws)

**Implementation:**
- Document policy in Nexova governance documentation
- Implement retention audits (frequency to be determined)
- Establish data purge procedures aligned with approved policy
- Reference policy in all data handling agreements

**Note:** This rule establishes the need for a retention policy; specific timelines require legal/compliance validation.

### Rule 6: Consent for Candidate Communications

**Scope:** Candidate outreach, recruitment emails, follow-ups

**Requirement:**
- Verify recipient preferences before sending recruitment communications
- Maintain a consent log for audit trail purposes
- Respect candidate preferences for communication type and frequency
- Implement opt-out mechanism for future communications

**Implementation:**
- Check candidate communication preferences in database before sending
- Log all outbound communications with timestamp and recipient
- Provide clear opt-out instructions in outbound messages
- Document consent status in audit trail

**Example:**
```
✗ Agent sends candidate communication without consent check
✓ Agent verifies candidate communication preferences before sending
✓ Agent logs communication in audit trail
```

---

## Implementation

### Checklist for Data-Handling Code

Before merging any PR that handles personal data:

- [ ] PII handling documented (masking or anonymization approach)
- [ ] Access control documented (read/write by role)
- [ ] Audit trail logs implemented
- [ ] Data retention approach documented
- [ ] Consent/preference procedures documented
- [ ] Deletion request process documented
- [ ] Data subject rights support documented

### Violations & Escalation

| Violation | Action | Escalation |
|-----------|--------|-----------|
| Unmasked PII in logs | Redact immediately | Security audit required |
| Unauthorized database access | Reject request | Escalate to CTO Sergio Molina |
| Missing audit trail | Halt operation | Implement logging before retry |
| Deletion request not processed | Escalate | HR + Legal review |

---

## References

- **Source:** `memory-bank/techContext.md` Data Privacy & Compliance section
- **Context:** BRIEFING.md (Nexova structure, departments handling personal data)
- **Contacts:** Patricia Solís (HR), Sergio Molina (CTO) for data governance coordination


# Tool Integration Guidelines

**Scope:** Integration with external APIs and tools: HubSpot, Zendesk, Gmail, Google Workspace
**Applies to:** Agents, services, data pipelines accessing external systems
**Last Updated:** 2026-11-19

---

## Overview

Nexova's existing technology ecosystem spans multiple disconnected systems. This guide documents recommended patterns for how agents and services **work with** these tools while maintaining operational consistency and audit trails.

**Current Connected Systems:**
- HubSpot (Sales CRM, 40% adoption rate)
- Zendesk (Support ticketing, legacy)
- Gmail (Email, candidate & client communication)
- Google Workspace (Docs, Sheets, Drive — informal data store)
- Custom ATS (2010s, internal candidate database)

---

## Rules

### Rule 1: HubSpot Integration Guidance

**Scope:** Services and agents reading/writing HubSpot data
**Tool:** HubSpot CRM API
**Applies to:** `services/api`, `agents/sales-agent`, `data/pipelines/`
**Status:** Integration documented in `memory-bank/techContext.md` Current Stack (40% adoption)

**Recommended Access Patterns:**

**Read Access (Supported):**
- Sales pipeline data (deals, stages, amounts)
- Account information (company name, industry, location)
- Contact basic info (name, email, company)

**Write Access (Recommended Restrictions):**
- Write deal status updates (stage, closed_won/closed_lost)
- Write activity logs (calls, emails, notes)
- Avoid: Modifying contact details, deleting records, modifying deal amounts

**Data Privacy Considerations:**
- Retrieve only necessary data (avoid excessive internal notes/sensitive fields)
- Implement access logging for audit trail
- Manage API rate limits appropriately

**Example:**
```
✓ Agent reads: deal_id=123, stage="Negotiation", amount=50000
✓ Agent updates: stage="Closed Won", timestamp=2026-11-19T12:00:00Z

✗ Agent reads: contact.internal_notes (may contain sensitive data)
✗ Agent deletes: deal_id=123 (not recommended)
```

**Verification:** Code review checks integration patterns; API calls logged for audit

**Note:** Specific rate limits and policy enforcement require validation with Nexova technology team.

---

### Rule 2: Zendesk Integration Guidance

**Scope:** Support agents, services reading/writing Zendesk tickets
**Tool:** Zendesk API
**Applies to:** `services/api`, `agents/support-agent`, `data/pipelines/`

**Recommended Access Patterns:**

**Read Access (Supported):**
- Ticket metadata (ID, status, priority, created_at, updated_at)
- Ticket content (issue_text, resolution)
- Customer identifier (company, support tier)
- SLA status (time to resolution, breach flag)

**Write Access (Recommended Restrictions):**
- Write ticket resolution (comment, status=closed)
- Write tags and priorities
- Write internal notes (for support team collaboration)
- Avoid: Modifying customer info, deleting tickets, changing SLA settings

**Data Privacy Considerations:**
- Ticket content may contain customer PII (company data, error messages)
- Implement access logging for audit trail
- Plan for log retention aligned with operational needs
- Mask sensitive information in logs and debugging

**SLA Tracking Context:**
- Current expectation: 24-hour max resolution (Roberto Díaz target)
- Currently averaging 48 hours
- Agents should track SLA status and prioritize accordingly
- Example: "Ticket created 24 hours ago, still open → approaching SLA target → prioritize"

**Example:**
```
✓ Agent reads: ticket_id=789, status="open", created_at="2026-11-18T12:00Z"
✓ Agent writes: comment="Resolution: KB article 42", status="closed"
✓ Agent calculates: resolution_time and tracks against SLA target

✗ Agent modifies: customer company name (not recommended)
✗ Agent exports: ticket content to external database without coordination
```

**Verification:** Code review checks integration patterns; SLA metrics tracked in progress.md

**Note:** Specific SLA policies and escalation procedures require coordination with Roberto Díaz and support operations team.

---

### Rule 3: Gmail Integration Guidance

**Scope:** Agents sending candidate/client communications
**Tool:** Gmail API
**Applies to:** `agents/recruitment-agent`, `agents/support-agent`, `services/email-service`

**Recommended Access Patterns:**

**Read Access (Supported):**
- Incoming candidate inquiries (from Nexova email inboxes)
- Previous correspondence context (thread history)
- Attachments (CVs only; personal identity documents should be avoided)

**Write Access (Recommended Pattern):**
- Send emails using predefined templates
- Recommended: Structured templates stored in `shared/email-templates/`
- Avoid: Freeform email generation without template review
- Recipient validation: Check database records before sending

**Data Privacy Considerations:**
- Minimize inclusion of candidate's personal data (phone, address) in email
- Respect candidate communication preferences
- Plan for secure attachment handling
- Implement consent verification procedures

**Rate Limiting Guidance:**
- Manage email sending rate appropriately
- Monitor API quota usage
- Respect Gmail API rate limits

**Templates (Recommended Structure):**
Maintain template collection for common communications:
- Interview invitation templates
- Rejection notification templates
- Support acknowledgment templates
- Issue resolution templates

**Example:**
```
✓ Agent sends recruitment communication using template
  Recipient: candidate@example.com (verified in DB)
  Personalization: [CANDIDATE_NAME], [JOB_TITLE]

✗ Agent sends freeform email: "Hi, we got your CV, we'll be in touch" (no template)
✗ Agent includes: "Your phone is +34 666 123 456" (unnecessary PII)
```

**Verification:** Code review checks email patterns; template usage documented

**Note:** Template catalog and communication policies should be coordinated with Javier Almeida (Recruitment) and Roberto Díaz (Support).

---

### Rule 4: Google Workspace Integration Guidance

**Scope:** Services and agents reading/writing Google Docs, Sheets, Drive
**Tool:** Google Workspace API
**Applies to:** `services/api`, `data/pipelines/`, automation flows

**Recommended Access Patterns:**

**Read Access (Supported):**
- Read course catalog (Google Sheets, Training department)
- Read shared calendars (training schedules, event dates)
- Read shared documents (templates, guidelines)

**Write Access (Recommended Restrictions):**
- Append operations to Google Sheets (avoid modifying/deleting existing data)
- Create new documents only in designated folders (`/data/exports/`)
- Avoid: Modifying or deleting shared documents owned by departments

**Architecture Note:**
- Google Sheets currently serves as informal data store (spreadsheets)
- This is temporary; plan for migration to centralized data infrastructure
- Do NOT establish new data stores in Google Sheets; use centralized systems instead

**Coordination Requirements:**
- Writing to department-owned Google Sheets should be coordinated with relevant team (Training department, HR, etc.)
- Example: Writing to course enrollment sheet should align with Elena Vargas' operations

**Example:**
```
✓ Data pipeline reads: Google Sheet "Course Catalog", extracts courses
✓ Data pipeline appends: Google Sheet "Enrollment History", adds new enrollment row

✗ Agent deletes: rows from course catalog sheet
✗ Service creates: new data store in Google Sheets (use centralized API instead)
```

**Verification:** Code review checks write patterns; shared drive access documented

**Note:** Specific approval workflows and data governance policies require coordination with department owners.

---

### Rule 5: Audit Trail for API Integrations

**Scope:** Integrations with HubSpot, Zendesk, Gmail, Google Workspace

**Requirement:**
- Document significant API operations for audit trail purposes
- Recommended log format:
  - Timestamp (when operation occurred)
  - Tool name (HubSpot, Zendesk, Gmail, Google Workspace)
  - Operation type (READ, WRITE, UPDATE)
  - Resource identifier (deal_id, ticket_id, sheet_name, etc.)
  - Agent/service initiating operation
  - Operation result (success/failure)

**Example Log Format:**
```json
{
  "timestamp": "2026-11-19T12:00:00Z",
  "tool": "zendesk",
  "operation": "READ",
  "resource": "ticket:789",
  "agent": "support-agent-v1",
  "status": "success",
  "purpose": "SLA tracking for ticket 789"
}
```

**Verification:** Code review confirms logging is implemented; audit logs reviewed regularly

---

## API Credentials & Security

**Requirement:**
- Never hardcode API credentials in code
- Store credentials in `.env` file (repo root, never committed)
- Use environment variables in services:
  ```python
  HUBSPOT_API_KEY = os.getenv("HUBSPOT_API_KEY")
  ZENDESK_API_TOKEN = os.getenv("ZENDESK_API_TOKEN")
  ```
- Rotate credentials quarterly

**Verification:** Code review checks for hardcoded secrets; pre-commit hooks reject commits with credentials

---

## Violations & Escalation

| Violation | Action | Escalation |
|-----------|--------|-----------|
| Unauthorized HubSpot write (modify deal amount) | Review in code review | CTO review recommended |
| Zendesk ticket deletion attempt | Review in code review | Coordinate with Roberto Díaz |
| Untemplatized email sent | Log and document | Email flagged for review |
| Google Sheets data store creation | Reject in code review | Propose data model in GitHub issue |
| Hardcoded API credentials | Reject PR | Regenerate credentials, use env vars |
| Missing audit trail documentation | Review implementation | Add logging before merge |

---

## References

- **Source:** `memory-bank/techContext.md` Current Technology Stack & Integration Requirements
- **Context:** BRIEFING.md (Roberto Díaz/Zendesk, Javier Almeida/HubSpot, Elena Vargas/Google Sheets)
- **Safety:** data-privacy.md (PII masking, consent, audit trails)
- **Workflow:** AGENTS.md (pre-commit Step 2, protected files)

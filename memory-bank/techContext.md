# Technical Context - Nexova Solutions

**Last Updated:** 2026-11-19  
**Version:** 1.0  
**Audience:** AI Engineers, Agents, Backend Services

---

## Current Technology Stack

### Legacy Systems (Existing)
| System | Purpose | Status | Issues |
|--------|---------|--------|--------|
| **Custom ATS** (2010s) | Candidate tracking | Active | Outdated, no APIs, single-threaded |
| **HubSpot CRM** | Sales pipeline, deal tracking | Active (40% adoption) | Inconsistent data, manual sync issues |
| **Zendesk** (legacy) | Support ticketing | Active | Disconnected from other systems, no KB |
| **Google Workspace** | Email, Docs, Sheets, Drive | Active | Unofficial data store; spreadsheets as databases |
| **Email (Gmail)** | Candidate & client comms | Active | No tracking, manual follow-ups |

### Infrastructure
- **Hosting:** On-premises + partial cloud (Google Cloud)
- **Databases:** Legacy SQL databases (custom ATS), no centralized data warehouse
- **Logging & Monitoring:** None (failures detected via user complaints)
- **CI/CD:** Manual deployments
- **APIs:** Minimal; most integrations are manual exports/imports

---

## Data Model (High-Level)

### Candidates (Selection Operations)
```
Candidate {
  id: UUID
  name: string
  email: string
  phone: string
  cv_text: string | file
  experience_years: int
  skills: string[] (free-form tags)
  languages: {lang: string, level: string}[]
  education: string[]
  current_status: "in-process" | "rejected" | "hired" | "inactive"
  created_at: timestamp
  updated_at: timestamp
  notes: string[] (manual consultant notes)
}

JobOpening {
  id: UUID
  client_id: UUID
  title: string
  description: string
  required_skills: string[]
  min_experience_years: int
  min_education_level: string
  budget_range: string
  created_at: timestamp
  status: "open" | "filled" | "cancelled"
}
```

### Support Tickets (Outsourced Support)
```
Ticket {
  id: UUID
  client_id: UUID
  issue_text: string
  channel: "phone" | "email" | "chat"
  created_at: timestamp
  resolved_at: timestamp | null
  resolution_time_hours: float
  agent_id: UUID
  sentiment: "positive" | "neutral" | "negative" | null
  resolution_text: string | null
}
```

### Training & Courses (Corporate Training)
```
Course {
  id: UUID
  title: string
  description: string
  duration_hours: int
  level: "beginner" | "intermediate" | "advanced"
  topics: string[]
  price_eur: float
  capacity: int
  schedule: datetime[]
}

Enrollment {
  id: UUID
  course_id: UUID
  learner_id: UUID (client employee)
  enrollment_date: timestamp
  completion_date: timestamp | null
  attendance_rate: float (0-1)
}
```

---

## Domain Vocabulary (Critical for Agents)

| Term | Nexova Definition | Context |
|------|-------------------|---------|
| **Consultor** | Recruitment specialist (40 people in Ops) | Selection operations |
| **Candidato** | Person applying for client's job opening | Selection + candidate DB |
| **CV / Currículum** | Resume or CV document (PDF, Word, text) | Input to CV scorer |
| **Proceso de selección** | End-to-end recruitment process for one job | May span weeks |
| **Briefing del cliente** | Initial client requirements for a job opening | Defines success criteria |
| **Oferta de empleo** | Job posting/opportunity | Target for candidate matching |
| **Entrevista** | Interview round (screening, technical, final) | Manual process, not tracked |
| **Entrenamiento** | Training course or program | Corporate training product |
| **SLA** | Service Level Agreement (24-hour target for support) | Support operations metric |
| **Ticket** | Support issue from outsourced client | Support operations unit of work |
| **Agente de soporte** | Support team member (30 people) | Handles tickets multi-channel |

---

## Nexova-Specific Constraints

### Data Privacy & Compliance
- **GDPR:** All personal data (candidates, employees, support customers) must comply
- **Data Retention:** Inactive candidates retained 1 year per Spanish labor law
- **Consent:** Marketing comms require explicit opt-in
- **Right to Delete:** Candidates can request removal from database

### Operational Constraints
- **Language:** Spanish primary (Valencia office), English secondary (Miami office)
- **Business Hours:** Spanish business hours (9-18h CET) for decision-making
- **Multi-currency:** EUR (Spain) and USD (Florida) for pricing
- **SLA for Support:** 24-hour max resolution time (currently failing)

### Integration Requirements
- **HubSpot API:** Read pipeline, write deal status updates
- **Google Workspace API:** Email notifications, calendar sync for training
- **Gmail:** Read incoming candidate messages, structured email templates
- **Zendesk API:** Read tickets, post resolutions, extract KB content

---

## Proposed Technology Stack (AI Engineering Decision)

> **Note:** The following technology choices are proposed recommendations for this AI Engineering project, not existing Nexova commitments. They should be validated with stakeholders and CTO Sergio Molina before implementation.

### For Agent Development (Proposed)
- **LLM:** Claude 3+ for multi-step reasoning (CV analysis, proposal generation)
- **Retrieval:** RAG over candidate database (semantic search for experience/skills)
- **Embeddings:** OpenAI or Mistral (Spanish-aware models preferred)
- **Agents:** Python + LangChain or similar for orchestration

### For Memory & Context
- **Memory Bank:** This folder (`memory-bank/`) — projectbrief, techContext, progress
- **Skills Registry:** `.agents/skills/` — reusable, documented capabilities
- **Rules Engine:** `.agents/rules/` — constraints & compliance per department

### For Validation
- **Manual Testing:** Key workflows (CV scoring, training recommendation) validated with real data
- **Metrics:** Precision/recall on domain-specific tasks (not generic benchmarks)
- **Audit Trail:** All agent decisions logged for compliance review

---

## Security & Access Patterns

| Resource | Access Level | Notes |
|----------|--------------|-------|
| **Candidate Database** | Consultors only (read), AI agents (read with masking) | PII sensitive |
| **Support Tickets** | Support team + managers | Contains customer PII |
| **Training Catalog** | Public read (catalog), managers (enrollment admin) | No personal data |
| **Internal HR Data** | HR team + employee self-service | Highest sensitivity |
| **CRM (HubSpot)** | Sales team (inconsistent), AI agents (read-only) | Deal-sensitive data |

---

## References & Sources

- **BRIEFING.md:** Full company and department descriptions
- **projectbrief.md:** This document's companion (business context)
- **progress.md:** Current work status and milestones
- **AGENTS.md:** Agent catalog and development workflow (at repo root)

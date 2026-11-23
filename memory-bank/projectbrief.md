# Project Brief - Nexova Solutions

**Last Updated:** 2026-11-19
**Status:** AI Engineering Project — Milestone 1 (Memory Bank & Agents Structure)

---

## Executive Summary

**Nexova Solutions** is an HR consulting and talent acquisition firm founded in 2011, headquartered in Valencia, Spain, with an expansion office in Miami, Florida. The company operates across three business lines:

1. **Headhunting** — mid-level and executive recruitment
2. **Outsourced Customer Support** — dedicated support teams for tech, retail, and financial services clients
3. **Corporate Training** — soft skills and leadership development

**Scale:** 120 employees | $8M USD annual revenue | 12-year track record

---

## Organizational Structure

| Department | Head | Team Size | Focus |
|-----------|------|-----------|-------|
| **Selection Operations** | Javier Almeida | 40 | CV screening, candidate matching, recruitment workflows |
| **Corporate Training** | Elena Vargas | 12 | Program design, course catalog, learner management |
| **Outsourced Support** | Roberto Díaz | 30 | Multi-channel support (phone, email, chat) for clients |
| **Sales & Business Dev** | Megan Clarke | 18 | Account management, prospecting, pipeline |
| **Marketing & Comms** | Carmen Ruiz | Small | Web presence, LinkedIn, newsletter, content |
| **Internal HR** | Patricia Solís | 4 | Employee onboarding, payroll, leave management |
| **Tech & Infrastructure** | Sergio Molina (CTO) | 6 | Legacy systems, ATS, integrations |
| **Executive** | Laura Mendoza (CEO) | — | Strategic direction, weekly reporting |

---

## Core Business Problems (AI Intervention Opportunities)

### Selection Operations (Primary Revenue Driver)
- **Pain:** Manual CV screening (30-80 CVs per consultant per process)
- **Need:** AI-powered CV scoring, semantic search over candidate database, real-time candidate tracking
- **Impact:** Reduce manual work, improve candidate-job fit, faster time-to-hire

### Corporate Training
- **Pain:** Catalog in PDF updated quarterly; inscriptions via Google Forms + spreadsheets
- **Need:** Searchable training catalog, AI-powered course recommendations, learner progress tracking
- **Impact:** Better discoverability, personalized learning paths, enrollment automation

### Outsourced Support (SLA Risk)
- **Pain:** 48-hour avg resolution time vs. 24-hour SLA; no centralized knowledge base
- **Need:** First-line chatbot (RAG-powered), centralized knowledge base, real-time visibility
- **Impact:** 40% automation target, SLA compliance, improved customer satisfaction

### Sales & Business Development
- **Pain:** Manual prospecting (LinkedIn exports to Excel), inconsistent CRM usage, lost follow-ups
- **Need:** Sales dashboard, prospecting automation, AI-powered proposal angles per prospect
- **Impact:** Deal velocity, reduced churn, better pipeline visibility

### Internal HR
- **Pain:** Email-based vacation requests, manual onboarding, no people analytics
- **Need:** HR portal, automated onboarding workflows, HR KPI dashboard, internal HR chatbot
- **Impact:** Employee experience, operational efficiency, data-driven HR

### Technology & Infrastructure
- **Pain:** Disconnected tools (HubSpot, legacy Zendesk, custom ATS, Google Workspace, spreadsheets)
- **Need:** Telemetry & logging, centralized data pipeline, real-time monitoring, internal architecture chatbot
- **Impact:** Operational visibility, faster incident response, informed decision-making

### Executive Leadership
- **Pain:** Manual weekly PDFs (4-8 hours per manager) with week-old data
- **Need:** Real-time executive dashboard, automated weekly summary, strategic alerting
- **Impact:** Faster decisions, unified business view, reduced admin overhead

---

## Key Constraints & Domain Requirements

- **Data Privacy:** Candidate and employee data must comply with GDPR (Spain/EU)
- **Multi-language:** Operations in Spanish and English
- **B2B Focus:** Users are professional teams with structured workflows, not consumers
- **Legacy Integration:** Must work with existing ATS, CRM (HubSpot), Zendesk, Google Workspace
- **Performance:** Selection operations (40 consultants) need responsive tools for daily use
- **SLA Compliance:** Support operations have hard 24-hour SLA targets

---

## Success Metrics (Defined per Agent)

| Agent/System | Key Metric | Target |
|-------------|-----------|--------|
| CV Scoring Skill | Precision of relevance scoring (manual validation) | >85% |
| Recruitment Agent | % of CVs screened by AI vs. manual | 80%+ |
| Support Chatbot | % of tickets resolved without escalation | 40%+ |
| Training Recommender | % of learners following recommended path | 60%+ |
| HR Portal | % of leave requests submitted via portal | 90%+ |

---

## References

- **Full BRIEFING:** See `../BRIEFING.md` for detailed department descriptions
- **Tech Stack:** See `techContext.md` for current systems and architecture
- **Project Status:** See `progress.md` for completed work and next steps

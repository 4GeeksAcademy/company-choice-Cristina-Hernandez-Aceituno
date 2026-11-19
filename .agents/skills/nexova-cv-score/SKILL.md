# nexova-cv-score Skill

**Last Updated:** 2026-11-19  
**Owner:** Javier Almeida (Selection Operations)  
**Purpose:** Reusable skill for scoring and evaluating candidate CVs against job opening requirements

---

## 1. Skill Overview

### Objective

Compare a candidate's CV (resume) against a specific job opening's requirements and produce a **scored assessment** indicating how well the candidate matches the position.

**Key Use:** Assist recruitment consultants (Consultores) in prioritizing candidates during the screening phase of a Proceso de Selección (recruitment process).

### When to Use This Skill

- **Selection Operations:** Consultants cribbing CVs (30-80 per process) need to rank candidates by fit
- **Recruitment Agent:** Automated CV screening before human review
- **Training Recommendation Engine:** Recommend upskilling for candidates who are close but gap some requirements
- **CLI Tool:** Manual CV scoring for quick assessments

### Reusability

This skill is **reusable across multiple contexts**:
- Used by human consultants (manual scoring)
- Used by automated agents (batch scoring)
- Used by tools (command-line, web API)
- Can be combined with other skills (e.g., "Score + Recommend Training")

---

## 2. Inputs & Format

### Input 1: Candidate CV

**Field Name:** `cv_data`  
**Type:** Object  
**Required:** Yes

```json
{
  "candidate_id": "UUID (optional, for tracking)",
  "candidate_name": "Name_[MASKED] (PII masked per data-privacy.md Rule 1)",
  "cv_text": "string (full CV/resume text, extracted from PDF/Word/text)",
  "experience_summary": {
    "total_years": "integer (1-50)",
    "roles": "string[] (list of job titles, e.g., ['Sales Manager', 'Account Executive'])",
    "industries": "string[] (e.g., ['SaaS', 'Fintech', 'Retail'])"
  },
  "skills": {
    "technical": "string[] (e.g., ['CRM', 'SQL', 'Python'])",
    "languages": "object[] (e.g., [{'lang': 'English', 'level': 'C1'}, {'lang': 'Spanish', 'level': 'Native'}])",
    "soft_skills": "string[] (e.g., ['Negotiation', 'Team Leadership'])"
  },
  "education": {
    "degree": "string (e.g., 'Bachelor in Business Administration')",
    "field": "string (e.g., 'Business')",
    "institution": "string (optional)"
  }
}
```

### Input 2: Job Opening Requirements

**Field Name:** `job_profile`  
**Type:** Object  
**Required:** Yes

```json
{
  "job_id": "UUID (for tracking)",
  "job_title": "string (e.g., 'Senior Sales Manager')",
  "client_name": "Client_[MASKED] (PII masked)",
  "job_description": "string (full job posting text)",
  "requirements": {
    "min_experience_years": "integer (minimum experience threshold)",
    "required_skills": "string[] (non-negotiable skills)",
    "preferred_skills": "string[] (nice-to-have skills)",
    "required_languages": "object[] (e.g., [{'lang': 'English', 'min_level': 'B2'}])",
    "min_education_level": "string (e.g., 'Bachelor degree')",
    "industry_preference": "string[] (e.g., ['SaaS', 'Tech'])"
  },
  "scoring_weights": {
    "experience": 0.30,
    "required_skills": 0.35,
    "preferred_skills": 0.15,
    "languages": 0.10,
    "education": 0.10
  }
}
```

### Input Validation

- Both `cv_data` and `job_profile` must be non-empty
- `candidate_id` and `job_id` are recommended for audit trail (data-privacy.md Rule 3)
- All PII (names, emails, phone) must be masked before input (data-privacy.md Rule 1)
- No hardcoded credentials or sensitive data in inputs
- Language levels must be one of: A1, A2, B1, B2, C1, C2, Native

---

## 3. Procedure (Step-by-Step)

### Step 1: Validate Inputs
- Confirm both `cv_data` and `job_profile` are present and well-formed
- Verify PII masking (no unmasked names, emails, phone numbers)
- Validate language levels (A1-C2 or Native)
- **Action if failed:** Return error with missing/invalid fields

### Step 2: Extract & Normalize Candidate Profile
- Parse CV text to extract:
  - Experience years (from job titles/dates)
  - Technical skills (from CV skills section)
  - Language proficiency (from education/international experience)
  - Education level (from degree, institution)
  - Soft skills (from job descriptions and achievements)
- Normalize skill names to match job profile vocabulary
  - Example: "Sales Ops" → "Sales Operations"
  - Example: "Advanced English" → "C1" language level
- Handle missing data gracefully (treat as "not mentioned, 0 score for this dimension")

### Step 3: Score by Dimension

#### Dimension 1: Experience (30% weight by default)
- **Calculation:** `(candidate_years - job_min_years) / job_min_years`, capped at 1.0
- **Example:** Job requires 5 years, candidate has 7 years → (7-5)/5 = 0.4, capped at 1.0 = **1.0 (exceeds)**
- **Example:** Job requires 5 years, candidate has 2 years → (2-5)/5 = -0.6, floor at 0 = **0.0 (insufficient)**
- **Example:** Job requires 5 years, candidate has 7 years → (7-5)/5 = 0.4 = **0.4 (partial)**

#### Dimension 2: Required Skills (35% weight by default)
- **Calculation:** `(matched_required_skills / total_required_skills)`
- **Example:** Job requires [CRM, SQL, Python], candidate has [CRM, Python] → 2/3 = **0.67**
- **Example:** Job requires [CRM], candidate has [CRM, Python, Sales] → 1/1 = **1.0 (all match)**
- Note: Matching is semantic (e.g., "Salesforce" matches "CRM")

#### Dimension 3: Preferred Skills (15% weight by default)
- **Calculation:** `(matched_preferred_skills / total_preferred_skills)`, or 0 if none preferred
- **Example:** Job prefers [Leadership, Agile], candidate has [Leadership] → 1/2 = **0.5**
- **Example:** Job prefers nothing → **1.0 (N/A, full score)**

#### Dimension 4: Languages (10% weight by default)
- **Calculation:** Match candidate's language level against job requirement
  - If required level met: **1.0**
  - If candidate's level < required: **0.0**
  - If candidate's level > required: **1.0** (exceeds)
- **Example:** Job requires Spanish B1, candidate has Spanish C1 → **1.0**
- **Example:** Job requires English C1, candidate has English B2 → **0.0 (insufficient)**
- If multiple languages required, score each and average

#### Dimension 5: Education (10% weight by default)
- **Calculation:** 
  - If candidate's degree meets/exceeds requirement: **1.0**
  - If candidate's degree below requirement: **0.5** (partial credit)
  - If candidate's degree not mentioned: **0.5** (assume partial)
- **Example:** Job requires Bachelor's degree, candidate has Master's → **1.0**
- **Example:** Job requires Bachelor's degree, candidate has High School → **0.5**

### Step 4: Calculate Weighted Score
- **Formula:** `final_score = (exp_score * 0.30) + (req_skills_score * 0.35) + (pref_skills_score * 0.15) + (lang_score * 0.10) + (edu_score * 0.10)`
- Scale to 0-100: `final_score_percent = final_score * 100`

### Step 5: Determine Fit Category
- **80-100:** Excellent fit (candidate is well-qualified)
- **60-79:** Good fit (candidate meets core requirements, may have skill gaps)
- **40-59:** Partial fit (candidate has some relevant experience, notable gaps)
- **0-39:** Poor fit (candidate does not meet core requirements)

### Step 6: Generate Evidence & Explanation
- For each dimension, provide:
  - **What matched:** List skills/languages/education that aligned
  - **What is missing:** List required items not found in CV
  - **Confidence:** How certain are we in the extraction (high if explicit, low if inferred)
  
  Example:
  ```
  Experience: 7 years (exceeds 5-year requirement) [HIGH CONFIDENCE]
  Required Skills: 2/3 matched (CRM ✓, Python ✓, SQL ✗ missing) [MEDIUM CONFIDENCE]
  Languages: English C1 (meets B2 requirement) ✓ [HIGH CONFIDENCE]
  Education: Bachelor's (meets Bachelor's requirement) ✓ [HIGH CONFIDENCE]
  ```

### Step 7: Log Audit Trail
- Record score decision per data-privacy.md Rule 3
  ```json
  {
    "timestamp": "2026-11-19T12:00:00Z",
    "agent": "recruitment-agent-v1",
    "action": "CV_SCORED",
    "candidate_id": "cand-123",
    "job_id": "job-456",
    "score": 72,
    "category": "Good fit",
    "purpose": "Screening batch for Job Opening 456"
  }
  ```

---

## 4. Output & Expected Result

### Output Format

```json
{
  "score": 72,
  "score_percent": "72/100",
  "fit_category": "Good fit",
  "dimensions": {
    "experience": {
      "score": 1.0,
      "weight": 0.30,
      "weighted_contribution": 0.30,
      "evidence": "Candidate 7 years vs. 5 required (exceeds by 2 years)"
    },
    "required_skills": {
      "score": 0.67,
      "weight": 0.35,
      "weighted_contribution": 0.23,
      "evidence": "2/3 matched: CRM ✓, Python ✓, SQL ✗"
    },
    "preferred_skills": {
      "score": 0.50,
      "weight": 0.15,
      "weighted_contribution": 0.08,
      "evidence": "1/2 matched: Leadership ✓, Agile ✗"
    },
    "languages": {
      "score": 1.0,
      "weight": 0.10,
      "weighted_contribution": 0.10,
      "evidence": "English C1 (meets B2 requirement)"
    },
    "education": {
      "score": 1.0,
      "weight": 0.10,
      "weighted_contribution": 0.10,
      "evidence": "Bachelor's in Business (meets requirement)"
    }
  },
  "summary": "Strong candidate with all required experience and most required skills. Missing SQL expertise, but otherwise well-qualified for Senior Sales Manager role.",
  "recommendation": "ADVANCE_TO_INTERVIEW",
  "gaps": ["SQL skill (required)", "Agile methodology (preferred)"],
  "strengths": ["7 years sales experience", "CRM & Python proficiency", "Fluent English & Spanish"],
  "audit_trail_id": "audit-789"
}
```

### Output Explanation

- **score (0-100):** Numeric fit score based on weighted dimensions
- **fit_category:** Human-readable category (Excellent/Good/Partial/Poor)
- **dimensions:** Breakdown by skill, experience, language, education
  - `score`: Dimension score (0-1)
  - `weight`: Percentage of total score (should sum to 1.0)
  - `weighted_contribution`: score × weight
  - `evidence`: Explanation of what matched/missed
- **summary:** Narrative explanation of fit
- **recommendation:** Suggested next action (ADVANCE_TO_INTERVIEW, REVIEW_GAPS, REJECT)
- **gaps:** List of missing required skills/experience
- **strengths:** List of matching qualities
- **audit_trail_id:** Reference to audit log (for data-privacy.md Rule 3)

---

## 5. Acceptance Criteria (Verification)

### Functional Criteria

| Criterion | How to Verify | Pass/Fail |
|-----------|---------------|-----------|
| **Score range valid** | Output score is 0-100 | ✓ Automated check |
| **Fit category matches score** | 80+ = Excellent, 60-79 = Good, etc. | ✓ Automated check |
| **Dimension calculations correct** | Each dimension score 0-1, weighted correctly | ✓ Automated check with example data |
| **Evidence provided** | Each dimension has explanation text | ✓ Manual review |
| **Audit trail logged** | JSON log created with candidate/job IDs | ✓ Manual review |
| **PII not exposed** | No unmasked names/emails in output | ✓ Manual review |

### Quality Criteria

| Criterion | How to Verify | Pass/Fail |
|-----------|---------------|-----------|
| **Consistency** | Same CV + Job should produce same score | ✓ Run twice, compare |
| **Interpretability** | Evidence is clear to recruitment consultant | ✓ Recruit review test |
| **Accuracy** | Score matches consultant's manual assessment (>80% agreement) | ✓ Blind comparison with 10 real CVs |
| **Semantic matching** | "Salesforce" matches "CRM" skill | ✓ Manual verification |
| **Language level matching** | "C1 English" correctly mapped to CEFR scale | ✓ Manual verification |

### Security Criteria

| Criterion | How to Verify | Pass/Fail |
|-----------|---------------|-----------|
| **No PII in input** | Candidate name/email masked | ✓ Automated regex check |
| **No PII in output** | Candidate ID only, no names | ✓ Automated regex check |
| **Audit trail complete** | Log includes WHO, WHAT, WHEN, WHY, HOW | ✓ Automated check |
| **No hardcoded credentials** | No API keys, passwords in skill code | ✓ Code review |

---

## 6. Complete Example

### Input: Candidate CV

```json
{
  "candidate_id": "cand-12345",
  "candidate_name": "Name_[MASKED]",
  "cv_text": "Senior Sales Manager with 7 years in SaaS and Fintech. Expert in Salesforce CRM, Python scripting, and team leadership. Fluent in English (C1) and Spanish (Native). MBA in Business Administration from Universidad Carlos III. Led teams of 5-10 across Europe and Miami.",
  "experience_summary": {
    "total_years": 7,
    "roles": ["Senior Sales Manager", "Sales Manager", "Account Executive"],
    "industries": ["SaaS", "Fintech"]
  },
  "skills": {
    "technical": ["Salesforce", "Python", "Excel", "Tableau"],
    "languages": [
      {"lang": "English", "level": "C1"},
      {"lang": "Spanish", "level": "Native"}
    ],
    "soft_skills": ["Team Leadership", "Negotiation", "Client Relations"]
  },
  "education": {
    "degree": "MBA",
    "field": "Business Administration",
    "institution": "Universidad Carlos III"
  }
}
```

### Input: Job Opening

```json
{
  "job_id": "job-98765",
  "job_title": "Senior Sales Manager",
  "client_name": "Client_[MASKED]",
  "job_description": "Lead a sales team of 8-12 to achieve revenue targets in EMEA region. Manage CRM (Salesforce), coordinate with technical teams (Python background required), and develop client relationships. Bilingual English/Spanish required.",
  "requirements": {
    "min_experience_years": 5,
    "required_skills": ["CRM", "Python", "SQL"],
    "preferred_skills": ["Leadership", "Agile"],
    "required_languages": [
      {"lang": "English", "min_level": "B2"},
      {"lang": "Spanish", "min_level": "B1"}
    ],
    "min_education_level": "Bachelor degree",
    "industry_preference": ["SaaS", "Tech"]
  },
  "scoring_weights": {
    "experience": 0.30,
    "required_skills": 0.35,
    "preferred_skills": 0.15,
    "languages": 0.10,
    "education": 0.10
  }
}
```

### Processing

#### Step 1: Validation ✓
- Inputs present and well-formed
- PII masked (names as "Name_[MASKED]", clients as "Client_[MASKED]")
- Language levels valid (C1, B2, B1, Native)

#### Step 2: Profile Extraction ✓
- Experience: 7 years (from CV)
- Required skills detected: CRM (Salesforce ✓), Python ✓, SQL ✗ missing
- Preferred skills detected: Leadership ✓, Agile ✗ missing
- Languages: English C1, Spanish Native
- Education: MBA (exceeds Bachelor requirement)

#### Step 3: Dimension Scoring

```
Experience:
  candidate_years = 7
  job_min_years = 5
  score = min((7-5)/5, 1.0) = min(0.4, 1.0) = 1.0 ✓ (exceeds minimum)

Required Skills (CRM, Python, SQL):
  matched: ["CRM", "Python"]
  missing: ["SQL"]
  score = 2/3 = 0.67

Preferred Skills (Leadership, Agile):
  matched: ["Leadership"]
  missing: ["Agile"]
  score = 1/2 = 0.50

Languages:
  English: candidate C1, required B2 → 1.0 ✓
  Spanish: candidate Native, required B1 → 1.0 ✓
  score = average(1.0, 1.0) = 1.0

Education:
  Candidate: MBA (exceeds Bachelor requirement)
  score = 1.0 ✓
```

#### Step 4: Weighted Score
```
final_score = (1.0 × 0.30) + (0.67 × 0.35) + (0.50 × 0.15) + (1.0 × 0.10) + (1.0 × 0.10)
            = 0.30 + 0.23 + 0.08 + 0.10 + 0.10
            = 0.81
            = 81/100
```

#### Step 5: Fit Category
81 falls in 80-100 range → **Excellent fit**

### Output: Score Result

```json
{
  "score": 81,
  "score_percent": "81/100",
  "fit_category": "Excellent fit",
  "dimensions": {
    "experience": {
      "score": 1.0,
      "weight": 0.30,
      "weighted_contribution": 0.30,
      "evidence": "Candidate 7 years vs. 5 required (exceeds by 2 years)"
    },
    "required_skills": {
      "score": 0.67,
      "weight": 0.35,
      "weighted_contribution": 0.23,
      "evidence": "2/3 matched: Salesforce/CRM ✓, Python ✓, SQL ✗ missing"
    },
    "preferred_skills": {
      "score": 0.50,
      "weight": 0.15,
      "weighted_contribution": 0.08,
      "evidence": "1/2 matched: Team Leadership ✓, Agile ✗ missing"
    },
    "languages": {
      "score": 1.0,
      "weight": 0.10,
      "weighted_contribution": 0.10,
      "evidence": "English C1 (exceeds B2 requirement), Spanish Native (exceeds B1 requirement)"
    },
    "education": {
      "score": 1.0,
      "weight": 0.10,
      "weighted_contribution": 0.10,
      "evidence": "MBA in Business Administration (exceeds Bachelor degree requirement)"
    }
  },
  "summary": "Excellent candidate. Exceeds experience requirement (7 vs. 5 years), bilingual at required levels, and holds an advanced MBA. Proficient in Salesforce CRM and Python. Only gap: SQL skill is not mentioned in CV, though could be learnable.",
  "recommendation": "ADVANCE_TO_INTERVIEW",
  "gaps": ["SQL skill (required—recommend screening interview on this)"],
  "strengths": [
    "7 years sales management experience",
    "Salesforce CRM expertise",
    "Python proficiency",
    "Bilingual English (C1) and Spanish (Native)",
    "MBA in Business Administration",
    "Leadership experience with teams of 5-10"
  ],
  "audit_trail_id": "audit-cv-score-20261119-cand12345-job98765"
}
```

### Next Actions
- **Consultant Action:** Contact candidate for initial screening
- **Training Path:** If candidate interested but needs SQL, recommend online course (available in Nexova training catalog)

---

## 7. Limitations & Data Protection

### Known Limitations

1. **CV Text Parsing:** Relies on well-formatted CV text. Handwritten CVs or unusual formats may not extract correctly. Confidence score lower for unstructured data.

2. **Semantic Matching:** Skill matching is semantic (e.g., "Salesforce" = "CRM"), but some domain-specific terms may not match. Example: "Lead generation platform" vs. "Sales automation" may not register as equivalent without manual review.

3. **Missing Skills Ambiguity:** If a CV doesn't mention SQL, it's unclear whether the candidate lacks the skill or simply didn't list it. Score assumes "not mentioned = not proficient" (conservative).

4. **Language Level Accuracy:** Self-reported language levels in CVs are not verified. A candidate claiming "C1 English" may not actually be C1. Skill cannot validate language proficiency.

5. **Industry/Role Context:** Scoring doesn't account for industry-specific nuances. Example: "CRM experience" in Retail (transaction-based) is different from "CRM experience" in SaaS (relationship-based), but skill treats both equally.

6. **Static Weights:** Default weights (30% experience, 35% required skills, etc.) may not apply to all roles. For specialized positions, weights should be adjusted manually.

### Data Protection Measures

#### Input Validation (data-privacy.md Rule 1: PII Masking)
- **Requirement:** All candidate names, emails, phone numbers MUST be masked before input
- **Implementation:** Reject inputs with unmasked PII; return error
- **Example:** Accept "Name_[MASKED]", reject "Juan García"

#### Access Control (data-privacy.md Rule 2)
- **Who can use this skill:** 
  - Javier Almeida (Selection Ops lead)
  - Recruitment consultants (40 consultants)
  - Automated recruitment agents (with logging)
  - Training recommendation engine (for upskilling paths)
- **Who cannot:** Support agents, marketing team, HR (not involved in selection)
- **Implementation:** Access controlled via role-based checks in service calling this skill

#### Audit Trail (data-privacy.md Rule 3)
- **Requirement:** Log all CV scoring decisions
- **What to log:**
  - `timestamp`: When score was computed
  - `agent`: Who/what scored (consultant ID, agent name)
  - `candidate_id`: Candidate UUID only (never name)
  - `job_id`: Job opening UUID
  - `score`: Final score
  - `purpose`: Why score was computed ("Initial screening", "Recommendation engine")
- **Example:**
  ```json
  {
    "timestamp": "2026-11-19T12:00:00Z",
    "agent": "consultant-javier-001",
    "candidate_id": "cand-12345",
    "job_id": "job-98765",
    "score": 81,
    "purpose": "Manual screening for Senior Sales Manager role"
  }
  ```

#### Right to Delete (data-privacy.md Rule 4)
- If a candidate requests deletion, CV text and scoring history should be anonymized/removed
- Skill output references only `candidate_id` (UUID), so removing actual CV data doesn't break audit trail
- **Process:** Coordinate with Selection Ops lead before deleting

#### No Credential Exposure
- Skill does NOT require API keys, passwords, or external service calls
- Operates entirely on input data provided (no external lookups)
- Safe to run without credential injection

---

## 8. Implementation Notes

### For AI Agents
- When using this skill in an agent workflow, follow data-privacy.md rules
- Mask PII in CV text before passing to skill
- Log all scoring decisions in audit trail
- If used for training recommendations, combine with `.agents/skills/training-recommendation` (future skill)

### For Manual Use (Consultants)
- Input CSV with candidate CVs (names masked)
- Input job requirements document
- Skill produces scored rankings
- Consultant reviews top candidates + gap analysis before contacting

### For CLI Tool Integration
```bash
# Example (pseudocode, not real implementation)
nexova-cv-score \
  --cv-file candidates-masked.json \
  --job-file job-requirements.json \
  --output scores.json \
  --audit-log scores-audit.log
```

### Dependencies (Proposed, Not Implemented)
- LLM for semantic skill matching (Claude 3+ recommended per techContext.md)
- Embedding model for skill similarity (optional, if doing deep semantic match)
- Logging framework for audit trail
- Database connection for fetching candidate/job profiles (if used in service)

---

## 9. References

- **BRIEFING.md:** "Operaciones de Selección" section — CV screening need
- **memory-bank/projectbrief.md:** Selection Operations: 40 consultants, 30-80 CVs/process
- **memory-bank/techContext.md:** Candidate data model, Domain Vocabulary (Consultor, Candidato, Oferta)
- **AGENTS.md:** Skill creation guidelines; skills are reusable, not mandatory per agent
- **.agents/rules/memory-access.md:** Vocabulary usage (Candidato, CV, Oferta de empleo)
- **.agents/rules/data-privacy.md:** 
  - Rule 1: PII masking (names, emails, phone)
  - Rule 2: Candidate database access control (Selection Ops + AI agents only)
  - Rule 3: Audit trail logging (WHO, WHAT, WHEN, WHY, HOW)
  - Rule 4: Right to Delete (soft delete, anonymization, retention)
- **.agents/rules/development.md:** 
  - Rule 3: Nexova domain modeling (use Nexova vocabulary)
  - Rule 5: Documentation requirements (references memory-bank, constraints, examples)

---

## Versioning

| Version | Date | Change |
|---------|------|--------|
| 1.0 | 2026-11-19 | Initial skill: CV scoring with 5 dimensions, fit categories, audit trail logging |

---

## Skill Status

✅ **Documented:** Complete specification with inputs, process, outputs, criteria, example, limitations  
✅ **Reusable:** Defined for multiple contexts (manual consultant, AI agent, training engine, CLI)  
✅ **Verifiable:** Acceptance criteria with automated and manual checks  
✅ **Data-Protected:** Aligns with data-privacy.md, audit trail, PII masking, access control  
⏳ **Implementation:** Pending Phase 5 (agent development)

---

**Last Reviewed:** 2026-11-19  
**Next Review:** After implementation in Phase 5 (agent development)

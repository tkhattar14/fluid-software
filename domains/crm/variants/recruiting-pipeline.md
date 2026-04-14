---
domain: crm
variant: recruiting-pipeline
status: illustrative
contributor: "Fluid Software"
version: 0.1

stages:
  seed:
    description: "Solo recruiter or founder hiring, <10 open roles"
    architecture:
      database: sqlite
      deployment: single-server
      pattern: monolith
  growth:
    description: "Small recruiting team, 10-50 open roles"
    architecture:
      database: postgresql
      deployment: api-server + workers
      pattern: modular-monolith
  scale:
    description: "Recruiting org, 50+ open roles, high-volume pipeline"
    architecture:
      database: postgresql-read-replicas
      deployment: api + workers + search-index
      pattern: event-driven

entities:
  - name: candidate
    fields: [name, email, phone, current_role, skills, location, resume, source]
    relationships: [applications, interviews, notes]

  - name: job
    fields: [title, department, location, type, salary_range, status, hiring_manager]
    relationships: [applications, interviewers]

  - name: application
    fields: [candidate, job, stage, source, applied_date, status]
    relationships: [candidate, job, interviews, scorecards]

  - name: interview
    fields: [application, interviewer, type, scheduled_date, scorecard, status]
    relationships: [application]

  - name: scorecard
    fields: [interview, criteria, ratings, recommendation, notes]
    relationships: [interview]

workflows:
  - name: candidate-pipeline
    trigger: application.created
    stages: [seed, growth, scale]
  - name: interview-scheduling
    trigger: application.stage == interview
    stages: [seed, growth, scale]
  - name: offer-management
    trigger: application.stage == offer
    stages: [growth, scale]

integrations:
  - name: job-boards
    examples: [linkedin, indeed, greenhouse-api]
    required: false
  - name: calendar
    examples: [google-calendar, outlook]
    required: true
  - name: email
    examples: [gmail-api, outlook-api]
    required: true
  - name: background-check
    examples: [checkr, certn]
    required: false
    stages: [growth, scale]

constraints:
  - "Candidate data subject to employment law and data protection (GDPR, CCPA)"
  - "Equal opportunity compliance — track demographics for reporting, not filtering"
  - "Data retention: candidate data should be purged after configurable period if not hired"
---

# Recruiting Pipeline CRM

> **Status: Illustrative.** This spec demonstrates the format. If you're a recruiter, talent lead, or HR professional, we'd value your contribution to upgrade this to a `real` spec.

## What Makes Recruiting CRM Different

The primary relationship is **candidate-to-job**, not person-to-company. One candidate can apply to multiple jobs. One job has many candidates. The pipeline is per-application, not per-person.

Time sensitivity is high. Good candidates get multiple offers. A slow pipeline loses candidates. The CRM needs to surface "who needs attention right now" instantly.

Compliance is non-negotiable. Employment law governs what you can track, how long you can keep it, and how decisions must be documented.

## Candidate Pipeline

```
APPLIED → SCREENED → INTERVIEW → FINAL ROUND → OFFER → HIRED
                                                   ↓
                                                REJECTED (with reason)
```

At seed stage, the founder is the hiring manager, interviewer, and recruiter. The "CRM" is a spreadsheet with columns for name, role, status, and notes.

At growth stage, multiple people are involved. The CRM needs handoff between recruiter (sourcing, screening) and hiring manager (interviews, decisions). Structured scorecards replace informal notes.

At scale, the pipeline needs search (find candidates by skill, location, history), bulk operations (reject 50 candidates for a closed role), and analytics (time-to-hire, source effectiveness, interviewer calibration).

## Common Mistakes

1. **No structured rejection reasons.** "Didn't feel right" is useless. Track: skills gap, culture fit, compensation mismatch, timing, role filled. This data drives sourcing strategy.
2. **Losing past candidates.** Someone rejected for Role A might be perfect for Role B six months later. The CRM should make past candidates searchable and re-engageable.
3. **Interview feedback in email/Slack.** If feedback isn't in the CRM, it doesn't exist for compliance purposes.

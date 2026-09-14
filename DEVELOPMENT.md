# GayatriBot-HR → GayatriHire Development Roadmap

**Status:** Active development blueprint  
**Repository:** `Gayatri-Education/gayatribot-Hr`  
**Target:** Evolve the current HR automation prototype into a commercially viable, multi-tenant hiring automation platform.

---

## 1. Purpose

GayatriBot-HR is the starting point for a larger product vision: **GayatriHire**.

The goal is **not** to build another generic HRMS or copy an existing ATS.

The target product is:

> **A high-volume hiring automation platform for Indian organisations that automates repetitive recruitment operations while keeping humans in control of hiring decisions.**

The platform should eventually cover:

```text
Candidate Acquisition
        ↓
Application
        ↓
Validation
        ↓
Eligibility
        ↓
CV / Skill Analysis
        ↓
Screening
        ↓
Assessment
        ↓
Interview Scheduling
        ↓
Interview Feedback
        ↓
Selection
        ↓
Offer
        ↓
Acceptance
        ↓
Joining
        ↓
Onboarding / HRMS Integration
```

This document is the **engineering source of truth for contributors**. Features should be implemented in the order described here rather than adding unrelated features on top of the current prototype.

---

# 2. Current Repository: Starting Point

The current repository already contains important building blocks:

```text
bot.py
database.py
bot_config.py
enrollment_filter.py
flask_app.py
offer_letter_sender.py
send_control.py
smtp_sender.py
smtp_service.py
archive_offer_logs.py
templates/
requirements.txt
```

The current README describes Google Sheets integration, SMTP automation, CV/LLM processing, offer-letter generation, and a Flask administration dashboard.

The current implementation is useful as a prototype, but it should **not be treated as production-ready SaaS infrastructure**.

The first engineering objective is therefore:

> **Stabilise and secure the existing system before adding major product functionality.**

---

# 3. Product Vision

## 3.1 Core promise

GayatriHire should eventually answer this business problem:

> "I have thousands or tens of thousands of applications. How can my HR team validate, screen, communicate with, schedule, select and onboard candidates without manually performing the same operations repeatedly?"

The platform should optimise for:

- high-volume recruitment
- automation
- operational visibility
- explainable candidate evaluation
- human approval
- candidate experience
- integrations
- security
- auditability
- measurable ROI

---

# 4. What We Are NOT Building

Do not turn this project into a collection of disconnected AI demos.

Avoid building features merely because they sound impressive:

- generic AI chatbot
- AI-generated HR slogans
- sentiment analysis without a business use case
- automatic candidate rejection by an LLM
- arbitrary resume summarisation
- unnecessary microservices
- unnecessary frontend frameworks
- features without tests
- features without a measurable workflow benefit

Every feature must answer:

1. What recruitment problem does this solve?
2. Who uses it?
3. What manual work does it remove?
4. What happens when it fails?
5. How is it tested?
6. How is it audited?
7. How does it fit the candidate lifecycle?

---

# 5. Non-Negotiable Engineering Principles

## 5.1 Human-in-the-loop

AI must assist hiring decisions, not silently make consequential employment decisions.

Bad:

```text
LLM → reject candidate
```

Good:

```text
CV
 ↓
Extraction
 ↓
Evidence
 ↓
Deterministic / configurable scoring
 ↓
Recruiter review
 ↓
Human decision
```

---

## 5.2 Fail closed for safety-critical operations

If an important dependency is unavailable:

```text
Enrollment source unavailable
Google API unavailable
Database inconsistent
Candidate identity ambiguous
Offer data incomplete
```

the system must **stop the affected operation** rather than assume a safe value.

Never do:

```python
except Exception:
    pass
```

Never turn:

```text
unknown
```

into:

```text
false
```

when false could cause an incorrect automated action.

---

## 5.3 No silent failures

Every failure must produce at least one of:

- structured log
- database event
- user-visible error
- retry
- alert
- failed job state

A failure that disappears into a log nobody checks is still a product failure.

---

## 5.4 Idempotency

Repeated execution must not create duplicate business actions.

Examples:

```text
same candidate + same email event
same candidate + same offer
same campaign + same recipient
same webhook + same event ID
```

must be safely deduplicated.

---

## 5.5 Auditability

Important actions must answer:

```text
WHO
WHAT
WHEN
WHY
BEFORE
AFTER
```

Example:

```text
Candidate: APP-18293
Action: Rejected
Actor: user_183
Reason: Interview score below threshold
Timestamp: 2026-09-14T15:42:11+05:30
```

---

## 5.6 Security by default

Never commit:

- passwords
- API keys
- service-account JSON
- SMTP credentials
- Flask secrets
- OAuth tokens
- production databases
- candidate PII dumps

Use environment variables / secret management.

---

# 6. Target Architecture

The existing system is a modular monolith. Keep it modular first.

Do **not** prematurely split everything into microservices.

Target architecture:

```text
                         ┌───────────────────────┐
                         │   Candidate Portal    │
                         └───────────┬───────────┘
                                     │
                         ┌───────────▼───────────┐
                         │   Recruiter Console   │
                         └───────────┬───────────┘
                                     │
                         ┌───────────▼───────────┐
                         │     API / Backend     │
                         └───────────┬───────────┘
                                     │
              ┌──────────────────────┼──────────────────────┐
              │                      │                      │
       ┌──────▼──────┐       ┌───────▼──────┐       ┌──────▼──────┐
       │ Candidate   │       │ Workflow     │       │ AI / CV     │
       │ Engine      │       │ Engine       │       │ Engine       │
       └──────┬──────┘       └───────┬──────┘       └──────┬──────┘
              │                      │                      │
              └──────────────────────┼──────────────────────┘
                                     │
                              ┌──────▼──────┐
                              │ PostgreSQL  │
                              └──────┬──────┘
                                     │
                           ┌─────────▼─────────┐
                           │ Job / Task Worker │
                           └─────────┬─────────┘
                                     │
             ┌───────────────────────┼────────────────────────┐
             │                       │                        │
        ┌────▼─────┐           ┌─────▼────┐             ┌─────▼─────┐
        │ Email    │           │ WhatsApp │             │ Calendar  │
        │ Provider │           │ Provider │             │ Provider  │
        └──────────┘           └──────────┘             └───────────┘
```

---

# 7. Target Technology Direction

Technology can evolve, but contributions should move toward the following baseline.

## Backend

Preferred:

- Python 3.11+
- FastAPI or a well-structured Flask API during migration
- Pydantic for validation
- SQLAlchemy
- Alembic migrations

## Database

Development:

- SQLite is acceptable temporarily

Production target:

- PostgreSQL

Required PostgreSQL features:

- transactions
- foreign keys
- indexes
- constraints
- JSONB where justified
- row-level tenant isolation strategy

## Background Jobs

Choose one and document the decision:

- Celery + Redis
- RQ + Redis
- Dramatiq
- a robust database-backed worker for early MVP

Do not run important long-lived business jobs inside Flask daemon threads.

## Frontend

A modern frontend may be introduced after backend contracts stabilise.

Possible direction:

- React
- Next.js
- TypeScript

Do not introduce a frontend rewrite before API/state contracts are stable.

## AI

AI providers must be abstracted behind interfaces.

Example:

```python
class ResumeAnalyzer:
    def analyze(self, document) -> ResumeAnalysis:
        ...
```

Possible implementations:

```text
OllamaResumeAnalyzer
OpenAIResumeAnalyzer
MockResumeAnalyzer
```

The core product must not depend directly on one model provider.

---

# 8. Development Phases

## PHASE 0 — Repository Reproducibility

### Objective

Make the current repository cloneable, installable and runnable by a new contributor.

### Tasks

- [ ] Verify every imported module exists.
- [ ] Remove stale documentation.
- [ ] Create `.env.example`.
- [ ] Move secrets/configuration to environment variables.
- [ ] Add deterministic dependency versions.
- [ ] Add startup validation.
- [ ] Add a development setup script.
- [ ] Add sample/test configuration.
- [ ] Add test database configuration.
- [ ] Add clear local run instructions.
- [ ] Fix README commands that do not work.
- [ ] Add CI workflow.

### Acceptance criteria

A contributor should be able to:

```bash
git clone <repo>
cd gayatribot-Hr
python -m venv .venv
pip install -r requirements.txt
cp .env.example .env
pytest
```

without needing production secrets.

---

# 9. PHASE 1 — Security Hardening

This phase blocks public production deployment until completed.

## Required

- [ ] Authentication
- [ ] Role-based authorization
- [ ] CSRF protection
- [ ] Secure session cookies
- [ ] Secret management
- [ ] Input validation
- [ ] Output escaping
- [ ] Rate limiting
- [ ] Security headers
- [ ] Audit logging
- [ ] Secure file access
- [ ] PII-safe logging

## Roles

Initial roles:

```text
ADMIN
HR_MANAGER
RECRUITER
HIRING_MANAGER
VIEWER
```

Permissions must be explicit.

Example:

```text
VIEWER
  view candidates

RECRUITER
  view + edit candidates
  manage screening
  communicate

HIRING_MANAGER
  review assigned candidates
  submit scorecards

HR_MANAGER
  approve offers
  manage workflows

ADMIN
  system configuration
  users
  integrations
```

## Critical security rules

No unauthenticated route may:

- send email
- send an offer
- approve/reject a candidate
- access applicant PII
- download candidate documents
- start/stop campaigns
- change global sending controls

---

# 10. PHASE 2 — Reliability & State Management

Replace timestamp-driven implicit state with explicit state machines.

## Candidate lifecycle

```text
APPLIED
  ↓
VALIDATING
  ↓
VALIDATED
  ↓
SCREENING
  ↓
SHORTLISTED
  ↓
INTERVIEW
  ↓
SELECTED
  ↓
OFFER_PENDING
  ↓
OFFER_SENT
  ↓
OFFER_ACCEPTED
  ↓
JOINING
  ↓
ONBOARDED
```

Alternative terminal states:

```text
REJECTED
WITHDRAWN
DUPLICATE
INELIGIBLE
```

Every transition must be validated.

Example:

```text
APPLIED → OFFER_SENT
```

must not be allowed.

---

# 11. PHASE 3 — Database Migration

Current SQLite architecture can remain for development, but production should migrate to PostgreSQL.

## Core entities

Recommended initial schema:

```text
tenants
users
roles
permissions

jobs
job_requirements
applications

candidates
candidate_documents
candidate_skills

screening_runs
screening_results
assessments

interviews
interview_scorecards

offers
offer_versions

email_messages
email_events
email_templates

campaigns
campaign_recipients

workflow_runs
workflow_events

audit_logs
integrations
webhook_events

system_settings
```

Every tenant-owned record must have:

```text
tenant_id
```

---

# 12. PHASE 4 — Multi-Tenancy

This is mandatory before commercial SaaS deployment.

Architecture:

```text
Tenant A
 ├── Users
 ├── Jobs
 ├── Candidates
 ├── Applications
 └── Campaigns

Tenant B
 ├── Users
 ├── Jobs
 ├── Candidates
 ├── Applications
 └── Campaigns
```

Never trust a `tenant_id` supplied by the browser.

Derive tenant context from authenticated identity/session/token.

Every query must enforce tenant scope.

Add automated tests attempting cross-tenant access.

Acceptance test:

```text
User from Tenant A
    ↓
attempts to access Candidate belonging to Tenant B
    ↓
403 or 404
    ↓
no data leakage
```

---

# 13. PHASE 5 — Candidate Engine

Build the central candidate system.

## Candidate profile

```text
Identity
Contact
Education
Experience
Skills
Documents
Applications
Communication
Interview history
Offers
Audit history
```

A candidate may have multiple applications.

Therefore:

```text
Candidate
   ├── Application A
   ├── Application B
   └── Application C
```

Do not duplicate the entire candidate record for every job.

---

# 14. PHASE 6 — Job & Eligibility Engine

Create configurable job requirements.

Example:

```yaml
job:
  title: Python Automation Intern

requirements:
  education:
    - BTech
    - BCA
    - MCA

  skills:
    required:
      - Python
      - Git

    preferred:
      - FastAPI
      - SQL

  availability:
    minimum_hours_per_week: 20
```

Eligibility should produce structured evidence.

Example:

```json
{
  "eligible": true,
  "score": 87,
  "checks": [
    {
      "requirement": "Python",
      "result": "pass",
      "evidence": "Python project listed in CV"
    }
  ]
}
```

---

# 15. PHASE 7 — CV / AI Engine

Build AI as an evidence-extraction system.

Pipeline:

```text
Upload / Drive
      ↓
File validation
      ↓
Virus / type check
      ↓
Text extraction
      ↓
OCR if required
      ↓
Document normalization
      ↓
LLM extraction
      ↓
Schema validation
      ↓
Skill matching
      ↓
Evidence generation
      ↓
Human review
```

Never store raw model output as the authoritative candidate record.

Store:

```text
model
model_version
prompt_version
timestamp
structured_result
confidence
source_document_hash
```

This makes AI results reproducible and auditable.

---

# 16. PHASE 8 — Communication Engine

Replace direct SMTP calls throughout the codebase with a single communication abstraction.

```python
class MessageService:
    def send(self, message: Message) -> SendResult:
        ...
```

Providers:

```text
SMTP
SendGrid
Amazon SES
Mailgun
etc.
```

Later:

```text
Email
WhatsApp
SMS
Push
```

## Message states

```text
QUEUED
PROCESSING
ACCEPTED
DELIVERED
BOUNCED
FAILED
CANCELLED
```

Do not call SMTP acceptance "delivered".

---

# 17. PHASE 9 — Workflow Engine

Recruitment should become configurable workflows.

Example:

```text
APPLICATION_RECEIVED
       ↓
VALIDATE
       ↓
ELIGIBILITY_CHECK
       ↓
SCREEN
       ↓
SHORTLIST
       ↓
ASSESSMENT
       ↓
INTERVIEW
       ↓
DECISION
       ↓
OFFER
       ↓
ACCEPTANCE
       ↓
ONBOARDING
```

A workflow action might be:

```json
{
  "type": "send_email",
  "template": "interview_invitation"
}
```

or:

```json
{
  "type": "create_task",
  "assignee": "hiring_manager"
}
```

or:

```json
{
  "type": "schedule_interview"
}
```

---

# 18. PHASE 10 — Candidate Portal

Candidate-facing experience:

```text
Apply
 ↓
Application ID
 ↓
Status
 ↓
Documents
 ↓
Assessments
 ↓
Interview scheduling
 ↓
Offer
 ↓
Acceptance
 ↓
Onboarding
```

Candidates should not need to contact HR for basic status information.

---

# 19. PHASE 11 — Interview System

Features:

- interview scheduling
- Google Calendar integration
- meeting link generation
- reminders
- interviewer assignment
- scorecards
- structured feedback
- decision history

Example scorecard:

```text
Technical ability       1–5
Problem solving         1–5
Communication           1–5
Role-specific skills    1–5
Overall recommendation  1–5

Comments:
...
```

---

# 20. PHASE 12 — Offer Management

Offer lifecycle:

```text
DRAFT
 ↓
PENDING_APPROVAL
 ↓
APPROVED
 ↓
GENERATED
 ↓
SENT
 ↓
VIEWED
 ↓
ACCEPTED / DECLINED / EXPIRED
```

Offer generation must never silently invent critical fields.

If required information is missing:

```text
BLOCK
```

not:

```text
GUESS
```

---

# 21. PHASE 13 — Analytics

Build analytics around business outcomes.

Examples:

```text
Applications
Qualified applications
Screening rate
Interview rate
Offer rate
Acceptance rate
Joining rate

Time-to-screen
Time-to-interview
Time-to-offer
Time-to-hire

Source effectiveness
Recruiter workload
Hiring funnel conversion
Candidate drop-off
Communication failures
```

A future executive dashboard should answer:

> "Where are candidates getting stuck?"

---

# 22. PHASE 14 — Integrations

Prioritise integrations by customer value.

### Tier 1

- Google Workspace
- Google Sheets
- Google Drive
- Google Calendar
- Gmail / SMTP

### Tier 2

- WhatsApp Business API
- Microsoft 365
- Zoom / Meet
- job boards

### Tier 3

- HRMS
- payroll
- attendance
- background verification
- assessment platforms

Build integration interfaces instead of hard-coding vendors into business logic.

---

# 23. PHASE 15 — Observability

Every production deployment needs:

```text
logs
metrics
traces
health checks
alerts
```

Required health endpoints:

```text
/health
/ready
/health/database
/health/queue
/health/email
/health/google
```

Example:

```json
{
  "status": "degraded",
  "database": "healthy",
  "queue": "healthy",
  "email": "failed",
  "google": "healthy"
}
```

---

# 24. PHASE 16 — Testing

Minimum test pyramid:

```text
             E2E
            /   \
         API     UI
        /          \
 Integration     Integration
       \            /
        Unit Tests
```

Required tests:

### Unit

- eligibility
- state transitions
- email templates
- offer generation
- rate limiting
- parsing
- permission checks

### Integration

- database
- Google APIs
- email provider
- file storage
- AI provider

### Security

- authentication
- authorization
- CSRF
- tenant isolation
- IDOR
- file access
- injection
- rate limiting

### Failure tests

Simulate:

```text
Google unavailable
SMTP unavailable
database locked
queue worker dies
duplicate webhook
duplicate email request
missing enrollment source
malformed CV
missing candidate email
invalid offer data
```

---

# 25. CI/CD

Every pull request should run:

```text
lint
format check
type check
unit tests
integration tests
security scan
dependency scan
```

Recommended tooling:

```text
ruff
black
mypy
pytest
bandit
pip-audit
```

For frontend:

```text
eslint
prettier
TypeScript
vitest
playwright
```

A PR should not merge if critical checks fail.

---

# 26. Contribution Workflow

## Branch naming

Use:

```text
feature/<short-name>
fix/<short-name>
security/<short-name>
refactor/<short-name>
docs/<short-name>
test/<short-name>
```

Examples:

```text
feature/candidate-search
fix/email-idempotency
security/dashboard-auth
refactor/email-service
test/tenant-isolation
```

## Commit messages

Use:

```text
feat:
fix:
security:
refactor:
test:
docs:
chore:
```

Examples:

```text
feat: add candidate eligibility engine
fix: prevent duplicate offer dispatch
security: require authorization for bulk actions
test: add cross tenant access tests
```

---

# 27. Pull Request Requirements

Every PR must contain:

```text
Problem
Solution
Files changed
Database changes
API changes
Security impact
Tests added
Manual test performed
Known limitations
```

PR checklist:

- [ ] No secrets committed
- [ ] Tests pass
- [ ] Error paths handled
- [ ] No silent exception swallowing
- [ ] Logs do not expose PII unnecessarily
- [ ] Authorization considered
- [ ] Tenant isolation considered
- [ ] Idempotency considered
- [ ] Database migrations included if required
- [ ] Documentation updated
- [ ] Backward compatibility considered

---

# 28. Definition of Done

A feature is **not complete** because the code runs.

It is complete only when:

```text
Code
 +
Tests
 +
Error handling
 +
Security
 +
Logging
 +
Documentation
 +
Migration
 +
Observability
```

are complete.

---

# 29. Issue Labels

Recommended GitHub labels:

```text
phase-0
phase-1
phase-2
phase-3
phase-4
phase-5
phase-6
phase-7
phase-8

bug
security
performance
backend
frontend
database
ai
email
integration
documentation
testing

good-first-issue
help-wanted
blocked
needs-design
breaking-change
```

---

# 30. Contribution Priorities

## P0 — Blockers

These must be fixed before production use:

- authentication
- authorization
- hardcoded secrets
- CSRF
- tenant isolation
- fail-open safety conditions
- duplicate sending
- state consistency
- secure file access
- audit logging

## P1 — Core Product

- candidate engine
- jobs
- eligibility
- workflow engine
- communication engine
- interview management
- offer management

## P2 — Growth

- candidate portal
- WhatsApp
- analytics
- integrations
- AI improvements
- assessment platform

## P3 — Advanced

- marketplace integrations
- advanced automation builder
- predictive analytics
- enterprise SSO
- advanced compliance
- large-scale optimisation

---

# 31. Recommended First 20 GitHub Issues

Create these issues first.

### Issue 01
**Create reproducible development environment**

### Issue 02
**Introduce `.env.example` and secure configuration**

### Issue 03
**Remove hardcoded Flask secret**

### Issue 04
**Implement admin authentication**

### Issue 05
**Implement RBAC**

### Issue 06
**Implement CSRF protection**

### Issue 07
**Implement audit logging**

### Issue 08
**Make enrollment safety checks fail closed**

### Issue 09
**Implement email idempotency**

### Issue 10
**Implement atomic candidate processing lock**

### Issue 11
**Add database transactions and SQLite WAL**

### Issue 12
**Create candidate state machine**

### Issue 13
**Create communication service abstraction**

### Issue 14
**Create background job worker**

### Issue 15
**Create candidate/application data model**

### Issue 16
**Create configurable eligibility engine**

### Issue 17
**Create job/requisition model**

### Issue 18
**Create interview scheduling model**

### Issue 19
**Create offer lifecycle**

### Issue 20
**Create tenant model and tenant isolation tests**

---

# 32. AI Coding Agent Instructions

AI coding agents working on this repository must follow these rules.

## Before changing code

1. Read `DEVELOPMENT.md`.
2. Inspect the relevant existing code.
3. Identify dependencies.
4. Identify database impact.
5. Identify security impact.
6. Identify failure modes.
7. Search for existing implementations before creating duplicates.

## Before implementing

Produce a short plan:

```text
Files to modify
Files to create
Database changes
API changes
Tests
Risks
```

## After implementing

Run:

```text
tests
lint
type checks
security checks
```

Then report:

```text
Implemented
Tests passed
Tests failed
Known limitations
Next recommended task
```

Never claim a feature works without testing it.

---

# 33. Do Not Rewrite Working Systems Without Evidence

Contributors should avoid:

```text
"Let's rewrite everything in FastAPI."
"Let's replace SQLite immediately."
"Let's rebuild the frontend."
"Let's convert to microservices."
```

unless an issue specifically requires it.

Migration should be incremental:

```text
Existing system
      ↓
Introduce abstraction
      ↓
Test abstraction
      ↓
Migrate consumers
      ↓
Remove old implementation
```

This reduces regression risk.

---

# 34. Commercial Product Milestones

## Milestone A — Reliable Internal Platform

```text
✓ Secure
✓ Testable
✓ Reproducible
✓ Reliable email
✓ Candidate lifecycle
✓ Offer lifecycle
✓ Admin dashboard
```

## Milestone B — MVP SaaS

```text
✓ Multi-tenancy
✓ User management
✓ Jobs
✓ Candidates
✓ Screening
✓ Workflow
✓ Candidate portal
✓ Billing-ready architecture
```

## Milestone C — Commercial Beta

```text
✓ Google integrations
✓ WhatsApp
✓ Calendar
✓ Analytics
✓ AI screening
✓ Audit
✓ Monitoring
✓ Backup / recovery
```

## Milestone D — Production SaaS

```text
✓ Security review
✓ Privacy/compliance review
✓ Load testing
✓ Disaster recovery
✓ Monitoring
✓ Customer support
✓ Documentation
✓ Billing
✓ SLA process
```

---

# 35. Product Success Metrics

Do not measure only GitHub activity.

Measure:

```text
Applications processed
Applications automatically validated
HR hours saved
Time-to-screen
Time-to-interview
Time-to-offer
Offer acceptance rate
Candidate completion rate
Communication failure rate
Automation success rate
Human intervention rate
Cost per processed candidate
```

The north-star operational metric should eventually be something similar to:

> **Candidates successfully moved through the hiring funnel per HR operator per day.**

---

# 36. Final Engineering Direction

The evolution should look like this:

```text
CURRENT
GayatriBot-HR
Python + SQLite + Google Sheets + SMTP
             │
             ▼
PHASE 1
Secure + Reliable
             │
             ▼
PHASE 2
Candidate + Job + Application Engine
             │
             ▼
PHASE 3
Workflow + Communication Engine
             │
             ▼
PHASE 4
AI-assisted Screening
             │
             ▼
PHASE 5
Candidate Portal + Interview System
             │
             ▼
PHASE 6
Multi-Tenant SaaS
             │
             ▼
PHASE 7
Integrations + Analytics
             │
             ▼
TARGET
GayatriHire
High-Volume Hiring Automation Platform
```

---

# 37. The Most Important Rule

**Do not build features faster than the system can become reliable.**

The goal is not:

> "More code."

The goal is:

> **"A trustworthy hiring platform that can process real candidates for real organisations without losing data, sending duplicate or incorrect communications, exposing applicant information, or making unexplained hiring decisions."**

Every contributor is responsible for moving the repository toward that standard.

---

## Initial Development Order

Start here:

```text
01. Reproducible environment
02. Configuration / secrets
03. Tests + CI
04. Authentication
05. RBAC
06. CSRF + security hardening
07. Audit logging
08. Fail-closed safety controls
09. Database/state consistency
10. Email idempotency
11. Background worker
12. Candidate state machine
13. Candidate/application schema
14. Job + eligibility engine
15. Communication abstraction
16. Workflow engine
17. Interview system
18. Offer lifecycle
19. Multi-tenancy
20. Candidate portal
21. AI/CV engine
22. Integrations
23. Analytics
24. Commercial SaaS infrastructure
```

**No contributor should skip directly to AI features, WhatsApp, analytics, or cosmetic dashboard work while P0 security and reliability issues remain unresolved.**

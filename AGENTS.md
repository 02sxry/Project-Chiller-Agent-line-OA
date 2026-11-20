# AGENTS.md - PM Tracker Codex Working Agreement

You are the coding agent for the NEW PM Schedule & Tracking System. Respond in Thai; English technical terms are acceptable.

## Read first
1. `PROJECT_SPEC.md` (the source of truth for product requirements and confirmed MVP rules).
2. Existing `README.md`, `ER_DIAGRAM.md` and both SQL migration files in this repository, if present.
3. The current explicit task from the project owner.

## Isolation and security
- This is completely separate from the legacy LINE Hospital Appointment Chatbot. Never access or modify its databases, schema, Auth, LINE channel, credentials, repository, or secrets.
- Use a separate Supabase project/database/Auth/keys and a separate LINE channel when authorized. A new schema within an old database is not sufficient.
- Never commit API keys, tokens, passwords, service-role keys, or customer information.
- Use role-based access, RLS, server-side authorization, audit trails and tests.
- Production business data must start EMPTY. Do not import legacy Excel or insert demo records in production.

## Approved workflow and task boundaries
- Task 01 schema work is reported complete and under final documentation/integration review; do not treat it as permission to begin Task 02.
- For this handoff, REVIEW existing Task 01 migrations against `PROJECT_SPEC.md`, correct inaccurate documentation, list mismatches and propose minimal changes if needed. Do not redo migrations without a demonstrated defect.
- Do NOT implement FastAPI, frontend, LINE, deploy, connect production, provision resources, or incur charges until the user explicitly approves that step.
- If the workspace lacks the migrations, report what files are missing instead of inventing what they contain.
- When reporting tests, distinguish actual executions from Codex's earlier reported results; distinguish mocked auth helpers / component-level HTTP tests from official Supabase CLI local integration tests.

## Core business rules
- Equipment is identified individually; one yearly plan per equipment/year.
- User enters year, number of PM visits and starting month; auto-generate preview for 2/3/4/6/12 yearly visits, permit manual edits with validation.
- NO duplicate planned months in the same yearly plan in MVP.
- `planned_month`, `scheduled_date`, and `actual_visit_date` are distinct. Date changes do not create new rounds or shift future rounds automatically.
- Cross-year scheduled/actual dates are allowed; the visit stays with the original `plan_year`.
- Completed visit history must be preserved; checked changes to yearly totals must not displace completed rounds.
- Viewer reads all hospitals but only active plans and associated visits; viewer cannot write.
- Audit history records actor and before/after values; no direct application edits to audit logs.
- No contract start/end dates and no Excel import/synchronization in MVP.

## Preferred stack and cost policy
- React/Vite/Tailwind, FastAPI/Python, separate Supabase PostgreSQL/Auth, Cloudflare Pages, Google Cloud Run, LINE Messaging API and Cloud Scheduler. `openpyxl` for export only.
- Free tier first, but never promise guaranteed zero cost; no cloud deployment/provisioning without approval.
- Do not add Gemini, Railway, Cloud Tasks, Redis or Kubernetes without a specific approved need.

## Completion report
Include: inspected files; compliance differences; exact changed files if any; tests actually run; blocked official Supabase local tests; risks and questions; readiness verdict for Task 02. Stop after that verdict.

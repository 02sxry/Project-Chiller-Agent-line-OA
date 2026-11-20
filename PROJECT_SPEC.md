# PM Schedule & Tracking System - Project Specification

## 1. Background and goal
Create a small Thai-language PM (Preventive Maintenance) tracking web application for equipment installed at hospitals. The owner previously checked three separate Excel files for monthly work. This NEW application replaces that habit with one dashboard and an internal LINE OA notification bot. It is completely separate from the owner's existing hospital-appointment chatbot.

Expected volume: initially three user-defined equipment types, up to 15 PM jobs per month, typical maintenance cadence every 3-4 months depending on each device's yearly plan. Prefer an affordable, simple, maintainable system, not a large CMMS or ERP.

## 2. Confirmed product decisions
- Start from ZERO business data in a new Supabase database: no seeded manufacturers, hospitals, equipment, visits or plans.
- Owner manually adds all records from the web app. Do NOT import or synchronize the old Excel files in MVP.
- Database becomes sole source of truth. Excel export may be added, but Excel import is out of scope.
- NO contract start/end date fields and NO contract-expiration module.
- Store yearly PM plans per actual piece of equipment: plan year, visits per year, preferred starting month, auto/manual schedule mode and notes.
- Generate planned PM months automatically, allow owner to review and adjust before saving.
- Track planned month, scheduled date and actual visit date separately.
- Store each year's visit status/history; owner can inspect past years and copy a plan into the next year without copying completed status.
- LINE OA is for the internal team, initially one internal LINE group, NOT the 19 hospital groups in the legacy chatbot.
- Thai UI, usable on both desktop and mobile. Easy manual data entry, with spreadsheet-like tables where beneficial.

## 3. Main features
### Dashboard
- Monthly jobs by hospital/equipment, unassigned dates, scheduled, postponed, overdue and completed.
- Next month's jobs and yearly planned/completed/remaining totals.
- Filters: year, month, hospital, manufacturer, equipment type, model, status.
- No fabricated statistics or demo records when database is empty.

### Equipment master data
- Add/edit/archive manufacturers, equipment types, hospitals and individual devices.
- Device fields include: manufacturer, equipment type, hospital, model, optional serial number, optional equipment code, optional room, notes, active flag.
- One hospital may have many devices. Several devices can share model names. Identifiable device ID is primary; serial number may be absent.
- Prevent accidental duplicates without relying solely on optional serial numbers.

### Yearly planner
- Enter equipment, calendar year, visits/year, and starting PM month.
- Auto-generate for evenly divisible 12-month counts; examples: 4 visits/year starting January = January/April/July/October; 3 starting February = February/June/October; 2 starting March = March/September.
- Support 2/3/4/6/12 visits per year by default, plus manual mode when intervals are not evenly divisible or the contractual month pattern differs.
- Present a twelve-month preview and require confirmation before saving.
- Handle start-month rollover and numbering within the selected year explicitly: propose a business rule and seek owner approval rather than silently changing the year or losing visits.
- One yearly plan per actual equipment per year. Each visit should have a stable ID and a unique sequence number within that yearly plan.

### PM visits and history
- States: planned, scheduled, postponed, completed, cancelled.
- planned_month is a month (1..12) tied to the yearly plan; scheduled_date is the actual appointment; actual_visit_date is when work occurred.
- Rescheduling never creates a duplicate visit, automatically changes planned_month, or shifts later planned months.
- Completion requires the actual visit date; completed visits drive the 'done' count. Define cancelled-visit treatment separately and display it clearly.
- Preserve edit history including user/time/reason, especially for date or status changes and past-year visits.
- Protect historical completed work from inadvertent deletion or overwriting.

### LINE OA
- One new LINE Messaging API channel, initially one internal team group, independent of legacy project.
- Monthly PM digest with hospital/device/date or 'date not set', and status totals.
- Upcoming PM reminders (e.g. 7 days before scheduled visit) and alerts for work requiring a date.
- Simple rule-based text inquiries or menu: this month, next month, remaining this year. No AI model needed for MVP.
- Use Reply API for incoming queries when valid reply token is available and Push API for scheduled notifications.
- Deduplicate notifications, log results/errors and use quotas carefully, including recipient-count-based group Push accounting where applicable.
- LINE signatures, group authorization and permitted senders must be verified.

### Export and security
- Excel export with openpyxl for schedules/yearly reports. No Excel import in MVP.
- Supabase Auth roles: Admin (write) and Viewer (read).
- PostgreSQL RLS, FastAPI authorization, validation, audit logs, safe handling of keys and secure scheduler endpoints.
- No real production business sample data. Backups and restoration strategy are mandatory before ongoing use.

## 4. Proposed data entities (design, not yet migrations)
- manufacturers(id, name, description, created_at, updated_at)
- equipment_types(id, name, description, is_active)
- hospitals(id, name, province, contact_name?, contact_phone?, notes, created_at, updated_at)
- equipment(id, manufacturer_id, equipment_type_id, hospital_id, model_name, serial_number?, equipment_code?, room_name?, notes, is_active, created_at, updated_at)
- pm_yearly_plans(id, equipment_id, plan_year, pm_times_per_year, start_pm_month, schedule_mode, notes, created_at, updated_at)
- pm_visits(id, yearly_plan_id, visit_number, planned_month, scheduled_date?, actual_visit_date?, status, postponement_reason?, technician_name?, notes?, completed_at?, created_at, updated_at)
- notification_logs, audit_logs, app_settings, user_profiles (add only if necessary and justify schema)

Recommended constraints: UNIQUE(equipment_id, plan_year), UNIQUE(yearly_plan_id, visit_number), relevant FKs and indexes, status/date checks. Never store year 2569 directly as a Gregorian date; use Gregorian year in database and Thai Buddhist Era conversion for display.

## 5. Target architecture
Frontend: React + Vite + Tailwind CSS on Cloudflare Pages Free.
Backend: FastAPI on Google Cloud Run (scale to zero).
Storage/Auth: a new isolated Supabase PostgreSQL Free project with Supabase Auth.
Scheduled checks: Google Cloud Scheduler triggering protected FastAPI endpoints in Asia/Bangkok timezone.
Messages: LINE Messaging API to the internal team.
Optional export: openpyxl.
No Railway, Gemini, Cloud Tasks, Redis, Kubernetes, standing worker processes, legacy chatbot integration, or Excel import for MVP.

Cloud account costs and free-tier limitations must be verified as of implementation time. Do not claim guaranteed zero cost. Explain the risk of Supabase inactivity pauses, Google Cloud billing requirements, LINE messaging limits, logs and egress charges.

## 6. Five core UI areas
1. Dashboard: this month's PM, overdue, postponed, completed, next month; filters and editable drill-down.
2. Equipment Database: manually add manufacturers, equipment types, hospitals and devices, with quick-add, search and archive.
3. PM Yearly Planner: choose equipment/year/visits per year/start month; generated calendar with manual edits, preview and save; copy previous year.
4. PM Schedule & History: monthly/yearly/list views; reschedule, set actual visit date and status; audit history.
5. Settings & Notifications: internal LINE group, reminder timing, Excel export, notification logs, access/roles.

Always show useful Empty States, not sample data, before user has added business records.

## 7. Delivery phases
- Phase 1: System design only. Architecture, ER/schema proposal, five wireframes, yearly PM rule and edge cases, data-entry flow, LINE strategy, costs, risks, tests and phased roadmap. No code or deployment.
- Phase 2: After explicit approval, application code, empty DB migrations, auth, equipment management, yearly planner, visits, dashboard, tests and Excel export.
- Phase 3: After approval, LINE webhook/reply/push, notifications, scheduler, quotas and logs.
- Phase 4: After approval, deployment to approved cloud accounts, security/backup checks, tests and monitoring.

## 8. Acceptance criteria
- New DB starts empty, and users can manually enter master and yearly plan data without Excel.
- Auto scheduling generates the expected preview and never duplicates yearly visits.
- Owner can edit scheduled day, record actual completed date, view completed vs remaining annual visits and inspect history.
- Moving an actual visit across a month/year does not silently alter the contractual/planned round.
- Dashboard supports filters, clear no-data states, and concise monthly overviews.
- Internal LINE can deliver summaries/reminders and answer simple schedule questions without an LLM.
- Access control, audit logs, quota monitoring and backups are designed before go-live.

## 9. No action without approval
Do not write code, create a cloud resource, connect accounts, execute SQL, migrate data, or deploy until the owner reviews and explicitly approves Phase 1.

## 8. Confirmed Task 01 MVP decisions (2026-11-20)
- The PM Tracker MUST use an independent Supabase project, database, Auth configuration and keys, and a separate LINE channel from the legacy appointment chatbot. A separate PostgreSQL schema alone is NOT sufficient isolation.
- The production business tables start empty. No Excel import/sync or contract start/end dates.
- Viewer may read all hospitals and only ACTIVE yearly plans and visits belonging to active plans; viewer cannot write. Admin manages data. Unauthenticated and unauthorized users must not read application data.
- Planned months must be UNIQUE within a yearly plan in the MVP; NO duplicate-month exception or approval workflow in the MVP.
- Scheduled and actual visit dates may fall in a different calendar year from plan_year. The visit remains attributed to its original yearly plan.
- Admin may change pm_times_per_year only through impact-checked operations that preserve completed visit history. Reject reductions that would displace completed visit numbers or invalidate completed historical records. Never silently erase completed or cancelled historical records.
- Audit log records actor, prior and new values, timestamp, and the reason where relevant; application users cannot directly rewrite audit history.
- Validate RLS/JWT/Data API using an official Supabase local stack before production deployment. Testing against separate component services or simulated DB auth helpers is useful but is not a substitute for that final integration verification.
- The user has NOT authorized production connection, paid resource provisioning, or deployment.

## 9. Current implementation handoff (reported, not independently verified)
- Codex reports SQL migrations `supabase/migrations/20261120000000_create_pm_tracker.sql` and `supabase/migrations/20261120010000_finalize_pm_tracker.sql` in `/workspace/output`, plus `ER_DIAGRAM.md`, tests and README.
- Codex reports 71 PostgreSQL tests and 42 HTTP/Auth/PostgREST/Kong tests passing. These results need independent file review; official `supabase start` testing has not yet passed due to Docker disk-image limits.
- Before Task 02: review both migrations against this spec; update existing README to forbid shared Supabase usage and duplicate-month exceptions; verify actual project isolation when provisioning an independent environment; repeat official Supabase integration tests before deployment.
- Do NOT recreate or overwrite existing Task 01 migrations simply because this documentation was previously missing. Propose only necessary fixes with a diff and tests.

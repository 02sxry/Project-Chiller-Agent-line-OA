# Task 01 final spec alignment — handoff to Codex

1. Place `PROJECT_SPEC.md` and `AGENTS.md` at the ROOT of the PM Tracker Git repository (same folder as its README and `supabase/`).
2. Confirm the two migrations from previous Task 01 work are actually in this repository. If they exist only in `/workspace/output`, copy/commit them into the PM Tracker repository without rewriting their contents.
3. Compare migrations, RLS/RPC, ER diagram and README with `PROJECT_SPEC.md` section 8.
4. Correct README statements permitting shared Supabase projects or duplicate-month exceptions; the MVP explicitly forbids both.
5. Report minimal SQL changes needed, if any, with evidence. Do not modify a passing migration solely to match wording.
6. Keep 71 PostgreSQL + 42 HTTP test counts as historical reported results unless rerun. Official Supabase CLI local full-stack testing remains a release gate, not a prerequisite for reviewing docs.
7. Do not start Task 02 or provision/deploy cloud resources.

# plan.md

**Project:** Employee Task Monitoring System (`[PRODUCT NAME]` — not yet chosen)
**Companion documents:** `project-context.md` (problem, roles, monitoring formulas, data model, open questions) · `README.md` (stack, scripts, conventions) · `AGENTS.md` (binding engineering rules)
**Status:** Plan only. No application code exists. Phase 0 has not started.
**Last updated:** 2026-09-28

**Locked decisions** (from `project-context.md` §12): Vite + React 19 SPA · Tailwind CSS v4 · Supabase (Postgres/Auth/RLS/Storage) · fresh build, no port of another product's interface · Board + Table views · Conventional Commits v1.0.0 with branch prefixes matching the commit type.

---

## 1. Goal-to-feature matrix

### 1.1 The goal, decomposed

| ID | Goal element | Verbatim |
| --- | --- | --- |
| **G0** | One place to assign tasks to employees | "give a manager one place to assign tasks to employees" |
| **G1** | Who is working on what | "(1) who is working on what" |
| **G2** | Which tasks are finished | "(2) which tasks are finished" |
| **G3** | Which tasks are overdue or stalled | "(3) which tasks are overdue or stalled" |

Anything that maps to `—` is either cut or explicitly deferred to §7. There is no third option: a feature that serves no goal element does not ship.

### 1.2 Mapping

| # | Feature | G0 | G1 | G2 | G3 | Verdict | Phase |
| --- | --- | :-: | :-: | :-: | :-: | --- | --- |
| 1 | Authentication, roles, and org scoping | ● | ● | ● | ● | **Keep** — the "one place" and "shared" require real auth; every goal element depends on it | 1 |
| 2 | Row-level access rules (RLS + column grants) | ● | ● | ● | ● | **Keep** — a monitoring system that leaks employee data is unusable with staff | 1 |
| 3 | Data-access module (single boundary, mock + Supabase adapters) | ● | ● | ● | ● | **Keep** — enables; also the mechanism that makes the mock→real swap auditable | 0 |
| 4 | Task model: title, description, assignee, due date, priority, status | ● | ● | ● | ● | **Keep** — the subject of all four goal elements | 2 |
| 5 | Task creation and assignment (manager) | ● | ○ | ○ | ○ | **Keep** — G0 | 2 |
| 6 | Reassignment and unassignment, with history | ● | ● | | ● | **Keep** — reassignment churn is itself a monitoring signal | 2 |
| 7 | Configurable status model (Not started / In progress / Blocked / Done) | ○ | ● | ● | ● | **Keep** — "working on it" and "finished" are both status claims | 1–2 |
| 8 | Activity event log written by DB triggers | | ● | ● | ● | **Keep** — the raw signal behind "actively worked on" and "stalled" | 1–2 |
| 9 | `last_activity_at` maintained by trigger | | ● | ● | ● | **Keep** — required by "working on it" and "stalled" | 2 |
| 10 | Derived flags: `is_active`, `is_overdue`, `is_stalled` (single SQL view) | | ● | ● | ● | **Keep** — the product's core computation | 4 |
| 11 | Kanban board with drag between status columns | ○ | ● | ● | | **Keep** — status changes are how work is reported; it is the "at a glance" view of G1/G2 | 2 |
| 12 | Board filters (assignee, priority, due, stalled) | | ● | ● | ● | **Keep** — turns the board into a monitoring tool rather than a to-do list | 2 |
| 12b | Board **Table** view (sortable, same filters) | | ● | ● | ● | **Keep** — scanning and sorting across tasks (by last activity, due date, stalled) without leaving the board. **Timeline and Map are cut** — they answer "when"/"where", not who/what/done/overdue | 2 |
| 13 | Employee "My Tasks" view with status updates | | ● | ● | | **Keep** — the *only* way G1/G2 can be true; without employee input, the manager is guessing | 3 |
| 14 | Checklist with tick items | | ● | ● | | **Keep** — activity type and progress-within-task | 3 |
| 15 | Comments | | ● | ● | | **Keep** — activity type, and where "blocked" gets explained | 3 |
| 16 | File/image attachments as proof of work | | ● | | | **Keep** — activity type; explicitly client-requested | 3 |
| 17 | Reopen Done (manager) | ● | | ● | | **Keep** — keeps the completion number honest | 2 |
| 18 | Manager Dashboard: attention queues (overdue / stalled / blocked) | | ○ | ○ | ● | **Keep** — the single most important screen for G3 | 4 |
| 19 | Manager Dashboard: "who is working on what" strip | | ● | ○ | ○ | **Keep** — G1 on one screen | 4 |
| 20 | Manager Dashboard: completed-today feed | | | ● | | **Keep** — a monitoring tool that only shows bad news gets ignored | 4 |
| 21 | Team page: per-employee assigned / in-progress / done / overdue / stalled counts | | ● | ● | ● | **Keep** — the per-person view of G1–G3 | 4 |
| 22 | Stalled-threshold setting (organization) | | | | ● | **Keep** — makes G3 tunable, otherwise the number is arbitrary | 4 |
| 23 | Transparency: employee sees the same flags, activity, and their own numbers | | ● | ● | ● | **Keep** — a stated principle and a trust precondition | 3–4 |
| 24 | Inbox (unassigned quick capture + manager triage) | ● | | | | **Keep** — a task that is never assigned is invisible monitoring-wise; G0 hygiene | 5 |
| 25 | Schedule (calendar of due dates, read + filters) | | | | ● | **Keep, reduced** — due dates are overdue's input; calendar drag is deferred | 5 |
| 26 | Calendar drag-and-drop rescheduling | | | | ○ | **Defer** — real value, but loses to every monitoring feature on the list (§7) | — |
| 27 | Reports: per-employee, per-period completion + overdue + stalled | | ○ | ● | ● | **Keep** — G2/G3 over time; the "did the period go well" question | 6 |
| 28 | CSV export of a report | | | ● | ● | **Keep** — client-requested; how the manager uses the data outside the app | 6 |
| 29 | On-time completion rate | | | ● | ○ | **Keep** — one extra column, materially changes the reading of the headline rate | 6 |
| 30 | Multiple boards per organization | | | | | **Defer** — one default board satisfies the goal; schema supports more | — |
| 31 | Labels | | | | | **Defer** — useful for filtering, not needed to answer G1–G3. Schema present, minimal UI in Phase 2, full label management deferred | 2 (thin) |
| 32 | Team/sub-team structure | | | | | **Defer** — schema present, dormant (Q5) | — |
| 33 | Task dependencies / subtasks / epic grouping | | | | | **Cut** — not needed to monitor; adds noise to the board | — |
| 34 | Custom fields | | | | | **Cut** — same | — |
| 35 | Social media planning, compose-post, platform selectors, AI captions, media library, content-approval templates | | | | | **Cut** — explicitly removed by the revised brief; must not reappear | — |
| 36 | Automation rules | | | | | **Defer** — out of scope per brief | — |
| 37 | Slack / Teams / email / third-party integrations | | | | | **Defer** — out of scope per brief | — |
| 38 | Presence, login times, idle tracking, screen capture, location | | | | | **Cut, permanently** — violates the surveillance boundary (§6.1 of `project-context.md`) | — |
| 39 | Native mobile app | | | | | **Defer** — out of scope; responsive web only | — |
| 40 | In-app / email notifications | | | | | **Defer** (Q12) — not required by the goal; risks scope creep in both directions | — |
| 41 | WCAG contrast helper + focus/keyboard system | | | | | **Keep** — accessibility is part of "done", and the client may choose colours | 0, every phase |
| 42 | Light and dark themes | | | | | **Keep** — already decided | 0 |
| 43 | Global quick-capture shortcut | ● | | | | **Keep (thin)** — small, directly serves G0 | 5 |
| 44 | Global search (`/`) | | ● | ● | ● | **Keep** — "which tasks mention the supplier invoice" is a monitoring question. Own tasks for an employee, org-wide for a manager, enforced by RLS | 4 |

**Verdict tally:** 28 keep (2 of them thin), 8 defer, 7 cut, 1 permanent cut.

### 1.3 Features deliberately *not* in this plan, and why

A monitoring product is easy to grow into a productivity suite. These are the most likely accidental additions, recorded now so they are recognized as scope errors when they appear:

- Rich Gantt/timeline view — answers "when", which is not one of G1–G3.
- Time tracking / timesheets — answers "how long", not "done or not done"; starts to look like attendance measurement.
- Employee performance scoring, rankings, or leaderboards — pressures employees to close tasks dishonestly and is a trust problem under §6.2. Reports are factual per-employee summaries, not comparisons.
- Custom fields and subtasks — no monitoring value.
- A second "activity" concept (logins, sessions, device info) — §6.1.
- Notification preferences and digests — Q12; revisit only if the client says the manager will not open the app daily.

---

## 2. Proposed folder structure

**One npm package, one Vercel project.** Supabase is the backend, so there is no second application to maintain; server-side work that needs the service-role key becomes a Vercel function.

```
/
├── AGENTS.md                       # binding engineering rules
├── README.md
├── Design.md                       # SUPPLIED LATER. Not written in this pass.
├── project-context.md
├── plan.md
├── package.json
├── .env.example                    # VITE_DATA_SOURCE, VITE_SUPABASE_URL, VITE_SUPABASE_ANON_KEY
├── .commitlintrc.json               # Conventional Commits v1.0.0
├── .prettierrc
├── eslint.config.js                # includes the no-direct-backend-import rule
├── .github/workflows/ci.yml        # typecheck, lint, unit tests, commit-lint on the PR range
│
├── supabase/
│   ├── migrations/
│   │   ├── 0001_schema.sql         # tables, constraints, indexes
│   │   ├── 0002_functions.sql      # app_now, current_org_id, current_role, is_manager, can_view_task
│   │   ├── 0003_rls.sql            # policies + column-level grants
│   │   ├── 0004_triggers.sql       # activity engine, last_activity_at, done/reopen/blocked guards
│   │   ├── 0005_views.sql          # monitoring_task_state, employee_task_rollup
│   │   ├── 0006_storage.sql        # private bucket + storage policies
│   │   └── 0007_seed.sql
│   ├── seed.sql                    # demo org, 1 manager, 2 employees, statuses, board, tasks
│   └── tests/                      # RLS matrix: one assertion per permission row
│
├── src/
│   ├── main.tsx
│   ├── App.tsx
│   ├── routes/                     # route table + guards
│   │   ├── guards.tsx              # requireAuth, requireManager, redirect logic
│   │   └── paths.ts
│   ├── components/
│   │   ├── ui/                     # Button, Dialog, Menu, Select, Toast, Table, Tooltip…
│   │   ├── a11y/                   # FocusRing, LiveRegion, SkipLink, ContrastBadge
│   │   └── layout/                 # AppShell, Sidebar, Topbar, PageHeader
│   ├── features/
│   │   ├── auth/                   # SignIn, AcceptInvite, ResetPassword, PrivacyNotice
│   │   ├── dashboard/              # manager home, attention queues, who-is-working strip
│   │   ├── team/                   # employee cards, employee task list
│   │   ├── tasks/                  # task detail, editor, checklist, comments, attachments, activity
│   │   ├── boards/                 # board view, table view, column, card, filters
│   │   ├── my-tasks/               # employee home
│   │   ├── inbox/                  # quick capture + triage
│   │   ├── schedule/               # month grid, agenda list, due-date editing
│   │   ├── reports/                # period selector, rollup table, export button
│   │   ├── settings/               # people, statuses, boards, general (stalled threshold)
│   │   └── dev/                    # /dev/contrast — WCAG contrast test page
│   ├── hooks/                      # useTasks, useTask, useSession, useMediaQuery, useKeyboard
│   ├── lib/
│   │   ├── data/                   # ← THE ONLY PLACE THE BACKEND IS CALLED
│   │   │   ├── index.ts            # public API surface for the app
│   │   │   ├── types.ts            # domain types + zod schemas
│   │   │   ├── repositories/       # tasks.ts users.ts boards.ts statuses.ts comments.ts
│   │   │   │                      # attachments.ts activity.ts reports.ts settings.ts
│   │   │   └── adapters/
│   │   │       ├── supabase/       # real implementation (maps rows ⇄ domain types)
│   │   │       └── mock/           # in-memory, interface prototyping only
│   │   ├── monitoring/             # PURE functions mirroring the SQL view (for optimistic
│   │   │                           #   UI + unit tests); SQL remains the source of truth
│   │   ├── supabase/               # client singleton; server-only admin client
│   │   ├── dnd/                    # the ONLY @hello-pangea/dnd imports live here
│   │   ├── a11y/                   # contrast.ts (WCAG relative luminance, meetsContrast)
│   │   ├── i18n/                   # en-PH / en; date & timezone helpers
│   │   └── utils/                  # dates, csv, formatting, ids
│   └── styles/
│       ├── index.css               # Tailwind v4 @theme tokens, light/dark
│       └── tokens.ts               # shared color/space/radius tokens, incl. turquoise primary
│
├── e2e/                            # Playwright: acceptance scenarios, 3 accounts
└── tests/                          # unit/component: vitest + testing-library
```

### 2.1 The data-access contract

The surface components are allowed to use. Load-bearing signatures; the rest is ordinary CRUD.

```
DataSource { auth, users, settings, statuses, boards, tasks,
             comments, attachments, activity, reports }

tasks.list(filter)        assigneeIds, statusIds, due(overdue|today|week|unset),
                          labels, priority, stalledOnly, search, sort, page
tasks.get(id)             task + assignee + status + counts, or NotFound (never Forbidden)
tasks.create(input)       returns the new task
tasks.update(id, patch)
tasks.setStatus(id, statusId, { blockedReason? })
tasks.assign(id, userId | null)
tasks.reopen(id, reason)
tasks.archive(id) / tasks.restore(id)

activity.forTask(id)      typed, human-renderable events
reports.rollup({ period, userIds? })
reports.exportCsv({ period, userIds? })   → runs as the user's session, so RLS applies
```

**Rules encoded in this structure:**

1. `src/lib/data/` is the only place `@supabase/supabase-js` is imported from application code. Enforced by lint rule `no-restricted-imports` on `@supabase/supabase-js` outside `src/lib/`.
2. `src/lib/dnd/` is the only place `@hello-pangea/dnd` is imported from, so a library swap is one file.
3. `src/lib/monitoring/` mirrors the SQL formulas in TypeScript for optimistic rendering and unit tests. If the two ever disagree, **SQL wins** and the TS version is corrected — the TS copy is never an independent source of truth.
4. Service-role usage is confined to `src/lib/supabase/admin.ts`, which is server-only (Vercel function), never imported into `src/`.
5. Any mutation the caller is not permitted to make resolves to **NotFound**, not Forbidden, so a denied request never confirms that a record exists.

### 2.2 Route table

```
/sign-in   /invite/:token/accept   /reset-password   /privacy-notice
/                     → manager: /dashboard      employee: /my-tasks
/dashboard   /team   /team/:userId/tasks   /board   /inbox   /schedule
/reports    /reports/export   /settings/{people,statuses,boards,general}
/tasks/:taskId      ← ONE shared route for both roles; permission comes from RLS
/my-tasks           /my-tasks/:taskId        /403   /404
/dev/contrast       (dev/staging only)
```

There is deliberately no separate manager-only task route: a duplicated route is a duplicated thing to forget to guard. One route, and RLS decides what renders.

---

## 3. Phased build order

### How the phases work

- **Each phase is usable end to end — interface, backend, and permissions — before the next begins.** A phase is not "done" because a screen renders; it is done when a manager and two employees can use it in production-like conditions with real RLS.
- **Every monitoring checkpoint is run against the Supabase adapter.** The mock adapter can never satisfy a checkpoint.
- **Multi-user checkpoints** require **one manager account and two employee accounts on separate browsers or devices** (separate browser profiles on one machine are acceptable when two devices are unavailable; three separate browser profiles is the default). Session sharing in one browser does not count.
- **Tags:** `[DESIGN]` depends on `Design.md`; `[A11Y]` keyboard/focus/contrast is part of the definition of done, not extra; `[RLS]` touches a database access rule.
- **Phases 0–4 are the product.** Phases 5–7 extend it. If time is cut, ship through Phase 4 and cut cleanly at a point where the goal is already met.

### Phase dependency graph

```
Phase 0 ──► Phase 1 ──► Phase 2 ──► Phase 3 ──► Phase 4 ──► Phase 5 ──► Phase 6 ──► Phase 7
  setup      auth/RLS    tasks/      My Tasks    monitoring   Inbox/     reports/    harden/
                                   boards      dashboard    Schedule   export      deploy
```

Phase 2 and Phase 3 have a partial overlap: the manager can create and assign in Phase 2, but nothing is *demonstrable* as monitoring until Phase 3 gives employees a way to report status.

---

### Phase 0 — Project setup and foundations

**Serves:** G0–G3 (enabler). Nothing user-facing, but everything later depends on it.
**Depends on:** nothing.
**Blocked by:** `Design.md` for any visual work. Phase 0 proceeds on logic/tooling only; visual tokens are filled in when `Design.md` lands.

**Tasks**

1. VS Code project: Vite + React 19 + TypeScript strict, path aliases, Prettier, ESLint 9 flat config, and the `dev` / `build` / `preview` / `typecheck` / `lint` / `test` / `test:e2e` / `verify` scripts.
2. **Commit discipline from the first commit:** `@commitlint/cli` + `@commitlint/config-conventional`, husky `commit-msg` + `pre-commit`, lint-staged on staged files, `commit-and-tag-version` for changelog/versioning, and a CI job that re-lints the commit range of every pull request. Branch prefixes (`feat/`, `fix/`, …) match the commit types. A convention nobody can fail is not a convention, so the tooling goes in before the first feature commit, not at the end.
3. Tailwind v4 set up with `@tailwindcss/vite` and an `@theme` token layer containing only the decided values: turquoise primary, flat surfaces, light + dark themes. Placeholder tokens for the rest, marked as awaiting `Design.md`.
4. `[A11Y]` Baseline accessibility scaffolding: skip link, visible focus ring component, `LiveRegion` for announcements, semantic landmarks, `<html lang>`.
5. `[A11Y]` `contrast.ts` + `ContrastBadge` (§10 of `project-context.md`) with unit tests for known WCAG ratios, and the `/dev/contrast` page registered.
6. Routing skeleton with route guards stubbed (`requireAuth`, `requireManager`).
7. **Data-access module skeleton** per the §2.1 contract: `types.ts`, repository interfaces, adapter interface, and a `mock` adapter with in-memory fixtures for the demo org (1 manager, 2 employees, 1 board, ~15 tasks across statuses and dates). `supabase` adapter stubbed with `throw new Error('not implemented')` so the missing implementation is loud, not silent.
8. `VITE_DATA_SOURCE` switch + a visible dev-only "MOCK DATA" banner + a build-time guard that prevents monitoring routes running on mock.
9. Lint rule forbidding direct backend imports outside `src/lib/data/`, and a second rule confining `@hello-pangea/dnd` imports to `src/lib/dnd/`.
10. Supabase project created; `.env.example` written; dev/staging/prod project separation decided.
11. `AGENTS.md` written with the four binding rules (data boundary, `[DESIGN]` dependency, a11y-as-done, commit conventions).
12. Test harness: Vitest + Testing Library + Playwright, with the multi-account fixture (manager + 2 employees) and a seeding script that creates a deterministic demo org with known dates for overdue/stalled scenarios.
13. `README.md` written and kept current — it is the first thing a new contributor and the client will read.

**Testing checkpoint (P0)**

- `typecheck`, `lint`, and `test` all pass on a clean clone; CI runs them.
- A deliberately malformed commit message is **rejected** by the `commit-msg` hook, and CI rejects the same message in a test branch. A bad commit cannot land.
- Build succeeds; the app boots to a placeholder route in both themes.
- Changing `VITE_DATA_SOURCE=supabase` produces a visible, loud failure (not a blank screen).
- `contrastRatio('#000', '#fff')` returns 21; `contrastRatio('#777', '#fff')` returns < 4.5 and the badge reports fail.
- Keyboard-only traversal of the shell reaches every landmark and control with a visible focus ring.
- **No monitoring behaviour exists yet — this checkpoint proves foundations only.**

---

### Phase 1 — Authentication, roles, data model, and access rules

**Serves:** G0–G3. This is the phase that makes it a *shared* system instead of a personal to-do list. It is second, not last, for exactly that reason.
**Depends on:** Phase 0.
**Blocked by:** `Design.md` for the auth screens' visual design (work proceeds on logic and migrations).
**Must resolve before finishing:** Q1 (Done verification), Q3 (employee task creation), Q4 (manager as assignee), Q5 (teams in v1), Q6 (attachment limits), Q11 (multiple managers), Q14 (privacy notice).

**Tasks**

1. `[RLS]` Migrations: all tables in §8.1 of `project-context.md`, constraints, indexes (including the partial unique index on one current assignment per task, and indexes on `due_at`, `last_activity_at`, `status_id`, `assignee_id`).
2. `[RLS]` Supabase Auth: email + password, **invite-only** (no public sign-up). Manager invites a user; the invite creates the `profiles` row with a role. First-login password reset flow.
3. `[RLS]` Helper functions `current_org_id()`, `current_role()`, `is_manager()`, `can_view_task()`; RLS enabled on every table; policies exactly as the table in §8.4 of `project-context.md`.
4. `[RLS]` **Column-level grants on `tasks`** for employees: `title, description, status_id, column_id, position, blocked_reason` only. `assignee_id`, `due_at`, `priority`, `last_activity_at`, `done_at`, `started_at` are not granted. This is the boundary that makes least privilege real rather than cosmetic.
5. `[RLS]` Triggers: activity-event writer on tasks/checklist/comments/attachments/labels; `last_activity_at` stamp; `started_at` on first entry to a non-not-started status; `done_at` on entry to a done status and clearing on exit; "reopen guard" rejecting non-manager transitions out of a done status; "blocked guard" requiring `blocked_reason`; employee self-assign-only on insert.
6. `[RLS]` Seeded statuses per org from `statuses`; default settings row (`stalled_days=3`, `active_window_hours=24`, `require_done_review=false`, …).
7. `[RLS]` `monitoring_task_state` view with `security_invoker = true` — the §5 formulas implemented in SQL, and nothing else implementing them.
8. `[RLS]` Private Storage bucket `task-attachments`, path convention `{org_id}/{task_id}/{uuid}-{name}`, signed URLs, upload policy keyed on `can_view_task`, size/type limits from settings.
9. `[RLS]` `employee_task_rollup` view and `employee_period_rollup()` function (manager-readable only).
10. Data-access layer: `supabase` adapter implemented for `users`, `settings`, `statuses`, `boards`; row ⇄ domain mapping; typed error handling; a `SessionProvider` and session refresh.
11. `[DESIGN]` `[A11Y]` Sign in, accept-invite, reset-password, and first-login **privacy notice** screens (Q14), with full keyboard support and visible focus.
12. `[DESIGN]` Route guards live: unauthenticated → sign-in; employee hitting a manager route → 403 page (not a silent redirect with no explanation); authed user landing on `/dashboard` (manager) or `/my-tasks` (employee).
13. `[RLS]` Test suite for the permissions matrix — one automated test per row of the matrix in §3.2, run as SQL against Postgres, not as UI tests.
14. `[DESIGN]` People settings: list employees, invite, deactivate, change role. Deactivation prompts for reassignment of open tasks (Q9).

**Testing checkpoint (P1)** — *multi-user, 1 manager + 2 employees, three separate browsers*

- Employee A creates a task assigned to self (if Q3 = yes) → succeeds. The same insert with `assignee_id = Employee B` → **rejected by the database**, with the error surfaced in the UI, not a silent failure.
- Employee A attempts `UPDATE tasks SET due_at = ...` on their own task → **rejected by column grants**. Confirmed by direct SQL client attempt, not only by hiding the UI control.
- Employee A queries all tasks → receives only their own. Verified by running a `select * from tasks` in the browser console as Employee A and inspecting the result set.
- Employee A inserts a row into `activity_events` directly → rejected (no INSERT policy).
- Manager sees all tasks. Employee A cannot see Employee B's task by ID, including by direct URL to `/tasks/:id` → **not found**, not forbidden (§2.1 rule 5).
- Reassign a task; the old assignment row has `is_current=false` with `unassigned_at` set, and the activity log shows both events.
- **A manager action does not refresh an assignee's freshness.** Employee A has a task with an old `last_activity_at`. The manager extends its due date and renames it → both events appear in the log, and `last_activity_at` is **unchanged**. A pure board reorder writes **no** event at all. (§5.2.2)
- Boundary test: with `SET LOCAL app.now`, a task at `stalled_days − 1 minute` is not stalled and at `stalled_days + 1 minute` is, using the production view.
- Reset a password, sign in on all three browsers simultaneously; sessions are independent.
- `npm run typecheck && lint && test` green; full RLS matrix suite green.

**Exit criteria:** three people can hold concurrent sessions, the data is genuinely shared, and the access rules are proven by the database rather than by the interface.

---

### Phase 2 — Tasks and boards (manager side)

**Serves:** G0, G1, G2, G3.
**Depends on:** Phase 1.
**Blocked by:** `Design.md` for all interface work (logic may proceed).

**Tasks**

1. Task repository in the data layer: create, edit, soft-delete, restore, list with filters (assignee, status, priority, label, due window, stalled-only, unassigned), get-by-id with comments/checklist/attachments/activity in one round trip.
2. Manager task create/edit form: title, description, assignee, due date, priority, status, labels, board placement. Inline validation; a task cannot be saved without a title.
3. Assignment: assign, reassign, unassign, with assignment history written by trigger. Bulk assign from the Inbox later (Phase 5).
4. Reopen action on a done task, with the reason captured in the activity log.
5. `[DESIGN]` Board: default board, columns derived from `statuses`, cards showing title, assignee, due date with overdue styling, priority, label chips, checklist progress, comment/attachment counts.
6. `[DESIGN]` Drag-and-drop between columns via the `src/lib/dnd/` adapter and `@hello-pangea/dnd`: optimistic move, rollback on error, position written as fractional midpoint.
7. `[A11Y]` Keyboard drag path: lift, move, drop, cancel via keyboard with a live-region announcement of the result. Tested with the keyboard only.
8. `[DESIGN]` Board filters (assignee, priority, label, due window, "only stalled") and a filter state in the URL so a filtered board can be linked.
8b. `[DESIGN]` Board **Table view**: the same task set in a sortable table (last activity, due date, status, assignee, stalled), sharing the Board's filters and URL state. Column headers are real `<th scope>`, sorting is keyboard-operable, and the row count is asserted to match the Board view. **No Timeline and no Map view.**
9. `[DESIGN]` Task detail: all fields, checklist, comments, activity log rendering (human-readable sentences generated from `activity_events`).
10. `[A11Y]` Empty states, loading states, and error states for every list and form; focus moves sensibly on route change; dialogs trap focus and restore it on close.
11. Data-access adapter completion for `tasks`, `boards`, `activity`.
12. `AGENTS.md` check: no direct backend import anywhere in `src/` outside `src/lib/data/`.

**Testing checkpoint (P2)** — *multi-user, 1 manager + 2 employees, three separate browsers*

- Manager creates and assigns a task to Employee A; it appears in the manager's board in the correct column, and the assignment history records who assigned it and when.
- Manager drags the card to In progress: status, `last_activity_at`, `started_at` all update, and one `status_changed` activity event exists. No duplicate events on re-render or refresh.
- Manager reorders a card and refreshes in all three browsers: order persists and is identical everywhere — and **no activity event was written** for the reorder.
- Manager switches to the **Table view** and sorts by last activity, due date, and stalled; the sort and filters persist in the URL, and the row count matches the Board view.
- Manager attempts to remove a card: soft-deleted, recoverable, and it disappears from all boards immediately.
- Keyboard-only: a card can be moved between columns and the change is announced; the drag works with no mouse and no touch. The Table view is fully operable with arrow keys.
- Employee A reloads: sees their own task in My Tasks shell; Employee B does not see it. (Full My Tasks UI is Phase 3; here we confirm the read boundary.)
- Overdue styling: a task with a past `due_at` shows the overdue treatment on the card without a page refresh after the minute rolls over (derived at read time, not a stored flag).

**Exit criteria:** the manager can run the whole task lifecycle on the board, and the activity log is provably complete for every change made through the interface.

---

### Phase 3 — Employee "My Tasks" and work reporting

**Serves:** G1, G2 — and without this phase, **G1 and G2 are guesses.** The manager cannot monitor what employees never report.
**Depends on:** Phase 2. Can run in parallel with Phase 2's task-detail work.
**Blocked by:** `Design.md` for all interface work.

**Tasks**

1. My Tasks page grouped by urgency: Overdue / Due today / In progress / Not started / Blocked / Done this week. Groups derive from `monitoring_task_state`, not from separate queries that could disagree.
2. Inline status actions: Start, Mark blocked (reason required), Mark done. Each writes a status change, stamps `last_activity_at`, and adds an activity event.
3. Checklist: add, rename, reorder, tick, untick. A tick is an activity event — this is the mechanism that lets a long task show "being worked on" even while the status stays In progress.
4. Comments: post, edit own within a short window, soft-delete. Each is an activity event.
5. Attachments: upload to the private bucket, list with name/size/type, preview images inline, download via signed URL, remove own. Enforce the settings limits server-side (Q6).
6. `[DESIGN]` Transparency: the employee sees their own overdue, stalled, and active badges, and their own completion numbers for the current period, computed from the same view the manager uses.
7. `[A11Y]` Full keyboard operation of My Tasks: every inline action reachable and operable by keyboard; focus not lost after a status change; changes announced in the live region.
8. `[DESIGN]` Employee-visible empty states that are informative ("Nothing overdue. Next: report card layout by Friday") rather than decorative.
9. Guard: no route, link, or menu item on the employee side exposes another employee's data (verified by walking every employee route).

**Testing checkpoint (P3)** — *multi-user, 1 manager + 2 employees, three separate browsers*

- Employee A sees their tasks; Employee B's tasks are absent from the network responses, not merely hidden with CSS.
- Employee A ticks a checklist item; `last_activity_at` advances; the activity log shows "Checklist item ticked" with the item's text.
- Employee A marks a task Blocked with no reason → rejected with an inline message. With a reason → saved and visible in the activity log.
- Employee A marks a task Done → `done_at` set; the task moves to Done-this-week; the manager's board reflects it on refresh.
- Employee A cannot find a control to change the due date or the assignee; and a direct API attempt to do so is rejected by the database (re-run the P1 column-grant test against a *real* task id).
- Employee A opens their own task detail and sees the same activity history the manager sees.
- Every My Tasks action completes with the keyboard only, on all three browsers.

**Exit criteria:** an employee can do their entire job in the app, and every action they take produces exactly the activity signal the manager will rely on.

---

### Phase 4 — Manager Dashboard, Team page, and the monitoring rules

**Serves:** G1, G2, G3. **This is the phase where the goal is met.**
**Depends on:** Phase 3 (there must be real activity to monitor).
**Blocked by:** `Design.md` for all interface work. Must resolve Q1 and Q2.

**Tasks**

1. Implement/verify `monitoring_task_state` against the §5 formulas: `is_active` (24 h window), `is_overdue` (end-of-day in the org timezone), `is_stalled` (`stalled_days`, includes blocked), `is_blocked`, `is_done`, `days_since_activity`, `hours_overdue`. Each formula gets a unit test with a fixed clock.
2. `employee_task_rollup` view: open, active, overdue, stalled counts per employee. Manager-readable only.
3. Dashboard — **Needs attention now**: Overdue, Stalled, and Blocked queues, each with a count and a short list; stalled split into *stalled & blocked* vs *stalled & moving*; plus an "unassigned for N days" nudge from the Inbox.
4. Dashboard — **Who is working on what**: per-employee strip with active count, current task, time since last activity, due-today count. This is G1 on one screen.
5. Dashboard — team pulse counters and a completed-today feed.
6. Team page: one card per active employee with assigned / in progress / done (current period) / overdue / stalled counts, current task, last activity, and a link to that employee's filtered task list.
7. Organization settings: `stalled_days` (1–30), `active_window_hours`, `require_done_review` (if Q1 = yes), timezone. Changing a setting visibly re-derives the Dashboard with no deploy and no cache.
8. Timezone handling for due dates: end-of-day in the org timezone (Q10).
9. Manager actions surfaced from the attention queues: reassign, extend due date (with the change logged), reopen, comment, mark done.
10. `[A11Y]` Relative-time strings are never the only signal ("4d ago" is paired with the absolute timestamp in a `title`/tooltip and in the text for screen readers).
11. Performance: the Dashboard must be a bounded number of queries against indexed columns; add index coverage and verify with `EXPLAIN` on a seeded org of realistic size.
12. Global search (`/`): own tasks for an employee, org-wide for a manager, enforced by RLS rather than by filtering client-side. Searches title, description, and comments.
12b. Delete the mock adapter's route through this feature: `monitoring` code paths must not be reachable with `VITE_DATA_SOURCE=mock` (guarded by a build-time check).

**Testing checkpoint (P4)** — *multi-user, 1 manager + 2 employees, three separate browsers; a fixed clock is used to test thresholds*

- Set `stalled_days = 2`. Employee A has a task last touched 3 days ago → appears in the Stalled queue. Employee B's task touched 1 day ago → does not. Change the setting to 5 → neither appears. All without a deploy.
- A task with a past `due_at` and status In progress appears in Overdue with an accurate "3 days overdue" string; move it to Done → it leaves both queues immediately.
- An employee who has done nothing in 2 days appears in the "who is working on what" strip with "2 days ago", and the manager can jump straight to that task.
- A task with a future due date, last activity 5 days ago, assigned, not done → **stalled but not overdue**. A task with a past due date, activity yesterday → **overdue but not stalled**. Both combinations are reachable and correct.
- Blocked + stale appears in the blocked-stalled subgroup, distinct from stalled-moving.
- Employee A opens their own My Tasks in their own browser and sees the same stalled/overdue badge on the same task that the manager sees. The numbers match, character for character.
- `require_done_review` toggled on: a new status appears, employees can no longer move a task to Done, and the completion numbers do not count pending items.
- Global search: the manager finds a task by a word in its **comment**; Employee A searching for the same word does not see it. Verified in the network payload, not by whether it is hidden with CSS.
- Dashboard loads in under 1 second against a seeded org of 1 manager and 50 employees with 5,000 tasks.

**Exit criteria:** the manager answers G1, G2, and G3 from the Dashboard alone, and the employee cannot see a different truth.

---

### Phase 5 — Inbox and Schedule

**Serves:** G0 (capture before assignment) and G3 (due-date view).
**Depends on:** Phase 4.
**Blocked by:** `Design.md` for all interface work. Must resolve Q3 and Q6 if not already resolved.

**Tasks**

1. Inbox capture: a single input, global shortcut `n`, creates an unassigned task (`assignee_id = null`, status `not_started`).
2. Manager Inbox triage: assign, edit, discard; sort by age; flag unassigned tasks older than `stalled_days` as "never assigned".
3. Exclude unassigned tasks from every workload count and from the report denominators; prove it with a test (Q3).
4. Schedule: month grid + agenda list, manager per-employee filter, employee own-tasks-only.
5. Due-date editing from the calendar (manager only; employees cannot move their own deadline).
6. Pinned "overdue backlog" strip above the calendar, because a month grid hides past dates.
7. `[A11Y]` Calendar is a real `<table>`/grid with proper headers; each day cell is focusable; due-date editing is operable by keyboard; moving a date with the keyboard announces the new date.
8. **Calendar drag-and-drop is deferred (§7).** Ship read + keyboard editing first.

**Testing checkpoint (P5)** — *multi-user*

- `n` from anywhere opens capture; a saved task appears in the manager Inbox and in nobody's My Tasks and nobody's workload count.
- A capture from 5 days ago shows the "never assigned" flag; assigning it clears the flag and the flag disappears from the Dashboard nudge.
- A task moved in the calendar updates the board card's due label, and vice versa.
- An employee attempts to change a due date from the calendar → no control, and a direct API attempt is rejected.
- The month grid is navigable by keyboard only; screen-reader announces the correct date after an edit.
- Reports (not yet built) are unaffected: a test asserts unassigned tasks never appear in any denominator.

**Exit criteria:** nothing gets captured and lost, and due dates have one home that stays consistent with the board.

---

### Phase 6 — Reports and export

**Serves:** G2, G3 over time — the "how did this period go" question.
**Depends on:** Phase 4 (metrics), Phase 5 (exclusion rules). Must resolve Q7, Q8, Q9.
**Blocked by:** `Design.md` for the report screen.

**Tasks**

1. `employee_period_rollup(org, user, period_start, period_end)` implementing §5.5 exactly: `assigned_in_period`, `completed_in_period`, `due_in_period`, `completion_rate = completed/due` with "—" for an empty denominator, `on_time_rate`, `overdue_count`, `stalled_count`, `open_tasks`, `active_tasks`.
2. Period selector: week / month / quarter / custom, computed in the organization timezone, defaulting to the pay cycle once §2.1 is known (Q7), month before that.
3. Report screen: per-employee table with the metrics, a small chart set, and a per-employee drill-down to the underlying task list with the same filters applied.
4. Reopen semantics: a reopened task drops out of the period it was completed in, and the UI says so (Q8). A visible "figures recalculate as work changes" note.
5. Deactivated employees: retained in history, excluded from active counts, with a clear label (Q9).
6. CSV export: one row per task with computed flags plus a summary block; generated from the same RLS-filtered query as the screen, so the export cannot contain anything the manager could not see.
7. `[A11Y]` Report tables have proper headers and scope; the export button announces completion; charts have text equivalents.
8. No leaderboard, no scoring, no ranking decoration (§1.3).

**Testing checkpoint (P6)** — *multi-user, seeded data with known counts*

- With 10 tasks due in the period, 7 completed in the period → `completion_rate = 70%`; `due_in_period = 10`; the CSV row count matches the table row count.
- Empty denominator renders "—", never "0%".
- A task due in the period but completed in the next period counts in neither the numerator nor the denominator of the first period's rate until it is due — the exact behavior asserted against hand-computed numbers.
- A reopened task disappears from the period where it was completed, and the change is visible immediately in both the table and the CSV.
- An unassigned task never appears in any employee's numbers.
- An employee requests `/reports` → route guard blocks, and a direct data-layer call returns no rows (RLS, not just the guard).
- CSV opens in a spreadsheet with the expected columns, one row per task, and contains no data for any employee outside the manager's organization.

**Exit criteria:** the completion number is reproducible by hand from the exported file, and the export contains nothing the manager could not see on screen.

---

### Phase 7 — Hardening, accessibility audit, and deployment

**Serves:** G1–G3 reliably, for the real client.
**Depends on:** Phases 0–6. Must resolve Q13 (client details) and Q14 (privacy notice and employee communication) before launch.

**Tasks**

1. Full acceptance-scenario suite (§4) automated in Playwright with three accounts.
2. Permission regression suite: every row of the §3.2 matrix in `project-context.md` has an automated test, so a future migration cannot quietly widen access.
3. `[A11Y]` Audit against WCAG 2.1 AA: automated scan (axe) on every route in both themes, plus a manual keyboard and screen-reader pass on the Dashboard, Board, My Tasks, and Task detail. Contrast assertions in CI for both themes and for every user-chosen colour.
4. Responsive pass: 360 px, 768 px, 1280 px, 1440 px. The board may scroll horizontally; status must never be conveyed by colour alone.
5. Performance: route-level code splitting, image handling, a real-size seed benchmark, and a target of LCP < 2.5 s on a mid-range Android over 4G.
6. Security review: no service-role key in the client bundle; storage bucket is private; signed URL TTLs; rate limiting on sign-in; CSP headers; dependency audit.
7. Data lifecycle: purge of soft-deleted tasks and orphaned attachments after a retention window, an admin-run script with a confirmation step, and a documented retention period (Q6, §6.4).
8. Vercel deployment: preview deployments per branch, production environment variables, custom domain, error monitoring, a real uptime check on the health endpoint.
9. Data backup and restore drill for the Supabase project; a written note on what happens if the client loses access.
10. Seed a **Demo organization** (fictional employees, synthetic data) so the client can explore without real employee data in the system before anyone is onboarded (Q14).
11. Handover: a short manager guide (what each number means, how to change the stalled threshold, how to export) and a short employee guide (what is recorded about my work, what is not, and that nothing is tracked outside the app). The employee guide is a product requirement under §6.1/§6.2, not a courtesy.

**Testing checkpoint (P7)**

- All acceptance scenarios in §4 pass in CI on every pull request, in both themes.
- Zero high-severity axe violations on any route; no keyboard trap anywhere; focus is visible on every interactive element in both themes.
- A fresh manager and two fresh employees complete the full lifecycle on real devices (a phone and a laptop, not emulators) following only the written guides.
- The client signs off on the Dashboard answering G1/G2/G3 in under 10 seconds, using a script of their own questions.
- Rollback tested: restore the database from backup into a scratch project and confirm the app works against it.

**Exit criteria:** a manager can log in on a Monday morning and trust what the screen tells them, and an employee can read their own guide and find no hidden tracking.

---

## 4. Goal acceptance scenarios

Executable tests that prove the goal. Each names the accounts involved. **A = Manager, B = Employee One, C = Employee Two**, each in a separate browser. All are also automated in Phase 7.

**Shared setup:** a fresh organization with `stalled_days = 3`, `active_window_hours = 24`, `require_done_review = false`, org timezone fixed to `[TIMEZONE]`. Where a scenario depends on elapsed time, it uses a controllable clock (or seeded `last_activity_at`) rather than waiting.

### (a) The manager assigns a task and the employee sees it

1. Sign in as **A**. Open Inbox, capture a task titled "Prepare weekly inventory report" assigned to **B**, due in 3 days, priority normal.
2. Assert in the database: one `tasks` row with `assignee_id = B`, one `task_assignments` row with `is_current = true`, one `activity_events` row `assignee_changed`, `last_activity_at` ≈ now.
3. In **B**'s browser (separate session, refreshed within seconds), open My Tasks. The task is present in "Not started", with the correct title and due date.
4. **C** refreshes My Tasks. The task is **not** present, and is absent from **C**'s network responses — not merely hidden.
5. **A** opens the board: the card is in the Not started column with B's avatar.
6. Open the task detail as **B**: the activity log shows "Assigned by A".

**Pass:** steps 2–6 hold, and the task is invisible to **C** at the data layer.

### (b) The employee moves it to In progress and the manager's dashboard reflects it

1. From the state in (a), **B** clicks "Start" on the task (or drags the card, if using the board).
2. Assert: `status_id` = in-progress, `started_at` set, `last_activity_at` = now, one `status_changed` activity event.
3. **B** ticks 2 of 5 checklist items. Assert two more activity events and `last_activity_at` advanced.
4. **A** opens the Dashboard → "Who is working on what" shows **B** with 1 active task, the task title, and "just now" / "2 min ago".
5. **A** opens the board → the card sits in the In progress column with 2/5 checklist progress.
6. The Team page card for **B** shows in-progress 1, and the employee strip and the board agree with each other.
7. Repeat the tick after 26 hours (time-shifted seed): the task leaves the "active" count while remaining In progress — the signal is activity recency, not status.

**Pass:** steps 2–6 hold, and step 7 shows the active flag expires on its own.

### (c) An untouched task is flagged as stalled after N days

1. `stalled_days = 3`. **A** assigns **B** a task "Reconcile supplier invoices", and nobody touches it.
2. Time-shift the clock to +2 days 23 h. **A** opens the Dashboard: the task is **not** in the Stalled queue.
3. Time-shift to +3 days 00 h 01 m. **A** refreshes: the task is in the Stalled queue, showing "3 days ago" and the assignee's name. No deploy, no cron, no refresh job — it is derived at read time.
4. **B** opens My Tasks: the **same** task carries the same stalled badge (transparency).
5. **B** adds a comment. The task immediately leaves the Stalled queue for both **A** and **B**.
6. **A** changes `stalled_days` to 5 in settings. Time-shift to +4 days. The task reappears as stalled. Then to 1. The threshold is live, with no deploy.
7. Assert a task with `assignee_id = null` (in the Inbox) is **not** counted as stalled by the formula, and is instead surfaced by the "never assigned" nudge.

**Pass:** the boundary is exact at N, the employee sees the same thing, one comment clears it, and the threshold is configurable live.

### (d) A past-due task shows as overdue

1. **A** assigns **B** a task with `due_at` = today at 23:59:59 in the org timezone.
2. At 23:30 local: the task is in "Due today" for **B**, and **not** overdue anywhere.
3. At 00:00:30 local the next day: **A**'s Dashboard Overdue count is 1; the card shows "1 day overdue"; the Team page shows B's overdue count 1.
4. **B** moves it to In progress: still overdue (overdue is about the due date, not the status).
5. **B** marks it Done: it leaves the overdue queue and the counts on both sides, on refresh.
6. Assert a `done` task with a past due date is not overdue — the formula excludes done statuses.
7. Assert the boundary uses the org timezone, not the browser's: set the browser to a different timezone and confirm no change in the result.

**Pass:** steps 2–7 hold; the flag is derived, exact, and identical for manager and employee.

### (e) The employee marks it Done and the completion numbers update

1. In a seeded period, **B** has 4 tasks with `due_at` inside the period, all not yet done → `due_in_period = 4`, `completion_rate = "—"`.
2. **B** marks 3 of them Done. Assert each gets `done_at = now` and a `status_changed` event.
3. **A**'s Dashboard shows the completed-today feed with the 3 task titles.
4. **A** opens Reports for the period: **B**'s `due_in_period = 4`, `completed_in_period = 3`, `completion_rate = 75%`, `open_tasks` = 1.
5. **B** opens their own numbers on My Tasks: the same 3 of 4 and 75%. Identical.
6. **A** reopens one of the three. `done_at` clears. The rate drops to 50% and the reopened task reappears in the open list. The activity log shows both the completion and the reopen.
7. **C**'s numbers are unchanged by any of **B**'s activity.
8. Assert an unassigned task contributes to nobody's `due_in_period`.

**Pass:** the number is hand-checkable, moves correctly in both directions, and is identical for both roles.

### (f) An employee cannot see another employee's tasks

1. **B** knows the URL of one of **C**'s task IDs (obtainable from the database, not the UI).
2. **B** navigates to it → a **not-found** page, not a blank screen and not a crash — and **not a 403**. A 403 would confirm that the task id is real, which is itself a leak and an enumeration vector. No task content in the response body, no title in `<title>`, no data in the network payload. (A **manager** attempting the same URL gets a genuine 403 on a resource they can see but may not change.)
3. **B** runs `supabase.from('tasks').select('*')` in the browser console → only **B**'s rows return.
4. **B** runs `supabase.from('tasks').select('*').eq('id', '<C task id>').single()` → an empty result, enforced by RLS.
5. **B** attempts to read `activity_events` for **C**'s task → empty.
6. **B** attempts `update tasks set assignee_id = <B>, due_at = <far future>` on their own task → rejected by column grants. Confirmed with a direct SQL client, not only by the absence of a UI control.
7. **B** attempts to insert a task assigned to **C** → rejected.
8. **B** requests `/reports`, `/settings`, `/team` → blocked, and a direct data-layer call returns nothing.
9. **B** attempts to read the attachment storage path of **C**'s task directly → signed URL request refused.
10. **B** attempts to set `last_activity_at` on their own task → rejected by column grants, and no activity event appears.

**Pass:** all ten attempts fail **at the database**, verified with a SQL client as well as in the UI, and no response distinguishes "does not exist" from "not yours". A UI-only block fails this scenario.

### (g) The manager exports a completion report

1. **A** opens Reports, selects the period, and clicks Export CSV.
2. The file contains: one summary row per employee, then one row per task with columns including title, assignee, status, `due_at`, `done_at`, `is_overdue`, `is_stalled`, `days_since_activity`.
3. Row counts equal the on-screen counts for the same period.
4. The completion figures in the CSV equal the on-screen figures and equal the values in scenario (e).
5. Reassigning a task and re-exporting shows the new assignee and a fresh assignment timestamp; the previous assignment is retained in the export's history column or the activity log.
6. **B** requests `/reports/export` directly → 403, and the underlying query returns zero rows for them.
7. Open the CSV in a spreadsheet: no formula injection (a task titled `=cmd|...` is escaped), correct encoding, and no employee outside the organization.
8. A manager who is a member of no other organization sees only their own organization's data in the export.

**Pass:** the file is a faithful, complete, safe reflection of what the manager can see on screen, and nobody else can produce it.

### Cross-scenario integrity check

Run after (a)–(g): for every task touched in the scenarios, the manager's view and the assignee's view of `status`, `due_at`, `priority`, flags, and the activity log are identical field for field, and `activity_events` contains exactly one event per user action with no duplicates and no missing entries.

---

## 5. Risks and assumptions

### 5.1 Risks

| # | Risk | Likelihood | Impact | Mitigation | Owner phase |
| --- | --- | --- | --- | --- | --- |
| R1 | **Trust collapse.** Employees find out the system tracks more than tasks, or believe it does, and start gaming it — closing tasks without doing them | High if mismanaged | Severe — the product's data becomes worthless | No invasive tracking, ever (§6.1); publish what is recorded; employee sees identical data; manager guidance on reading the signals; no per-person scores | 1, 4, 7 |
| R2 | **Stalled threshold is wrong for this client** — too noisy, or too slow | Medium | High — the flagship metric loses credibility | Configurable 1–30; validate the default in week 1 with the actual manager; ask the client before launch (Q2) | 4 |
| R3 | **Employees never update status**, so the dashboard is a wall of "Not started" | High | High — G1/G2 fail in practice | My Tasks is the employee's first screen with one-tap actions; blocked requires a reason; onboarding tells employees the manager can already see the task; manager follows up on Not-started-and-due-soon | 3, 7 |
| R4 | **Privacy/employment-law exposure** if deployed without notice or in a jurisdiction with stricter rules | Medium | Severe — regulatory and reputational | Privacy notice at first login, employee guide, `require_done_review` option, data-retention and purge tooling, explicit client flag that this is not legal advice (§6.4, Q14) | 1, 7 |
| R5 | **RLS gap** — one missing policy leaks employee data across the org | Medium | Severe | Permissions matrix has one automated test per row; `security_invoker` views only; a permission regression suite in CI; security review in Phase 7 | 1, 7 |
| R6 | **`last_activity_at` drift** — a path that updates a task without writing an activity event makes a stalled task look active, or vice versa | Medium | High — the core signal becomes untrustworthy | All activity written by `SECURITY DEFINER` triggers, never by clients; a reconciliation test that diffs `tasks` against `activity_events`; column grants block clients from setting `last_activity_at` | 1, 4, 7 |
| R7 | **Timezone and clock errors** make due dates fire at the wrong hour | Medium | Medium | Due dates stored as `timestamptz` computed from end-of-day in the org timezone; fixed-clock tests; a browser-timezone test in scenario (d) | 2, 4 |
| R8 | **Scope creep back toward the old product** (a content planner, a social field, an automation rule) | High given the project's history | Medium — schedule and coherence | The cut list in §1.2 is written down; a template resembling a removed feature is a scope error; the goal matrix is the acceptance test for any new feature proposal | all |
| R9 | **`@hello-pangea/dnd` incompatibility or unmaintained fork** | Low–Medium | Medium | v18.0.1 supports React 18/19 and is Apache-2.0; all usage isolated in `src/lib/dnd/`; a swap is one file; `@dnd-kit` is the named fallback | 0, 2 |
| R10 | **Mock data reaches a demo or a stakeholder** and is mistaken for the real system | Medium | High — credibility | Dev-only banner; build-time guard preventing monitoring features under the mock adapter; Demo org with obviously synthetic data; mock adapter removed before launch | 0, 4, 7 |
| R11 | **Attachment cost and abuse** (large files, wrong content) | Medium | Medium | 10 MB cap, type allowlist, private bucket, signed URLs with short TTL, quota per org, retention purge | 3, 7 |
| R12 | **Small-team reality**: 4 employees, 2 devices, a shared tablet. "Separate browsers" testing is impractical for the client, and a shared device breaks per-user sessions | High | Medium | Test accounts per person; explicit note in the employee guide to not share logins; `is_active` derived from activity, not presence, so a shared device is not treated as a monitoring signal | 1, 7 |
| R13 | **Client-specific unknowns** (pay cycle, working days, holidays) block period logic | High until Q13 is answered | Medium | Phase 0 default of calendar month; open questions surfaced rather than assumed | 1, 6 |
| R14 | **Vercel + Supabase auth friction** (session refresh, redirects, preview environments pointing at production data) | Medium | Medium | Route guards tested in preview; separate Supabase branches per environment; no production seeding in preview | 0, 7 |
| R15 | **The dashboard is a wall of red** and the manager stops reading it | Medium | Medium | Attention queues are capped with "and N more"; stalled and blocked are separated so a single loud problem does not mask the rest; completed feed keeps the screen useful on a good day | 4 |

### 5.2 Assumptions

Stated so they can be falsified rather than discovered late.

1. The client will use the system **daily** as the source of truth for task state. If not, the product cannot work, and R3 applies.
2. Employees have **device and account access** for the period being monitored.
3. One organization per deployment is sufficient; `org_id` is present everywhere so multi-org is a configuration change, not a rewrite.
4. Team size is small (tens, not thousands). Aggregation is done with indexed SQL, not a warehouse.
5. The organization timezone can be configured; if the team spans timezones, v1 uses one org timezone and per-user timezones are deferred.
6. Email delivery is not required (Q12). If the manager will not open the app daily, a digest email becomes a requirement and moves out of "out of scope" — flagged for the client, not assumed.
7. Attachment volume is modest (hundreds of MB per organization per month). Storage cost is discussed with the client before launch.
8. Statuses are simple linear states; workflows with multiple reviewers or parallel approval are out of scope.
9. The client accepts a factual report rather than a performance-scoring tool, and will not ask for rankings later. If they do, it is a scope conversation, not a feature request to just build.
10. The manager is a hands-on supervisor who assigns work directly, not a middle manager routing work between other managers.
11. `require_done_review` stays off unless Q1 says otherwise; if it turns on, the Dashboard needs a new queue for pending items, which is a Phase 4 change.
12. No requirement for real-time push updates; a 30–60 second poll or a refresh-on-focus is acceptable for a team this size.

---

## 6. What is deliberately deferred

Deferred means: named, scoped, and agreed to be *after* the phases above. Not "maybe someday."

| Item | Why deferred | Triggers the revisit |
| --- | --- | --- |
| **Calendar drag-and-drop rescheduling** | Loses to every monitoring feature; the keyboard-accessible due-date editor covers the need | Manager asks for it after real use |
| **Multiple boards** | One default board answers G1–G3 | A second distinct workflow appears (e.g. recurring vs. project work) |
| **Team/sub-team structure** | Not needed to answer the goal (Q5) | Client has genuinely separate teams with separate managers |
| **Full label management** | Schema present; thin UI in Phase 2 | More than ~8 labels in real use |
| **In-app and email notifications** | Not in the goal; adds an integration surface (Q12) | Manager says they will not open the app daily |
| **Automation rules** | Explicitly out of scope per the brief | Only after a stable reports baseline exists |
| **Slack / Teams / integrations** | Explicitly out of scope per the brief | Client requirement |
| **Native mobile app** | Explicitly out of scope per the brief | Only after sustained web-app usage |
| **Per-user timezones** | One org timezone is enough for a small local team | Team becomes genuinely distributed |
| **Custom fields, subtasks, task dependencies** | No monitoring value | Never, on current goals |
| **Employee scoring, rankings, leaderboards** | Undermines §6.2 trust; corrupts the completion rate | Never, on current goals |
| **Time tracking / timesheets / attendance** | Drifts toward surveillance; answers a different question | Never, on current goals |
| **Presence, login times, idle tracking, screen capture, location** | Violates the surveillance boundary | Never. Permanently out of scope |
| **Gantt / timeline / workload heatmap** | Answers "when", not "who/what/done/overdue" | Only if the client's real work demands it |
| **Public share links, SSO/SAML, granular notification prefs** | Unnecessary for a small team | Client IT requirement |
| **Multi-org / white-label** | Single-org is sufficient | Second client |
| **Native offline / optimistic-conflict resolution beyond rollback** | Small team, server available | Offline requirement appears |
| **Data warehouse / BI connector** | Aggregates are small enough for SQL | Data volume or analysis complexity outgrows the app |

---

## 7. Test strategy, fixtures, rollout, and sizing

### 7.1 Four test layers

| Layer | Tool | What it proves | Why it exists |
| --- | --- | --- | --- |
| **Unit** | Vitest | The §5 formulas as pure functions, with fixed inputs and boundary values at exactly N | The maths, isolated from everything |
| **SQL / RLS** | SQL assertions against Postgres | One test per row of the permissions matrix in `project-context.md` §3.2 | **The layer that matters most.** A UI-only block is not enforcement; the database is what stops a query bug from leaking one employee's rows to another |
| **Component** | Vitest + Testing Library | Status actions, flag rendering, empty/loading/error states, focus behaviour, contrast in both themes | Behaviour of a component without a browser or a database |
| **E2E** | Playwright | The nine acceptance scenarios in §4 | The goal itself, end to end |

**How the multi-user requirement is satisfied in CI.** Playwright's `browser.newContext()` gives each account isolated cookies and storage, so one automated run *is* the "one manager and two employees on separate browsers" checkpoint — not a simulation of it. The manual version on three real browsers or profiles still runs before Phase 7 exits, because the client-facing checkpoint is about the real thing, not a harness.

**No clock mocking, and no waiting three days.** See `project-context.md` §5.8: thresholds are asserted either with `SET LOCAL app.now` (exact boundaries, against production SQL) or with backdated seed rows. Both exercise the real code path. A test that mocks the formula is not a test of the formula.

### 7.2 Migration order

`0001_schema` → `0002_functions` (`app_now`, `current_org_id`, `current_role`, `is_manager`, `can_view_task`) → `0003_rls` (policies + column grants) → `0004_triggers` (the activity engine from `project-context.md` §5.2.1, `last_activity_at` freshness rules from §5.2.2, done/reopen/blocked guards) → `0005_views` (`monitoring_task_state`, `employee_task_rollup`, `employee_period_rollup`) → `0006_storage` (private bucket + storage policies) → `0007_seed`.

Each migration is independently re-runnable. **Phase 1 is the highest-risk phase in the plan** — it is the one that has to be right before anything else is worth building.

### 7.3 Seed data

One demo organization, 1 manager + 2 employees, ~15 tasks chosen so that **every formula in §5 has a fixture**. Deterministic IDs, so assertions can name a task rather than searching for one.

The fixture set must contain: due in 2 days / due today / due yesterday; `last_activity_at` at 1 hour / 2 days / 4 days / never; all four statuses; one task stalled-but-not-overdue and one overdue-but-not-stalled; one unassigned Inbox item 5 days old; one blocked-and-stalled; one reopened; one with a checklist, comments, and an attachment; one task owned by the manager.

The same seed is the **Demo org** the client explores during rollout, with fictional people and synthetic data only.

### 7.4 Rollout to the client

| Step | What happens | Gate |
| --- | --- | --- |
| 1 | Demo org with fictional employees; the manager and team explore freely | The manager can explain every number on the Dashboard |
| 2 | Manager brief: what each metric means, how to change the stalled threshold, how to export | — |
| 3 | Team walkthrough with the written employee guide: exactly what is recorded, and what is not | Every employee acknowledges the privacy notice at first login |
| 4 | Real organization created; real employees invited; no real data yet | — |
| 5 | One week running both ways (paper or chat, and the app) | No discrepancy the team cannot explain |
| 6 | Cutover | Client sign-off: the manager answers their own three questions in under 10 seconds |

No real employee data enters the system before step 4. A product that observes performance and is introduced covertly loses the data quality it depends on.

### 7.5 Sizing

Very rough, for **one experienced full-stack developer** who already knows Supabase, RLS, and this stack.

| Phase | Days | | Phase | Days |
| --- | --- | --- | --- | --- |
| 0 | 3–4 | | 4 | 7–9 |
| 1 | 6–8 | | 5 | 4–5 |
| 2 | 7–9 | | 6 | 4–5 |
| 3 | 6–8 | | 7 | 5–7 |
| | | | **Total** | **~42–55** |

The schedule risks are not technical. They are `Design.md` slipping, and a change to the stalled threshold or the privacy-notice flow arriving after Phase 4 — both of which cost about a week, not a redesign. **Phases 0–4 are the product**; everything after is extension.

---

## 8. Working agreements for the build

1. **No scope resolution by assumption.** Anything on the §11 open-questions list in `project-context.md` that would change scope, cost, or behaviour is asked, not guessed.
2. **The cut list stays cut.** Social media planning, compose-post, platform selectors, AI captions, media library, content-approval templates, and the Timeline and Map board views do not reappear in any form.
3. **Monitoring is not surveillance.** A feature that would let a manager learn something about a person that the person cannot see about themselves is rejected at design time, not at review time.
4. **One data boundary.** Components never call the backend; the mock adapter never satisfies a monitoring checkpoint.
5. **`[DESIGN]` is a real dependency.** Interface tasks are not started against invented visual specifications. If `Design.md` slips, the slip moves UI work, not the data and logic work behind it.
6. **Accessibility is done, not polish.** No phase closes with an unresolved keyboard or contrast failure.
7. **A checkpoint is a checkpoint.** If a phase's multi-user test cannot be run with one manager and two employees on separate browsers, the phase is not done.
8. **Numbers are reproducible.** Every figure on the Dashboard and in Reports must be hand-checkable from the exported CSV, using the formulas written down in `project-context.md`.
9. **Conventional Commits, enforced.** commitlint runs in a `commit-msg` hook and again in CI. Do not bypass it with `--no-verify` — fix the message. Branch prefixes match the commit type.

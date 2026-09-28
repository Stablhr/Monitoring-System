# project-context.md

**Project:** Employee Task Monitoring System (working title — not a brand name; see §9 Originality)
**Status:** Planning document. No application code exists yet.
**Last updated:** 2026-09-28
**Companion document:** `plan.md` (build order, acceptance scenarios)

---

## 1. Overview and problem statement

A business owner managing a small team of employees has no reliable way to answer three questions:

1. **Who is working on what right now?**
2. **Which assigned tasks are actually finished?**
3. **Which tasks are late, or stuck and going nowhere?**

Today those answers come from asking people, from memory, or from a chat log nobody can search. The result is that work silently stalls: a task sits "in progress" for two weeks, the owner does not notice, and the deadline passes.

The previous direction for this project (Trello-style boards plus a social media content planner) has been revised. The social media scheduling and content planning features are removed entirely and must not be reintroduced. What remains — boards, cards, and a dashboard — is kept only because it is the minimum surface needed to answer the three questions above.

### 1.1 The problem stated positively

The system is a **task assignment and activity-monitoring system for a small team**. Its value is not that it manages tasks; it is that it produces a trustworthy, always-current answer to "is my team actually working, and what is stuck?"

Every feature in this plan is justified against that sentence. Anything that does not move it forward is cut or deferred to a clearly labelled later phase.

### 1.2 What this product deliberately is not

It is **not** an employee surveillance tool. The distinction is not cosmetic; it is a product boundary:

- It measures **work on tasks** (status changes, checklist ticks, comments, attachments, edits), which are actions a person performs deliberately in the app.
- It never measures **the person** (no screen capture, no keystroke logging, no location, no app-level activity tracking, no login-time tracking beyond session security).
- Every employee can see the same task data and the same activity history about their own work that the manager sees. Nothing about an employee is recorded in a place the employee cannot read.

This boundary is a functional requirement, not a values statement. It is enforced by the data model (§7): monitoring signals are **derived from task rows**, and there is no table in which a fact about a person can be stored that is invisible to that person.

---

## 2. Goal

> Give a manager one place to assign tasks to employees and to see, without asking anyone:
> **(1)** who is working on what, **(2)** which tasks are finished, and **(3)** which tasks are overdue or stalled.

### 2.1 Client details

These are **not yet known** and must not be invented. They are carried as placeholders and must be filled in with the client before the Reports phase, because headcount, timezone, and working-week assumptions affect period logic.

| Field | Value |
| --- | --- |
| Client name | `[CLIENT NAME]` |
| Business | `[BUSINESS NAME]` |
| Industry | `[INDUSTRY]` |
| Team size | `[NUMBER OF EMPLOYEES]` |
| Operating timezone | `[TIMEZONE — e.g. Asia/Manila]` |
| Working days | `[e.g. Mon–Fri]` |
| Working hours | `[e.g. 09:00–18:00]` |
| Billing cycle / pay period | `[e.g. semi-monthly]` |

**Rule for future work:** if a decision would depend on one of these values and the value is unknown, the decision goes to §11 Open Questions rather than being assumed silently.

### 2.2 Success conditions

The goal is met when a manager can open the Dashboard, and within one screen and no more than ~10 seconds, answer questions 1, 2 and 3; and can open any employee and confirm or correct the answer. The Plan's acceptance scenarios (§`plan.md`) encode this as executable tests.

---

## 3. Roles and permissions

### 3.1 Role model

Exactly two roles in the first release:

- **Manager** — one or more accounts. Sees all tasks in their organization, assigns and reassigns, reopens completed work, manages boards, statuses, settings and user accounts, views reports and exports.
- **Employee** — sees only tasks where they are the current assignee. Updates status, ticks checklist items, comments, attaches files.

A **multi-team** concept (one employee on several teams) is modelled but **not enforced in the first release** — see §11 Q5.

### 3.2 Permissions matrix

| Capability | Manager | Employee | Enforced at |
| --- | --- | --- | --- |
| Create a task (unassigned) | Yes | **Open question** (Q3) | RLS INSERT |
| Create a task assigned to self | Yes | **Open question** (Q3); default assumption = yes, self-assign only | RLS INSERT `WITH CHECK` + trigger |
| Create a task assigned to someone else | Yes | No | RLS INSERT + column grants |
| View all tasks in organization | Yes | No | RLS SELECT |
| View own assigned tasks | Yes | Yes | RLS SELECT |
| View unassigned (inbox) tasks | Yes | No | RLS SELECT |
| Edit title / description / labels / checklist | Yes | Own tasks only | RLS + column grants |
| Change task status | Yes | Own tasks only | RLS + column grants |
| Change assignee (assign / reassign / unassign) | Yes | No | Column grant (`assignee_id` not granted) |
| Change due date | Yes | No | Column grant (`due_at` not granted) |
| Change priority | Yes | No | Column grant |
| Mark a task Done | Yes | Own tasks only | RLS + column grants |
| Reopen a task from Done | Yes | No | Trigger rejects employee transitions out of a done status |
| Comment | Yes | Own tasks only | RLS SELECT + INSERT |
| Attach / remove file | Yes | Own tasks only | RLS + Storage policy |
| View activity log | Yes (all) | Own tasks only | RLS SELECT on `activity_events` |
| View activity log *of other people* | Yes | No | RLS SELECT |
| Delete / archive a task | Yes (archive) | No | RLS UPDATE/DELETE (manager only) |
| Purge archived tasks | Yes | No | RLS DELETE (manager only) |
| Create / rename / archive boards and columns | Yes | No | RLS |
| Reorder cards on a board | Yes | Own cards only | RLS + column grant (`position`) |
| Create / edit / deactivate statuses | Yes | No | RLS |
| Change organization settings (incl. stalled threshold) | Yes | No | RLS |
| Invite / deactivate users, change roles | Yes | No | RLS + server-side admin path |
| View reports | Yes | No | RLS |
| Export report CSV | Yes | No | RLS (export reads through the same policies) |
| Read own profile | Yes | Yes | RLS |
| Read another employee's profile | Yes | No | RLS |

**Rule:** the interface hides what a user cannot do, but hiding is not enforcement. Every row above marked "Enforced at" must have a corresponding database policy or column grant. A permission that exists only in the UI does not count as implemented.

### 3.3 Least privilege, concretely

- Enforcement lives in Postgres (Supabase row-level security + column-level grants), not in React.
- The browser never receives a row the user is not allowed to read. There is no endpoint that returns "all tasks, filtered client-side".
- Service-role credentials are used **only** in server-side, audited code paths (user invitation, CSV export of large ranges). They must never be shipped to the client.

---

## 4. Core areas

Each area is described well enough to build without further design input. All interface work additionally depends on `Design.md` (see §10).

### 4.1 Manager Dashboard

The manager's home screen. Its job is triage: *what needs my attention right now?*

Sections, in priority order:

1. **Needs attention now** — three queues, each with a count and a short list:
   - *Overdue* (§5.3)
   - *Stalled* (§5.4)
   - *Blocked* — tasks whose status is `blocked`, split by whether they have gone stale (so a manager can see whether "Blocked" is being used as a parking space rather than a signal)
2. **Who is working on what** — a per-employee strip: name, active task count, current task title, time since last activity, today's due count. This is the literal answer to goal question (1). Only employees with at least one open task appear.
3. **Team pulse counters** — assigned / in progress / done / overdue / stalled totals for the current period.
4. **Unassigned / Inbox overflow** — count with a link, so tasks captured but never assigned do not hide.
5. **Completed today / this week** — a short feed, so the manager sees progress, not only problems. A monitoring tool that only shows bad news gets ignored.

Every number is a link into the filtered list behind it. Every number is computed by the same server-side function the employee's own view uses, so the two can never disagree.

### 4.2 Team page

One card per employee (active employees only, most-loaded first or alphabetical — a setting):

- Name, role, title, avatar
- **Assigned** (open tasks), **In progress**, **Done** (current period), **Overdue**, **Stalled** counts
- Currently active task + last activity time ("12 min ago", "4 days ago")
- Link to that employee's task list, pre-filtered
- Manager-only extra: a *Reassign all* bulk action, deferred (see `plan.md`)

The Team page is the "is this person actually working?" screen. It shows **task activity**, never presence, login times, or device data.

### 4.3 Boards

Kanban-style. Columns map to statuses (§5.1).

- One default board per organization ("All Tasks"); additional boards can be created later, and a task belongs to at most one board.
- Cards show: title, assignee avatar, due date with overdue styling, priority, label chips, checklist progress (e.g. 2/5), comment and attachment counts, and a stalled/overdue indicator.
- Drag a card between columns to change status. This writes: status change, `last_activity_at`, `done_at`/`started_at` side effects, and an activity event — in one transaction.
- Filters: assignee, priority, label, due (overdue / today / this week / no date), and "only stalled".
- Managers may rename columns; the underlying status is shared across boards.
- Keyboard equivalent: focus a card, use arrow keys / space to lift, move, and drop, with a live-region announcement. Drag-and-drop without a keyboard path does not count as done.

### 4.4 My Tasks (employee home)

The employee's landing page. Deliberately narrow — an employee should see their work, not the org's.

- Grouped: **Overdue**, **Due today**, **In progress**, **Not started**, **Blocked**, **Done this week**.
- Inline controls: start, block (with a required reason note), mark done, tick checklist items, add comment, attach file.
- "Mark done" is one click and needs no manager approval by default (§5.1, Q1).
- The same stalled/overdue badges the manager sees are shown here, including for the employee's own stalled tasks. Transparency is a feature, not a leak.
- Explicitly absent: the ability to see colleagues' tasks, other people's names on the Team page, reports, or organization settings.

### 4.5 Inbox

Quick capture for tasks that exist before they are assigned to anyone.

- A single input, created from anywhere via a global shortcut (`n`).
- Captured tasks have `assignee_id = null`, status `not_started`, and are **excluded** from every employee's workload and from the completion rate denominator (see §5.5).
- Manager triage view: assign, edit, or discard. Unassigned tasks older than the stalled threshold get a "never assigned" nudge in the Dashboard's attention section.
- Optionally, employees may be allowed to submit to the Inbox rather than assign directly (Q3).

### 4.6 Schedule

Calendar view of due dates.

- Month view and week/agenda view. Employees see their own tasks; managers can switch to a per-employee view.
- Drag a card to another day to change the due date (manager only — employees cannot move their own deadlines, §3.2).
- Overdue backlog rendered as a pinned strip above the calendar, because a month grid hides past dates.
- **Lower priority than monitoring features.** If the calendar drag-reschedule threatens a later phase, ship the read-only month grid first and the drag later.

### 4.7 Task detail

A drawer or full page, depending on `Design.md`. Contains:

- Title, description (rich text, plain text minimum)
- Status (with allowed transitions), assignee, due date, priority
- Labels (org-wide, colour-coded)
- Checklist items with progress
- Comments (threaded flat, with edit/delete of your own within 5 minutes or by manager)
- Attachments: files and images, uploaded to private storage, served via short-lived signed URLs
- Activity log: chronological, human-readable ("Aries moved this to In progress", "Checklist item 2 ticked", "Reopened by manager"), derived from `activity_events`

The activity log is a transparency mechanism first: it is the same list the manager reads. It must read as a factual history, not as a surveillance report — no "idle time" entries, no session data.

### 4.8 Reports

Per-employee and per-period completion summaries, manager only, with CSV export.

- Period selector: week / month / quarter / custom range. Default = the current pay cycle once §2.1 is filled in; month until then.
- Metrics (§5.5, §5.6), one table + one small chart set.
- CSV export produces one row per task with the computed flags, plus a summary block. Built from the same RLS-filtered query the screen uses, so the export can never contain something the manager could not see on screen.
- No cross-employee ranking screen presented as a league table; the report is a factual summary per employee for a period. (Presentational choice, consistent with §9.)

---

## 5. Monitoring definitions (normative)

These are the contract. Build and test against these formulas literally. All timestamps are `timestamptz`. "Now" is the server clock. All flags are **derived at query time** — they are never stored as columns, so they cannot go stale or disagree with the source data.

### 5.1 Status model

Default statuses, seeded per organization and editable by the manager:

| key | name | colour role | `is_done` | Notes |
| --- | --- | --- | --- | --- |
| `not_started` | Not started | neutral | false | default entry state |
| `in_progress` | In progress | primary (turquoise) | false | sets `started_at` on first entry |
| `blocked` | Blocked | warning | false | requires a reason note |
| `done` | Done | success | true | sets `done_at` |

Model: a `statuses` table per organization, each row carrying `is_done`. Any status the manager creates is treated as non-done unless flagged. Deleting a status that is in use is disallowed; archiving is allowed and existing tasks keep their status row.

**Does "Done" need manager verification?**
**Decision (default): No.** The employee marks a task Done; the manager can reopen it, and the reopen is recorded in the activity log. Rationale: a verification step inserts a second hidden queue ("awaiting approval") that would itself need monitoring, and it delays the exact signal the product exists to give ("what is finished"). Verification also creates pressure on employees to under-report completion, which corrupts the completion rate.

**Escape hatch:** an organization setting `require_done_review` (default `false`) is modelled. If enabled, a status `done_pending_review` is activated and `done` is reachable only by managers; the completion rate (§5.5) counts only `is_done` statuses, so pending items are visible as *not yet done* and do not inflate the metric. This is a setting flip, not a redesign.

**Flagged as an open question (Q1).**

### 5.2 Activity and `last_activity_at`

An **activity event** is a row in `activity_events`. Event types:

`task_created`, `status_changed`, `reopened`, `assignee_changed`, `due_date_changed`, `priority_changed`, `title_edited`, `description_edited`, `checklist_item_added`, `checklist_item_ticked`, `checklist_item_unticked`, `comment_added`, `attachment_added`, `attachment_removed`, `label_changed`, `label_added`, `label_removed`.

**`last_activity_at`** is stored on `tasks` and maintained by a database trigger that stamps it to `now()` on every activity event for that task. It is initialized to the task's `created_at` on insert.

```
last_activity_at(task) = max( created_at(activity_events where task_id = task.id) )
                        , tasks.created_at
```

**Trigger-based, not client-based.** The client must not be able to set `last_activity_at` directly (it is not in any client's column grant list), so a task cannot be kept "fresh" by an unlogged write.

**Visible activity (goal question 1):** a task is considered *actively worked on* when it has had activity within the **active window**.

```
is_active(task) = task.last_activity_at >= now() - interval '24 hours'
                  AND task.status_id is not a done status
```

The active window is an organization setting `active_window_hours`, default **24**. A task assigned but never touched has `last_activity_at = created_at`, so it is not active from the start — which is the correct, visible signal.

### 5.3 Overdue

```
is_overdue(task) = task.due_at IS NOT NULL
                   AND task.due_at < now()
                   AND task.status is NOT a done status
```

- `due_at` is a timestamp, not a bare date. A task due "today" is due at **23:59:59 in the organization's timezone** (§2.1), so a task does not become overdue at midnight in the database's timezone while the employee still thinks it is a normal day.
- Overdue is a *state*, and the UI must show **how long** it is overdue (e.g. "3 days overdue"), because a binary flag hides urgency differences.
- Overdue count on a card is derived, never typed by hand.

### 5.4 Stalled

```
is_stalled(task) = task.assignee_id IS NOT NULL
                   AND task.status is NOT a done status      -- includes not_started, in_progress, blocked
                   AND task.last_activity_at < now() - (settings.stalled_days || ' days')::interval
```

- `settings.stalled_days` is an organization setting. **Default N = 3 calendar days.** Recommended range 1–30, enforced in the settings form.
- `blocked` tasks are **included** in stalled, because a blocked task with no follow-up for a week is the most common form of silent failure. The UI separates *stalled & blocked* from *stalled & moving-blocked* so the manager can tell the difference.
- Default is calendar days, not working days, for v1 — working-day math needs a holiday calendar the client has not supplied. Flagged (Q2).
- Stalled is recomputed on read. A task that becomes stalled at 00:01 appears in the Dashboard with no scheduled job. The only scheduled job in v1 is the optional digest email, which is deferred.

### 5.5 Completion rate

Two clearly separated metrics, because they answer different questions and mixing them is the most common way a report like this misleads.

**Per-employee, for a period P** (a half-open interval `[P.start, P.end)`):

```
assigned_in_period(emp, P)   = count of tasks where
                                 task_assignments.user_id = emp
                             AND task_assignments.assigned_at >= P.start
                             AND task_assignments.assigned_at <  P.end
                             AND task_assignments.is_current = true
                             AND task.deleted_at IS NULL

completed_in_period(emp, P)  = count of tasks where
                                 task.assignee_id = emp
                             AND task.done_at >= P.start
                             AND task.done_at <  P.end
                             AND task.deleted_at IS NULL

due_in_period(emp, P)        = count of tasks where
                                 task.assignee_id = emp
                             AND task.due_at >= P.start
                             AND task.due_at <  P.end
                             AND task.deleted_at IS NULL
                             AND task.status is NOT a done status   -- "was due", at query time
```

**Headline metric — completion rate:**

```
completion_rate(emp, P) = completed_in_period / due_in_period
                          (0 if due_in_period = 0; render as "—", not "0%")
```

Rationale: the denominator is work that was *expected* in the period (due in the period), and the numerator is work *finished* in the period. This is a ratio of outcomes to expectations, which is what a manager asks for. Using `assigned_in_period` as the denominator produces rates that can exceed 100% and is rejected.

**Supporting metrics:**

```
on_time_rate(emp, P)      = count(done tasks with done_at <= due_at AND done_at in P)
                            / due_in_period(emp, P)

overdue_count(emp, P)     = count of tasks where is_overdue(task) AND assignee = emp
stalled_count(emp, P)     = count of tasks where is_stalled(task) AND assignee = emp
open_tasks(emp)           = count of tasks where assignee = emp
                            AND status is NOT a done status
                            AND deleted_at IS NULL
```

**Period rules, stated exactly:**

- Default period = current calendar month in the organization timezone.
- A period boundary is computed in the organization's timezone, then converted to UTC for the query.
- A task appears in the *assignment* bucket of the period containing its current assignment timestamp; a reassignment moves it (and its history is retained in `task_assignments`).
- A task appears in the *completion* bucket of the period containing `done_at`; a reopen clears `done_at` and removes it from that bucket retroactively. Reports are recomputed live, never snapshotted — a manager looking at last month today sees last month as it actually stands.
- Reopened work therefore *reduces* the historical completion number. This is intentional and should be stated in the UI ("figures recalculate; reopened tasks are removed from the period they were completed in").
- Empty denominators render as "—".

### 5.6 Workload

```
open_tasks(emp)  = open task count (see above)
active_tasks(emp) = count where is_active(task) AND assignee = emp
```

Displayed as: open count, with overdue and stalled shown as sub-counts, and "oldest open task" as the pressure indicator. Workload is used for **balance**, not for ranking employees by output; there is no leaderboard.

### 5.7 Derived-only, never stored

`is_overdue`, `is_stalled`, `is_active`, `is_blocked`, `open_tasks`, `completion_rate`, `workload` are computed by a Postgres view/function (`monitoring_task_state`, `employee_period_rollup`) that both the manager UI and the employee UI read. Consequences:

- The manager and the employee can never see different numbers for the same task.
- There is no table in which "this employee stalled" is recorded as a fact about a person — the same transparency rule as §1.2, enforced structurally.

---

## 6. Principles (binding)

### 6.1 Activity-based, not surveillance
Monitoring reads task activity only. **Not implemented, ever, in this product:** screen capture, webcam, keystroke or mouse logging, GPS or location tracking, clipboard monitoring, browser history, app-usage/Idle tracking, badge-in/out attendance, network traffic inspection, or any background collection that the employee does not perform inside the app. A stall is inferred from absence of *task* activity — the absence of a work action, not the absence of the person.

### 6.2 Transparency
An employee can see, about their own tasks: the same status, assignee, due date, priority, checklist, comments, attachments, and activity log the manager sees; the same overdue, stalled, and active flags; and the same completion numbers for themselves that appear in the manager's report. There is no employee-visible-invisible data. If a future feature would create such data, it is out of scope by this rule.

### 6.3 Least privilege
Enforced in the database (§3.2, §7.4). The UI is a convenience layer over the enforcement, never the enforcement itself.

### 6.4 Data privacy
The system stores employee names, work contact details, task text, comments, and uploaded files. That is personal data about identified individuals.

- **Flag for the client (not legal advice):** before launch, the client must be informed of the data privacy obligations that apply to their business and location. If the business operates in the **Philippines**, this includes the **Data Privacy Act of 2012 (RA 10173)** and the implementing rules and issuances of the National Privacy Commission — in particular the obligations around lawful processing, the required **privacy notice and consent** for processing personal data, the **retention and disposal** of records, **security safeguards** for the personal data in the system's custody, and the **rights of the data subject** (access, correction, objection, erasure). Other jurisdictions have their own equivalents. The client should obtain advice appropriate to their situation; this document is a product plan and not legal advice.
- Product controls supporting those obligations, to be built in: retention and purge of archived tasks and attachments, a visible privacy notice acknowledged at first login, employee access to and correction of their own profile, and an audit trail of access to files.
- Because the system exists to observe employee performance, the client should be told plainly that employee notice and agreement is a practical precondition, and that a covert deployment would damage both the tool and the trust the tool depends on.

### 6.5 Originality
Boards, lists, cards, and drag-and-drop are established patterns and are used. Nothing is copied from another product: not a name, logo, wordmark, icon set, illustration, illustration style, marketing copy, or brand colour palette. Icons come from an open-source set (planned: Lucide) with its licence recorded. The turquoise primary, the flat treatment, and the light/dark theming are this product's own; `Design.md` will specify the rest and must not restate any other product's identity.

---

## 7. Technical constraints and stack

### 7.1 Editor and platform
VS Code. Deployed on Vercel.

### 7.2 Frontend
- **React 19** + **Vite** + **TypeScript**.
- **Tailwind CSS v4** (CSS-first `@theme` tokens, `@tailwindcss/vite` plugin) — decision recorded because v4 moves the token definition into CSS and the plan's theming/contrast work depends on it.
- Routing, forms, and validation: chosen in Phase 0, no framework lock-in assumed.
- **Drag and drop: `@hello-pangea/dnd`.** Reasons: it is the maintained community fork of `react-beautiful-dnd` (v18.0.1, Apache-2.0) and declares support for React 18 and 19, which keeps us on current React; it has a real keyboard-accessible drag mode with screen-reader announcements built in, which is required here (keyboard support is part of "done", not polish); and it is a drop-in for the canonical Kanban interaction model, so the board is cheap and correct. **Caveat and boundary:** it is a list/board library, not a calendar library — the Schedule drag-reschedule (§4.6) will not use it, and will use a small purpose-built pointer/keyboard interaction on day cells. All `@hello-pangea/dnd` usage is confined to one adapter module (`src/lib/dnd/`) so a future React or library migration is a single-file change.
- Visual direction is **pending `Design.md`**. No visual specifications are invented in this plan. Already decided: flat, professional, business look; **no glassmorphism, no neumorphism**; **turquoise primary**; **light and dark themes**; text readable on any background.

### 7.3 Backend
**Required.** A manager cannot monitor employees if each person's data lives in their own browser, so a localStorage-only build is unacceptable here. Authentication, roles, and a shared database with per-row access rules land in Phase 1, not at the end.

**Supabase (default choice):** Postgres + Auth + Row Level Security + Storage.

Why this fits the product specifically:

1. **Row-level security is the requirement.** "Employees see their own, managers see all" is a per-row predicate. Postgres RLS expresses that directly in the database; it is enforced no matter which client code, query, or future integration touches the data.
2. **The data is relational and the reports need joins.** Tasks, assignments, activity, comments, and a period rollup are relational by nature. A single SQL view computes `is_overdue`, `is_stalled`, and the completion rate consistently for the manager and the employee — the property §5.7 depends on. In a document store, that same guarantee requires duplicating logic in application code, which is exactly where the two views would drift apart.
3. **Atomic writes matter for monitoring.** A status change must update `tasks.status_id`, `tasks.last_activity_at`, `tasks.done_at`, and insert an `activity_events` row together or not at all. If the activity log can be lost, the product's core signal is unreliable. A single Postgres transaction gives this; Firestore would need careful batching.
4. **Cheap reads for dashboards.** Postgres computes counts server-side; a manager dashboard that is one indexed aggregate query does not bill per document read.

Firebase was considered and rejected for this product, not in principle: Firestore rules *can* enforce per-document access, but (a) the period rollup and the derived monitoring flags would have to be computed and duplicated in client code, risking exactly the manager/employee disagreement we are trying to prevent, and (b) per-document read pricing makes an always-open monitoring dashboard noticeably more expensive over time than a small, indexed SQL query.

### 7.4 Data access layer (binding architectural rule)

**No component calls the backend directly.** All database access goes through one module.

```
src/lib/data/
├─ types.ts               # shared domain types
├─ index.ts               # the public API components are allowed to import
├─ repositories/          # tasks.ts, users.ts, boards.ts, comments.ts, …
└─ adapters/
   ├─ supabase/           # real implementation
   └─ mock/               # in-memory implementation for interface prototyping only
```

- Components import from `src/lib/data`. A lint rule or code-review convention enforces the boundary.
- The adapter is chosen by an env flag (`VITE_DATA_SOURCE=supabase | mock`). Mock mode renders a persistent dev-only banner.
- **The mock adapter may be used only for early interface prototyping, and every monitoring feature is incomplete until the Supabase adapter replaces it.** "Dashboard works" against the mock adapter does not satisfy any checkpoint in `plan.md`; each monitoring checkpoint names the Supabase adapter explicitly.
- The mock adapter is useful for one thing only: designing and building interfaces before auth and RLS exist. It is not a demo mode, not a fallback, and it will not be present in the deployed app.

### 7.5 Deployment
Vercel for the frontend. Supabase for the database, auth, and storage. Vercel environment variables for the Supabase URL/anon key; the service-role key is server-side only. Deployment is Phase 7.

---

## 8. Data model (proposed)

Postgres, Supabase. `org_id` is present on every business table so access rules are uniform and a future multi-organization deployment stays possible. All timestamps are `timestamptz`. Soft delete via `deleted_at` where noted.

### 8.1 Tables

**`organizations`**
`id uuid PK` · `name text` · `slug text unique` · `created_at` · `created_by uuid → profiles.id`

**`profiles`** (one row per user; `id` = `auth.users.id`)
`id uuid PK` · `org_id` · `email citext unique` · `full_name text` · `role text check in ('manager','employee')` · `title text` · `avatar_url text` · `is_active bool default true` · `created_at` · `deactivated_at` · `accepted_privacy_notice_at`

Role lives on the profile rather than in a separate table for v1; a `roles` join table is deferred (see `plan.md`).

**`teams`** — modelled, not enforced in v1
`id uuid PK` · `org_id` · `name` · `description` · `created_by` · `archived_at` · `created_at`
**`team_members`** — `team_id` · `user_id` · `role_in_team` · `joined_at` · `left_at`
*(Q5: whether v1 uses teams at all. If v1 is single-team, these tables exist but no interface uses them.)*

**`statuses`**
`id uuid PK` · `org_id` (nullable = system default) · `key text` · `name text` · `color_token text` · `position int` · `is_done bool default false` · `is_blocked bool default false` · `is_active bool default true` · `created_at`

**`boards`**
`id uuid PK` · `org_id` · `name` · `description` · `is_default bool` · `created_by` · `archived_at` · `created_at`

**`board_columns`**
`id uuid PK` · `board_id` → boards · `status_id` → statuses · `name_override text` · `position int`
Unique on `(board_id, status_id)`.

**`tasks`**
`id uuid PK` · `org_id` · `board_id` → boards (nullable; null = not on a board, still in My Tasks) · `column_id` → board_columns (nullable) · `title text not null` · `description text` · `status_id` → statuses not null` · `assignee_id` → profiles (nullable = Inbox) · `reporter_id` → profiles` · `created_by` → profiles` · `due_at timestamptz` · `priority text check in ('low','normal','high','urgent') default 'normal'` · `position real` (fractional; midpoint insert, periodic rebalance) · `last_activity_at timestamptz not null` · `started_at timestamptz` · `done_at timestamptz` · `blocked_reason text` · `created_at` · `updated_at` · `deleted_at`

**`task_assignments`** (assignment history; also the source of `assigned_in_period` in §5.5)
`id uuid PK` · `task_id` → tasks · `user_id` → profiles · `assigned_by` → profiles` · `assigned_at` · `unassigned_at` (nullable) · `is_current bool`
Partial unique index: one `is_current = true` row per task.

**`task_checklist_items`**
`id uuid PK` · `task_id` · `title text` · `is_done bool` · `position int` · `created_by` · `completed_at` · `completed_by` · `deleted_at`

**`comments`**
`id uuid PK` · `task_id` · `author_id` · `body text` · `created_at` · `updated_at` · `edited_at` · `deleted_at`

**`attachments`**
`id uuid PK` · `task_id` · `uploader_id` · `storage_path text` · `file_name text` · `mime_type text` · `size_bytes bigint` · `kind text check in ('image','file')` · `created_at` · `deleted_at`

**`labels`** · `id` · `org_id` · `name` · `color_token` · `created_at`
**`task_labels`** · `task_id` · `label_id` (composite PK)

**`activity_events`** (append-only)
`id bigint PK` · `org_id` · `task_id` · `actor_id` → profiles` · `type text` · `field_name text` · `old_value jsonb` · `new_value jsonb` · `created_at`

**`settings`**
`id uuid PK` · `org_id` · `key text` · `value jsonb` · `updated_at` · `updated_by` · unique `(org_id, key)`

Default settings and their defaults:

| key | default | used by |
| --- | --- | --- |
| `stalled_days` | `3` | §5.4 |
| `active_window_hours` | `24` | §5.2 |
| `require_done_review` | `false` | §5.1 |
| `organization_timezone` | `[TIMEZONE]` | §5.3, §5.5 |
| `default_report_period` | `month` | §5.5 |
| `max_attachment_mb` | `10` | §4.7 |
| `allowed_attachment_types` | images, pdf, office docs, plain text | §4.7 |
| `default_due_days` | `3` | Inbox capture |

### 8.2 Relationships

```
organizations 1─* profiles
organizations 1─* teams 1─* team_members *─1 profiles
organizations 1─* statuses
organizations 1─* boards 1─* board_columns *─1 statuses
organizations 1─* tasks *─1 statuses        (current status)
                tasks *─1 board_columns     (board position, nullable)
                tasks *─0..1 profiles       (assignee_id, nullable = Inbox)
                tasks 1─* task_assignments *─1 profiles
                tasks 1─* task_checklist_items
                tasks 1─* comments *─1 profiles (author)
                tasks 1─* attachments *─1 profiles (uploader)
                tasks *─* labels
                tasks 1─* activity_events *─1 profiles (actor)
organizations 1─1 settings (key/value)
```

### 8.3 Views and functions

- `monitoring_task_state` — `task_id`, `is_active`, `is_overdue`, `hours_overdue`, `is_stalled`, `days_since_activity`, `is_blocked`, `is_done`. `security_invoker = true` so RLS still applies to the underlying rows. This view is the only place the §5 formulas are implemented, and it is the single source of truth for both roles.
- `employee_task_rollup` — per-employee open / active / overdue / stalled counts. Manager-only readable.
- `employee_period_rollup(org, user, period_start, period_end)` — the §5.5 metrics. Manager-only readable.
- `next_due_at` trigger helper for the "due today means end of day in the org timezone" rule.
- Activity event writer: `SECURITY DEFINER` triggers on tasks, checklist items, comments, attachments, and labels. Clients **insert into these tables; the database writes the log**, so history cannot be forged or lost by a client bug.

### 8.4 Access rules per table (RLS summary — full policies written in Phase 1 migrations)

Helper functions, `SECURITY DEFINER STABLE`: `current_org_id()`, `current_role()`, `is_manager()`, `can_view_task(task_id)`.

| Table | SELECT | INSERT | UPDATE | DELETE |
| --- | --- | --- | --- | --- |
| `organizations` | same org | manager | manager | — |
| `profiles` | self, or any profile in org if manager | self (signup path) / manager (invite path) | self (limited fields) or manager | — |
| `teams`, `team_members` | manager (v1) | manager | manager | manager |
| `statuses` | same org | manager | manager | manager (if unused) |
| `boards`, `board_columns` | same org | manager | manager | manager |
| `tasks` | `is_manager() OR assignee_id = auth.uid()` | manager any; employee self-assign only | row: `is_manager() OR assignee_id = auth.uid()`; columns: see below | manager |
| `task_assignments` | if `can_view_task(task_id)` | manager | manager | manager |
| `task_checklist_items` | if `can_view_task(task_id)` | if `can_view_task` and (manager or assignee) | manager or assignee | manager or creator |
| `comments` | if `can_view_task(task_id)` | if `can_view_task` and (manager or assignee) | author (own) or manager | author (soft) or manager |
| `attachments` | if `can_view_task(task_id)` | if `can_view_task` and (manager or assignee) | — | uploader or manager |
| `labels`, `task_labels` | same org | manager | manager | manager |
| `activity_events` | if `can_view_task(task_id)` | **none** (triggers only, `SECURITY DEFINER`) | **none** | **none** |
| `settings` | same org (read) | manager | manager | — |

**Column-level grants on `tasks`** (Postgres column privileges, enforced independently of RLS and of any client code):

- Manager role: full `UPDATE` on the table.
- Employee: `UPDATE` only on `title, description, status_id, column_id, position, blocked_reason` — and RLS still restricts the *rows* to their own tasks. `assignee_id`, `due_at`, `priority`, `org_id`, `last_activity_at`, `done_at`, `started_at` are **not** granted, so an employee cannot silently reassign a task, move a deadline, or forge freshness.
- `last_activity_at`, `started_at`, `done_at` are maintained by triggers, never by clients.

Additional rules:

- **Reopen guard:** a `BEFORE UPDATE` trigger rejects a status change *out of* a done status for a non-manager, and rejects any employee transition into a status with `is_done = true` when `require_done_review` is on.
- **Blocked guard:** moving to a status with `is_blocked = true` requires a non-empty `blocked_reason`.
- **Storage:** private bucket `task-attachments`, path `{org_id}/{task_id}/{uuid}-{file_name}`; no public URLs; short-lived signed URLs; upload/size/type limits from `settings`; delete policy mirrors the attachment row's uploader-or-manager rule.
- **Deactivation, not deletion:** deactivating a profile keeps their task history intact (a manager must be able to see who did the work last quarter) and removes them from assignee pickers.

---

## 9. Removed and out of scope

### 9.1 Removed entirely (do not plan, design, or reintroduce)

- Social media scheduling and content planning of any kind
- Compose-post / publish modals
- Platform selectors (Facebook, Instagram, X/Twitter, LinkedIn, TikTok…)
- AI caption or copy generation
- Media library / asset library for posts
- Content-approval pipelines as a default template
- Social-specific labels, copy, tone-of-voice fields, hashtag fields, or audience fields

If a template resembling any of these appears in a later phase, it is a scope error.

### 9.2 Out of scope for the first release (explicitly)

- Automation rules ("when status becomes X, then…")
- Slack, Teams, email-digest, or any third-party integration
- Native mobile app (the web app must be responsive; that is not the same as a native app)
- Any invasive tracking of any kind (§6.1)
- Gantt / timeline view, workload heatmaps, multi-org support, custom role definitions, public share links, granular notification preferences, SSO/SAML, audit-log UI for admins

---

## 10. Design dependency

`Design.md` will be supplied after this planning pass, based on a UI/UX reference image. It does not exist yet.

**Binding rule:** every user-interface task in `plan.md` is tagged `[DESIGN]` and depends on `Design.md`. A phase may not be signed off with unresolved interface work; where `Design.md` is not yet available, the corresponding work is limited to data and logic layers, and the UI is built when the document lands.

Already decided and not to be re-opened: flat professional business look; no glassmorphism; no neumorphism; turquoise primary; light and dark themes; text readable on any background.

**Planned accessibility contrast helper** (specified here because it is a logic/utility task, not a visual one): a `contrastRatio(fg, bg)` utility implementing WCAG 2.1 relative luminance, with

- `contrastRatio` and `meetsContrast(ratio, size, weight)` helpers,
- an `A11yContrastBadge` component that renders a live swatch/foreground pair with its ratio and a pass/fail label,
- thresholds: **4.5:1** normal text, **3:1** large text (≥24px, or ≥18.66px bold) and UI component boundaries,
- applied to user-chosen label, priority, and status colours — when a user picks a colour, the system shows the ratio against the current theme's background and warns (or auto-darkens) below threshold,
- automated contrast assertions in the test suite, in both themes.

---

## 11. Open questions

Do not resolve these silently where the answer changes scope, cost, or what gets built. Each has a recommended default so the plan can move, but the default is a placeholder until the client or the user confirms.

| # | Question | Why it matters | Recommended default | Blocks |
| --- | --- | --- | --- | --- |
| **Q1** | Does "Done" require manager verification? | Adds a hidden approval queue that itself needs monitoring; changes the status model, the Dashboard, and the completion-rate denominator | **No.** Employee marks Done; manager can reopen. `require_done_review` setting exists, default off | Status model, Dashboard, Reports |
| **Q2** | What is the default stalled threshold N, and calendar days or working days? | The single most sensitive number in the product; too low = noise, too high = misses problems | **3 calendar days**, configurable 1–30. Working days need a holiday calendar the client has not supplied | §5.4, Dashboard, Team page |
| **Q3** | Can employees create their own tasks, or only submit to the Inbox for assignment? | Changes RLS on `tasks` insert, the completion-rate denominator, and whether an employee can be "overloaded by choice" | **Yes, self-assign only**; no assigning to others | RLS, Inbox, My Tasks, Reports |
| **Q4** | Can the manager also be an assignee of tasks? | Affects whether the manager appears in workload views and whether manager tasks count in reports | **Yes**, but the manager's own tasks are excluded from their own report by default | Reports, Team page, RLS |
| **Q5** | Can one employee belong to multiple teams, and is "team" even used in v1? | Determines whether `teams`/`team_members` are live or dormant, and whether RLS is org-scoped or team-scoped | **Single team in v1**; tables exist, no UI. Team-scoped access deferred | Data model, RLS, Team page |
| **Q6** | How are attachments stored and limited? (max size, allowed types, retention, private vs. signed URLs, image preview, virus scanning) | Affects storage cost, RLS on the storage bucket, and a retention obligation under §6.4 | Supabase private bucket, signed URLs, 10 MB, images + PDF + office docs, no scanning in v1, purge on task purge | Task detail, Phase 1 storage policies |
| **Q7** | Period default: calendar month, or the client's pay/billing cycle? | Changes the headline completion number the client will judge the product by | Month until §2.1 is filled in, then the pay cycle | Reports |
| **Q8** | Should a reopened task be removed from the historical period where it was counted as complete? | Affects whether last month's report changes after today | **Yes**, remove it and say so in the UI. Alternative: keep it and add a separate "reopened" column | Reports, CSV |
| **Q9** | What happens to a deactivated employee's open tasks? | Affects workload reports and whether work silently disappears | Reassignment prompt at deactivation; tasks stay assigned to the deactivated profile until reassigned; excluded from active counts, retained in history | Team page, Reports |
| **Q10** | Do due dates need time-of-day, or is end-of-day enough? | Affects the timezone rules in §5.3 and every overdue test | **End of day in the org timezone**; a true deadline time is deferred | Overdue formula, Schedule |
| **Q11** | Multiple managers — is one enough, and is there a manager-of-managers need? | Affects `role` modelling and any future permission split | **Yes, multiple managers, same permissions** | Roles, RLS |
| **Q12** | Are comment notifications required (in-app, email), or is checking the app enough? | Notifications are not in the goal; email is an integration and out of scope | **No notifications in v1**; in-app only. Revisit only if the client says the manager will not open the app | Inbox, Dashboard |
| **Q13** | Client, business, industry, team size, timezone, and pay cycle (§2.1) | Headcount and timezone affect period logic, the Team page, and the real-data testing checkpoint | Leave as placeholders | Reports, Phase 1 test fixtures, Phase 7 |
| **Q14** | Is employee notice/agreement a precondition of deployment? | A product that observes performance is deployed very differently from one that is silently introduced | **Yes.** Manager informs the team, privacy notice acknowledged at first login, Demo org for training | Phase 1, Phase 7 |

---

## 12. Decision log

| Date | Decision | Rationale |
| --- | --- | --- |
| 2026-09-28 | Social media planning removed; product is monitoring-only | Revised brief |
| 2026-09-28 | Supabase over Firebase | Per-row RLS, relational reports, atomic activity writes, cheap dashboard reads (§7.3) |
| 2026-09-28 | `@hello-pangea/dnd` for boards | React 19 support, built-in keyboard DnD, canonical Kanban model; isolated in an adapter (§7.2) |
| 2026-09-28 | All monitoring flags derived, never stored | Manager/employee can never disagree; nothing about a person is stored invisibly (§5.7) |
| 2026-09-28 | Activity log written by DB triggers, read-only to clients | History cannot be forged or lost (§8.3) |
| 2026-09-28 | Employee/manager field split via Postgres column grants, not just RLS | RLS cannot restrict columns; the assignment/due-date/priority boundary must be real (§8.4) |
| 2026-09-28 | Soft delete + profile deactivation instead of hard delete | Preserves history a manager needs; supports retention obligations (§6.4) |
| 2026-09-28 | Default Done = no manager verification | A verification queue is itself unmonitored work and biases the completion rate (Q1) |

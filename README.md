🗂️ **[PRODUCT NAME]** — Employee Task Monitoring System

> **Status: pre-implementation.** This repository currently holds planning documents only. Phase 0 has not started. The structure below is the agreed target layout.
> **The product name has not been chosen.** A working name may be used internally, but it must not be the name, wordmark, logo, or brand identity of any existing product.

---

## 📖 Project Overview

A small business owner needs to answer three questions without asking anyone:

1. **Who is working on what right now?**
2. **Which assigned tasks are actually finished?**
3. **Which tasks are overdue, or stuck and going nowhere?**

Today those answers come from chasing people, from memory, or from a chat log nobody can search. Work silently stalls: a task sits "in progress" for two weeks, nobody notices, and the deadline passes.

This system is a **task assignment and activity-monitoring system for a small team**. Its value is not that it manages tasks; it is that it produces a trustworthy, always-current answer to those three questions — and gives the manager one place to assign work.

### 🎯 The one-sentence product rule

> Every feature must answer one of: *who is working on what*, *which tasks are finished*, *which tasks are overdue or stalled*. A feature that answers none of these is cut or deferred.

### 🛡️ The surveillance boundary

This is a functional requirement, not a values statement.

Monitoring reads **task activity** — status changes, checklist ticks, comments, attachments, and edits. It never measures **the person**.

| Never implemented | Why |
| --- | --- |
| Screen capture, webcam, keystroke or mouse logging | Measures the person, not the work |
| GPS or location tracking | Same |
| Browser history, clipboard, network inspection | Same |
| App-usage or "idle" tracking, badge-in/out attendance | Same — a stall is inferred from the absence of *work*, not the absence of the *person* |
| Login-time tracking beyond session security | Same |

**Transparency:** an employee can see, about their own tasks, exactly what the manager sees — the same status, due date, priority, checklist, comments, attachments, activity log, and the same overdue, stalled, and active badges. Nothing about an employee is recorded anywhere the employee cannot read. Structurally, no monitoring flag is stored: every flag is derived at query time from task rows, so there is no table in which a fact about a person can hide.

**Least privilege:** enforced in the database with row-level security and column-level grants, not in the interface. A permission that exists only in the UI does not count as implemented.

**Data privacy:** the system stores employee names, contact details, task text, comments, and uploaded files — that is personal data about identified individuals. Before launch the client must be informed of the privacy obligations that apply to their business and location. If the business operates in the **Philippines**, this includes the **Data Privacy Act of 2012 (RA 10173)** and the implementing rules and issuances of the National Privacy Commission — lawful processing, a **privacy notice and consent**, **retention and disposal** of records, **security safeguards** for personal data in the system's custody, and the **data subject's rights** (access, correction, objection, erasure). Other jurisdictions have their own equivalents. *This is a flag for the client, not legal advice — they should obtain advice appropriate to their situation.*

### 👥 Roles

| Role | Can |
| --- | --- |
| **Manager** | Create and assign tasks, see every employee's tasks and progress, reassign, reopen completed work, manage boards/statuses/settings/people, view reports, export |
| **Employee** | See only their own tasks, update status, comment, attach files as proof of work, create tasks assigned to themselves |

### 🏢 Client details

Not yet supplied. These placeholders are deliberate and must be filled in with the client — do not invent them.

| Field | Value |
| --- | --- |
| Client name | `[CLIENT NAME]` |
| Business | `[BUSINESS NAME]` |
| Industry | `[INDUSTRY]` |
| Team size | `[NUMBER OF EMPLOYEES]` |
| Operating timezone | `[TIMEZONE — e.g. Asia/Manila]` |
| Working days | `[e.g. Mon–Fri]` |
| Billing / pay cycle | `[e.g. semi-monthly]` |

---

## 🔑 Key Features

- **Dashboard** *(manager)* — What needs attention right now: overdue, stalled, and blocked queues, each split so one loud problem does not mask the rest. Below that, a "who is working on what" strip with each employee's current task and time since last activity, team pulse counters, a completed-today feed, and a nudge for tasks captured but never assigned. Every number links into the filtered list behind it.
- **Team** *(manager)* — One card per employee: assigned, in progress, done this period, overdue, and stalled counts, the current task, and how long since any activity. Answers "is this person actually working?" from task activity only — never presence, login times, or device data. No rankings, scores, or leaderboards.
- **Boards** — Kanban board where columns are task statuses, plus a sortable table view for scanning across tasks. Drag a card between columns to change status; the keyboard path and a screen-reader announcement are part of the definition of done, not polish. Cards show assignee, due date, priority, labels, checklist progress, comment and attachment counts, and stalled/overdue indicators. Filters for assignee, priority, label, due window, and "only stalled" are reflected in the URL so a filtered board can be linked.
- **My Tasks** *(employee)* — The employee's home, grouped by urgency: Overdue, Due today, In progress, Not started, Blocked, Done this week. One-tap Start, Block (reason required), and Mark done, with inline checklist, comments, and attachments. The same stalled/overdue badges the manager sees appear here.
- **Task detail** — Description, assignee, due date, status, priority, labels, checklist, comments, file and image attachments, and a human-readable activity log. The activity log is a transparency mechanism first: it is the same list the manager reads, and it reads as a factual history of work, not as a surveillance report.
- **Inbox** — Quick capture (`n` from anywhere) for tasks that exist before they are assigned. Captured tasks are excluded from every employee's workload and from report denominators, so an unassigned task never silently distorts a completion number. Unassigned tasks that go stale get a "never assigned" nudge on the Dashboard.
- **Schedule** — Month grid and agenda of due dates, with a manager per-employee filter and a pinned overdue backlog above the grid, because a month grid hides past dates. Due dates can be edited here (manager only); dragging cards between days is a documented later phase.
- **Reports** *(manager)* — Per-employee, per-period completion summaries with an on-time rate, overdue and stalled counts, a drill-down to the underlying tasks, and a CSV export built from the same access-controlled query as the screen, so the export can never contain something the manager could not see.

### 🚫 Deliberately not built

Removed on purpose — do not reintroduce, and do not treat a proposal for any of these as a small addition:

- Social media scheduling or content planning of any kind
- A compose-post / publish modal
- Platform selectors (Facebook, Instagram, X, LinkedIn, TikTok…)
- AI caption or copy generation
- A media library
- Content-approval pipelines as a default template
- Social-specific labels, copy, hashtag, or audience fields
- Employee scores, rankings, or leaderboards
- Time tracking, timesheets, or attendance
- Presence, idle/away tracking, screen capture, or location

Out of scope for the first release: automation rules, Slack/Teams/third-party integrations, native mobile apps, Gantt/timeline views, multiple boards, sub-teams, and custom fields.

---

## 🛠️ Tech Stack

### Frontend

| Layer | Technology |
| --- | --- |
| Framework | React 19 (SPA, Vite) |
| Language | TypeScript 5 (strict) |
| Build tool | Vite |
| Routing | React Router (data router) |
| Server state | TanStack Query v5 |
| Data validation | Zod |
| Styling | Tailwind CSS v4 + `@tailwindcss/vite` |
| Drag & drop | `@hello-pangea/dnd` 18 |
| Icons | Lucide React |
| Forms | React Hook Form |
| Linter | ESLint 9 (flat config) + Prettier |
| Unit / component tests | Vitest + Testing Library |
| End-to-end tests | Playwright (multi-context) |
| Commit hooks | commitlint + husky + lint-staged |

**Why `@hello-pangea/dnd`:** it is the maintained community fork of `react-beautiful-dnd` (v18.0.1, Apache-2.0) and declares support for React 18 and 19, so the project stays on current React. It has a real keyboard-accessible drag mode with screen-reader announcements built in, which this product requires. It is a list/board library, not a calendar library, so the Schedule uses a small purpose-built keyboard interaction instead — and every import is confined to `src/lib/dnd/` so a future swap touches one file.

**Why a Vite SPA rather than a server-rendered framework:** this is a private, authenticated, highly interactive board with no SEO requirement. Server rendering buys nothing here, and a client-side-only data-access module keeps the RLS session context in exactly one place. Any service-role work that genuinely needs a server (large CSV ranges, invitations) becomes a small Vercel function.

### Backend

| Layer | Technology |
| --- | --- |
| Platform | Supabase |
| Database | PostgreSQL (with Row Level Security) |
| Auth | Supabase Auth (email + password, invite-only, no public sign-up) |
| Authorization | RLS policies + Postgres column-level grants |
| File storage | Supabase Storage, private bucket, short-lived signed URLs |
| Derived metrics | SQL views and functions (`monitoring_task_state`, `employee_period_rollup`) |

**Why Supabase over a document database:** three properties this product cannot work without.

1. **Row-level security is the actual requirement.** "Employees see their own, managers see all" is a per-row predicate. Postgres expresses it in the database, so it holds no matter which client code, query, or future integration touches the data.
2. **The monitoring numbers must be identical for both roles.** `is_overdue`, `is_stalled`, the completion rate, and workload are computed once in a single SQL view read by both the manager and the employee UI. In a document store the same guarantee requires duplicating that logic in application code — which is exactly where the two views would drift apart and the manager and employee would see different numbers for the same task.
3. **Activity writes must be atomic.** A status change updates the task, stamps `last_activity_at`, sets or clears `done_at`, and inserts an activity event, or none of it happens. If the activity log can be lost, the product's core signal is unreliable. That is one transaction in Postgres.

**Why column-level grants in addition to RLS:** RLS can restrict *rows* but not *columns*. The guarantee that an employee cannot change a task's assignee, due date, priority, or freshness timestamp is a Postgres column privilege, enforced independently of any client code. Employee `UPDATE` on `tasks` is granted only on `title, description, status_id, column_id, position, blocked_reason`.

**Why the activity log is write-protected:** all `activity_events` rows are written by `SECURITY DEFINER` triggers, and clients have no INSERT, UPDATE, or DELETE policy on that table. History cannot be forged by a buggy or malicious client, and clients cannot hand-set `last_activity_at` to keep a stalled task looking fresh.

**No localStorage.** All application state lives in the shared database. A manager cannot monitor employees if each person's data lives in their own browser.

### Tooling

| Layer | Technology |
| --- | --- |
| Commit message linting | `@commitlint/cli` + `@commitlint/config-conventional` |
| Git hooks | husky (`commit-msg`, `pre-commit`) |
| Changelog / version | `commit-and-tag-version` (Conventional Commits → SemVer) |
| Database migrations | Supabase CLI (`supabase/migrations/*.sql`) |

---

## 📁 Project Structure

```
monitoring-system/
├── AGENTS.md                       # Binding rules for agents and contributors
├── README.md
├── Design.md                       # visual + interaction source of truth
├── project-context.md              # Problem, roles, monitoring formulas, data model
├── plan.md                         # Goal-to-feature matrix, phases, acceptance scenarios
├── package.json
├── .env.example                    # VITE_DATA_SOURCE, VITE_SUPABASE_URL, VITE_SUPABASE_ANON_KEY
├── .commitlintrc.json
├── eslint.config.js
├── .prettierrc
├── .github/
│   └── workflows/ci.yml            # typecheck, lint, unit tests, commit-lint on PR range
│
├── supabase/
│   ├── migrations/                 # 0001_schema … 0007_seed
│   ├── seed.sql                    # Demo org: 1 manager, 2 employees, known dates
│   └── tests/                      # RLS matrix assertions (one per permission row)
│
├── src/
│   ├── main.tsx
│   ├── App.tsx
│   ├── routes/                     # Route table + guards (requireAuth, requireManager)
│   │   ├── paths.ts
│   │   └── guards.tsx
│   ├── components/
│   │   ├── ui/                     # Button, Dialog, Menu, Select, Toast, Table, Tooltip…
│   │   ├── a11y/                   # FocusRing, LiveRegion, SkipLink, ContrastBadge
│   │   └── layout/                 # AppShell, Sidebar, Topbar, PageHeader
│   ├── features/
│   │   ├── auth/                   # SignIn, AcceptInvite, ResetPassword, PrivacyNotice
│   │   ├── dashboard/              # Manager home: attention queues, who-is-working strip
│   │   ├── team/                   # Employee cards, employee task list
│   │   ├── tasks/                  # Task detail, editor, checklist, comments, attachments, activity
│   │   ├── boards/                 # Board view, Table view, columns, cards, filters
│   │   ├── my-tasks/               # Employee home
│   │   ├── inbox/                  # Quick capture + manager triage
│   │   ├── schedule/               # Month grid, agenda list, due-date editing
│   │   ├── reports/                # Period selector, rollup table, CSV export
│   │   ├── settings/               # People, statuses, boards, general (stalled threshold)
│   │   └── dev/                    # /dev/contrast — WCAG contrast test page
│   ├── hooks/                      # useTasks, useTask, useSession, useMediaQuery
│   ├── lib/
│   │   ├── data/                   # ← THE ONLY PLACE THE BACKEND IS CALLED
│   │   │   ├── index.ts            # Public API surface for the app
│   │   │   ├── types.ts            # Domain types + Zod schemas
│   │   │   ├── repositories/       # tasks, users, boards, statuses, comments,
│   │   │   │                      # attachments, activity, reports, settings
│   │   │   └── adapters/
│   │   │       ├── supabase/       # Real implementation
│   │   │       └── mock/           # In-memory. Interface prototyping only.
│   │   ├── monitoring/             # Pure TS mirror of the SQL formulas (optimistic + unit tests)
│   │   ├── supabase/               # Client singleton; server-only admin client
│   │   ├── dnd/                    # The ONLY @hello-pangea/dnd imports live here
│   │   ├── a11y/                   # contrast.ts — WCAG relative luminance, meetsContrast
│   │   └── utils/                  # dates, csv, formatting
│   └── styles/
│       ├── index.css               # Tailwind v4 @theme tokens, light/dark
│       └── tokens.ts
│
├── e2e/                            # Playwright: acceptance scenarios, 3 accounts
└── tests/                          # Vitest: unit + component
```

### Rules this structure encodes

1. `src/lib/data/` is the only place `@supabase/supabase-js` is imported from application code. Enforced by an ESLint `no-restricted-imports` rule. Components import from `src/lib/data`.
2. `src/lib/dnd/` is the only place `@hello-pangea/dnd` is imported from.
3. `src/lib/monitoring/` mirrors the SQL formulas for optimistic rendering and unit tests. **When the two disagree, SQL wins** — the TypeScript copy is never an independent source of truth.
4. The mock adapter may be used only for early interface prototyping. **Every monitoring feature is incomplete until the Supabase adapter replaces it**, and a build-time guard prevents monitoring routes from running on mock data. "The dashboard works" against the mock does not satisfy any checkpoint.

### Routes

```
/sign-in   /invite/:token/accept   /reset-password   /privacy-notice
/                        → manager: /dashboard      employee: /my-tasks
/dashboard   /team   /team/:userId/tasks   /board   /inbox   /schedule
/reports    /reports/export   /settings/{people,statuses,boards,general}
/tasks/:taskId      ← ONE shared route for both roles; permission comes from RLS
/my-tasks           /my-tasks/:taskId        /403   /404
/dev/contrast       (dev/staging only)
```

---

## ⚙️ Prerequisites

- **Node.js 20 or newer**
- **npm** 10+
- **Git** 2.40+
- A **Supabase project** (local via the Supabase CLI, or a hosted project) with the database password, `anon` key, and `service_role` key
- The **Supabase CLI** — <https://supabase.com/docs/guides/cli>
- A Vercel account (only for deployment)

---

## 🚀 Getting Started

1. **Clone the repository**

   ```bash
   git clone <repository-url>
   cd monitoring-system
   ```

2. **Install dependencies**

   ```bash
   npm install
   ```

3. **Link the Supabase project**

   ```bash
   npx supabase login
   npx supabase link --project-ref <project-ref>
   ```

   For a fully local database instead: `npx supabase start`.

4. **Create the environment file**

   ```bash
   cp .env.example .env.local
   ```

   Fill in:

   | Variable | Where to get it | Notes |
   | --- | --- | --- |
   | `VITE_SUPABASE_URL` | Supabase → Project Settings → API | Safe to expose in the browser |
   | `VITE_SUPABASE_ANON_KEY` | Supabase → Project Settings → API | Safe to expose; RLS is what protects the data |
   | `VITE_DATA_SOURCE` | — | `supabase` normally; `mock` for interface prototyping only |

   > The **service-role key must never be placed in a `VITE_`-prefixed variable or committed.** `VITE_*` values are bundled into the client and are public. Service-role use is confined to server-only Vercel functions.

5. **Apply migrations and seed the demo data**

   ```bash
   npm run db:migrate
   npm run db:seed
   ```

   The seed creates a demo organization with one manager and two employees, plus tasks engineered to hit every monitoring formula: due in 2 days, due today, due yesterday; last activity 1 hour / 2 days / 4 days / never; all four statuses; one unassigned inbox item five days old; one blocked-and-stalled; one reopened; one with a checklist, comments, and an attachment.

6. **Start the dev server**

   ```bash
   npm run dev
   ```

   The app runs at <http://localhost:5173>. Sign in at `/sign-in` with a seeded account. A manager lands on `/dashboard`; an employee lands on `/my-tasks`.

   The **demo organization contains fictional people and synthetic data only.** Real employee data does not enter the system until the team has been told what it records and has acknowledged the privacy notice.

7. **Deploy**

   ```bash
   npx vercel
   ```

   Set `VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY`, and `VITE_DATA_SOURCE=supabase` in the Vercel project. Preview deployments are created per branch; never seed preview environments with production data.

---

## 📜 Available Scripts

| Command | Description |
| --- | --- |
| `npm run dev` | Start the Vite dev server with HMR |
| `npm run build` | Type-check and build for production |
| `npm run preview` | Serve the production build locally |
| `npm run typecheck` | `tsc --noEmit` |
| `npm run lint` | Run ESLint |
| `npm run lint:fix` | Run ESLint with autofix |
| `npm run format` | Format with Prettier |
| `npm run test` | Unit and component tests (Vitest) |
| `npm run test:watch` | Vitest in watch mode |
| `npm run test:e2e` | End-to-end acceptance scenarios (Playwright) |
| `npm run test:rls` | Row-level-security matrix tests against the database |
| `npm run db:migrate` | Apply pending Supabase migrations |
| `npm run db:seed` | Seed the demo organization |
| `npm run db:reset` | Drop, re-migrate, and re-seed |
| `npm run db:types` | Regenerate TypeScript types from the database schema |
| `npm run verify` | `typecheck` + `lint` + `test` — run before pushing |

---

## 🌿 Git Branching Workflow

```
main
├── feat/<short-description>
├── fix/<short-description>
├── refactor/<short-description>
├── perf/<short-description>
├── test/<short-description>
├── docs/<short-description>
├── chore/<short-description>
└── hotfix/<short-description>       # urgent production fixes
```

| Type | Pattern | Example |
| --- | --- | --- |
| Feature | `feat/<short-description>` | `feat/board-keyboard-drag` |
| Bug fix | `fix/<short-description>` | `fix/overdue-card-styling` |
| Refactor | `refactor/<short-description>` | `refactor/split-data-repositories` |
| Performance | `perf/<short-description>` | `perf/rollup-index` |
| Test | `test/<short-description>` | `test/rls-matrix` |
| Documentation | `docs/<short-description>` | `docs/phase-4-checkpoint` |
| Chore | `chore/<short-description>` | `chore/update-deps` |
| Hotfix | `hotfix/<short-description>` | `hotfix/stalled-flag-guard` |

**Branch prefixes always match the commit type**, so branch and commit history read consistently.

### Starting a new feature

```bash
# 1. Make sure main is up to date
git checkout main
git pull origin main

# 2. Create your branch
git checkout -b feat/board-keyboard-drag

# 3. Work on your changes, then commit
git add .
git commit -m "feat(boards): add keyboard drag between status columns"

# 4. Push
git push origin feat/board-keyboard-drag
```

### Merging back to main

Open a pull request (`feat/... → main`), wait for CI, and request a code review. Squash-merge is acceptable — the PR title becomes the commit message, so keep it Conventional.

---

## 📝 Commit Message Convention

This project follows **[Conventional Commits v1.0.0](https://www.conventionalcommits.org/en/v1.0.0/)**.

```
<type>[optional scope][!]: <description>

[optional body]

[optional footer(s)]
```

| Element | Rule |
| --- | --- |
| `<type>` | Required. A noun, from the table below |
| `<scope>` | Optional. A noun in parentheses describing a section of the codebase |
| `!` | Optional, immediately before the colon. Marks a breaking change |
| `<description>` | Required. Short summary, immediately after the colon and space |
| `<body>` | Optional, starts one blank line after the description. Explains **why**, not what |
| `<footer>` | Optional, one blank line after the body. Git-trailer style |

### Types

| Type | Description | SemVer effect |
| --- | --- | --- |
| `feat` | New feature | MINOR |
| `fix` | Bug fix | PATCH |
| `perf` | Performance improvement | PATCH |
| `refactor` | Code restructure, no behaviour change | — |
| `test` | Adding or updating tests | — |
| `docs` | Documentation updates | — |
| `style` | Formatting only, no logic change | — |
| `chore` | Dependencies, configuration, tooling | — |
| `ci` | CI pipeline and Vercel configuration | — |
| `build` | Build system configuration | — |
| `revert` | Reverts a previous commit | — |
| `BREAKING CHANGE` | Footer, or `!` on any type | MAJOR |

### Scopes

Use the feature area: `auth`, `data`, `tasks`, `boards`, `dashboard`, `team`, `my-tasks`, `inbox`, `schedule`, `reports`, `settings`, `dnd`, `storage`, `a11y`, `design-system`, `db`.

### Writing the message

- Description: imperative mood ("add", not "added" or "adds"), lowercase, **no trailing period**, 72 characters or fewer.
- Body: free-form. Write the reasoning a reviewer would otherwise have to ask for.
- Footers use a token with `-` instead of whitespace: `Refs: #123`, `Reviewed-by: Name`, `Co-authored-by: Name`.
- A breaking change is marked **either** with `!` after the type/scope **or** with a `BREAKING CHANGE:` footer. In this project that means a schema migration or a change to the data-access interface.
- `BREAKING CHANGE` must be uppercase; type, scope, and description are case-insensitive but must be consistent.

### Examples

```bash
git commit -m "feat(boards): add keyboard drag between status columns"
git commit -m "fix(attachments): reject uploads over the configured size limit"
git commit -m "chore: update react to v19"
git commit -m "docs: add phase 4 testing checkpoint"
git commit -m "test(rls): cover column grants on tasks"
```

With a body and a breaking-change footer:

```
feat(data)!: return NotFound instead of Forbidden for unauthorized tasks

Returning 403 to a non-manager confirmed that a task id exists, which
leaked the existence of another employee's work. The manager still gets
a real 403; everyone else gets 404.

BREAKING CHANGE: `tasks.get` resolves to NotFound for any caller who is
not permitted to read the task, regardless of role.
```

Revert:

```bash
git commit -m "revert: drop the redundant stalled recompute in the board query

Refs: 676104e, a215868"
```

### Enforcement

Conventional Commits is enforced by tooling, not discipline:

- `@commitlint/config-conventional` runs in a husky `commit-msg` hook.
- lint-staged runs ESLint and Prettier on staged files in a `pre-commit` hook.
- CI re-lints the commit range of every pull request.

A convention nobody can fail is not a convention. Do not use `--no-verify` to get around it; fix the message.

---

## 🤝 Contributing

1. Branch off `main` using the naming above.
2. Follow the commit message convention.
3. Open a pull request into `main` and request a code review before merging.
4. **Accessibility is part of done, not polish** — no phase closes with an unresolved keyboard-navigation, focus-visibility, or contrast failure.
5. **Interface work follows `Design.md`.** Do not invent visual specifications. A task the document does not cover is a gap to raise, not one to guess at.
6. **Every feature must map to a goal element** in the matrix in `plan.md`. If it maps to none, it is cut or deferred.

More detail for agents and contributors lives in [`AGENTS.md`](./AGENTS.md).

---

## 📚 Further Documentation

| Document | Contents |
| --- | --- |
| [`project-context.md`](./project-context.md) | Problem statement, roles and permissions matrix, the eight core areas, the normative monitoring formulas, the data model, access rules, and open questions |
| [`plan.md`](./plan.md) | Goal-to-feature matrix, folder structure, phased build order, goal acceptance scenarios, risks, and deferrals |
| [`AGENTS.md`](./AGENTS.md) | Binding engineering rules and conventions |
| [`Design.md`](./Design.md) | Colour tokens for both themes, typography, spacing, the manager Dashboard specification, component anatomy, motion, and accessibility requirements |

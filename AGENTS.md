# AGENTS.md

Binding rules for anyone — human or agent — working in this repository. If a rule here conflicts with a habit or a "quick win", the rule wins.

**Project:** Employee Task Monitoring System (`[PRODUCT NAME]` — not yet chosen)
**Status:** pre-implementation. Planning documents only; Phase 0 has not started.
**Read first:** `project-context.md` (what and why), `plan.md` (order and scope).

---

## 1. What this product is

A manager assigns tasks to employees and sees, without asking anyone:

1. **Who is working on what**
2. **Which tasks are finished**
3. **Which tasks are overdue or stalled**

That is the entire product. Every feature is judged against it.

---

## 2. The four binding rules

### 2.1 One data-access boundary

**Components never call the backend directly.** All database access goes through `src/lib/data/`.

- Application code imports from `src/lib/data`. An ESLint `no-restricted-imports` rule bans `@supabase/supabase-js` outside `src/lib/` and bans `src/lib/data` from being bypassed. Do not weaken this rule.
- The adapter is selected by `VITE_DATA_SOURCE` (`supabase` | `mock`).
- **The mock adapter is for early interface prototyping only. Every monitoring feature is incomplete until the Supabase adapter replaces it.** A monitoring checkpoint is never satisfied against mock data, and a build-time guard prevents monitoring routes from running on it. The mock adapter will not exist in the deployed app.
- Mock mode shows a persistent dev-only banner. It is not a demo mode and not a fallback.
- `src/lib/monitoring/` holds a TypeScript mirror of the SQL formulas for optimistic rendering and unit tests. **When the TypeScript and SQL versions disagree, SQL wins.** The TS copy is never an independent source of truth.
- The service-role key is server-only. It must never appear in a `VITE_`-prefixed variable, never be committed, and never reach the client bundle.

### 2.2 Monitoring is not surveillance

Monitoring reads **task activity** — status changes, checklist ticks, comments, attachments, and edits. It never measures **the person**.

**Never implement, in any form:** screen or webcam capture, keystroke or mouse logging, clipboard monitoring, GPS or location tracking, browser history, network inspection, app-usage or "idle"/away tracking, badge-in/out attendance, or login-time tracking beyond session security.

A stall means *no work happened on this task*. It does not mean *this person was not present*.

### 2.3 Transparency and least privilege

- Every employee can see, about their own tasks, exactly what the manager sees — including the overdue, stalled, and active badges, and their own completion numbers.
- Never create a record about a person that the person cannot read. This is enforced structurally: **all monitoring flags are derived at query time from task rows, never stored.** Do not add a table or column that stores "this employee is idle" or similar.
- Authorization is enforced in the **database** — RLS policies plus Postgres column-level grants — never in the interface alone. A permission that exists only in the UI does not count as implemented.
- RLS restricts rows, not columns. Field boundaries for employees (notably `assignee_id`, `due_at`, `priority`, `last_activity_at`, `done_at`, `started_at` on `tasks`) are enforced with **column-level `GRANT`**, not with application code.
- `activity_events` has no client INSERT, UPDATE, or DELETE policy. Triggers write it. Do not add one, and do not let a client set `last_activity_at`.
- When a non-manager requests a resource they may not read, the API returns **404, not 403** — a 403 confirms the resource exists and leaks. Managers still get a real 403.
- One automated test per row of the permissions matrix in `project-context.md` §3.2, run against Postgres. A new policy without a test is unfinished.

### 2.4 Accessibility is done, not polish

- Keyboard navigation and a visible focus state are part of the definition of done for every interactive element, in every phase.
- No interface task is signed off with an unresolved keyboard, focus, or contrast failure.
- Status is never conveyed by colour alone.
- Targets: **4.5:1** for normal text, **3:1** for large text (≥24px, or ≥18.66px bold) and UI component boundaries, in **both** light and dark themes.
- User-chosen colours (labels, priorities, statuses) are checked with the WCAG contrast helper in `src/lib/a11y/contrast.ts`, and the UI shows the ratio and a pass/fail state. `/dev/contrast` is the test page.
- Drag and drop always has a keyboard equivalent. The board's lift/move/drop/cancel path and its screen-reader announcements are required, not optional.

---

## 3. Visual work depends on `Design.md`

`Design.md` exists and is the visual and interaction source of truth: tokens, typography, spacing, the Dashboard specification, component anatomy, motion, and accessibility. Its colour ratios were verified with the WCAG 2 formula.

- **Do not invent visual specifications.** No colours beyond those in `Design.md`, no spacing values, no component styling guesses, no layout decisions.
- Every interface task in `plan.md` is tagged `[DESIGN]` and is built against it. **A task the document does not cover is a gap to raise, not licence to guess** — say so instead of inventing a value.
- Use the semantic tokens (`bg-surface`, `text-ink`, `border-strong`, …). Because they are theme-aware, components write no `dark:` variants. There is no `tailwind.config.js`; tokens live in `src/styles/index.css` behind **`@theme inline`**, without which dark mode silently renders light.
- **Already decided and not to be re-opened:** flat, professional, business look; **no glassmorphism; no neumorphism**; **turquoise primary**; light and dark themes; text readable on any background.

**Originality:** boards, lists, cards, and drag-and-drop are established patterns and are fine to use. Do not copy any other product's name, logo, wordmark, icon set, illustration, illustration style, marketing copy, or brand colour palette. This codebase is a **fresh build** — do not reintroduce a port of another product, its CSS, its wrapper class names, or its component names. Icons come from Lucide with its licence recorded.

---

## 4. Scope discipline

**Cut — do not reintroduce, in any form:** social media scheduling or content planning, compose-post/publish modals, platform selectors, AI caption or copy generation, a media library, content-approval pipelines as a default template, social-specific labels/copy/hashtag/audience fields, employee scores, rankings, or leaderboards, time tracking, timesheets, attendance, Gantt/timeline views, multiple boards, sub-teams, and custom fields.

**Out of scope for the first release:** automation rules, Slack/Teams/third-party integrations, native mobile apps, notifications, public share links, SSO/SAML, granular notification preferences.

**A new feature proposal must name the goal element it serves.** The matrix in `plan.md` §1.2 is the acceptance test. A feature that maps to none is cut or deferred.

**Do not resolve ambiguities silently** when the answer changes scope, cost, or behaviour. The open questions live in `project-context.md` §11. Recommended defaults exist so work can proceed — but a default is a placeholder, not a decision.

---

## 5. Phase discipline

- Work in phase order. Each phase is **usable end to end — interface, backend, and permissions** — before the next begins.
- A phase is not done because a screen renders. It is done when a manager and two employees can use it with real RLS.
- **Every monitoring checkpoint runs against the Supabase adapter.** The mock adapter never satisfies a checkpoint.
- **Multi-user checkpoints require one manager account and two employee accounts on separate browsers or devices** (three separate browser profiles is the minimum acceptable substitute). Sharing one browser does not count.
- If a phase's checkpoint cannot be run, the phase is not done.
- Phases 0–4 are the product. If time is cut, ship through Phase 4 and cut cleanly — the goal is already met there.

---

## 6. Numbers must be reproducible

Every figure on the Dashboard and in Reports must be hand-checkable from the exported CSV, using the formulas written down in `project-context.md` §5.

- The SQL view is the only implementation of the monitoring formulas. Do not reimplement `is_overdue`, `is_stalled`, `is_active`, workload, or the completion rate in application code for a *correctness* decision.
- The manager and the employee read the same view, so the two can never show different numbers for the same task.
- Reports are recomputed live, never snapshotted. A reopened task leaves the period it was counted complete in, and the UI says so.
- Empty denominators render as "—", never "0%".

---

## 7. Commands

```bash
npm run dev          # dev server
npm run typecheck    # tsc --noEmit
npm run lint         # ESLint
npm run test         # unit + component
npm run test:e2e     # Playwright acceptance scenarios
npm run test:rls     # RLS matrix against the database
npm run verify       # typecheck + lint + test — run before pushing
```

`typecheck`, `lint`, and `test` must pass before any phase checkpoint is claimed. Do not commit generated output, `.env*` files, or `node_modules`.

---

## 8. Commits and branches

**[Conventional Commits v1.0.0](https://www.conventionalcommits.org/en/v1.0.0/)** is mandatory, enforced by commitlint in a husky `commit-msg` hook and re-linted in CI on every pull request. Full details in `README.md` §"Commit Message Convention".

```
<type>[optional scope][!]: <description>

[optional body]

[optional footer(s)]
```

- Types: `feat` `fix` `perf` `refactor` `test` `docs` `style` `chore` `ci` `build` `revert`
- Scopes: `auth` `data` `tasks` `boards` `dashboard` `team` `my-tasks` `inbox` `schedule` `reports` `settings` `dnd` `storage` `a11y` `design-system` `db`
- Description: imperative, lowercase, no trailing period, ≤ 72 characters. Body explains **why**.
- Breaking change: `!` before the colon **or** a `BREAKING CHANGE:` footer. Here that means a migration or a data-access interface change.
- Branches: `feat/ fix/ refactor/ perf/ test/ docs/ chore/ hotfix/` — **the branch prefix always matches the commit type.**
- Do not use `--no-verify` to bypass commitlint. Fix the message.

```bash
git commit -m "feat(boards): add keyboard drag between status columns"
git commit -m "fix(attachments): reject uploads over the configured size limit"
git commit -m "test(rls): cover column grants on tasks"
git commit -m "docs: add phase 4 testing checkpoint"
```

---

## 9. Where things live

| Need | Path |
| --- | --- |
| Backend calls | `src/lib/data/` only |
| SQL formulas (source of truth) | `supabase/migrations/0005_views.sql` → `monitoring_task_state` |
| Database schema + policies | `supabase/migrations/` |
| Drag-and-drop imports | `src/lib/dnd/` only |
| WCAG contrast helper | `src/lib/a11y/contrast.ts` |
| Theme tokens (Tailwind v4 `@theme`) | `src/styles/index.css`, `src/styles/tokens.ts` |
| Route table and guards | `src/routes/` |
| Feature UI | `src/features/<area>/` |
| Demo fixture data | `supabase/seed.sql` |

**Naming:** the product is not named yet. Do not use another product's name anywhere in code, comments, class names, or package metadata, and do not reintroduce a third-party project's naming.

---

## 10. Quick self-check before calling anything done

- [ ] Does this feature map to a goal element in `plan.md` §1.2?
- [ ] Did I add a rule that the employee cannot see about themselves?
- [ ] Is every permission enforced in the database, with a test in the matrix suite?
- [ ] Is it fully keyboard-operable with a visible focus state, in both themes?
- [ ] Do the numbers match the SQL formulas, and are they reproducible from the CSV?
- [ ] Is any interface work matching `Design.md` rather than a guess, and is any new value recorded there instead of invented here?
- [ ] Did I run `npm run verify`?
- [ ] Is the commit message Conventional Commits, and the branch prefix matching?

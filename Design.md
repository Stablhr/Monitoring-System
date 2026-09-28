# Design System: Employee Task Monitoring App

Working name: TBD. This document is the visual and interaction source of truth for the app described in `project-context.md` and `plan.md`.

**How the reference image is used.** The manager dashboard reference shows a good *structure* for this kind of screen: stat cards on top, an employee overview table, a progress summary, and a recent activity feed. We use it as a guide for what information appears where. We do **not** copy its look. Colors, icons, shadows, avatars, branding, and sample data are our own (see section 2).

---

## 1. Design Principles

1. **Answer the manager's question first.** The dashboard exists to show who is working on what, what is finished, and what is late or stalled. The most important information gets the most visual weight.
2. **Flat and professional.** Crisp 1px borders, minimal shadow, opaque surfaces. No glassmorphism, no neumorphism, no gradients used as decoration.
3. **Never color alone.** Every status has an icon and a text label. Color supports meaning, it never carries it by itself.
4. **Every number is real.** Nothing on screen is invented or rounded into something the data does not say (see section 11).
5. **Calm, readable, fast to scan.** Generous spacing, a small number of type sizes, and consistent alignment so a manager can check the team in seconds.
6. **Transparent to employees.** Employees see the same information about their own tasks that the manager sees. The interface never hides what is being recorded.

---

## 2. What We Take From the Reference and What We Do Not

| Reference element | What we take | What we change |
|---|---|---|
| Sidebar nav | The idea of a persistent left nav with a few clear destinations | Our own labels and icons, role-based items, light surface instead of a dark navy panel, no tagline or illustration |
| Greeting header + date pill + user menu | A friendly header with a period selector and the current user | The date pill becomes a **period selector** (This week, This month, Custom) that controls the stats and progress summary |
| Four stat cards | A row of headline numbers with a short context line | Different card set (section 7.3), small monochrome icons instead of large colored circles, trend shown as text |
| Employee Overview table | The core idea: one row per employee with current task, progress, status, and last active | Attention-first sorting, a stalled/overdue treatment, progress bars only when computable, initials avatars |
| Task Progress donut | A summary chart with a legend and counts | Segments are mutually exclusive stored statuses only; legend is the accessible source of truth |
| Recent activity feed | An icon, a sentence, and a relative time per event | Each item links to its task; icons plus text labels |
| Quick Actions panel | Fast access to common actions | Reduced to actions the sidebar does not already offer (section 7.7) |
| Photo avatars, name, tagline, logo, sample data | Nothing | Initials avatars, our own name and logo, our own copy |
| Navy sidebar, saturated blue primary, soft drop shadows | Nothing | Turquoise system below, border-based cards |

---

## 3. Color

Two themes, light and dark, defined as CSS variables and switched with a `data-theme` attribute (default follows `prefers-color-scheme`). Every text and control pair below was checked with the WCAG 2 relative luminance formula. Ratios are listed so they can be re-verified.

### 3.1 Light theme

| Token | Hex | Use | Verified contrast |
|---|---|---|---|
| `--bg` | `#F3FBFA` | Page background | |
| `--surface` | `#FFFFFF` | Cards, table, inputs, modals | |
| `--surface-alt` | `#E4F4F2` | Sidebar, hover fill, table header | |
| `--border` | `#CDEBE7` | Decorative dividers and card outlines only | Not for control boundaries |
| `--border-strong` | `#668582` | Input, checkbox, and button-outline boundaries | 4.01:1 on surface, 3.81:1 on bg |
| `--ink` | `#132A29` | Primary text | 15.10:1 on surface |
| `--ink-muted` | `#4A6664` | Secondary text | 6.22:1 on surface, 5.49:1 on surface-alt |
| `--ink-faint` | `#557170` | Placeholders, timestamps | 5.27:1 on surface, 5.02:1 on bg, 4.65:1 on surface-alt |
| `--brand` | `#087570` | Primary buttons, links, active nav text, focus ring | White on brand 5.54:1; on surface 5.54:1 |
| `--brand-hover` | `#065F5B` | Primary button hover | White on it 7.51:1 |
| `--brand-light` | `#D2F3F0` | Selected row, active nav background | Brand text on it 4.70:1 |
| `--brand-vivid` | `#0DABA3` | **Decorative only**: logo mark, large illustrations | 2.85:1 on white, so never for text, meaningful icons, or control boundaries |

### 3.2 Dark theme

| Token | Hex | Use | Verified contrast |
|---|---|---|---|
| `--bg` | `#111B1A` | Page background | |
| `--surface` | `#1A2827` | Cards, table, inputs, modals | |
| `--surface-alt` | `#223432` | Sidebar, hover fill, table header | |
| `--border` | `#2C403D` | Decorative dividers and card outlines only | |
| `--border-strong` | `#5F7C78` | Control boundaries | 3.37:1 on surface, 3.88:1 on bg |
| `--ink` | `#E6F2F0` | Primary text | 13.31:1 on surface, 11.41:1 on surface-alt |
| `--ink-muted` | `#9DB7B3` | Secondary text | 7.16:1 on surface, 6.14:1 on surface-alt |
| `--ink-faint` | `#8AA29E` | Placeholders, timestamps | 5.62:1 on surface, 4.82:1 on surface-alt |
| `--brand` | `#2DC9C0` | Primary buttons, links, active nav, focus ring | 7.43:1 on surface, 8.55:1 on bg |
| `--on-brand` | `#062321` | Text on a brand-filled button | 8.05:1 on brand |
| `--brand-hover` | `#5AD9D1` | Primary button hover | Dark text on it 9.67:1 |
| `--brand-light` | `#173C3A` | Selected row, active nav background | Brand text on it 5.86:1 |

### 3.3 Status system

Task status is one of four **stored** values. Overdue and stalled are **derived flags** (definitions in `project-context.md`), so a task can be In progress and Stalled at the same time.

| Status or flag | Meaning | Icon (line style) | Light text / background | Dark text / background | Chart color (light / dark) |
|---|---|---|---|---|---|
| Not started | Assigned, no work yet | Dashed circle | `#4A5A59` / `#ECF0F0` (6.31:1) | `#B5C7C4` / `#26332F` (7.47:1) | `#6B7F7C` / `#8FA5A2` |
| In progress | Being worked on | Half-filled circle | `#1F5FBF` / `#E4EEFC` (5.20:1) | `#8DB8FF` / `#1B2E4D` (6.75:1) | `#1F5FBF` / `#8DB8FF` |
| Blocked | Cannot continue | Pause or ban | `#8A5A00` / `#FDF1D6` (5.29:1) | `#F2C55C` / `#3A2E10` (8.18:1) | `#B7791F` / `#F2C55C` |
| Done | Finished | Check in circle | `#1B7A4B` / `#E3F5EC` (4.71:1) | `#6FD9A0` / `#17362A` (7.58:1) | `#1B7A4B` / `#6FD9A0` |
| Overdue (flag) | Past due and not Done | Clock with alert | `#B42335` / `#FDE6E8` (5.46:1) | `#FF8A96` / `#47202A` (6.19:1) | `#B42335` / `#FF8A96` |
| Stalled (flag) | No activity for N days | Snooze or hourglass | `#A8481F` / `#FFE9DF` (4.98:1) | `#FFA37A` / `#442A1E` (6.75:1) | `#C2571F` / `#FFA37A` |

Chart colors have at least 3:1 contrast against the surface in both themes.

### 3.4 Color rules

- **Brand turquoise means "action or navigation" only**: buttons, links, selection, focus. It never represents a task status. This keeps "in progress" (blue) and "done" (green) from being confused with the brand color.
- Status colors mean status only. Do not reuse them for decoration.
- Any user-chosen background (board colors, label colors) must run through the shared contrast helper that picks black or white text using the WCAG formula, with a fallback for mid-tones where neither reaches 4.5:1.
- Never hardcode `white` or `black` text next to a variable background.

---

## 4. Typography

| Family | Role |
|---|---|
| **Space Grotesk** | Page titles, card headings, large stat numbers |
| **Inter** | All UI text: table cells, labels, buttons, inputs, nav |
| **JetBrains Mono** | Timestamps, counts, IDs |

| Style | Size / weight |
|---|---|
| Page title | 24px, Space Grotesk 700 |
| Card heading | 16px, Space Grotesk 600 |
| Stat number | 32px, Space Grotesk 700 |
| Body and table text | 14px, Inter 400/500 |
| Labels and meta | 13px, Inter 500 |
| Mono meta (times, counts) | 12px, JetBrains Mono 400 |

Minimum text size is 12px. Line height 1.5 for body text, 1.2 for headings and numbers. Body text can be scaled to 200% without loss of content or function.

---

## 5. Spacing, Shape, Elevation

- **Spacing** uses a 4px base: 4, 8, 12, 16, 20, 24, 32.
- **Radius:** 6px for controls and pills, 8px for cards and table containers, 12px for modals.
- **Cards** have a `1px solid var(--border)` outline, an opaque `--surface` fill, and no shadow at rest. Clickable cards gain `--shadow-sm` on hover.
- **Elevation scale:**
  - `--shadow-sm`: `0 1px 2px rgba(19,42,41,0.05)`
  - `--shadow-md`: `0 2px 8px rgba(19,42,41,0.08)` for menus and popovers
  - `--shadow-lg`: `0 4px 16px rgba(19,42,41,0.12)` for modals and drawers
  In dark theme, use the same scale with `rgba(0,0,0,0.35)`, `0.45`, and `0.55`.
- No `backdrop-filter`, no translucent panels, no inset or dual-direction shadows.

---

## 6. Layout and Responsive Behavior

- **Sidebar:** 240px wide, `--surface-alt` background, full height. Collapses to a 72px icon rail between 768px and 1279px, and to a slide-in drawer below 768px.
- **Content area:** `--bg` background, padding 24px, maximum content width 1440px, 16px to 24px gaps between cards.
- **Breakpoints:** 1280px and up is the full two-column dashboard. 768px to 1279px stacks the right column under the table, in a two-up grid. Below 768px everything is a single column and table rows become stacked cards.
- **Desktop-first for managers, usable on a phone for employees.** The employee My Tasks screen must work at 360px width.
- **Navigation is role-based.**
  - Manager: Dashboard, Team, Boards, Inbox, Schedule, Reports, Settings.
  - Employee: My Tasks, Boards (own tasks only), Schedule, Inbox.

---

## 7. Manager Dashboard Specification

Order of regions on desktop, top to bottom, left to right:

### 7.1 Header
- Greeting based on time of day plus the user's name, with one line of supporting text (for example, "3 tasks need your attention").
- **Period selector** (This week default, This month, Custom range). It controls the stat cards and the progress summary. It does not change the "needs attention" list, which is always live.
- Primary button **Add task** (see 7.7). User menu at the far right with name and role.

### 7.2 Attention count
The supporting text in the header states how many tasks are overdue or stalled right now, linking to the Needs attention panel. If the count is zero, it says so plainly.

### 7.3 Stat cards (row of four)

| Card | Value | Context line |
|---|---|---|
| Completed | Tasks marked Done in the period | Change versus the previous period, as text ("+3 vs last week") |
| In progress | Tasks currently In progress | "Across N employees" |
| Overdue | Tasks past due and not Done | "Oldest: 5 days" |
| Stalled | Tasks with no activity for N days | "Threshold: N days" (links to settings) |

Deliberate deviation from the reference: **Total Employees** is not a headline card because it rarely changes and does not help the manager act. The employee count appears in the Employee Overview heading instead ("8 employees").

Card anatomy: 20px monochrome line icon in `--ink-muted`, label (13px), number (32px), context line (13px `--ink-muted`). Each card is a button that filters the Employee Overview to matching tasks. Trend text says what direction is good or bad in words, and never relies on an up or down arrow color alone (for Overdue, an increase is bad).

### 7.4 Needs attention panel (new, not in the reference)
Placed at the top of the right column. A list of overdue and stalled tasks, most severe first. Each item shows the task title, the employee, a reason chip ("Overdue 2 days" or "Stalled 4 days"), and buttons **Open** and **Reassign**. Empty state: "Nothing needs attention right now." This panel exists because the goal of the system is to surface problems without the manager having to hunt for them, and the reference dashboard has no stalled concept.

### 7.5 Employee Overview table (left, main area)

Columns: **Employee**, **Current task**, **Progress**, **Status**, **Last active**, and a row chevron.

- **Employee:** initials avatar (32px), name, role or title.
- **Current task:** the employee's most recently active In progress task, with its board name beneath. If they have more, show "+2 more" linking to their task list. If none, show "No active task".
- **Progress:** a bar plus a percent **only when it can be computed** (checklist items done divided by total). Otherwise show a dash and rely on the status pill. Never display an invented percent. The bar fill is `--brand`, the track is a neutral tint, and the percent is always shown as text.
- **Status:** the stored status pill (icon + label). If the task is overdue or stalled, a second flag pill appears beside it.
- **Last active:** relative time ("12 min ago") in mono text, with the exact date and time available on hover and to screen readers. Values of N days or more use the stalled color and an icon.
- **Row treatment for problems:** a 3px left border in the overdue or stalled color, plus the flag pill. The row background stays neutral.
- **Default sort:** needs attention first (overdue, then stalled, then oldest last-active), then alphabetical. Users can re-sort by any column.
- **Filter chips above the table:** All, Needs attention, In progress, Done.
- **Interaction:** the whole row is a focusable link to the employee's detail. Enter opens it. Row height 56px, minimum touch target 44px.
- **Semantics:** a real `<table>` with header cells and `scope`, not a grid of divs. On narrow screens each row becomes a stacked card with the same fields.

### 7.6 Task progress and recent activity (right column)

**Task progress (donut plus legend)**
- Segments are the four **stored** statuses only: Not started, In progress, Blocked, Done. They are mutually exclusive, so they always add up to the total. Overdue and stalled are shown as separate counts beneath the legend, not as segments.
- Center label: overall completion rate using the exact formula from `project-context.md` (Done divided by assigned, for the selected period), with the label "Completion".
- A 2px surface-colored gap separates segments. The legend lists each status with its icon, name, and count, and is the accessible version of the chart. Provide a "View as table" toggle.

**Recent activity**
- The latest 6 to 8 events from the activity log. Each shows a status icon, one sentence ("Maria marked Design banner as Done"), and a relative time. Each item links to its task. A "View all" link opens the full activity list.
- Updates without a full page reload (a 60-second refresh is acceptable for the first version).

### 7.7 Actions
- **Add task** is the single primary button, in the header.
- The reference's Quick Actions panel is dropped because View Reports and Settings duplicate the sidebar. If an action panel is kept, it holds only actions not in the sidebar, such as **Invite employee**.

### 7.8 States
- **Loading:** flat gray skeleton blocks matching each region's shape, no spinners over content.
- **Empty (no employees):** "Invite your first employee" with a primary button.
- **Empty (no tasks):** "Create the first task and assign it to someone."
- **Error:** an inline message in the affected card with a Retry button. One failing region does not blank the page.

---

## 8. Core Components

- **Button.** Primary: `--brand` fill with white text (dark theme: `--brand` fill with `--on-brand` text), hover `--brand-hover`. Secondary: `--surface` fill with `1px solid var(--border-strong)`. Ghost: no border, `--brand` text. Danger: a text-only or outlined red button that fills only on confirmation. Height 40px, radius 6px, `transform: scale(0.98)` while pressed.
- **Input.** `--surface` fill, `1px solid var(--border-strong)`, placeholder in `--ink-faint`, focus ring (section 10). Labels are always visible above the field, never placeholder-only.
- **Status pill.** Icon (14px) plus label (13px, 500), background and text from section 3.3, radius 6px, height 24px.
- **Avatar.** 32px circle with initials. Background is chosen deterministically from a fixed set of eight muted tints, and the text color comes from the contrast helper. Photos are an optional later addition.
- **Progress bar.** 8px tall, radius 4px, fill `--brand`, always accompanied by a text percent.
- **Table.** Header row on `--surface-alt`, 1px row dividers in `--border`, hover fill `--surface-alt`, selected row `--brand-light`.
- **Filter chip.** 32px tall pill, `--border-strong` outline, selected state `--brand-light` fill with brand text and a check icon.
- **Card.** See section 5.
- **Modal and drawer.** Opaque `--surface`, `1px solid var(--border)`, `--shadow-lg`, 12px radius, a dimmed opaque-black overlay at 40% opacity, focus trapped inside, Escape closes.
- **Toast.** Bottom right, `--surface` with a border, icon plus text, announced through a polite live region, auto-dismiss after 5 seconds unless it contains an action.
- **Icons.** One open-source line icon set (`lucide-react`), 1.5px stroke, 16px inline and 20px in cards. Do not use any other product's icons.

---

## 9. Interaction and Motion

- **Task movement.** Drag and drop between status columns on Boards, with a full keyboard alternative (a "Move to status" menu on every card). Drag must never be the only way to change status.
- **Transitions:** 150 to 200ms, opacity and transform only. Progress bars may animate width. Never animate shadows or filters.
- **Reduced motion:** honor `prefers-reduced-motion` by disabling all non-essential transitions.
- **Hover and focus:** every interactive element has distinct resting, hover, focus, and active states. States are shown by fill, border, or text changes, never by shadow depth alone.

---

## 10. Accessibility Requirements

Target: **WCAG 2.2 AA**.

- Text contrast is at least 4.5:1 (3:1 for text of 24px or larger, or 18.66px bold). Control boundaries and meaningful graphics are at least 3:1. The palette in section 3 already meets this. Placeholder text meets 4.5:1 too.
- **Focus ring:** 2px solid `--brand` with a 2px offset, always visible on keyboard focus (5.28:1 on bg in light theme, 8.55:1 in dark).
- **Status is never color alone.** Icon and text label always appear together.
- **Target size:** at least 24 by 24px for every control, aiming for 40px or more on primary controls.
- **Keyboard:** full operation without a mouse, logical tab order, skip-to-content link, visible focus in every state.
- **Screen readers:** semantic table markup, labeled icon buttons, live regions for toasts and activity updates, chart data available as a table.
- **Tooltips** are never the only way to reach information. Anything in a tooltip (exact timestamps, threshold explanations) is also reachable by keyboard focus.
- Test with keyboard-only navigation and one screen reader before each phase is called done.

---

## 11. Data Display Rules

These protect the goal of the system, which is trustworthy monitoring.

1. **One source of truth for every metric.** Overdue, stalled, completion rate, and workload use only the formulas defined in `project-context.md`. The same helper functions feed the stat cards, the table, the donut, and reports so they can never disagree.
2. **No invented numbers.** If a value cannot be computed (for example, progress on a task with no checklist), show a dash, not an estimate.
3. **Chart segments never overlap.** The reference shows an "Overdue" slice next to "In Progress", which would double count a task that is both. Ours use stored statuses only.
4. **Numbers reconcile.** The completion rate in the donut center must match Done divided by total in the legend. (The reference image itself does not: it shows 75% beside counts that work out to 60%.) Add a test that checks this.
5. **Timestamps** are stored in UTC and shown in the viewer's timezone, relative for recent events and absolute on hover.
6. **Transparency.** Any metric shown about an employee is also visible to that employee.

---

## 12. Other Screens (to be extended when references are provided)

- **Team:** a grid or list of employees, each with assigned, in progress, done, and overdue counts, opening a per-employee task list. Reuses the dashboard's avatar, pill, and table patterns.
- **Boards:** Kanban columns by status. Cards show title, assignee avatar, due date, and any overdue or stalled flag.
- **My Tasks (employee):** a single list grouped by status with large touch targets and one-tap status changes.
- **Task detail:** description, assignee, due date, status, priority, checklist, comments, attachments, and the activity log, which is the evidence trail for monitoring.
- **Reports:** per-employee and per-period summaries with CSV export, using the same metric helpers.
- **Inbox and Schedule:** quick capture with assignment, and a due-date calendar with a keyboard-accessible reschedule action.

---

## 13. Implementation Mapping

Define tokens as CSS variables and expose them to Tailwind, so components use semantic classes and both themes work from one set of class names.

```css
/* index.css */
:root {
  --bg: #F3FBFA;
  --surface: #FFFFFF;
  --surface-alt: #E4F4F2;
  --border: #CDEBE7;
  --border-strong: #668582;
  --ink: #132A29;
  --ink-muted: #4A6664;
  --ink-faint: #557170;
  --brand: #087570;
  --brand-hover: #065F5B;
  --brand-light: #D2F3F0;
  --on-brand: #FFFFFF;
}
[data-theme="dark"] {
  --bg: #111B1A;
  --surface: #1A2827;
  --surface-alt: #223432;
  --border: #2C403D;
  --border-strong: #5F7C78;
  --ink: #E6F2F0;
  --ink-muted: #9DB7B3;
  --ink-faint: #8AA29E;
  --brand: #2DC9C0;
  --brand-hover: #5AD9D1;
  --brand-light: #173C3A;
  --on-brand: #062321;
}
```

```js
// tailwind.config.js (theme.extend)
colors: {
  bg: 'var(--bg)', surface: 'var(--surface)', 'surface-alt': 'var(--surface-alt)',
  border: 'var(--border)', 'border-strong': 'var(--border-strong)',
  ink: { DEFAULT: 'var(--ink)', muted: 'var(--ink-muted)', faint: 'var(--ink-faint)' },
  brand: { DEFAULT: 'var(--brand)', hover: 'var(--brand-hover)', light: 'var(--brand-light)', on: 'var(--on-brand)' },
},
fontFamily: {
  display: ['"Space Grotesk"', 'sans-serif'],
  sans: ['Inter', 'sans-serif'],
  mono: ['"JetBrains Mono"', 'monospace'],
},
```

Status colors follow the same pattern (`--status-done-text`, `--status-done-bg`, and so on) using the values in section 3.3 for each theme.

---

## 14. Do Not

- Do not use `backdrop-filter`, translucent panels, gradient fills, or inset and dual-direction shadows.
- Do not use `--brand-vivid` (`#0DABA3`) for text, meaningful icons, or control boundaries. It fails contrast on white.
- Do not use brand turquoise to represent a task status.
- Do not communicate status with color alone.
- Do not show a percent, count, or trend that is not derived from the shared metric helpers.
- Do not copy another product's name, logo, icons, illustrations, sample data, or brand colors.
- Do not add tracking that employees cannot see, and do not design any screen around surveillance features such as screen capture or keystroke logging.

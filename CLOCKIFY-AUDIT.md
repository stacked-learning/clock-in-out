# Clockify product and workflow audit

Audit date: 13 October 2026, Australia/Sydney. Prepared to guide a custom clock-in/out and project time tool.

Clockify combines personal time capture, project costing, team attendance, approvals, resource planning and client invoicing. A credible alternative needs consistent time records and permissions underneath all these views. Reproducing the screens alone would leave the hardest operational problems unresolved.

## Scope and evidence

I inspected the signed-in Clockify web application in Chrome on your desktop, in the **Stacked Learning** workspace. You were signed in as the workspace owner. The banner reported **7 days left in trial**, with sample data enabled. Four sample teammates and your owner account appeared on the Team screen. Sample figures below are demonstrations, not actual business performance.

Evidence labels used in this report:

- **Observed:** read directly from your signed-in application, including screen contents, existing sample records, menus and unsaved forms.
- **Documented:** verified against current official Clockify pages; not exercised in your account.
- **Recommendation:** a design or implementation decision for your tool, not a claim about Clockify internals.

This was a broad, read-only product audit. I opened existing records and inspected blank creation forms, then closed the forms without saving. I did not create time entries, run timers, approve requests, send reminders, change workspace policies, remove sample data, connect integrations or purchase a plan. Native Windows/macOS/Linux clients and mobile apps were researched through documentation, not installed or tested. This is not a penetration test, code audit, performance benchmark or payroll compliance assessment. Export choices were inspected; exported files were not downloaded and reconciled. Permission controls were inspected as owner; enforcement under separate member accounts remains untested.

Visual inspection covered the tracker, summary reporting and activity settings. The first screenshot attempt timed out; subsequent screenshots succeeded. Most detailed evidence came from the rendered page and accessibility content. Some asynchronously loading tabs only exposed navigation during inspection; these are identified below rather than counted as completed workflow tests.

## Product structure

Observed navigation divides work into three groups:

| Group | Modules | Purpose |
|---|---|---|
| Daily work | Time Tracker, Calendar, Schedule, Expenses, Time off | Capture work and plan availability |
| Analyze | Dashboard, Reports | Understand time, attendance and money |
| Manage | Kiosks, Approvals, Projects, Team, Clients, Tags, Invoices | Maintain records and control workflow |

Workspace settings have a separate navigation: General, Time tracking, Finances, Time planning, User activity, Kiosk, Security, Accounts, Permissions, Integrations & add-ons, and Import/Export. Timesheet is a configurable view and was disabled in this workspace.

## Screen-by-screen findings

| Area | Direct observation | Implication for your tool |
|---|---|---|
| Time Tracker | Description, project, tags, billability, start/end, date, duration and Add; menu contains Timer, Manual and Break modes. Personal history was empty. | Keep the daily action fast, with explicit current status and a correction workflow. |
| Timesheet | Workspace setting was off; tracker offers Enable Timesheet. Settings explain that project is required in this view. | A weekly grid is useful for retrospective project hours. The live grid itself was not inspected. |
| Calendar | Week/day choices, teammate selector, date range, planned labels and zoom controls. Current personal week was empty. | Show tracked time separately from plans; support overnight work clearly. Dragging and saving were not tested. |
| Dashboard | Project grouping, Only me scope, date selector, total time, top project/client and most tracked activities. Personal view had no data. | Make personal versus team scope unmistakable, including in empty states. |
| Summary report | Team/client/project/task/tag/status/description/kiosk filters; date range; total, billable and amount; grouping; rounding; estimate display; print/share/invoice actions. | Use one calculation engine across cards, charts, tables and exports. |
| Detailed report | Entry-level descriptions, project/task/client, tags, user, billability, start/end/date and duration; row selection; Time audit and Add time for others controls. | Corrections and bulk administration belong beside the records, subject to permissions and approval locks. |
| Weekly report | Project rows across Monday–Sunday, daily and weekly totals, grouping and rounding. | Provide quick reconciliation against detailed entries. |
| Export menu | PDF, CSV, Excel and Customization. | CSV should be a first release feature; use stable IDs and explicit timezone/units. |
| Report families | Time, Team and Expense report choices. Team reports include Attendance and Assignments. | Project reporting and attendance are different views with different calculations. |
| Attendance | Start, end, capacity, work, overtime, undertime, balance, break and time off; date/team filters and grouping. | Work-span duration, recorded work and payable time must be explained separately. |
| Projects | Four sample projects; active/client/access/billing filters; search, export and creation control. | Maintain project lifecycle and prevent archived projects from breaking history. |
| Project tasks | Named tasks, search and Add new Task input. | Tasks belong to a project and should not be selectable under a different project. |
| Project access | Public/private visibility and Add members. Public projects allow everyone to track time. | Define membership separately from visibility of confidential costs. |
| Project settings | Client, colour, billable default, hourly billable/cost rates, manual/task-based estimate, optional recurring reset, inclusion of non-billable time. | Persist effective rates and budget rules rather than recalculating all history from today's settings. |
| Project status | Estimate, remaining, billable/non-billable time and expenses, amount, progress and task breakdown. | Progress must state which hours consume the estimate. |
| Project forecast | Day/week/month choices; completed, forecasted, scheduled and estimate; link to current schedule. | Forecasting depends on reliable planned assignments and estimates. |
| Schedule | Project/team timelines, search, milestones, zoom, publication state and capacity options. Team view displayed open and overloaded hours. | Planning needs capacity and leave data, not just coloured calendar blocks. |
| Approvals | Timesheet/expense selector; Pending, Unsubmitted and Archive; manager column, period totals, remind/approve-all controls. | Missing submission and pending review need separate queues. |
| Approval detail | Existing pending sample request with submitted/break/billable/leave/amount/cost totals, project split and entry details; Edit Timesheet, Reject and Approve actions. | Preserve what was submitted and reviewed; require reasons for correction/rejection. Actions were not executed. |
| Team | Full, Limited, Groups and Reminders tabs; filters by status/rates/role/group; user cost and billable rates. Owner, member, team manager and project manager roles visible. | Membership, manager scope and commercial visibility require a deliberate permission model. Limited-user, group and reminder workflows were not tested. |
| Clients | Search, active filter, add input, address and currency, edit/menu controls. | Clients group projects and carry billing metadata. |
| Tags | Empty state explaining tagging, with tag-name input and Add. | Optional tags provide reporting dimensions without expanding the project hierarchy. |
| Kiosks | Empty-state workflow: configure assignees/authentication; clock in/out and breaks; attendance reports. | A shared-device experience needs a simple employee-facing screen separate from administration. |
| Kiosk form | Required name/assignees, default work and break projects, logout duration and authentication checkbox. | Include device/session expiry and a recovery path for forgotten clock-outs. No kiosk was created or launched. |
| Expenses | Personal list had no results. Blank form includes teammate/date, required project/category/amount, currency, note, billability and receipt attachment. | Capture expenses independently but associate them with projects and approval periods. |
| Time off | Requests, Timeline, Balance, Policies and Holidays navigation. Sample requests demonstrated pending, approved and rejected states. | Leave affects capacity and attendance and needs policy-aware approval. |
| Leave balance | Policy selection, accrued/used/available columns and export. | Use a transaction ledger for entitlement and explain how pending requests affect balances. Policy editor, holidays and accrual execution were not inspected. |
| Invoices | Client/status/date/amount/balance filters; sample sent and overdue invoices; ID, issue/due dates, amount and balance. | Invoice status and amount due are separate from time approval. |
| Invoice detail | Existing sample invoice with service/product items, quantities, prices, tax checkboxes, discount/tax percentages, totals, import-time/expenses, PDF download, payment and recurrence controls. | Snapshot billing data and prevent the same time from being billed twice. Payments and recurrence were not tested. |

Observed routes include `/tracker`, `/calendar`, `/dashboard`, `/projects`, `/teams`, `/clients`, `/tags`, `/kiosks`, `/expenses`, `/time-off`, `/scheduling`, `/approvals`, `/invoices`, `/reports/summary`, `/reports/detailed`, `/reports/weekly` and `/reports/team/attendance`. Existing sample project, invoice and approval detail pages were also inspected. These are web application routes, not proposed API endpoints.

## Workspace configuration found

These are current settings in the inspected trial workspace, not universal Clockify defaults.

| Configuration | Observed value |
|---|---|
| Week start / daily work capacity | Monday / 8 hours |
| Overtime calculation period | Daily |
| New project defaults | Public and billable enabled |
| Timesheet view | Disabled |
| Break tracking | Enabled; no default break project |
| Time approval | Enabled, weekly; semi-monthly and monthly also offered |
| Duration format | Full `hh:mm:ss` |
| Labels | Client / Project / Task |
| Custom entry fields | None configured |
| Required fields | Project/task unchecked; tag/description controls shown disabled |
| Report rounding rule | Nearest 15 minutes; report rounding checkbox separately available |
| Historical time/expense locking | No date lock configured |
| Force timer | Disabled |
| Project/task favourites and favourite entries | Enabled |
| Currency / number / currency display | USD / `3.456,78` / `123 USD` |
| Workspace fallback billable/cost rate | Both 0 USD |
| Cost rates, billable rates and task rates | Enabled |
| Expenses and expense approval | Enabled; weekly approval |
| Invoicing | Enabled; company address absent |
| Leave and scheduling | Enabled |
| Scheduled billable/cost amounts | Disabled |
| Activity monitoring, GPS and screenshots | Disabled |
| Data region | Global; region edit disabled in this session |
| SSO / SCIM | Not configured; Enterprise controls disabled |
| Audit log | Enterprise control disabled |

The activity screen states that screenshot capture can run every five minutes during a desktop timer. Its audit-log controls list time entries, projects, clients, tags and expenses. Availability controls were observed; recording behaviour was not tested.

### Permission observations

Rates and amounts are restricted to admins. Anyone can change billability and create projects, clients, tasks and tags. Regular users can see all tracked time and expenses and access Team Dashboard. Everyone's schedule assignments and leave outside their teams are not enabled for regular users. Assignment creation is admins-only; project status is visible to anyone.

Timesheet and expense approvers are admins/team managers. Editing others' time and expenses is admins-only. Invoice management is admins-only; a specific-member option is available. Kiosk launch is admins-only in the inspected settings.

**Recommendation:** begin your tool with own-record visibility for ordinary staff, scoped team access for managers, and restricted costs/rates. Add broader visibility only where the business needs it. These findings establish configured policy, not proof that every API enforces it.

## Checks and notable behaviours

1. **Report totals reconcile at the screen level.** Summary, Detailed and Weekly each showed 66 hours for 12–18 October. Project subtotals were 14 + 13 + 22 + 17 = 66 hours. Summary and Detailed showed 41 billable hours and 491 USD. Export parity remains untested.
2. **Estimate scope matters.** Project Alpha showed a 400-hour estimate, 285 billable hours, 115 remaining and 357 total hours. The 71.25% progress equals 285/400, not 357/400. A reader needs to know whether non-billable work consumes a budget.
3. **Invoice arithmetic reconciles.** The inspected sample invoice showed subtotal 764.20 USD, discount 76.42, tax 68.78 and total 756.56. Calculation: 764.20 - 76.42 + 68.78 = 756.56. The presence of a 10% sample tax is not evidence of configured Australian tax compliance.
4. **Personal and team scope differ.** Tracker and personal dashboard were empty while team reports had sample data. This is consistent with the selected scope, not demonstrated data loss.
5. **Overnight entries are explicit.** Entry and attendance views show `+1` beside an end time after midnight. Your model needs complete timestamps and an explicit local date policy.
6. **Work hours can differ from elapsed span.** One sample teammate's attendance row showed a 18:00–04:00 span with 16 recorded work hours. The sample entries include overlapping activity intervals. This is not proof of a calculation defect; a product should flag overlap and distinguish summed project time from attendance span.
7. **Future undertime can confuse a live dashboard.** On Tuesday 13 October, the current-week attendance report included later weekdays with capacity but no work and counted undertime. The full-week summary showed 154 undertime hours. Recommend separate elapsed-to-date and full-period views before drawing conclusions about missing work.
8. **Localisation needs intentional setup.** This Australian workspace currently uses USD and a comma decimal separator. Your version should explicitly choose currency, number format, timezone, week start and work capacity during onboarding.
9. **Advanced entitlement remains relevant during trial.** Security and audit-log controls were disabled despite broad trial access. Do not treat trial visibility as permanent plan entitlement.

## Integrations and migration

Observed integration settings list QuickBooks, Jira, Google Calendar, Outlook Calendar and other web-app tracking integrations, plus Marketplace. No add-ons were installed. The QuickBooks card explicitly says it is incompatible with QuickBooks Payroll; do not assume an accounting integration is a complete payroll integration.

Observed import UI accepts CSV/TSV/SSV/PSV up to 10 MB, offers users/client-project/timesheet templates and supports exporting time entries for a chosen year. No import was performed.

The official API documents header-based API/add-on authentication, pagination, webhooks and separate ordinary/report regional endpoints. Any future integration should handle pagination, regional routing, retries and duplicate events. Its documented 50 requests/second limit is specifically for an add-on on a workspace using `X-Addon-Token`; it should not be generalised to all credentials. [Clockify API documentation](https://docs.clockify.me/).

Recommendation: start migration with an exported file, map stable source IDs, preview errors and totals, and import idempotently. Keep original timestamps, descriptions, project/task mappings, billability and approval/invoice provenance where the source actually provides them. Missing fields should remain identifiable rather than being silently invented.

## Current pricing and feature boundaries

Official pricing checked 13 October 2026. USD per seat/month; annual figures require annual billing. [Clockify pricing](https://clockify.me/pricing).

| Plan | Annual billing | Monthly billing | Main boundary |
|---|---:|---:|---|
| Free | 0 | 0 | Up to five users; core tracking |
| Basic | 3.99 | 4.99 | Administration, kiosk, imports, rates and exports |
| Standard | 5.49 | 6.99 | Approvals, attendance, leave and invoices |
| Pro | 7.99 | 9.99 | Scheduling, budgets, expenses and productivity features |
| Enterprise | 11.99 | 14.99 | SSO, SCIM and audit log |

Active and invited users occupy paid seats, including owners and managers; deactivated users do not. The trial is not a reliable representation of Free. Prices are a dated comparison, not a purchase quote.

## Features researched beyond the desktop browser

The official timer documentation describes real-time/manual capture, breaks, favourite entries, offline sync, idle detection, reminders, Pomodoro, automatic tracking, calendar connections and timer restrictions. Their native-device operation was not exercised here. [Timer features](https://clockify.me/features/timer).

Clockify documents native desktop clients for Windows, macOS and Linux alongside mobile apps and browser extensions. Desktop-specific convenience features justify a later native companion only if those workflows matter to your team. [Desktop time tracking](https://clockify.me/desktop-time-tracking).

The documented Timesheet view provides weekly hour entry by project/task. The observed workspace switch was off, so the actual editable grid and save behaviour remain untested. [Timesheet features](https://clockify.me/features/timesheet).

The documented approval workflow includes submissions, scoped manager review, archived approved requests, locking and reminders. I inspected request contents and controls but did not execute transitions. [Approval features](https://clockify.me/features/approval).

## Build priorities

Confirmed by you: the tool should support both staff clock-in/out and project time. You subsequently added Xero invoicing, computer clients, an Android app, an iOS PWA and intelligent daily-note imports. These are required deliverables. The separate build specification and INTEGRATIONS-AND-CAPTURE.md convert this into concrete acceptance criteria. They supersede the original optional/later placement of native clients and accounting integration below; the table records the original audit recommendation.

| Release | Deliverable | Reason |
|---|---|---|
| 1: Reliable daily operation | Sign-in, staff and roles, clock-in/out, breaks, project/manual time, history, weekly totals, corrections, approval and CSV | Covers the usable operational loop |
| 2: Management depth | Leave, capacity, attendance exceptions, budgets, rate history, richer exports and shared-device kiosk | Improves supervision and cost control |
| 3: Commercial workflows | Expenses, invoice snapshots, payment tracking, accounting integrations and planning | Builds on trusted approved records |
| 4: Selected parity | Native/offline client, activity features, advanced identity provisioning and extensions | Only implement where the team has a demonstrated need |

Application authorisation and an audit trail should exist from release 1 even if the comparator gates some controls by plan. Desktop activity/GPS/screenshot collection is not necessary for an ordinary time tool; treat it as separate optional scope.

### Further verification before production

Use a designated test workspace to exercise timer start/stop/recovery, manual corrections, kiosk authentication, cross-midnight/DST handling, duplicate requests, submission/rejection/resubmission, approved-entry locks, member/manager access enforcement and exported totals. Test native offline sync and integrations only if included in scope. Current sample observations cannot establish these guarantees.

The workspace directory was empty at audit start. No existing application was available to compare against Clockify or modify. This deliverable is the investigation and build blueprint, not an implemented replacement.

# Clockify-style tool: proposed build specification

Prepared 13 October 2026. This is a proposed implementation design derived from the companion audit. It does not describe Clockify's private database or architecture. Confirmed scope: employee clock-in/out, project time, Xero invoicing from logged hours, computer clients, an Android app, an iOS PWA, and intelligent daily-note imports. See [the expanded integration and capture specification](INTEGRATIONS-AND-CAPTURE.md) for these requirements and the worked note example.

## Intended experience

A staff member signs in, sees whether they are clocked out, working or on break, and completes the next action in one click. They can associate work with a project, review their day/week, request corrections and submit a period. A manager sees outstanding corrections, missing submissions, pending approvals and attendance exceptions. An administrator maintains staff, projects, policies and export settings.

Project timers and attendance must have an explicit relationship. Choose whether project timers generate attendance or sit within a separate shift. For a mixed tool, recommend a shift containing work/break intervals and project allocations, with checks that allocations do not double-count attendance. Manual weekly project hours may lack precise attendance timestamps; label them accordingly.

## First release screens

| Screen | Required behaviour |
|---|---|
| Today | Current state, start time, elapsed duration, project, Clock in/out and Start/end break, latest entries |
| My time | Daily records, weekly totals, project allocations, correction requests and submission status |
| Review | Pending timesheets/corrections; submitted values; approve/reject with explanation |
| Attendance | Team work, breaks and exceptions; elapsed-to-date default; filter by person/date/status |
| Projects | Clients, projects, tasks, active/archive status and membership |
| Team | Staff, scoped manager relationships, roles and capacity |
| Settings | Australia/Sydney timezone, AUD option, Monday week start, display format and tracking/approval rules |
| Export | Preview approved hours, chosen period, timezone and columns; download CSV |
| Note import | Paste/upload daily Markdown or ingest through a common API; review parsed times, proposed projects, ambiguity and duplicates before committing |
| Billing / Xero | Map clients, review approved unbilled hours and rates, create a draft Xero invoice and reconcile its status |
| Device quick timer | Favourite/recent projects and one-action start/stop/switch on computers, Android and iOS PWA; visible offline queue and sync status |

Use Clockify's familiar navigation, terminology, layout patterns and core interaction sequences to reduce staff training. Preserve a clear personal/team scope indicator, visible saved/pending/error states and meaningful empty states. Prefer named controls over unlabeled icon-only actions. On mobile, prioritise daily clock actions over wide management tables. Use our own product name, logo and assets. See UI-FAMILIARITY.md for the required screen mapping and usability checks.

## Proposed data model

Every workspace-owned entity needs a workspace ID and server-side authorisation. Proposed entities:

| Entity | Essential fields/relationships |
|---|---|
| Workspace | Name, timezone, currency, week start, capacity policy, tracking/approval configuration |
| User / Membership | Identity, active status, role, manager/team scope; preserve membership history |
| Client / Project / Task | Workspace, names, lifecycle, membership; task belongs to project |
| Shift / Interval | User, actual start/end instants, local context, work/break type, source, status, revision |
| Project allocation | User, project/task, description, tags, billability, time interval or duration-only basis |
| Correction request | Target revision, proposed change, reason, requester, reviewer, decision |
| Timesheet submission | User, local period boundaries, submitted snapshot, status, reviewer and timestamps |
| Audit event | Actor, action, target, timestamp, before/after or revision reference, reason |
| Rate version | Billable/cost amount, currency, scope and effective date |
| Leave transaction | Policy, user, accrual/use/adjustment/reservation, amount and effective date |
| Export batch | Period, calculation version, record revisions and generated timestamp |

Required additional entities: source document/version, import batch/candidate, project alias/matching rule, device operation/outbox, Xero organisation/contact mapping, billing batch/line allocation and external invoice link. Later entities: expense/receipt, assignment/milestone, kiosk/device session and expanded payment handling. Detailed integration fields and lifecycle are specified in INTEGRATIONS-AND-CAPTURE.md.

Store time instants in UTC and retain the relevant IANA timezone; derive local period boundaries explicitly. Store durations as integer seconds and currency in decimal-safe units. Keep local wall-clock input alongside resolved instants where needed to explain corrections. Avoid a schema that only stores `09:00` and `17:00` without dates/timezone.

## State and calculation rules

Daily attendance states: clocked out → working → on break → working → clocked out. Define whether clocking out during a break ends both intervals. Reject overlapping open shifts for the same person at the chosen workspace/account scope.

Submission states: draft → submitted → approved, or submitted → rejected → revised → resubmitted. Approved records require an auditable reopening/correction process. Preserve the originally approved snapshot and reason for any revision.

Rules for the initial build:

1. One authoritative running state; repeated clicks/network retries cannot create duplicate shifts.
2. A timer display is derived from server timestamps. Reloading or closing the browser does not erase elapsed time.
3. Record start/end and work/break durations separately. Paid versus unpaid breaks must be a configured business rule.
4. Detect overlapping timed project allocations. Duration-only timesheets cannot be presented as verified clock attendance.
5. Flag negative/implausible durations and missing clock-outs; provide a correction request rather than silently altering time.
6. Handle midnight and daylight-saving boundaries using actual elapsed instants. Explain how an overnight shift contributes to daily reports.
7. Keep raw duration unchanged when displaying rounded billing/export values. State rounding scope and direction.
8. Apply effective rates and policy versions deliberately. Historic approved figures should not change merely because a default rate changes.
9. For live attendance, distinguish elapsed-to-date capacity from full-period planned capacity. Future weekdays must not imply missed work in the default live view.
10. Do not infer payable hours or award-specific overtime from project hours alone. Payroll rules require a separate validated business specification if needed.

## Permission design

| Role | Initial access |
|---|---|
| Staff | Own time and corrections; assigned/public projects; own submission status |
| Team manager | Named team's time and review queues; approve within scope; costs only if explicitly granted |
| Project manager | Assigned project progress and allocations; separate permission for approval |
| Admin/owner | Workspace administration, exports and policy management; all sensitive changes recorded |

Enforce ownership and scope on the server for every read and write, including exports and search. Hiding a navigation item is only presentation. Deactivating a user must preserve historical time and immediately prevent future access as designed. Kiosks need narrowly scoped sessions and must not expose administrative account access.

## Meaningful acceptance criteria

| Scenario | Expected result |
|---|---|
| Double-click Clock in or retry a timed-out request | One shift; second response returns existing operation/state |
| Reload while working | Running state and elapsed duration recovered from authoritative timestamps |
| Start/end break | Work/break totals reconcile to shift intervals under the configured break rule |
| 23:00–02:00 overnight shift | Three actual hours, clear next-day indicator and consistent allocation across reports |
| DST change in Australia/Sydney | Duration reflects elapsed instants; local display shows actual offset context |
| Two tabs stop one timer | One close operation; both views converge without duplicate records |
| Overlapping project allocation | Flag or reject according to policy; no unexplained double-counted attendance |
| Correct an approved entry | Direct mutation blocked; authorised correction creates a recorded revision |
| Staff requests another user's record/export | Denied regardless of guessed URL or ID |
| Manager requests a different team's submission | Denied unless explicitly granted that scope |
| Reject and resubmit a timesheet | Reason preserved, new revision reviewable, old submission retained |
| Export approved period | CSV totals match detailed records and dashboard for the same scope/policies |
| Rate changes after approval | Historic approved report/invoice figures remain reproducible |
| View current week early in week | Future days identified separately from elapsed attendance exceptions |
| Connection lost during clock action | UI shows uncertain/pending state and reconciles safely on reconnect |

These are implementation tests for your future app, not tests already run against Clockify.

## Architecture and delivery

A shared server API, relational data store and background job worker should support the web app, computer clients, Android app and iOS PWA. Each client uses the same time-entry and approval rules. Computer operating-system coverage and the actual framework, authentication provider and hosting remain to be confirmed.

Use database transactions/constraints for running-state consistency, idempotency keys for clock actions, revision checks for edits and a shared calculation library/service for report/export totals. Background jobs handle imports and Xero sync/reconciliation. Offline capture and queue reconciliation are part of the required device experience; platform behaviour must be validated on actual target devices.

Suggested implementation order:

1. Confirm attendance/project relationship, identity provider, hosting and team scope; document time/approval policies.
2. Implement membership, projects and authoritative clock/break state with history and audit events.
3. Add daily-note import with review, project matching, manual records/corrections, period submission, manager review and approved CSV export.
4. Deliver computer quick-timer clients, Android app and iOS PWA using the shared API and offline operation queue.
5. Add Xero contact mapping, reviewed billing batches, draft invoice creation and status reconciliation.
6. Verify concurrency, authorisation, imports, device offline recovery, midnight/DST and billing/export totals with realistic test data.
7. Add leave, kiosk, budgets, expenses and planning according to confirmed priorities. Xero and the device clients are confirmed deliverables, not optional parity extras.

## Decisions still needed

- Whether project allocations must sit within clocked attendance, and how retrospective project hours outside a shift are handled. Both attendance and project time are confirmed requirements; Xero invoicing is now confirmed scope.
- Number of staff and manager/team boundaries; one workspace or multiple organisations.
- Which computer operating systems must be supported, and whether a shared kiosk is also required. Android app and iOS PWA are confirmed.
- Existing sign-in provider and deployment environment.
- Paid/unpaid break rules, manual corrections, approval frequency and period locking.
- Xero organisation(s), contact/project mappings, billing rates, account/tax codes and invoice grouping; any additional payroll destination and export schema.
- Whether current Clockify data needs migration, and which history must be preserved.
- Import destination is confirmed as the custom app; decide which additional personal logging formats/API clients must be supported beyond Markdown notes.
- Approved project/task taxonomy and aliases, and whether optional LLM processing should run locally or through an authorised hosted service.

No delivery duration or cost is promised before these choices and the implementation scope are settled. The immediate next deliverable can be a working first release built around the validated daily clock and approval loop.

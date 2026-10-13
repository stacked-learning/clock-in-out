# Clockify UI familiarity requirements

Confirmed requirement, 13 October 2026: honour the Clockify app's familiar UI and workflows to reduce the cost of staff learning a replacement. This is a product requirement alongside attendance, project time, device clients, daily-note import and Xero billing.

## Design principle

Keep the main navigation, labels, screen organisation and daily interaction sequences recognisable to existing Clockify users. Apply our own product identity and assets. Make necessary differences visible where they affect time, approvals or billing; familiarity must not conceal a change in behaviour.

## Navigation and screen mapping

| Clockify pattern | Required treatment in our app |
|---|---|
| Left sidebar, with daily work followed by Analyze and Manage | Preserve the grouping and familiar order; show only implemented and authorised modules |
| Time Tracker | Keep this name; make it the main quick-entry screen with description, project/task, tags, billability, duration and start/stop/manual controls |
| Timesheet | Use a recognisable weekly grid with project/task rows, daily columns and totals |
| Calendar | Preserve day/week switching, date navigation and chronological time blocks |
| Dashboard | Familiar totals and project/client breakdowns; make personal/team scope explicit |
| Reports | Preserve Summary, Detailed and Weekly tabs, date selection, filters, grouping, totals and Export |
| Projects | Preserve list/search/filter patterns and task, access, status and settings organisation where implemented |
| Team, Clients and Tags | Retain the familiar names and management-list patterns |
| Approvals | Preserve Pending, Unsubmitted and Archive; keep period totals and explicit Approve/Reject actions |
| Invoices | Keep the familiar label; place Xero connection/mapping, invoice preview and external status here |
| Kiosks, Expenses, Time off and Schedule | Retain familiar labels and patterns when delivered; do not show nonfunctional navigation merely for visual parity |
| Workspace settings | Separate workspace policy/configuration from personal preferences, following the familiar administrative structure |

## Daily interaction compatibility

- Keep project/task selection, recent/favourite activities and timer/manual modes in familiar locations.
- Use a compact entry bar on desktop and a simplified version on phones.
- Keep chronological history and daily/weekly totals close to capture controls.
- Show start/end dates and next-day indicators for overnight work.
- Retain familiar table filters and export placement across management screens.
- Use recognisable blue primary actions, light surfaces, restrained borders, spacing and hierarchy as the initial visual direction. Exact styling should be checked against the reference screens during implementation.
- Maintain clear labelled controls, keyboard access, focus states and usable touch targets rather than reproducing observed accessibility shortcomings.

Attendance is an explicit additional concept: show **Clocked out / Working / On break** and distinguish **Clock in/out** from project **Start/Stop** where the shift and project allocation are separate. Switching a project should not accidentally clock the employee out.

## Fit the new features into familiar places

| New capability | Placement and behaviour |
|---|---|
| Daily-note import | An Import notes action on Time Tracker/Timesheet opens a review screen using familiar time-entry rows and project selectors |
| Intelligent project matching | Suggestions appear beside the project/task controls with a short reason; ambiguous matches require a choice |
| Xero billing | Invoices contains a reviewed Create Xero draft flow; show the linked Xero invoice and status |
| Desktop/Android/iOS quick capture | Use the same project labels, favourites and timer semantics as the main app |
| Offline operation | Add a visible Pending sync indicator and recovery action without changing the meaning of Start/Stop |

## Implementation and validation

Before implementing each core screen, compare its layout and interaction sequence with the audited Clockify reference. Keep a screen mapping checklist covering terminology, navigation, action placement, filters, empty states and errors. Validate browser layouts and target device clients separately.

Usability acceptance: an existing Clockify user should be able to find a project, start and stop time, enter retrospective hours, inspect a weekly report and submit a timesheet without guided retraining. A manager should find pending approvals and review them through a familiar sequence. Record task completion, errors and points of confusion with representative users; treat material confusion as a design issue to fix before rollout.

Compatibility covers the UI and user workflows. It does not imply compatibility with Clockify's APIs or native clients. The custom app retains its own data model and the confirmed integrations.

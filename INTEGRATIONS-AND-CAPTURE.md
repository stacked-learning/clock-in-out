# Xero, device clients and intelligent time capture

Confirmed requirements, 13 October 2026: staff attendance plus project time, Xero client invoicing from logged hours, quick project timers on computers, an Android app, an iOS PWA and import of informal daily notes produced by your own logging methods. This document extends BUILD-SPEC.md. It is a design specification, not an implemented or connected integration.

## One time ledger, several capture methods

Monday.com is an additional confirmed integration. Its boards/items can supply linked work structure for project/task selection and note matching. Our app retains the canonical time ledger; Xero remains the invoice destination. See MONDAY-INTEGRATION.md for mapping and sync choices. Updating Monday's native Time Tracking values is not supported by its documented API; optional hours summaries use a separately configured supported destination.

Computer clients, Android, the iOS PWA, Markdown notes and future personal logging tools should all submit to the same intake and time ledger. Capture method is provenance, not a separate set of hours. Time enters as a timer record or a reviewed import, then follows corrections, submission, approval and billing.

Recommended pipeline:

`Capture → normalise → validate times → match project → detect overlap/duplicates → review → commit → submit/approve → select unbilled hours → Xero draft`

Confirmed import destination: your own Clockify-style app. Notes, personal logging tools and device timers commit into its canonical ledger. Synchronisation to the existing Clockify service is outside the confirmed scope. Xero is the separate invoice destination.

## Xero billing workflow

Xero is the invoice/accounting destination. The custom app remains authoritative for captured time, approval history and which entries have been billed.

1. Connect a Xero organisation using OAuth; bind the selected organisation to the workspace. Store credentials on the server, not in staff client bundles.
2. Map each internal client to a confirmed Xero contact. A project/client display name is not a reliable accounting identifier. Allow several projects to map to the same contact.
3. Configure the billing rate, currency, invoice grouping, revenue account and tax code. Business owners supply these values; do not infer rates or tax treatment from prose.
4. Select approved, billable, unbilled entries for a client and date range. Preview raw duration, billing rounding, quantity, rate and resulting amount. Review invoice descriptions separately from internal notes.
5. Freeze a billing batch, reserving the selected entries so another batch cannot invoice them concurrently.
6. Create a draft sales invoice in Xero, store its ID and the entry-to-line allocations, and reconcile the result. Release reservations only when failure is known; an uncertain network outcome requires lookup/reconciliation before retry.
7. Display linked invoice status. Review/send invoices through an explicit billing workflow; note ingestion must not itself issue an invoice.

Xero's invoice API supports sales invoices (`ACCREC`), contact IDs and line items with quantity, unit amount and account information. Proposed mapping: hours become Quantity; hourly billing rate becomes UnitAmount. Use an explicit draft status for the initial workflow. [Xero invoice API](https://developer.xero.com/documentation/api/accounting/invoices).

Proposed safeguards: stable billing-batch IDs, a unique link from time allocations to billed quantity, transactional reservations, API idempotency where supported and reconciliation after uncertain outcomes. Xero documents replaying cached results for the same idempotency key; a durable local ledger is still needed beyond provider replay windows. [Xero idempotent requests](https://developer.xero.com/documentation/guides/idempotent-requests/idempotency/).

Maintain decimal precision when converting minutes to hours. A 20-minute entry is one third of an hour, not exactly 0.33; define rounding and verify invoice totals against approved time. Group only entries with compatible client, currency, rate and tax/account treatment. Preserve all source IDs even when one invoice line aggregates many entries. Partial billing, cancelled drafts and corrections to already-issued invoices need explicit states and must not silently reset “unbilled.”

OAuth design: clients authenticate with the custom app, whose server handles Xero. If a native client ever connects directly to Xero, Xero requires PKCE for public desktop/mobile apps that cannot keep a client secret. Verify current minimum scopes and app entitlement during implementation. [Xero OAuth overview](https://developer.xero.com/documentation/guides/oauth2/overview).

Do not require Xero Projects for this workflow: internal projects can be represented in sales invoice descriptions while invoices target Xero contacts. Xero Projects synchronisation would be a separately specified feature.

## Device clients

| Surface | Required quick-capture experience |
|---|---|
| Windows client | System-tray icon and quick timer menu: favourite/recent projects, searchable project/task picker, Start, Pause, Resume, Stop and Switch project; current project, elapsed time and sync status; offline queue |
| macOS client | Menu-bar icon with the same quick timer actions, project picker, current project, elapsed time and sync status; offline queue |
| Android app | Project picker, recent/favourites, one-action timer and break control, running-state recovery and offline queue; optional widget/notification shortcut |
| iOS PWA | Home-screen install guidance, responsive quick timer, cached project list, durable local pending operations, foreground sync and manual retry |
| Web management app | Review notes, timesheets, corrections, team attendance and billing; available from clients through a management link |

All surfaces share a user identity and current timer. Switching projects closes the preceding project allocation and starts the next without an unexplained gap; the attendance shift remains intact. Closing the UI must not stop a timer unless the user chooses that policy.

Never depend on a continuously running JavaScript loop to record time. Persist actual action timestamps and render elapsed time from the start instant when the UI resumes. Device-local operations need durable IDs, timestamps, timezone/offset, device ID, sequence and expected record revision. Server reconciliation resolves concurrent actions and surfaces clock skew or overlapping offline work for review.

For iOS PWA, use sync-on-open/resume, connectivity recovery while open and an explicit retry control as the guaranteed path. Background Sync is not universally supported, so it must be an enhancement rather than a prerequisite. Verify behaviour on supported iOS versions and installed-PWA mode. Pending offline data needs a recoverable export path; do not imply browser storage guarantees permanent retention. [MDN Background Synchronization API](https://developer.mozilla.org/en-US/docs/Web/API/Background_Synchronization_API).

Windows and macOS are confirmed desktop targets. Windows uses a system-tray entry point; macOS uses a menu-bar entry point. Packaging, app signing and distribution are separate from the web app. Pick the actual client framework and supported OS versions during implementation.

### Desktop live timer and pause/resume

Selecting a project from the tray/menu-bar picker starts recording immediately. Favourite and recent projects should be available without opening the main app. Optional task and description details can be filled in through the compact timer panel. The timer visibly distinguishes **Running**, **Paused** and **Stopped**, including the current project and accumulated recorded duration.

Project timer states: `Stopped → Running → Paused → Running → Stopped`. Stop is also available directly from Running or Paused. Starting another project closes the previous session and starts the selected project atomically; prompt only if a conflict or policy requires resolution.

- **Start:** create a timer session and its first active interval.
- **Pause:** close the active interval immediately; keep the session/project available for resuming. Paused time does not accumulate project hours or billing quantity.
- **Resume:** create another active interval in the same session. Preserve the gap as paused time; do not fill it retrospectively.
- **Stop:** close any active interval and finalise the session with the sum of its active intervals. Stopping a paused session adds no time.
- **Switch project:** finalise the current project session and start the chosen project, with the attendance shift unchanged.

Example: start at 09:00, pause at 09:20, resume at 09:35 and stop at 10:00 records **45 minutes** over two active intervals. The 15-minute pause is excluded. Persist these intervals, not just a single start/end pair, so reports and billing can reproduce the total.

Pausing project recording does not automatically start an attendance break or clock someone out. Those remain separate labelled actions. Mirror Pause/Resume in the web and mobile timer controls and synchronise the state across clients.

Recording continues until the user pauses or stops, including while the timer panel is closed. Hiding the panel or closing the management window leaves the tray/menu-bar client available. Do not silently stop on app exit, restart, sleep, idle or loss of connectivity; persist the running start/state and recover it. Flag long-running or sleep-spanning sessions for review when appropriate, with any automatic stopping policy an explicit future setting rather than the default. A paused session remains paused on restart.

Online state is authoritative on the server; unsynchronised local operations remain visibly pending. Concurrent actions across devices must resolve without overlapping active timers or lost pause intervals. Closing the UI does not require a continuously ticking background counter to preserve duration.

## Daily-note import: offline first, optional language model

For the example format, a language model is unnecessary for date/time extraction. Use a deterministic parser for filenames such as `2026-10-12.md`, clock ranges, AM/PM and duration arithmetic. Support additional separators/24-hour notation as explicitly tested formats. Reject or flag missing/ambiguous dates, ambiguous AM/PM, invalid end times and overlapping intervals.

For project matching, maintain a workspace dictionary of client/project aliases, abbreviations, recurring task phrases and user-scoped rules. Exact unique aliases can match offline. Fuzzy matching proposes candidates; collisions, multiple topics and new names remain reviewable. People mentioned in a note provide context but are not automatically clients or projects.

An optional local or authorised hosted LLM may summarise descriptions and propose project/task IDs from the allowed catalogue. It cannot invent IDs, durations, billability, rates, attendance or invoice actions. Retain original text; validate structured output against the parser and catalogue. Treat note text as content, not instructions to the model or tools. Keep an entirely offline rule-based path available. Processing location and the provider remain undecided.

Recommended review UI:

- Original note beside proposed entries, with parsed times and total.
- Proposed project/task, matching reason and status such as exact alias, suggested, ambiguous or unresolved.
- Clear unknown billability until a configured rule or reviewer resolves it.
- Gap and overlap indicators; no automatic creation of lunch/break entries from silence.
- One-click correction and optional “remember this alias” rule, scoped to the appropriate user/workspace.
- Commit reviewed entries, then normal submission/approval. Import acceptance is not timesheet approval.

Preserve `source_document_id`, content version/hash, filename/date, original line text, line anchor, parsed timestamps, proposed/final project IDs, parser/rule/model version, review outcome and committed record ID. Whole-file hashes detect identical imports; line matching and record links reconcile edited files. If an already-imported line changes, propose a correction to its linked record rather than appending another entry.

Existing timers may cover the same interval as a note. The importer should propose adding context or replacing/correcting a linked entry rather than silently creating duplicate hours. Retrospective notes are project-time evidence; attendance reconstructed from them should remain labelled retrospective and go through a separate explicit review policy.

## Worked example: 12 October 2026

Timezone assumption: Australia/Sydney. Project names below are candidate taxonomy, not confirmed existing projects or accounting contacts. All billability is unresolved.

| Time | Duration | Suggested allocation | Review needed |
|---|---:|---|---|
| 08:30–09:00 | 30 min | BRSC / President's Gala Dinner / camera support | Confirm project and billability |
| 09:00–10:30 | 90 min | PGD invitation/agenda work; tablet/music support also mentioned | Mixed topics: keep full interval pending confirmation or request an explicit split; never invent sub-durations |
| 10:30–11:30 | 60 min | BRSC / website or newsletter / visibility groups | Confirm whether newsletter is a separate project or website task |
| 11:30–11:50 | 20 min | SL / module marketing / meeting | Confirm whether internal or client billable |
| 11:50–12:00 | 10 min | SL / module publishing / Vimeo upload | Confirm project and billability |
| 13:00–15:00 | 120 min | BRSC / President's Gala Dinner / backdrop design and production | Confirm project and billability |
| 15:00–16:00 | 60 min | BRSC / website / updates | Confirm project and billability |
| 16:00–17:30 | 90 min | BRSC / President's Gala Dinner / voucher design | Confirm project and billability |

Exact logged total: **480 minutes = 8 hours**. First start to final end spans nine hours. **12:00–13:00 is an unlogged gap**, not an asserted lunch break. No overlapping intervals appear in this example.

Provisional grouping: 4 hours of clearly PGD-described intervals; 1.5 hours of mixed PGD/tablet work awaiting review; 2 hours of BRSC website/newsletter work; 0.5 hours of SL work. If the reviewer assigns the entire mixed interval to PGD, PGD becomes 5.5 hours. Do not present that conditional total as confirmed.

No Xero amount can be calculated until client mapping, approved billable allocations, rate, currency and tax/account configuration are supplied. Source notes contain internal names/context; prepare invoice-facing summaries instead of copying all internal text automatically.

## Additional acceptance tests

| Scenario | Expected result |
|---|---|
| Import the supplied note | Eight candidate entries, exactly 480 minutes, one 60-minute unlogged gap |
| Mixed-topic 90-minute line | Flagged; no invented time split |
| Import same document again | No duplicate committed entries |
| Edit previously imported line | Proposed correction linked to original record |
| Import overlaps a running/finished timer | Reviewable reconciliation, no automatic double counting |
| Alias maps to multiple projects | Requires explicit choice; no silent match |
| LLM returns unknown project or altered duration | Reject output; preserve parser result and request review |
| Close/reopen device UI while timing | Correct elapsed time from persisted start; no required background loop |
| Start project from Windows tray/macOS menu bar | Timer starts without opening the main app; project and running status are visible |
| Start 09:00, pause 09:20, resume 09:35, stop 10:00 | Exactly 45 minutes recorded; 15-minute pause excluded |
| Stop an already paused timer | Finalises existing active intervals and adds zero paused time |
| Pause project recording during an attendance shift | Attendance remains working unless a separate break/clock-out action is taken |
| Restart while running or paused | Restores the persisted state; paused gaps are never counted as project work |
| Hide the timer panel | Recording continues; tray/menu-bar controls remain accessible |
| Two devices act offline | Durable operations reconcile; conflicting intervals remain visible |
| iOS has no background sync | Pending changes sync on foreground use and can be retried |
| Invoice request times out after creation | Reconcile/retry same batch; exactly one linked invoice |
| Two billers select same approved hours | Reservation prevents duplicate billing |
| Twenty-minute billing entry | Decimal conversion and displayed total follow configured precision policy |
| Xero contact/currency/account missing | Block invoice creation with an actionable mapping/configuration message |

## Delivery sequence

Deliver the ledger and approvals first, then reviewed note import and quick device capture, then Xero draft invoicing using those approved records. These are required parts of the initial product programme. Advanced AI, native monitoring, a full local invoicing engine, additional accounting systems and Xero Projects are separate optional expansions.

Outstanding choices: actual project aliases/catalogue, supported Windows/macOS versions, Xero organisation/contact/rate mappings, invoice grouping and optional local/hosted LLM preference. These do not prevent defining the common ledger and deterministic importer.

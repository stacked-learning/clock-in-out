# Xero, device clients and intelligent time capture

Confirmed requirements, 13 October 2026: staff attendance plus project time, Xero client invoicing from logged hours, quick project timers on computers, an Android app, an iOS PWA and import of informal daily notes produced by your own logging methods. This document extends BUILD-SPEC.md. It is a design specification, not an implemented or connected integration.

## One time ledger, several capture methods

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
| Computer client | Favourite/recent projects; start/stop/switch; running status; small persistent window or tray access where supported; optional keyboard shortcut; offline queue |
| Android app | Project picker, recent/favourites, one-action timer and break control, running-state recovery and offline queue; optional widget/notification shortcut |
| iOS PWA | Home-screen install guidance, responsive quick timer, cached project list, durable local pending operations, foreground sync and manual retry |
| Web management app | Review notes, timesheets, corrections, team attendance and billing; available from clients through a management link |

All surfaces share a user identity and current timer. Switching projects closes the preceding project allocation and starts the next without an unexplained gap; the attendance shift remains intact. Closing the UI must not stop a timer unless the user chooses that policy.

Never depend on a continuously running JavaScript loop to record time. Persist actual action timestamps and render elapsed time from the start instant when the UI resumes. Device-local operations need durable IDs, timestamps, timezone/offset, device ID, sequence and expected record revision. Server reconciliation resolves concurrent actions and surfaces clock skew or overlapping offline work for review.

For iOS PWA, use sync-on-open/resume, connectivity recovery while open and an explicit retry control as the guaranteed path. Background Sync is not universally supported, so it must be an enhancement rather than a prerequisite. Verify behaviour on supported iOS versions and installed-PWA mode. Pending offline data needs a recoverable export path; do not imply browser storage guarantees permanent retention. [MDN Background Synchronization API](https://developer.mozilla.org/en-US/docs/Web/API/Background_Synchronization_API).

Computer OS coverage remains to be confirmed; Windows is available in the current environment. Packaging, app signing and distribution are separate from the web app. Pick the actual client framework after deciding supported OSs and native shortcuts.

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
| Two devices act offline | Durable operations reconcile; conflicting intervals remain visible |
| iOS has no background sync | Pending changes sync on foreground use and can be retried |
| Invoice request times out after creation | Reconcile/retry same batch; exactly one linked invoice |
| Two billers select same approved hours | Reservation prevents duplicate billing |
| Twenty-minute billing entry | Decimal conversion and displayed total follow configured precision policy |
| Xero contact/currency/account missing | Block invoice creation with an actionable mapping/configuration message |

## Delivery sequence

Deliver the ledger and approvals first, then reviewed note import and quick device capture, then Xero draft invoicing using those approved records. These are required parts of the initial product programme. Advanced AI, native monitoring, a full local invoicing engine, additional accounting systems and Xero Projects are separate optional expansions.

Outstanding choices: actual project aliases/catalogue, computer OS coverage, Xero organisation/contact/rate mappings, invoice grouping and optional local/hosted LLM preference. These do not prevent defining the common ledger and deterministic importer.

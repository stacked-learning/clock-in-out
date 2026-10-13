# Monday.com integration specification

Confirmed requirement, 13 October 2026: connect our Clockify-style app to Monday.com. This is a proposed integration design, not an implemented account connection. Sync direction and the selected boards/columns remain to be confirmed.

## Relationship between systems

Monday.com can supply work structure and external references. Our app owns time capture, pause/resume intervals, note imports, corrections, approval and billing allocations. Xero receives reviewed invoice data. A Monday item is not automatically a Xero contact or a billable task; retain separate mappings.

Recommended first workflow: choose selected boards, map their work items to internal projects/tasks, then select those linked tasks in our familiar timer UI on web, Windows tray, macOS menu bar, Android and iOS PWA. Preserve an Open in Monday link on the task and recorded entries. Keep native internal projects available alongside connected ones.

## Configurable mapping

| Monday structure | Proposed use |
|---|---|
| Account/workspace | Connection identity and board selection, not automatic membership in our workspace |
| Board | Default candidate for an internal project; allow a different mapping |
| Group | Optional category/phase; optionally a project if the board represents several projects |
| Item | Default task linked to a stable external item ID |
| Subitem | Child task where used; preserve its parent reference |
| Selected people/status/date fields | Optional assignment/status/deadline context; mappings configured per board |

Board schemas vary. Inspect selected boards and preview mappings before importing. If a board's items are projects rather than tasks, allow item-to-project and subitem-to-task mapping. Do not hard-code all board names as projects. Match by connection/account and stable IDs, not title; titles can change or collide.

Show source labels and mapping status. Include linked names/aliases in intelligent note matching, but keep ambiguous aliases reviewable. A status such as Done must not silently stop a timer, approve hours or issue an invoice.

## Connection and permissions

Add Monday.com under Workspace settings → Integrations. An authorised workspace administrator connects the intended Monday account, selects boards, previews mappings and starts import. Staff devices use our API; they do not receive the integration credential.

Use the current OAuth flow and only scopes required by enabled features. Monday documents board-read/write and webhook scopes, and now provides an OAuth 2.1 flow with expiring access tokens, refresh and revocation. Verify the chosen flow, scopes and entitlement at implementation time. [OAuth and permissions](https://developer.monday.com/apps/docs/oauth), [OAuth 2.1 migration](https://developer.monday.com/apps/docs/migrating-to-the-new-oauth-flow).

Access to a board through an administrator's connection does not automatically authorise every employee to see its contents. Define internal project membership and permitted metadata explicitly. Lost permission, revoked access or disconnection should mark mappings unavailable/stale while preserving time and invoice history.

## Sync behaviour

Proposed initial mode: inbound work structure with linked time in our app. Optional outbound summaries and full bidirectional metadata synchronisation require the user's chosen scope and field-ownership rules.

- Initial import is paginated and previewable, with stable external ID mappings and restartable checkpoints.
- Rename updates the mapped title while retaining the ID, time history and invoice descriptions already frozen in billing batches.
- Moving an item updates its parent association according to configured mapping, preserving existing time provenance.
- Archived/deleted items become unavailable for new selection by default; historical time remains intact.
- Record source revision/timestamp and last successful sync. Expose errors and retry controls.
- Use supported board event webhooks plus periodic reconciliation where appropriate. Validate the webhook endpoint according to the event type, enqueue work, handle duplicates/out-of-order delivery and re-fetch authoritative accessible data before applying changes. Monday documents board webhook events and URL verification. [Webhooks](https://developer.monday.com/api-reference/reference/webhooks).
- Pin a supported API version, minimise requested fields, paginate and back off according to returned limit information. Limits include complexity and request constraints; budget integration jobs instead of polling per running timer. [Rate limits](https://developer.monday.com/api-reference/docs/rate-limits).

## Time write-back limitation and options

Monday's native Time Tracking column supports reading, filtering and creating the column, but timer values cannot be updated or cleared through the API. This means direct native session write-back and remotely starting/stopping that column are not promised. Reading native sessions is possible, but importing them would need its own provenance and deduplication rules. [Time Tracking API](https://developer.monday.com/api-reference/reference/time-tracking).

If outbound hours are requested, propose an explicitly mapped Numbers column such as Approved hours from our app, plus a linked report. Confirm the target column type/schema and write permission before implementation. Monday provides typed column mutations; do not treat every column as writable. [Columns API](https://developer.monday.com/api-reference/reference/columns).

Recommended outbound policy: publish approved duration as a derived total to a dedicated app-owned column, not an increment on top of the previous value. Define whether totals cover all time or a chosen period and whether non-billable time is included. Display the last sync and calculation basis. Draft/paused/pending entries should not look like approved billable hours.

Our ledger remains authoritative for that summary. Manual edits to the dedicated summary column are detected and surfaced; define overwrite/review behaviour before enabling automatic writes. Optional item updates containing summaries require separately enabled update-write scope and a frequency policy to avoid one message per timer action.

Full two-way metadata sync also needs field ownership, revision checks, loop suppression and explicit conflict resolution. It must not imply two-way native timer session sync, given the documented limitation.

## Data additions

Connection: account ID, installing user reference, encrypted credential reference, granted scopes, enabled features, connection health and selected boards.

Mapping: external board/group/item/subitem IDs, internal project/task IDs, board-specific field mapping, source URL, parent mapping, source revision and lifecycle.

Sync job/event: durable operation ID, connection/source entity, event version or deduplication fingerprint, direction, retry state, last successful result and error.

Outbound summary: target item/column, internal entry revision set, approval basis, period, duration units, published value and last verified external value.

## Acceptance checks

| Scenario | Expected result |
|---|---|
| Import the same board twice | Existing IDs map to the same projects/tasks; no duplicate records |
| Two items share a name | Distinct IDs remain distinct |
| Item renamed/moved | Mapping changes without losing recorded hours |
| Item archived/deleted | New selection disabled, history and invoices retained |
| Start/pause/resume linked task from tray | Same timer ledger semantics; source link retained |
| Note refers to a Monday task alias | Exact unique mapping or reviewable candidate; no invented item |
| Staff lacks internal project access | Connected metadata cannot bypass app permissions |
| Credential revoked or API unavailable | Clear stale/error state; existing time remains usable |
| Webhook delivered twice/out of order | No duplicate import or stale overwrite |
| Rate limit reached | Deferred retry with checkpoint; no tight retry loop |
| Native time column requested as outbound destination | Unsupported path explained; offer configured summary/report option |
| Outbound summary retried after timeout | Reconciled absolute value; hours not added twice |
| Approved hours later corrected | Audited recalculation and revised summary; issued Xero invoice unaffected until explicit billing correction |

## Decisions pending

Which Monday account/boards to use; board/group/item mapping; inbound only versus outbound summaries versus bidirectional metadata; authoritative fields; summary columns and period; whether existing Monday-recorded sessions need migration. These choices can be configured independently of our shared time ledger.

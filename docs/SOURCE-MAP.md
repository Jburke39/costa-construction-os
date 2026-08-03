# SOURCE MAP — Costa Construction OS Prototype v0.1

Every behaviour in `index.html` traced to the canonical source that governs it,
plus an explicit list of the places where the prototype had to make an assumption
because the source records the fact as an open decision.

Source priority applied, as instructed:

1. **COS-04** controls the invoice workflow.
2. **COS-05** controls prototype functionality.
3. **COS-06** controls data, audit and permission semantics.
4. **COS-01** and **COS-02** control the organization.
5. **COS-08** identifies unresolved facts and decisions.
6. The existing baseline HTML controls the starting implementation unless it
   conflicts with the canonical documents.

## Documents read

All six were read in full, read-only, from the Drive folder
`Alex Costa → Construction_OS_Operating_Foundation_and_Invoice_Intelligence_v0.1`.
No Drive file was created, edited, moved, renamed or replaced.

| ID | Title | Drive file ID |
| --- | --- | --- |
| COS-01 | Master Operating Model | `12XM_RCakKaT1k6u2VOos91E9HxSwFKGG` |
| COS-02 | Organization, Accountability and Authority | `15ECpJgTAuxxfaj9Wv5n8OomFmHJKsMHT` |
| COS-04 | WF-001 Invoice Approval Operating Standard | `1QL2C9lkLUPkkxVUqp7Lkbxl-BQjDyS1E` |
| COS-05 | Invoice Intelligence MVP Product Specification | `1VYwYpa6KGrwSzdhotpuM_mfkXQoTG0Yw` |
| COS-06 | Invoice Data Model and Permissions | `1BSY7UP-fh4Qq5k0h4A7E-qxO0fGsKot4` |
| COS-08 | Alex Validation and Decisions Packet | `1P40Hpzygx7ETd5HylFw2WLf8adxjaHPW` |

Baseline application:
`~/.codex/.chatgpt-projects/g-p-6a4bb21d442c8191adc9674cfd2f3788/costa-construction-os/index.html`
(684 lines), imported unmodified as the first commit on `main` so that all
prototype work reads as a diff.

---

## 1. Canonical lifecycle — COS-04

COS-04: *Received → Registered → Extraction Review → Validation → Exception
Review (if needed) → Pending Project Approval → Pending Financial Approval →
Pending Executive Approval (only when configured) → Approved → Scheduled for
Payment → Paid → Closed.*

Implemented verbatim as `MAIN_STAGES` (12 states). `Exception Review` and
`Executive Approval` are marked conditional and are skipped by `nextStage()`
unless their entry condition holds — an open exception, or a configured
executive tier.

Branch states, from the COS-04 state table:
`Missing Documentation`, `Information Requested`, `On Hold`, `Disputed`,
`Rejected`, `Duplicate`, `Cancelled`, and `Superseded by revision` (COS-04:
*"Resubmission creates linked new version, never overwrites"* — the original is
retained and flagged, never edited).

**Baseline conflict resolved in favour of COS-04.** The baseline used a
9-stage path with no Received, Registered, Extraction Review, Exception Review,
Cancelled or resubmission concept. It was replaced.

## 2. Validation checks — COS-04 "Validation process"

Seventeen checks (`VC-01` … `VC-17`) map to the COS-04 validation table. Each
returns **pass**, **fail** or **not evaluated**. "Not evaluated" is used wherever
Costa has not supplied the governing fact, and is never presented as a pass.

| Check | COS-04 row | Notes |
| --- | --- | --- |
| VC-01 Vendor identity | Vendor | Presence only. Vendor master, active and blocked status are Not configured. |
| VC-02 Project scope | Project | Vendor-permitted-on-project is Not configured. |
| VC-03 Contract, PO or commitment | Contract/PO | Fails to EX-01 when absent. |
| VC-04 Cost code | Cost code | Canonical cost-code list is Not configured. |
| VC-05 Mathematics and tax | Mathematics | Recomputes gross + tax − retainage only when all three are supplied. |
| VC-06 Duplicate fingerprint | Duplicate | Vendor + number only. There is no document hash — stated in the UI. |
| VC-07 Supporting evidence | Supporting evidence | Requirement by invoice type is a prototype assumption (COS-08 Q13). |
| VC-08 Receipt or work confirmation | Supporting evidence | Prototype-defined exception PX-01. |
| VC-09 Lien waiver and insurance | Insurance/lien waiver | COS-04 requires these *only when the contract or jurisdiction requires*, which is Unknown. |
| VC-10 Change order | Change order | Declaration only; no change-order register exists. |
| VC-11 Commitment balance | Budget/commitment | Evaluated only where a sample commitment snapshot exists. |
| VC-12 Budget impact | Budget/commitment | Evaluated only where a sample budget snapshot exists. |
| VC-13 Retainage | Retainage mismatch | Always "not evaluated" — the contract retainage rule is Unknown (COS-08 Q14). |
| VC-14 Payment terms | Payment terms | Always "not evaluated" — terms are Unknown (COS-08 Q15). |
| VC-15 Vendor banking change | Banking change | Fails to EX-10 and blocks approval. |
| VC-16 Timeliness | Late invoice | Fails to EX-13 when received after the due date. |
| VC-17 Extraction confidence | Extraction | Always "not evaluated" — no document is processed. |

## 3. Exception catalog — COS-04 "Exception handling"

`EX-01` … `EX-14` are implemented with the exact label, severity, owner role and
SLA from the COS-04 table, plus the required action text.

Two **prototype-defined** codes are flagged as such everywhere they appear, in
the UI and in the CSV export:

- `PX-01` Missing receipt or work confirmation
- `PX-02` Missing required supporting evidence

COS-04 has no code for these, but its match policy requires the evidence, so the
prototype raises them rather than silently ignoring a missing control.

## 4. Blocking rule

COS-04: *"Blocking exceptions prevent approval until resolved or formally
overridden."* It does not say which severities block.

**Prototype assumption, flagged in the UI:** Critical and High block; Medium is
advisory. `authorityFor(inv,'advance')` refuses to advance while any blocking
exception is open, the button is disabled with the reason shown, and
`applyDecision` re-checks before mutating state.

Overriding is a named, logged decision restricted to Controller or Executive
Approver. Overriding a **Critical** exception additionally requires a different
human from the one who opened it — COS-04's second-review requirement.

## 5. Authority, roles and segregation of duties — COS-02 and COS-06

- Roles are the COS-06 permission-matrix roles.
- Each lifecycle state carries the accountable role; only that role (or the
  Project Executive standing in for the PM, or the Controller for payment status)
  may advance it.
- The **Auditor** role is read-only, per COS-06.
- COS-04 maker-checker: the actor who registered an invoice may not approve it,
  and no actor may record approvals at two different tiers on the same invoice.
- Denied attempts write a `permission.denied` audit event (COS-05 audit-event
  list) rather than failing silently.

### Exception resolution is scoped to the owning role

COS-04 assigns every exception type a single accountable owner. Resolving an
exception asserts that the missing evidence now exists, so **only the role that
owns that exception type may resolve it**. `authorityFor(inv, action, exception)`
takes the exception itself and checks `exception.ownerRole`.

The Controller and the Executive Approver are deliberately **not** given a blanket
resolve permission. They have the separate **override** path, which records a
named decision to proceed *without* the evidence and stores the exception as
`overridden`, never as `resolved`. The two are different facts and the audit
trail keeps them distinct.

Because EX-07 is owned by the Contracts Manager, a Contracts Manager persona was
added — otherwise that exception would have had no one able to close it.

### EX-10 bank-information change is a two-person control

COS-04: *"FBI-aligned out-of-band verification using a previously trusted contact
plus independent second human approval."* COS-02 repeats it: *"Two-person
independent verification; no email-only approval."*

EX-10 therefore cannot be resolved by any single action. It carries
`twoPersonControl: true` and closes only when **both** recorded steps exist:

1. `out_of_band_verification` — recorded by the Vendor Master Owner, with a
   rationale describing the previously known contact used.
2. `independent_review` — recorded by a **different** authorized human (Vendor
   Master Owner, Controller or Executive Approver).

Until both exist the exception stays `open` and, being Critical, keeps blocking
approval. Each step stores actor, timestamp and rationale, and each writes its own
audit event (`control.verified`, `control.second_review`). Completion writes
`exception.resolved` naming both people.

### The catalog is the authority source, never the saved record

An exception stored in a browser is a **copy**. A session written by an earlier
build can carry stale metadata, or lack a control flag that did not exist yet.
Reading `exception.twoPersonControl` off that copy would silently weaken the
control — which is exactly what happened between the first remediation and this
one.

Severity, owner, SLA, blocking status and two-person status are therefore always
derived from `EXCEPTIONS[exception.code]` through the accessors `exSeverity()`,
`exOwnerRole()`, `exSla()`, `exIsBlocking()` and `exIsTwoPerson()`. Those
accessors are used in the authority checks, the action rendering, the two-person
progress rendering, the decision application, the dashboard, the AVA output and
the CSV exports. The persisted value is only ever a fallback for a code the
catalog does not know.

### Why v2 normalization rather than a v3 migration

Bumping the schema to v3 would be worse, not better. `load()` refuses a session
whose schema version does not match and falls back to sample data — so a bump
would strand every existing v2 session, which is precisely the "silently discard
an existing session" outcome to avoid.

Instead, `normalizeState()` runs on every session that is loaded from storage
**and** every session that is imported. It is backward compatible and lossless:

- **Restored from the code**: label, severity, owner role, SLA, blocking status,
  two-person status, prototype-defined flag.
- **Guaranteed shapes**: `controlSteps` becomes an array if absent; `exceptions`,
  `comments`, `history`, `attachments`, `declarations`, `payment`, `priorities`
  and `feedback` are coerced to their expected types; `revision` defaults to 1.
- **Never touched**: the exception's detail text, status, who opened it and when,
  who resolved it and why, every recorded control step, every comment, every
  audit event, every priority, every feedback answer, and the configuration.

The repair is reported to the user on the Session data page and, for imports, in
the import result — it is not silent.

### Legacy single-action closures require remediation without rewriting history

A two-person control that an earlier build allowed one person to close is a real
problem: the control was never actually satisfied. But the closure is also a
recorded human decision, and rewriting it would destroy an audit fact.

Normalization therefore preserves the original status, actor, rationale and
audit history, but marks the control `remediationRequired`. It is treated as
effectively open and blocking until a Vendor Master Owner records the out-of-band
verification and a different authorized human records the independent review.
Completion appends `control.remediated`; it does not overwrite the historical
resolution fields.

### Authority is re-checked at confirmation

`authorityFor` is evaluated twice: when the action is offered, and again inside
`applyDecision` at the moment the human confirms. The actor, the record and the
exception can all change while the modal is open. A confirmation that is no
longer authorized is refused in the modal and written as `permission.denied`.

## 6. Executive approval tier — COS-02

COS-02's approval-authority matrix records the pilot threshold for every
category as **ALEX DECISION REQUIRED**. The prototype therefore ships the
executive step **disabled**, exposes it as configuration on the Construction OS
page, restricts the change to Controller or Executive Approver, and writes a
`configuration.changed` audit event — COS-04: *"Threshold values are
configuration records … not hard-coded application logic."*

No dollar threshold is invented anywhere.

## 7. Audit events — COS-05 and COS-06

Event names come from the COS-05 audit-event list and are classified using the
COS-06 audit-event taxonomy (`identity`, `document`, `data`, `workflow`,
`decision`, `configuration`, `integration`).

Implemented: `invoice.received`, `invoice.registered`, `extraction.skipped`,
`validation.completed`, `exception.opened`, `exception.resolved`,
`exception.overridden`, `route.assigned`, `state.changed`, `approval.approved`,
`approval.rejected`, `approval.disputed`, `approval.held`, `comment.added`,
`permission.denied`, `configuration.changed`, `payment_status.received`,
`invoice.closed`.

`extraction.completed` is deliberately **not** emitted. Nothing is extracted, so
`extraction.skipped` is recorded with an explicit note that a human typed every
field.

Every event carries actor, timestamp, prior state, new state, class and, for
intake, a correlation ID (COS-06: *"a correlation ID for one end-to-end
transaction"*).

## 8. AVA contract — COS-04 and COS-02

COS-04's AVA executive summary contract requires: vendor; project; amount; type;
PO/contract match; budget and remaining-commitment impact; exceptions and
evidence; risk level; required human decision; recommended action with reasons;
alternatives; approval deadline — and *"AVA must state uncertainty and may not
describe a recommendation as an approval."*

`avaForInvoice()` returns verdict, reasons, evidence used, required human action
and uncertainty. `avaBriefing()` produces the dashboard briefing across the five
required headings. Both are labelled **Prototype recommendation**, and both state
that the output comes from deterministic rules in the browser — no model ran, so
no confidence score is displayed.

COS-02's human-versus-AI matrix is reproduced on the Construction OS page,
including the "AVA never" column.

The sample record `INV-SAMPLE-0005` reproduces the COS-05 worked example: a
$48,200 subcontractor invoice exceeding its remaining commitment by $8,200 while
billing an unapproved change order, on which AVA recommends HOLD.

## 9. Organization — COS-01 and COS-02

All ten departments D-01 to D-10 are rendered with the COS-01 mission,
accountable executive, core KPIs and cadence. The accountability ladder is the
COS-02 structure. AVA appears as bounded support and never as an accountable
executive.

**Alex is labelled exactly as instructed:** *Alex Costa — Pilot Product Owner /
Executive Test User — actual role and authority to validate.* Nothing in the
prototype asserts that Alex is the CEO, because no source confirms it — COS-08
question 1 is still asking.

Every accountable role shows *holder unknown, to validate*.

## 10. Data model — COS-06

Invoice records carry the COS-06 core fields, plus line-level references,
declarations, commitment and budget snapshots, exceptions, comments, payment
status and an append-only history. Corrections are additive: a corrected
resubmission creates a linked new record and marks the original `superseded`.
Payment credentials are absent entirely, per COS-06's model principles.

## 11. What the baseline contributed

Preserved and improved rather than discarded:

- Single self-contained `index.html`, embedded CSS and JS, no build step.
- Browser-local `localStorage` persistence with CSV and full-session JSON export
  and import.
- Page shell, sidebar navigation, invoice drawer, decision modal, toast, role
  simulator, editable priority board and feedback capture.
- Light warm-white / black / Costa-orange visual language.
- The prominent prototype-only disclosure.

Replaced because it conflicted with a canonical document: the 9-stage lifecycle,
free-text exceptions, unenforced blocking, absent role authority, the "Alex Costa
— Executive" label, and the absence of AVA.

---

## Assumptions register — every place the prototype had to choose

Each of these is surfaced in the application itself, not only in this file.

| # | Assumption | Why it was needed | Source status |
| --- | --- | --- | --- |
| 1 | Critical and High exceptions block approval; Medium is advisory | COS-04 requires blocking behaviour but no severity cut | To validate |
| 2 | Required evidence per invoice type, derived from the COS-04 match policy | COS-08 Q13 unanswered | To validate |
| 3 | Lien waiver and insurance assumed required for subcontractor billings | COS-04 says "only when the contract requires"; contracts unknown | To validate |
| 4 | Codes PX-01 and PX-02 for missing receipt and missing evidence | No COS-04 code exists | Prototype-defined |
| 5 | An actor may not approve at two tiers on one invoice | COS-04 forbids the submitter being sole approver; silent on repeat tiers | To validate |
| 6 | Duplicate fingerprint is vendor + invoice number | COS-04 also requires a document hash; no document exists | Reduced, stated |
| 7 | Risk level derived from open exception severity plus overdue status | COS-04 lists a risk score without a formula | To validate |
| 8 | Executive tier shipped off | COS-02 threshold is ALEX DECISION REQUIRED | Not configured |
| 9 | Sample vendors, projects, amounts, commitments and budgets | A prototype needs records to demonstrate | Synthetic, labelled SAMPLE |
| 10 | Sample personas for every role except Alex | COS-02 leaves all named roles open | Synthetic, labelled sample persona |

## Facts deliberately left blank

Approval thresholds and the authority matrix; named role holders; company model;
project types and geographies; active project count; monthly invoice volume and
peak; intake channels and current AP owner; accounting and PM platforms; vendor
master; cost-code and budget structure; PO and subcontract practice; permitted
non-PO categories; required documents by type; retainage, lien-waiver and
insurance practice; payment terms; recurring invoice problems; executive
reporting expectations; legal, insurer, lender, owner and audit constraints;
retention classes; hosting boundary; identity model.

These are COS-08 Section A and B items. They render as `To validate`,
`Not configured` or `Unknown`.

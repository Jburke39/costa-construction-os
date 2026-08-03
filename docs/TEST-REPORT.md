# TEST REPORT — Costa Construction OS Prototype v0.1

| Field | Value |
| --- | --- |
| Date | 2026-08-03 |
| Repository | `/Users/jackbutrke/costa-construction-os` |
| Branch | `feature/costa-construction-os-prototype-v0-1` |
| Artefact under test | `index.html` (single file, no build step) |
| Server | `python3 -m http.server 8777` serving the repository root |
| URL | `http://localhost:8777/index.html` |
| Browser | Chromium-based in-app browser, desktop 1280×720 and mobile 390×844 |
| Result | **15 of 15 required checks pass**, plus **9 of 9 permissions checks** and **10 of 10 upgrade checks**. 8 defects found and fixed in total. |

Testing was performed against the running application in a real browser. State
assertions were read back from `localStorage`, not from in-memory variables, so
every "pass" reflects what actually persisted.

---

## Required checks

### 1. Every page loads and navigation works — PASS

All eight sections were opened by clicking their navigation buttons.

| Page | Active section | Heading | Hash |
| --- | --- | --- | --- |
| Executive dashboard | `page-dashboard` | "See the business before it surprises you." | `#dashboard` |
| Invoice intake | `page-intake` | "Register a complete invoice record." | `#intake` |
| Approval queue | `page-queue` | "Move every invoice with intent." | `#queue` |
| Exceptions | `page-exceptions` | "Nothing gets approved around a control." | `#exceptions` |
| Construction OS | `page-organization` | "The company comes before the software." | `#organization` |
| Priorities | `page-priorities` | "Turn the blueprint into owned work." | `#priorities` |
| Alex feedback | `page-feedback` | "Tell us where this is wrong." | `#feedback` |
| Session data | `page-session` | "Your session lives only in this browser." | `#session` |

Exactly one section was active at a time. Deep links (`index.html#exceptions`)
open the correct page directly.

### 2. No console errors — PASS

`read_console_messages` returned **no logs at all**, and **no errors**, across
the entire session: initial load, all page navigations, invoice creation, every
workflow decision, all six exports, seven import attempts, two resets, a role
change and a configuration change.

Network trace: the only requests are `GET /index.html` and browser-internal
`data:` URIs for the native date-picker icon. **Zero external requests.**

### 3. A new invoice can be created — PASS

- Submitting an empty form is refused, an error list is shown, focus moves to the
  first missing field, and no record is written (`countUnchanged=true`).
- A completed form created `INV-1001` and wrote it to `localStorage`.
- The record entered the workflow, the drawer opened on it, and the queue,
  dashboard and stage counters updated.

### 4. Missing evidence creates an exception — PASS

A subcontract invoice submitted with no PO, no cost code, no lien waiver, no
insurance, no progress evidence and a received date after its due date produced:

| Code | Exception | Severity | Blocking |
| --- | --- | --- | --- |
| EX-01 | Missing PO or contract | High | Yes |
| EX-02 | Wrong project or cost code | Medium | No |
| EX-07 | Missing lien waiver or insurance | High | Yes |
| EX-13 | Late invoice | Medium | No |

The record routed to **Exception Review** rather than to approval.

### 5. Valid invoices move through approval stages — PASS

A fully evidenced invoice (`INV-1002`) with a PO and cost code raised zero
exceptions and routed straight to Project Approval. It was then walked end to
end by the correct role at each step:

```
Project Approval  → Financial Approval   (Sample Project Manager)
Financial Approval→ Approved             (Sample Controller)
Approved          → Scheduled for Payment (payment status only)
Scheduled         → Paid                  (payment status only)
Paid              → Closed
```

Final audit chain (10 events, all with prior and new state):

```
invoice.received > invoice.registered > extraction.skipped > validation.completed >
route.assigned > approval.approved > approval.approved >
payment_status.received > payment_status.received > invoice.closed
```

Payment scheduling and payment status were recorded separately from approval, as
COS-04 requires. Nothing claims a payment was executed.

### 6. Blocking exceptions prevent approval — PASS

On `INV-1001`, acting as the AP Coordinator who owns the stage, the advance
control was **disabled** with:

> "2 blocking exceptions must be resolved or formally overridden first: EX-01, EX-07."

Confirmed at three layers:

1. The button is disabled and states the reason.
2. `authorityFor()` refuses the action and writes a `permission.denied` event.
3. `applyDecision()` re-checks before mutating state, so a forced call still
   refuses.

Medium-severity exceptions correctly do **not** block: `INV-SAMPLE-0009` carried
an open EX-13 and still advanced.

Additional control checks:

- **Segregation of duties.** An actor holding the PM role registered an invoice
  and was then refused its approval: *"Segregation of duties: you registered this
  invoice, so you may not be its approver (COS-04 maker-checker)."*
- **Role gating.** The Controller could not act at Executive Approval; the AP
  Coordinator could not act at Project Approval.
- **Override control.** Override is limited to Controller or Executive Approver,
  and a Critical exception cannot be overridden by the person who opened it.

### 7. Request information, hold, dispute, rejection, duplicate, resume and resolution — PASS

Every branch was exercised on a live record and read back from storage:

| Action | Result |
| --- | --- |
| Request information | → `information_requested` |
| Resume | → `project_approval` |
| Place on hold | → `on_hold` |
| Resume | → `project_approval` |
| Dispute | → `disputed` |
| Resume | → `project_approval` |
| Raise exception (EX-06) | exception opened, approval then blocked |
| Resolve exception | exception closed, 0 open remaining |
| Mark duplicate | → `duplicate` |
| Corrected / resubmitted | original → `superseded`, new `INV-1003-R2` at `received`, linked both ways |
| Reject | → `rejected` |

The original record was never overwritten by the resubmission, per COS-04.

Every action refused to proceed without a rationale (verified: confirming with an
empty rationale shows an error and changes nothing).

### 8. Every transition creates an audit event — PASS

On the fully walked invoice, **10 of 10** events carry a prior state, a new
state, an actor, a timestamp and an event class. Denied attempts also write a
`permission.denied` event. Comments write `comment.added`. Configuration changes
write `configuration.changed` to every record.

Audit export for the test session contained **87 rows**.

### 9. Comments, priorities and feedback persist after reload — PASS

Written, then the page was fully reloaded, then read back from storage **and**
confirmed visible in the re-rendered UI:

| Item | In storage | Visible after reload |
| --- | --- | --- |
| Comment on `INV-SAMPLE-0001` | yes | yes |
| Priority "PERSIST-PRIORITY-TEST" | yes | yes |
| Feedback entry | yes | yes |
| Selected role | yes (`ap1`) | yes |

Counts survived the reload exactly (13 invoices, 9 priorities, 2 feedback).
Submitting feedback with every answer blank is refused and saves nothing.

### 10. CSV and JSON exports work — PASS

All six exports generated non-empty downloads with correct filenames:

| Export | File | Size |
| --- | --- | --- |
| Full session | `costa-construction-os-session-2026-08-03.json` | 84,635 B |
| Invoices | `costa-invoices-2026-08-03.csv` | 7,906 B |
| Exceptions | `costa-exceptions-2026-08-03.csv` | 5,604 B |
| Audit events | `costa-audit-events-2026-08-03.csv` | 19,786 B |
| Priorities | `costa-priorities-2026-08-03.csv` | 1,934 B |
| Feedback | `costa-feedback-2026-08-03.csv` | 817 B |

Contents were parsed and checked, not just counted:

- Invoice CSV: 13 data rows, 34 columns including the AVA recommendation, open
  exceptions, blocked flag and data source.
- Feedback CSV: header carries all five validation questions verbatim; the typed
  answer round-tripped intact.
- Audit CSV: 87 rows, oldest first.
- Session JSON: `app: costa-construction-os`, `schemaVersion: 2`, 13 invoices.

CSV files are quoted, CRLF-terminated and BOM-prefixed so Excel opens them
correctly.

### 11. JSON import validates before replacement — PASS

Seven cases. In every rejection case the existing session was **byte-for-byte
untouched** (verified as 13/9/2 before and after).

| Case | Result |
| --- | --- |
| Not JSON | Refused: "Not valid JSON. Nothing was changed." |
| Wrong application id | Refused, names the offending id |
| Wrong schema version | Refused: "Schema version is 1; this build requires 2." |
| Missing collections | Refused, lists each missing collection |
| Malformed records | Refused: "1 invoice record is missing an id, a status or an audit history. 1 invoice record uses a state this build does not recognise." |
| Valid, user declines | "Import cancelled. Nothing was changed." |
| Valid, user confirms | Imported; counts changed 13/9/2 → 2/1/0 |

The confirmation names what is being discarded before it is discarded.

### 12. Sample reset works — PASS

- Declining the confirmation changes nothing.
- Confirming restores exactly the 9 built-in sample invoices, 8 priorities and 1
  feedback entry; all IDs are `INV-SAMPLE-*`; the selected role is preserved; the
  dashboard re-renders.

### 13. Desktop and 390px mobile without horizontal overflow — PASS

At 390×844, on all eight pages, `document.scrollWidth === window.innerWidth` —
**no page-level horizontal overflow anywhere**, including the invoice drawer
(390px wide in a 390px viewport).

The only elements wider than the viewport are the workflow stage road and the
organization chart, and both sit inside their own `overflow-x: auto` containers
and scroll independently, which is the intended behaviour for wide content.

Desktop at 1280×720 verified by screenshot on the dashboard, queue, invoice
drawer and Construction OS pages.

### 14. Controls are keyboard accessible — PASS

| Check | Result |
| --- | --- |
| All form controls labelled | 45 of 45; **0 unlabelled** |
| Positive `tabindex` values | 0 (natural tab order preserved) |
| Skip link | Focusable, target exists |
| Visible focus ring | `:focus-visible` 3px, non-brand colour for contrast |
| Drawer | `role="dialog"`, `aria-modal`, `inert` + `aria-hidden` when closed |
| Focus on drawer open | Moves to the close button |
| Focus on modal open | Moves to the rationale field |
| Focus on close | Returns to the element that opened it |
| Escape | Closes modal, then drawer, then the mobile nav, in that order |
| Mobile menu | `aria-expanded` tracked correctly through open, close and Escape |
| Status indicators | Every status pill carries a text label, never colour alone; check results show ✓ / ✕ / – plus the words Pass, Fail, Not evaluated |
| Live regions | `role="alert"` on form errors, `role="status"` on toasts and import results |

### 15. GitHub Pages relative paths work — PASS

- **Zero** absolute paths. The only `href` in the document is `#main-content`.
- **Zero** `http://` or `https://` references anywhere in the file.
- No external stylesheet, script, font or image. The single file is entirely
  self-contained.
- Served from a nested subpath (`/_ghptest/costa-construction-os/index.html`,
  simulating `https://user.github.io/costa-construction-os/`) the app loaded,
  deep-linked to `#exceptions`, rendered all 10 exceptions and the catalog, and
  navigation continued to work.
- `.nojekyll` is present.

---

## Permissions controls — added after the Codex review

Codex reviewed commit `2babbab` and raised one release-blocking permissions
defect: exception resolution was open to a broad list of roles rather than to the
exception's own owner, and EX-10 was closable by a single action. Both are fixed.
The nine checks below prove it.

Test records: `INV-SAMPLE-0008` (sample EX-10) and two invoices created during
the run — one declaring a bank-detail change, one a subcontract missing its lien
waiver.

### P1. AP Coordinator cannot resolve EX-10 — PASS

Acting as the AP Coordinator on `INV-SAMPLE-0008`:

- No **Resolve** control is rendered at all for a two-person control.
- **Record out-of-band verification** is disabled: *"Only the Vendor Master Owner
  may record out-of-band verification. You are acting as AP Coordinator."*
- EX-10 remained `open` with `blocking = true`.

### P2. Vendor Master verification alone does not close EX-10 — PASS

The Vendor Master Owner recorded out-of-band verification with a rationale.
Afterwards:

- `EX-10.status = open` — **not** resolved.
- `controlSteps = [out_of_band_verification]` — one of two.
- The invoice stayed at Exception Review.

### P3. The same actor cannot perform both actions — PASS

Still acting as the Vendor Master Owner who recorded the verification, the
independent review control is disabled:

> "Segregation of duties: you recorded the out-of-band verification, so a
> different authorized human must record the independent review."

### P4. A second authorized actor completes the control — PASS

Acting as the Controller, the independent review was recorded. EX-10 then closed:

- `status = resolved`
- `controlSteps = out_of_band_verification:vm1 | independent_review:ctl1`
- distinct actors in the two steps: **2**

### P5. EX-10 blocks approval until both actions exist — PASS

On a freshly created invoice declaring a bank-detail change:

| Point | Advance control | Reason shown |
| --- | --- | --- |
| EX-10 open, no steps | disabled | "1 blocking exception must be resolved or formally overridden first: EX-10." |
| After verification only | disabled | same |
| After the independent review | enabled | — |

The invoice then returned from Exception Review to Validation and the transition
was written to the audit history.

### P6. Both actors, timestamps, rationales and state changes are in the audit history — PASS

```
control.verified       | Sample Vendor Master Owner — Vendor Master Owner (sample persona) | 2026-08-03T12:36:39.024Z
control.second_review  | Sample Controller — Controller (sample persona)                   | 2026-08-03T12:36:39.064Z
exception.resolved     | Sample Controller — Controller (sample persona)                   | 2026-08-03T12:36:39.064Z
```

- Every step stores actor, actor label, ISO timestamp and rationale
  (`everyStepHasTimestampAndRationale = true`).
- The `exception.resolved` event names both people.
- The stored resolution reads: *"Two-person control satisfied. Out-of-band
  verification by Sample Vendor Master Owner … Independent second review by
  Sample Controller …"*
- The Exception Review → Validation transition is logged with prior and new
  state.

### P7. Other exceptions are resolvable only by their configured owner — PASS

Every persona was tested against every exception. Only the owner role can resolve:

| Exception | Owner role | Personas able to resolve |
| --- | --- | --- |
| EX-01 Missing PO or contract | AP Coordinator | AP Coordinator only |
| EX-02 Wrong project or cost code | Project Manager | Project Manager, Project Executive (holds the PM role) |
| EX-03 Duplicate suspected | AP Coordinator | AP Coordinator only |
| EX-04 Exceeds commitment | Project Executive | Project Executive only |
| EX-05 Unapproved change billing | Project Executive | Project Executive only |
| EX-07 Missing lien waiver or insurance | Contracts Manager | Contracts Manager only |
| EX-13 Late invoice | AP Coordinator | AP Coordinator only |
| PX-01 Missing receipt confirmation | Project Manager | Project Manager, Project Executive |

The Controller and Executive Approver can resolve **nothing** they do not own.
On EX-01 (owned by AP) the Controller sees Resolve disabled and Override enabled;
overriding stored the exception as `overridden`, never `resolved`, so the record
does not claim the evidence was supplied.

A Contracts Manager persona was added during this fix. Without it EX-07 would
have had no one able to close it.

### P8. Authority is re-checked at confirmation, not only when the modal opens — PASS

The Executive Approver opened the independent-review modal while authorized, the
acting role was then switched to AP Coordinator, and the decision was confirmed:

- Refused in the modal: *"Not permitted. Only the Vendor Master Owner, Controller
  or Executive Approver may record the independent review."*
- `EX-10.status` unchanged at `open`, `controlSteps` still length 1.
- A `permission.denied` event was written, worded *"…was not authorized at
  confirmation time…"* so it is distinguishable from a denial at offer time.

### P9. Read-only and non-owning roles — PASS

Enabled controls on `INV-SAMPLE-0005` (open EX-04 and EX-05, owner Project
Executive), by persona:

| Persona | Enabled controls |
| --- | --- |
| Auditor | **0** |
| Project Manager | 0 |
| Vendor Master Owner | 0 |
| Contracts Manager | 0 |
| Project Executive | resolve EX-04, resolve EX-05 |
| AP Coordinator | 7 stage actions, no resolve |
| Controller | 10, including both overrides, no resolve |
| Executive Approver | 9, including both overrides, no resolve |

Re-running validation is now restricted to the AP Coordinator and the Controller.

### Regression after the permissions change

Re-verified: full lifecycle Validation → Project → Financial → Approved →
Scheduled → Paid → Closed; all eight pages render; no horizontal overflow at
390px; zero console errors; sample reset restores the shipped state.

## Upgrade path — added after the second Codex review

Codex verified the EX-10 workflow on a fresh session but found that **existing
schema-v2 sessions were not upgraded**. Their saved EX-10 records predate the
`twoPersonControl` flag, and the code read that flag off the persisted record —
so a returning user still got an enabled single-action "Resolve EX-10" button
and no two-person controls. Correct finding, and a genuine release blocker: the
people most likely to hit it are exactly the people who had already been testing.

Fixed by (a) deriving all control metadata from the canonical catalog rather than
the saved copy, and (b) normalizing every loaded and imported session.

Test fixture: a session written by hand in the exact shape a pre-remediation
build (`2babbab`) produced — an EX-10 with **no** `twoPersonControl` and **no**
`controlSteps`, plus comments, history, priorities and feedback to prove nothing
is lost.

### U1. The existing session is loaded, not discarded — PASS

```
invoices=1  priorities=1  feedback=1  schemaVersion=2  actor=vm1  execThreshold=30000
```

The user's role selection and executive-approval configuration survived.

### U2. The exception is normalized from the canonical catalog — PASS

| Field | Before (as saved) | After load |
| --- | --- | --- |
| `twoPersonControl` | *absent* | `true` |
| `controlSteps` | *absent* | `[]` |
| `blocking` | true | true |
| `ownerRole` | vendorMaster | vendorMaster |
| `severity` | Critical | Critical |
| `status` | open | open |

### U3. Existing user data is intact — PASS

Comment, audit event, priority, feedback entry, exception detail text,
`openedBy`, `openedAt` and the attachment name all survived byte-for-byte.

### U4. No "Resolve EX-10" action appears — PASS

`resolveButtonExists = false`. The Vendor Master Owner's controls on the legacy
record are:

```
verify_banking(ENABLED)  independent_review(disabled)  override_exception(disabled)
```

### U5. Vendor Master sees only out-of-band verification — PASS

- `verify_banking` enabled.
- `independent_review` disabled: *"The Vendor Master Owner must record
  out-of-band verification before an independent review can be recorded."*
- The two-person progress block renders (2 steps, both outstanding).
- The repair notice is shown to the user, not hidden.

### U6. EX-10 blocks approval on the legacy record — PASS

Acting as the AP Coordinator who owns the stage:
*"1 blocking exception must be resolved or formally overridden first: EX-10."*

### U7. The second independent action is still required — PASS

| Step | Result |
| --- | --- |
| Vendor Master records verification | `ok` |
| EX-10 after verification | still `open`, 1 of 2 steps |
| AP advance | still blocked |
| Same actor attempts step 2 | refused — segregation of duties |
| Controller records independent review | `ok` |
| EX-10 | `resolved`, 2 distinct actors |
| Invoice | left Exception Review for Validation |
| Legacy comment | still present |

Audit chain on the upgraded record:

```
invoice.received > control.verified > control.second_review > exception.resolved > state.changed
```

### U8. Pre-remediation *exports* are upgraded on import — PASS

Importing an export file written by the old build:

> "Session imported. 2 invoices, 1 priority and 0 feedback entries replaced the
> previous session. **Upgraded on import:** 2 exceptions in this saved session
> carried stale or missing control metadata and were restored from the canonical
> COS-04 catalog…"

The imported open EX-10 came through with `twoPersonControl=true`,
`controlSteps=[]`, `blocking=true`. Imported history and priorities preserved.

### U9. A legacy single-action closure is blocked pending remediation — PASS

The fixture included an EX-10 that the old build let one person "resolve" with
the rationale *"Looked fine to me."* and no control steps.

- The recorded decision is **preserved exactly**: status `resolved`, original
  `resolvedBy`, original resolution text. It is not rewritten.
- `legacySingleActionClose = true` and `remediationRequired = true` are set.
- The invoice remains approval-blocking until both remediation steps exist.
- The invoice drawer shows: *"Historically closed — remediation required."*
- Completing both steps appends `control.remediated` and preserves the original
  resolution actor, timestamp and rationale.
- The exceptions CSV carries a "Closed by single action under an earlier build"
  column.

### U10. Imported open EX-10 still needs two people — PASS

```
resolveExists=false  verifyEnabled=true  independentEnabled=false
```

### Regression after the upgrade change

On a fresh sample session: all eight pages render, no horizontal overflow at
390px, zero console errors, owner scoping still exact (EX-01 → AP only, EX-04 →
Project Executive only, PX-01 → Project Manager and Project Executive), full
lifecycle Project → Financial → Approved → Scheduled → Paid → Closed with 10
audit events, and all six exports non-empty.

## Defects found and fixed

All six were found by this testing and re-verified after the fix.

| # | Defect | Severity | Fix |
| --- | --- | --- | --- |
| 1 | AVA briefing produced ungrammatical output ("5 critical exceptions is open", "8 still opens") | Cosmetic, but it undermines executive credibility | Added a verb-agreement helper and corrected the affected sentences |
| 2 | Topbar title included the navigation icon glyph ("▦Executive dashboard") | Cosmetic | Title now reads only the text nodes of the nav button |
| 3 | Queue row sub-text ran inline with the primary text ("INV-SAMPLE-0002SAMPLE-RM-77814") | Readability | `.cell-stack` made a flex column; `.invoice-id` / `.subtext` set to block |
| 4 | Validation check titles ran inline with their detail ("VC-01 Vendor identityVendor name recorded…") | Readability | `.check-row strong` and `.check-detail` set to block |
| 5 | Actor avatar was clipped off the right edge at 390px | Layout | Avatar hidden below 560px; the role select takes the space |
| 6 | The invoice drawer stayed open when navigating to another page | Behaviour | `navigate()` now closes an open drawer |
| 8 | Existing schema-v2 sessions were not upgraded: saved EX-10 records lacked `twoPersonControl`, so a returning user still got a single-action "Resolve EX-10" button | **Release-blocking**, raised by the second Codex review of `efdbf22` | Control metadata is now always derived from the canonical catalog, never from the persisted copy; every loaded and imported session is normalized; legacy single-action closures are flagged rather than silently accepted |
| 7 | Exception resolution was open to a broad role list, and EX-10 could be closed by one person | **Release-blocking permissions defect**, raised by the Codex review of `2babbab` | Resolution scoped to `exception.ownerRole`; EX-10 made a two-person control; authority re-checked at confirmation; Contracts Manager persona added; the "Re-run validation" control is now visibly disabled for roles that may not use it |

One further change was made for correctness rather than as a defect fix: the seed
session is now written to storage on first load, so a reload before any edit
returns the same session instead of a freshly generated one.

## Not defects

- **Black screenshots partway through testing.** Caused by the Browser pane being
  hidden by the tooling, confirmed by the tool's own error message and by DOM
  geometry checks showing the page laid out correctly. Not an application fault.
- **"Approval queue 0" while acting as Alex.** Correct. Alex holds the Executive
  Approver role, the executive tier ships disabled, so no invoice is awaiting
  him. Switching to the Controller or Project Manager shows work.

## Coverage against the COS-05 acceptance suite

The prototype is not the MVP, so the 20 COS-05 acceptance tests cannot pass in
full — several require ground-truth documents, real extraction, notifications and
authentication, none of which exist here. The scenarios that the prototype *can*
demonstrate were exercised above:

| COS-05 test | Demonstrated |
| --- | --- |
| AT-001 valid invoice, full match | Yes — check 5 |
| AT-002 missing PO | Yes — EX-01 |
| AT-003 wrong project / cost code | Yes — EX-02 |
| AT-004 duplicate | Yes — EX-03 on `INV-SAMPLE-0007` |
| AT-005 / AT-006 exceeds PO or commitment | Yes — EX-04 |
| AT-007 unapproved change-order billing | Yes — EX-05 |
| AT-009 / AT-010 missing lien waiver or insurance | Yes — EX-07 |
| AT-011 vendor banking change | Yes — EX-10 on `INV-SAMPLE-0008` |
| AT-012 requires executive approval | Yes — configured tier, check under "Executive routing" |
| AT-014 mathematical discrepancy | Yes — EX-11 when gross, tax and retainage are supplied |
| AT-015 rejected | Yes — check 7 |
| AT-016 corrected and resubmitted | Yes — linked revision, original superseded |
| AT-017 disputed | Yes — check 7 |
| AT-018 on hold | Yes — check 7 |
| AT-019 cancelled | Yes — Controller/Executive only |
| AT-020 unauthorized approval attempt | Yes — denied and logged as `permission.denied` |
| AT-008 incorrect retainage | **No** — the contract retainage rule is Unknown, so the check reports "not evaluated" |
| AT-013 low-confidence extraction | **No** — nothing is extracted; reported honestly as "not evaluated" |

## Executive routing check

With the executive tier disabled (the shipped default), no invoice routes to an
executive. After the Controller enabled it at $25,000 in the prototype
configuration panel:

- `INV-SAMPLE-0001` at $68,900 routed Project → Financial → **Executive
  Approval**, where only Alex could act; the Controller was refused.
- `INV-SAMPLE-0009` at $15,650 skipped the executive step and went Financial →
  **Approved**.
- The change was written as a `configuration.changed` audit event.
- The AP Coordinator could not open the configuration at all.

The default was restored to disabled before committing.

## How to reproduce

```bash
cd ~/costa-construction-os && python3 -m http.server 8777
```

Then open `http://localhost:8777/index.html`. Use **Session data → Reset to
sample data** to return to the shipped state at any point.

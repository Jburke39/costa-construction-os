# BUILD STATUS — Costa Construction OS Prototype v0.1

Live status log. Updated at four milestones: (1) Drive sources inspected,
(2) first working build, (3) browser verification completed, (4) final local commit.

Nothing has been pushed, merged, deployed, published, or shared. No Google Drive
file has been created, edited, moved, renamed, or replaced — Drive was read only.

---

## Milestone 1 — Drive sources inspected

| Field | Value |
| --- | --- |
| Timestamp | 2026-08-03 07:42:39 EDT |
| Repository path | /Users/jackbutrke/Desktop/costa-construction-os |
| Branch | feature/costa-construction-os-prototype-v0-1 |
| Current commit | 696e105 (baseline import) |

**Completed**

- Located the baseline application at
  `/Users/jackbutrke/.codex/.chatgpt-projects/g-p-6a4bb21d442c8191adc9674cfd2f3788/costa-construction-os/index.html`
  (684 lines, ~80 KB, single self-contained file) and read it end to end.
- Read all six required Drive references, in full, read-only:
  COS-04 (canonical invoice workflow), COS-05 (MVP product specification),
  COS-06 (invoice data model and permissions), COS-01 (master operating model),
  COS-02 (organization and authority), COS-08 (Alex validation packet).
- Confirmed no pre-existing Git repository anywhere for this work. Created a new
  local repository, imported the baseline unmodified as commit 696e105 so that all
  prototype work appears as a reviewable diff, and branched to
  `feature/costa-construction-os-prototype-v0-1`.
- Completed a gap analysis of the baseline against the canonical documents.

**Gap analysis — baseline vs. canonical sources**

| Gap | Source that governs |
| --- | --- |
| No AVA at all (no briefing, no per-invoice recommendation, no evidence or uncertainty) | COS-04 AVA executive summary contract; COS-05 FR-011 |
| 9-stage lifecycle does not match the 12-state canonical lifecycle; Received, Registered, Extraction Review, Exception Review, Cancelled and Corrected/Resubmitted are all absent | COS-04 canonical lifecycle |
| Exceptions are free-text strings with no code, severity, owner or SLA | COS-04 EX-01…EX-14 |
| Blocking exceptions do not actually block approval (an invoice at Validation carrying an exception can still be advanced) | COS-04; COS-05 core user stories |
| No validation-check results surfaced anywhere | COS-04 validation process |
| No role/authority enforcement and no segregation of duties | COS-02; COS-06 permission matrix |
| No Construction OS organization view (D-01…D-10) | COS-01; COS-02 |
| Alex is labelled "Executive", which asserts authority the sources do not confirm | COS-08 item 1 is still an open question |
| Priorities board has no Outcome or Open decision field | Required experience |
| No structured Alex feedback capture (the five validation questions) | COS-08 |
| No reset to sample data; storage key is not schema-versioned | Persistence requirements |
| No invoice type, received date, supporting-document declarations or upload control | COS-04 required invoice information |

**In progress**

- Rewriting `index.html` against the canonical lifecycle, exception catalog,
  permission model and AVA contract.

**Blocked**

- Nothing.

**Files changed**

- `index.html`, `README.md`, `.nojekyll`, `SOURCE-AUDIT.md` — imported from the
  baseline, unmodified, at commit 696e105.
- `docs/BUILD-STATUS.md` — this file (new, uncommitted).

**Tests run and results**

- None yet. Browser verification runs at milestone 3.

**Decisions needed from Jack**

1. Repository location. No repo existed, so one was created at
   `~/Desktop/costa-construction-os`. Confirm or tell me where it belongs.
2. Every Costa-specific fact is genuinely unknown in the sources — approval
   thresholds, named employees, invoice volumes, evidence rules, accounting
   platform. These are rendered as `To validate` / `Not configured` / `Unknown`
   rather than invented. Confirm that is the behaviour you want in front of Alex.
3. COS-02 leaves the executive-approval tier as "ALEX DECISION REQUIRED", so the
   prototype ships that step disabled and configurable rather than guessing a
   dollar threshold.

---

## Milestone 2 — First working build

| Field | Value |
| --- | --- |
| Timestamp | 2026-08-03 08:16:11 EDT |
| Repository path | /Users/jackbutrke/costa-construction-os |
| Branch | feature/costa-construction-os-prototype-v0-1 |
| Current commit | 696e105 (baseline import; prototype work still uncommitted) |

**Completed**

- Rewrote `index.html` as a single self-contained file, 2995 lines, no build
  step and no dependencies. JavaScript parses clean under `node --check`.
- Implemented the COS-04 canonical 12-state lifecycle in full, plus the eight
  off-path states including Cancelled and Superseded-by-revision.
- Implemented 17 validation checks that return pass, fail or **not evaluated** —
  "not evaluated" is used wherever Costa has not supplied the governing fact and
  is never presented as a pass.
- Implemented the COS-04 exception catalog EX-01 to EX-14 with the exact
  severity, owner role and SLA, plus two prototype-defined codes flagged as such
  everywhere they appear.
- Implemented blocking: Critical and High exceptions prevent approval at three
  layers — disabled control with a stated reason, an authority check that logs a
  `permission.denied` event, and a re-check before any state mutation.
- Implemented COS-06 role permissions, the read-only Auditor, and COS-04
  maker-checker segregation of duties.
- Implemented AVA to the COS-04 summary contract: verdict, reasons, evidence
  used, required human action and uncertainty, on both the dashboard and every
  invoice — labelled a prototype recommendation, never an approval.
- Built eight screens: Executive dashboard, Invoice intake, Approval queue,
  Exceptions, Construction OS, Priorities, Alex feedback, Session data.
- Built the COS-01 organization view, all ten departments D-01 to D-10, with the
  COS-02 human-versus-AVA authority matrix.
- Shipped the executive approval tier **disabled and configurable** rather than
  inventing the threshold COS-02 leaves as ALEX DECISION REQUIRED.
- Nine synthetic sample invoices covering valid full match, missing PO, missing
  cost code, duplicate, commitment overrun, unapproved change order, vendor
  bank-detail change, late invoice and a fully paid record.

**In progress**

- Browser verification.

**Blocked**

- Nothing. One environment obstacle was worked around: the sandboxed static
  server cannot read `~/Desktop` (macOS TCC protection), so the repository was
  moved to `~/costa-construction-os`. Flagged for Jack below.

**Files changed**

- `index.html` — rewritten (uncommitted)
- `docs/BUILD-STATUS.md` — this file (new)

**Tests run and results**

- `node --check` on the extracted script: **pass**.
- Full browser verification not yet run.

**Decisions needed from Jack**

1. Repository location moved from `~/Desktop/costa-construction-os` to
   `~/costa-construction-os` because the sandboxed test server is denied read
   access to Desktop. Confirm or relocate.
2. Confirm the two prototype-defined exception codes (PX-01, PX-02) are wanted at
   all, or whether COS-04 should be amended to carry them.

---

## Milestone 3 — Browser verification completed

| Field | Value |
| --- | --- |
| Timestamp | 2026-08-03 08:16:11 EDT |
| Repository path | /Users/jackbutrke/costa-construction-os |
| Branch | feature/costa-construction-os-prototype-v0-1 |
| Current commit | 696e105 (prototype work still uncommitted at time of testing) |

**Completed**

- Served the application at `http://localhost:8777/index.html` and tested it in a
  real browser at desktop 1280x720 and mobile 390x844.
- Ran all 15 required verification checks. **15 of 15 pass.**
- Found six defects during testing, fixed all six, and re-verified each.
- State assertions were read back from `localStorage` rather than from in-memory
  variables, so every pass reflects what actually persisted.

**Verification results**

| # | Check | Result |
| --- | --- | --- |
| 1 | Every page loads, navigation works | Pass — 8 of 8 pages, deep links included |
| 2 | No console errors | Pass — zero console messages, zero external network requests |
| 3 | A new invoice can be created | Pass — empty submit correctly refused |
| 4 | Missing evidence creates an exception | Pass — EX-01, EX-02, EX-07, EX-13 raised |
| 5 | Valid invoices move through approval stages | Pass — walked Project to Closed, 10 audit events |
| 6 | Blocking exceptions prevent approval | Pass — enforced at three layers; SoD and role gating also verified |
| 7 | Request info, hold, dispute, reject, duplicate, resume, resolve | Pass — all eleven transitions |
| 8 | Every transition creates an audit event | Pass — 10 of 10, plus denied attempts logged |
| 9 | Comments, priorities, feedback persist after reload | Pass — in storage and re-rendered |
| 10 | CSV and JSON exports work | Pass — all six, contents parsed and checked |
| 11 | JSON import validates before replacement | Pass — 5 rejection paths, decline path, accept path |
| 12 | Sample reset works | Pass — decline and confirm both correct |
| 13 | Desktop and 390px mobile, no horizontal overflow | Pass — scrollWidth equals viewport on all 8 pages |
| 14 | Controls keyboard accessible | Pass — 45 of 45 labelled, focus managed, Escape ordered |
| 15 | GitHub Pages relative paths | Pass — zero absolute paths, verified from a nested subpath |

**Defects found and fixed**

1. AVA briefing produced ungrammatical output ("5 critical exceptions is open").
2. Topbar title included the navigation icon glyph.
3. Queue row sub-text ran inline with the primary text.
4. Validation check titles ran inline with their detail text.
5. Actor avatar was clipped off the right edge at 390px.
6. The invoice drawer stayed open when navigating to another page.

One correctness change, not a defect fix: the seed session is now written to
storage on first load, so a reload before any edit returns the same session.

**In progress**

- Documentation and the local commit.

**Blocked**

- Nothing.

**Files changed**

- `index.html` — rewritten and repaired
- `README.md` — rewritten
- `SOURCE-AUDIT.md` — header added marking it superseded by docs/SOURCE-MAP.md
- `docs/SOURCE-MAP.md`, `docs/TEST-REPORT.md`, `docs/PROTOTYPE-LIMITATIONS.md` — new
- `docs/BUILD-STATUS.md` — updated
- `.gitignore` — new

**Tests run and results**

Full results in [TEST-REPORT.md](TEST-REPORT.md). Summary: 15 of 15 required
checks pass; 6 defects found, fixed and re-verified; zero console errors.

**Decisions needed from Jack**

Unchanged from milestone 2, plus: review the seven prototype assumptions listed
in the assumptions register of `docs/SOURCE-MAP.md` before this goes to Alex.

---

## Milestone 4 — Final local commit completed

| Field | Value |
| --- | --- |
| Timestamp | 2026-08-03 08:18:52 EDT |
| Repository path | /Users/jackbutrke/costa-construction-os |
| Branch | feature/costa-construction-os-prototype-v0-1 |
| Current commit | bb29d4b (`bb29d4b38f6cc8015eb7f28d90733f685d76e129`) |
| Parent | 696e105 — unmodified baseline import |

**Completed**

- All implementation, verification, defect repair and documentation.
- Committed locally as bb29d4b. Working tree clean at the time of commit.
- `git status --short` immediately after commit: empty.

**In progress**

- Nothing.

**Blocked**

- Nothing.

**Files changed in this commit**

```
  .gitignore                    |    2 +
  README.md                     |  286 +++-
  SOURCE-AUDIT.md               |    8 +
  docs/BUILD-STATUS.md          |  229 +++
  docs/PROTOTYPE-LIMITATIONS.md |  152 ++
  docs/SOURCE-MAP.md            |  249 +++
  docs/TEST-REPORT.md           |  340 ++++
  index.html                    | 3420 ++++++++++++++++++++++++++++++++++-------
  8 files changed, 4064 insertions(+), 622 deletions(-)
```

**Tests run and results**

15 of 15 required browser checks pass. Zero console errors. Zero external
network requests. Six defects found during testing, all fixed and re-verified.
Full detail in [TEST-REPORT.md](TEST-REPORT.md).

**Nothing was pushed, merged, deployed, published or shared.** No Git remote is
configured. No Google Drive file was created, edited, moved, renamed or
replaced — Drive was read-only throughout.

**Decisions needed from Jack**

1. **Repository location.** No repository existed, so one was created. It sits at
   `~/costa-construction-os` rather than `~/Desktop` because the sandboxed test
   server is denied read access to Desktop by macOS privacy protection. Confirm
   or relocate.
2. **The seven prototype assumptions** in the assumptions register of
   `docs/SOURCE-MAP.md` — most importantly that Critical and High exceptions
   block approval while Medium does not. COS-04 requires blocking but does not
   state the severity cut.
3. **The two prototype-defined exception codes** PX-01 and PX-02. Either accept
   them or amend COS-04 to carry equivalents.
4. **Executive approval tier.** Shipped disabled. It needs a real, effective-dated
   threshold from Alex before the pilot, per COS-02.
5. **Whether to push and publish.** Nothing has been pushed. GitHub Pages would
   make the prototype reachable but would not make any data shared — worth being
   explicit with Alex about that before sending a link.

**Decisions needed from Alex**

1. COS-08 Section A, all 18 questions. The prototype makes their absence visible
   rather than guessing at answers.
2. Confirmation of his own role and decision authority — COS-08 question 1. He is
   labelled Pilot Product Owner / Executive Test User until he says otherwise.
3. The approval authority matrix and every monetary threshold.
4. Whether the required supporting documents per invoice type, and the lien
   waiver and insurance assumptions, match how Costa actually contracts.

---

## Milestone 5 — Codex review remediation

| Field | Value |
| --- | --- |
| Timestamp | 2026-08-03 08:41:44 EDT |
| Repository path | /Users/jackbutrke/costa-construction-os |
| Branch | feature/costa-construction-os-prototype-v0-1 |
| Reviewed commit | 2babbabbd9cc59d0dc0b4d7a29a0fa6b08a7e5ff |

Codex independently reviewed `2babbab` and found the build substantially correct
with **one release-blocking permissions defect**. The finding was correct and is
fixed. It mattered: as shipped, any of six roles could mark a control satisfied,
and the FBI-aligned banking-change control — the one that exists specifically to
stop payment-diversion fraud — could be closed by a single person.

**Completed**

1. **Exception resolution scoped to the owning role.** `authorityFor()` now takes
   the exception and checks `exception.ownerRole`. The previous broad list
   (AP, Controller, PM, PX, Vendor Master, Contracts) is gone.
2. **Controller and Executive Approver no longer resolve.** They keep the separate
   formal-override path, which stores the exception as `overridden`, never
   `resolved`, so the record never implies they supplied missing evidence.
3. **EX-10 made a two-person control.** It closes only when both
   `out_of_band_verification` (Vendor Master Owner) and `independent_review`
   (a different authorized human) are recorded, each with actor, timestamp and
   rationale, each writing its own audit event.
4. **EX-10 stays open and approval-blocking** until both actions exist.
5. **Authority re-checked at confirmation.** `applyDecision()` re-evaluates
   `authorityFor()` at the moment the human confirms, refuses in the modal, and
   logs a `permission.denied` event worded to distinguish it from a denial at
   offer time.
6. **Nine permissions tests added** and passing — see TEST-REPORT.md, section
   "Permissions controls".
7. Docs updated: TEST-REPORT.md, SOURCE-MAP.md and this file.

**Two supporting changes the fix required**

- **Contracts Manager persona added.** EX-07 is owned by the Contracts Manager and
  no persona held that role, so once resolution was correctly scoped, EX-07 would
  have been unresolvable by anyone. Eight personas now.
- **Re-run validation control gated in the markup.** It was already refused on
  click, but rendered enabled for roles that could not use it. It is now visibly
  disabled with the reason, and restricted to AP Coordinator and Controller.

**Blocked**

- Nothing.

**Files changed**

- `index.html` — authority model, two-person control, confirm-time recheck, UI
- `docs/TEST-REPORT.md` — nine permissions checks, seventh defect recorded
- `docs/SOURCE-MAP.md` — corrected permission model documented
- `docs/BUILD-STATUS.md` — this entry

**Tests run and results**

15 of 15 original checks still pass after regression. 9 of 9 new permissions
checks pass:

| # | Check | Result |
| --- | --- | --- |
| P1 | AP Coordinator cannot resolve EX-10 | Pass |
| P2 | Vendor Master verification alone does not close EX-10 | Pass |
| P3 | The same actor cannot perform both actions | Pass |
| P4 | A second authorized actor completes the control | Pass |
| P5 | EX-10 blocks approval until both actions exist | Pass |
| P6 | Both actors, timestamps, rationales and state changes in audit history | Pass |
| P7 | Other exceptions resolvable only by their configured owner | Pass |
| P8 | Authority re-checked at confirmation, not only at modal open | Pass |
| P9 | Auditor and non-owning roles have zero enabled controls | Pass |

Zero console errors. No horizontal overflow at 390px. Sample data reset to the
shipped state before committing.

**Nothing was pushed, merged, deployed, published or shared.** No Google Drive
file was touched. No global configuration was changed during this remediation —
`/Users/jackbutrke/.claude/launch.json` was left exactly as it was.

**Decisions needed from Jack**

Unchanged from milestone 4, plus two arising from this fix:

1. **Who may record the independent second review on EX-10?** The prototype
   accepts the Vendor Master Owner, Controller or Executive Approver, provided it
   is a different person from the verifier. COS-04 says "second human approval"
   without naming the role. Confirm the real list.
2. **The Contracts Manager role** now exists in the prototype solely because
   COS-04 assigns EX-07 to it. Confirm Costa actually has that role, or reassign
   EX-07 to whoever really chases lien waivers and insurance certificates.

---

## Milestone 6 — Second Codex review remediation (session upgrade)

| Field | Value |
| --- | --- |
| Timestamp | 2026-08-03 08:56:53 EDT |
| Repository path | /Users/jackbutrke/costa-construction-os |
| Branch | feature/costa-construction-os-prototype-v0-1 |
| Reviewed commit | efdbf22c8d6a61d2917381788362b9f01f9c05b9 |

Codex confirmed the EX-10 two-person control works on a **fresh** session, and
found the remaining blocker: **existing schema-v2 sessions were not upgraded.**
Saved EX-10 records predate the `twoPersonControl` flag, and the code read that
flag off the persisted record — so a returning tester still saw an enabled
single-action "Resolve EX-10" button and no two-person controls.

Correct finding, and the worst possible audience for it: the people with an
existing session are exactly the people who have already been testing.

**Completed**

1. **Persisted control metadata is never the authority source.** Added canonical
   accessors — `exSeverity`, `exOwnerRole`, `exSla`, `exLabel`, `exIsBlocking`,
   `exIsTwoPerson`, `exSteps`, `exStepDone` — all resolving through
   `EXCEPTIONS[exception.code]`. The saved value is only a fallback for a code
   the catalog does not know.
2. **Every read switched to those accessors**: authority checks, exception action
   rendering, two-person progress rendering, decision application, blocking
   calculation, risk derivation, dashboard, project risk, AVA briefing and
   per-invoice AVA, exceptions page, and the CSV exports.
3. **`normalizeState()` runs on every loaded and every imported session.** It
   restores label, severity, owner, SLA, blocking and two-person status from the
   code; guarantees `controlSteps` is an array; and coerces every collection to
   its expected shape. User-authored content is untouched.
4. **Kept schema v2 deliberately, and documented why.** `load()` discards a
   session whose schema version does not match, so bumping to v3 would strand
   every existing session — the exact outcome to avoid. Backward-compatible
   normalization upgrades in place instead, and reports what it repaired.
5. **Legacy single-action closures are blocked pending remediation, without
   rewriting history.** The original status, actor and rationale remain intact,
   while the invoice is treated as blocked until both required human steps are
   recorded. Completion appends `control.remediated`.
6. **Ten upgrade regression tests added and passing**, covering both a
   browser-loaded legacy session and an imported pre-remediation export.

**Blocked**

- Nothing.

**Files changed**

- `index.html` — canonical accessors, normalization, legacy-closure flag, UI warning
- `docs/TEST-REPORT.md` — ten upgrade checks, eighth defect recorded
- `docs/SOURCE-MAP.md` — catalog-as-authority, v2-normalization rationale, legacy closures
- `docs/BUILD-STATUS.md` — this entry

**Tests run and results**

| # | Check | Result |
| --- | --- | --- |
| U1 | Existing session loaded, not discarded | Pass |
| U2 | Exception normalized from the canonical catalog | Pass |
| U3 | Existing comments, history, priorities, feedback intact | Pass |
| U4 | No "Resolve EX-10" action appears | Pass |
| U5 | Vendor Master sees only out-of-band verification | Pass |
| U6 | EX-10 blocks approval on the legacy record | Pass |
| U7 | The second independent action is still required | Pass |
| U8 | Pre-remediation exports upgraded on import | Pass |
| U9 | Legacy single-action closure flagged, not silently accepted | Pass |
| U10 | Imported open EX-10 still needs two people | Pass |

Regression: 15 of 15 original checks and 9 of 9 permissions checks still pass.
Zero console errors. No horizontal overflow at 390px. Sample data reset to the
shipped state before committing.

**Nothing was pushed, merged, deployed, published or shared.** No Google Drive
file was touched. No global configuration was changed —
`/Users/jackbutrke/.claude/launch.json` was not modified in this remediation.

**Decision applied**

- Legacy single-action EX-10 closures remain preserved as historical facts but
  are blocked pending completion of the two-person remediation control.

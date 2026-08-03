# Costa Construction OS — Prototype v0.1

A single-file, browser-local prototype of the Construction OS invoice workflow,
built for Alex Costa to open, understand, test and argue with.

> **Prototype only.** Data is saved in this browser and is not shared with Jack,
> Alex, Google Drive, or a company system. Export the session to share it.
> There is no authentication, no live company data, no document processing, no AI
> model, no accounting integration, and no payment execution. Every sample record
> is synthetic and labelled `SAMPLE`.

**This is not the Construction OS.** It is one workflow — WF-001 invoice
approval — rendered concretely enough to be corrected.

---

## What it demonstrates

| Area | What you can do |
| --- | --- |
| **AVA executive intelligence** | An executive briefing and a per-invoice recommendation, each stating what needs attention, why it matters, the evidence used, the recommended human action, and what is missing or uncertain. AVA never approves anything. |
| **Invoice intake and approval** | Register a synthetic invoice with vendor, project, type, amount, all three dates, PO or contract reference, cost code, description, supporting-document declarations and a project approver. Then move it through the canonical workflow. |
| **Exception management** | Typed exceptions with the COS-04 code, severity, accountable role and target time. Critical and High block approval until a human resolves or formally overrides them. |
| **Executive monitoring** | Invoices requiring attention, pending approval dollars, exceptions, overdue items, project risks, today's priorities, recent activity and approval bottlenecks. |
| **The Construction OS organization** | The ten departments from COS-01, the authority ladder from COS-02, and AVA shown as bounded support that is never accountable. |
| **Business priorities and progress** | An editable board with title, outcome, owner, priority, status, progress and the open decision each item waits on. |
| **Alex's feedback** | Five structured questions, saved locally and included in both exports. |

## The workflow it implements

The canonical lifecycle from **COS-04**, in full:

```
Received → Registered → Extraction Review → Validation →
Exception Review (when required) → Project Approval → Financial Approval →
Executive Approval (only when configured) → Approved →
Scheduled for Payment → Paid → Closed
```

Plus the off-path states: Missing Documentation, Information Requested, On Hold,
Disputed, Rejected, Duplicate, Cancelled, and Superseded by a corrected
resubmission — which creates a linked new record and never overwrites the
original.

Every material action requires a rationale and records the prior state, the new
state, the actor and the timestamp.

## Run it locally

No build step, no dependencies, no install.

```bash
cd ~/costa-construction-os && python3 -m http.server 8777
```

Then open <http://localhost:8777/index.html>.

Opening `index.html` directly by double-clicking also works, with one caveat:
some browsers restrict `localStorage` on `file://` URLs, so nothing will persist.
Use the local server if you want your session to survive a reload.

## Test it in fifteen minutes

1. **Dashboard.** Read the AVA briefing. Note the "Missing information and
   uncertainty" panel — that list is the real output of this exercise.
2. **Approval queue → `INV-SAMPLE-0005`.** A $48,200 subcontractor invoice that
   exceeds its remaining commitment and bills an unapproved change order. Read
   the validation checks, the exceptions and the AVA recommendation. Try to
   approve it: you cannot, and the screen says exactly why.
3. **Switch role.** Use "Acting as" in the top bar. As the Controller you can
   override a control; as the AP Coordinator you cannot. As the Auditor you can
   do nothing at all. Try to approve something at the wrong stage and read the
   refusal.
4. **Invoice intake.** Create an invoice with no PO and no cost code, and watch
   EX-01 and EX-02 open. Then create one with everything supplied and walk it all
   the way to Closed.
5. **Exceptions.** See every open exception in one place, with the catalog
   underneath. The two codes marked *prototype-defined* are ones the prototype
   invented because COS-04 has no code for them — those especially need your
   verdict.
6. **Construction OS.** Check the ten departments against how the company
   actually works. Note that the executive approval tier ships **switched off**,
   because COS-02 still records every threshold as *ALEX DECISION REQUIRED*.
7. **Alex feedback.** Record what is wrong.
8. **Session data → Export.** Send Jack the file.

Use **Session data → Reset to sample data** whenever you want a clean start.

## How local storage works

- Your session is saved in this browser under `costa-construction-os.v2`, using
  `localStorage`. It never leaves the machine.
- It persists across reloads and across closing the browser.
- **Clearing your browser data deletes it permanently.** Private and incognito
  windows, "clear site data", and some privacy extensions will remove it.
- Different browsers, devices and profiles each hold a completely separate
  session. There is no sync.
- If the browser refuses to write, the app tells you and asks you to export
  rather than pretending it saved.
- The stored schema version is checked on load. A session written by a different
  version is not migrated and not deleted — sample data loads instead and the
  Session data page explains why.

## How Alex exports a session

**Session data → Export**, then send the file to Jack yourself. Nothing is
transmitted by the application.

| Export | Contains |
| --- | --- |
| Full session JSON | Everything — invoices, exceptions, comments, audit events, priorities, feedback, configuration |
| Invoices CSV | One row per invoice, including stage, age, blocked status, open exceptions and the AVA recommendation |
| Exceptions CSV | Every exception with code, severity, owner, target, status and resolution |
| Audit events CSV | Every recorded event in time order |
| Priorities CSV | Title, outcome, owner, priority, status, progress, open decision |
| Feedback CSV | All five validation questions and your answers |

**Import** accepts a full-session JSON export. It is validated first — the
application id, the schema version, the presence of every collection and the
shape of every invoice record. If validation fails, nothing changes and the
reason is shown. If it passes, you are told what is about to be discarded and
asked to confirm.

## Why GitHub Pages does not give shared monitoring

GitHub Pages is static file hosting. It serves the page; it runs no server code,
holds no database, performs no authentication and keeps no audit log.

Publishing makes the prototype **reachable**. It does not make the data
**shared**. Every visitor gets their own empty local session in their own
browser. If Alex records ten decisions, Jack sees none of them — until Alex
exports the session and sends the file.

Anyone who can reach the URL can also select any role from the "Acting as"
dropdown. That control simulates a role for testing; it is not a login and it
protects nothing.

## Deploying to GitHub Pages

The repository is Pages-ready: one self-contained `index.html`, no build step, no
external requests, and a `.nojekyll` file so Jekyll does not reprocess it. Paths
are relative, so it works from a project subpath such as
`https://<user>.github.io/costa-construction-os/` — this was tested.

Nothing has been pushed or deployed. When Jack decides to:

```bash
git remote add origin https://github.com/<user>/costa-construction-os.git
git push -u origin feature/costa-construction-os-prototype-v0-1
```

Then, in the repository, open **Settings → Pages**, set **Source** to *Deploy
from a branch*, choose the branch and the `/ (root)` folder, and save. The URL
appears there within a minute or two.

There is no deployment URL in this README because nothing has been deployed. Do
not add one until a real URL has been opened and checked.

## What a real shared backend would require

In rough order:

1. A server and database, so a record exists independently of anyone's browser.
2. SSO with MFA and administrator-controlled role assignment, so "Acting as"
   becomes authentication.
3. Permission enforcement on the server — today every rule is JavaScript the user
   could edit.
4. An append-only audit store, time-synchronised and outside the reach of the
   people it audits.
5. Document storage with immutable originals, content hashes, retention classes
   and legal hold.
6. Real extraction with confidence and source anchors, and a correction path that
   never overwrites the original.
7. Integrations with the accounting and project-management platforms, once COS-08
   questions 7 and 8 are answered.
8. Notifications and SLA escalation.
9. Backup, restore and a tested recovery objective.
10. Thresholds, evidence rules, exception severities and SLAs held as
    effective-dated, approved, audited configuration records.

Until items 1 to 4 exist, nothing this prototype produces can be treated as a
control.

## Known limitations

The full list is in [`docs/PROTOTYPE-LIMITATIONS.md`](docs/PROTOTYPE-LIMITATIONS.md).
The short version:

- No authentication, no sharing, no server, no integrations, no document
  processing, no AI model, no payment execution.
- Duplicate detection is vendor plus invoice number only — there is no document
  hash.
- SLA targets are displayed but nothing enforces, escalates or notifies.
- Extraction, retainage rules and payment terms are reported as *not evaluated*,
  never faked.
- Seven workflow rules are prototype assumptions rather than Costa facts, each
  flagged in the interface.
- Every unresolved COS-08 fact shows as `To validate`, `Not configured` or
  `Unknown` — including who holds each role and what the approval thresholds are.

## Documentation

| File | Purpose |
| --- | --- |
| [`docs/SOURCE-MAP.md`](docs/SOURCE-MAP.md) | Every behaviour traced to the canonical document that governs it, plus the full assumptions register |
| [`docs/TEST-REPORT.md`](docs/TEST-REPORT.md) | The 15 required verification checks, exact results, and the six defects found and fixed |
| [`docs/PROTOTYPE-LIMITATIONS.md`](docs/PROTOTYPE-LIMITATIONS.md) | What this is not, and what a real system would need |
| [`docs/BUILD-STATUS.md`](docs/BUILD-STATUS.md) | Build status log across the four milestones |
| [`SOURCE-AUDIT.md`](SOURCE-AUDIT.md) | The baseline application's own source audit, kept from the imported starting point |

## Canonical sources

Read-only references in Google Drive, folder
*Alex Costa → Construction_OS_Operating_Foundation_and_Invoice_Intelligence_v0.1*.
No Drive file was created, edited, moved, renamed or replaced.

- **COS-04** — WF-001 Invoice Approval Operating Standard *(controls the workflow)*
- **COS-05** — Invoice Intelligence MVP Product Specification *(controls functionality)*
- **COS-06** — Invoice Data Model and Permissions *(controls data, audit, permissions)*
- **COS-01** — Master Operating Model *(controls the organization)*
- **COS-02** — Organization, Accountability and Authority *(controls authority)*
- **COS-08** — Alex Validation and Decisions Packet *(identifies the open decisions)*

## Repository layout

```
costa-construction-os/
├── index.html                        # the entire application
├── .nojekyll                         # GitHub Pages: serve files as-is
├── README.md
├── SOURCE-AUDIT.md                   # from the imported baseline
└── docs/
    ├── SOURCE-MAP.md
    ├── TEST-REPORT.md
    ├── PROTOTYPE-LIMITATIONS.md
    └── BUILD-STATUS.md
```

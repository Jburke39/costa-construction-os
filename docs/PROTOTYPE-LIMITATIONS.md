# PROTOTYPE LIMITATIONS — Costa Construction OS v0.1

Read this before drawing any conclusion from the application.

This is a **prototype for validation**, not a system. Its job is to give Alex
something concrete enough to argue with. Everything below is a real limit, not a
disclaimer for its own sake.

---

## 1. What this is not

| Not | What is actually true |
| --- | --- |
| Not authenticated | The "Acting as" selector simulates a role. It is not a login. Anyone with the page can select any role. It enforces nothing outside this browser tab. |
| Not shared | Data lives in one browser profile. Two people using the prototype see two unrelated sessions. Jack cannot see Alex's session, and Alex cannot see Jack's. |
| Not monitored | Nothing is transmitted anywhere. There is no server, no database, no logging endpoint, no analytics. |
| Not integrated | No accounting system, project-management platform, vendor master, e-mail intake or ERP is connected. |
| Not storing documents | A selected PDF or image can be read locally for assisted entry. Document bytes and extracted text are not uploaded, persisted or exported. Only the filename and extraction summary remain. |
| Not running AI | AVA output is deterministic JavaScript over the records in the browser. No model runs. There is no confidence score because nothing produced one. |
| Not moving money | No payment is scheduled, released or executed. Payment status is a recorded value and nothing more. |
| Not connected company data | No company records are preloaded. Alex may enter real records for testing, but they remain only in his browser and are not verified against a Costa system. |

## 2. Data and persistence

- Storage is `localStorage` under the key `costa-construction-os.v3`, with up to
  seven rotating daily browser recovery snapshots.
- **Clearing browser data destroys the session permanently.** So do private and
  incognito windows, "clear site data", and some privacy extensions.
- Different browsers, devices and profiles each hold a separate session.
- Storage is roughly 5 MB. If a write fails the application says so, keeps
  working on screen, and tells you to export — it does not pretend to have saved.
- The current schema is normalized on load and import. Legacy v2 records are
  left untouched and ignored rather than silently mixed with the current session.
- Export is the only way to move a session. You send the file yourself.

## 3. Rules the prototype had to assume

Each of these is visible in the application, flagged as an assumption. None is a
Costa fact.

1. **Blocking severity.** COS-04 requires blocking exceptions to prevent
   approval, but does not say which severities block. The prototype treats
   Critical and High as blocking and Medium as advisory.
2. **Required evidence by invoice type.** Derived from the COS-04 match policy
   because COS-08 question 13 is unanswered.
3. **Lien waiver and insurance.** COS-04 requires these *only when the contract,
   policy or jurisdiction requires them*. The controlling contracts are unknown,
   so the prototype assumes they are required for subcontractor billings.
4. **Two prototype-defined exception codes** that do not exist in COS-04:
   `PX-01` missing receipt or work confirmation, `PX-02` missing required
   supporting evidence. Both are labelled prototype-defined everywhere.
5. **Segregation of duties.** COS-04 forbids the submitter being the sole
   approver. The prototype also prevents one actor approving at two tiers on the
   same invoice; that second rule is an interpretation.
6. **Risk level.** Derived from open exception severity plus overdue status.
   COS-04 references a risk score without defining a formula.
7. **Duplicate detection** uses vendor plus invoice number only. COS-04 also
   requires a document hash and fuzzy similarity; document bytes are not retained,
   so the check is weaker than the standard and says so.

## 4. Facts left deliberately blank

Nothing Costa-specific was invented. These appear as `To validate`,
`Not configured` or `Unknown`:

- Approval thresholds and the whole authority matrix — COS-02 records every tier
  as **ALEX DECISION REQUIRED**, so the executive approval step ships **switched
  off** rather than guessing a number.
- Named holders of every role, including who the CEO is.
- Company model, project types, geographies, contract types, active project
  count, typical project size.
- Monthly invoice volume, peak volume, intake channels, current AP owner.
- Accounting and project-management platforms, versions and API access.
- Vendor master, cost-code and budget structure, PO and subcontract practice,
  permitted non-PO categories.
- Retainage, lien-waiver, insurance and payment-term rules, and the contracts and
  jurisdictions that control them.
- Retention classes, hosting boundary, identity model.
- SLA targets — the COS-04 target times are displayed, but nothing enforces,
  escalates or notifies.

**Alex's own role is one of these open questions.** The prototype labels him
*Pilot Product Owner / Executive Test User — actual role and authority to
validate*, and never asserts he is the legal CEO, because no source says so.
COS-08 question 1 is still asking.

## 5. Where the prototype is thinner than the MVP

Against COS-05, the following functional requirements are **not** implemented,
because they cannot exist in a browser-local static page:

| COS-05 | Requirement | Status |
| --- | --- | --- |
| FR-001 | Authenticated upload, immutable original, content hash | Absent |
| FR-003 | Extraction with per-field confidence and source anchor | Partial — local PDF text extraction and OCR assist entry, but provide no field confidence or retained source anchor and require human review |
| FR-004 | Correction overlay with before / after / reason | Partial — corrections happen as a linked resubmission, not per field |
| FR-005 | Match against real vendor, project, contract, cost code, commitment | Partial — typed references only; no connected system verifies them |
| FR-012 | Notifications, reminders, SLA escalation | Absent |
| FR-015 | Receive downstream scheduled / paid status | Simulated locally |

Also absent: the 100+ ground-truth invoice pilot set, idempotent retry and
dead-letter handling, backup and restore, encryption, least-privilege
enforcement that survives outside the tab, and any real audit store.

## 6. What a shared, real version would require

The prototype deliberately stops short of these. They are the substance of the
next decision, not a formality:

1. **A backend.** A server and database that hold the record, so it exists
   independently of anyone's browser.
2. **Identity.** SSO with MFA, real user accounts, and role assignment that an
   administrator controls — so that "Acting as" becomes authentication rather
   than a dropdown.
3. **Server-side authority.** Permission checks enforced where the user cannot
   reach them. Today every rule in this file is enforced in JavaScript the user
   could edit.
4. **An append-only audit store.** Write-once, time-synchronised, exportable, and
   outside the reach of the people it audits.
5. **Document storage.** Immutable originals with content hashes, retention
   classes and legal hold.
6. **Production extraction**, with confidence, immutable source anchors and a
   human correction path that never overwrites the original.
7. **Integrations** with the accounting and project-management platforms, once
   COS-08 questions 7 and 8 are answered.
8. **Notifications and SLA escalation.**
9. **Backup, restore and a tested recovery objective.**
10. **The configuration itself** — thresholds, evidence rules, exception
    severities and SLAs as effective-dated, approved, audited records.

Until items 1 to 4 exist, no result produced here can be treated as a control.

## 7. GitHub Pages does not change any of this

Publishing to GitHub Pages makes the page **reachable**. It does not make the
data **shared**. Every visitor gets their own empty local session. A GitHub Pages
site is static file hosting: no server code, no database, no authentication, no
audit log. If two people need to see the same invoice, that requires section 6.

## 8. How to report what is wrong

Use the **Alex feedback** page. It asks the five questions that matter:

- What makes sense?
- What is missing?
- Where does this not match reality?
- What should AVA know?
- Which authority or evidence rule is wrong?

Then **Session data → Export** and send Jack the file. Nothing leaves the browser
until you do that yourself.

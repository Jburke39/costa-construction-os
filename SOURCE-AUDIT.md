# Source Audit — Costa Construction OS Command Tracker

> **Kept from the imported baseline.** This file is the source audit written for
> the earlier tracker build, preserved unchanged below for provenance. It is
> superseded for Prototype v0.1 by [`docs/SOURCE-MAP.md`](docs/SOURCE-MAP.md),
> which traces every behaviour to COS-01, COS-02, COS-04, COS-05, COS-06 and
> COS-08. Where the two disagree, SOURCE-MAP.md is correct — in particular the
> "nine-stage approval path" described below was replaced by the 12-state
> canonical lifecycle from COS-04.

## Requested source

`/mnt/data/COSTA CONSTRUCTION OS — COMMAND TRACKER.html`

## Availability in this build environment

The requested `/mnt/data` path was not present. The synced project workspace contained no `sources/` files or other copy of the uploaded HTML. The referenced prior conversation exposed an audit summary, but not the attachment bytes.

Therefore, this was **not** a line-by-line audit of the original HTML source. No claim is made that the original file was opened or modified in this workspace.

## Behaviors established by the available prior audit summary

The earlier tracker was reported to contain:

- A build-plan tracker.
- A basic manual invoice list.
- Browser-only persistence through `localStorage`.
- CSV export.
- Manual status cycling.
- Language describing the invoice list as an interim manual control rather than a finished invoice-intelligence system.

## What this build preserves

- A locally persisted build-priority tracker.
- A manual invoice list with sample data.
- Browser-only `localStorage` persistence.
- CSV exports for invoices and notes.
- Explicit invoice status transitions.

## What this build adds

- Structured invoice intake and evidence controls.
- A nine-stage approval and payment path.
- Exception, hold, dispute, duplicate, rejection, and resumption branches.
- Required decision notes, comments, and per-invoice audit histories.
- A role simulator for stage ownership testing.
- Executive exposure, exception, aging, project, and activity monitoring.
- A draft operating organization and accountability principles.
- Editable priorities, ownership, progress, and build sequencing.
- Process-feedback capture and full-session JSON export/import.
- Prominent disclosure that static GitHub Pages and local browser storage do not provide shared monitoring.

## Safety and truthfulness boundary

This package is a static workflow prototype. It does not include authentication, role enforcement, shared data, document storage, accounting integrations, or a server-side audit log. A GitHub Pages URL makes the prototype reachable; it does not make Alex's local test data visible to another person.


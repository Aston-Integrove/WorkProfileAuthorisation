# Proposal

## Why

Mines track operating licences in Excel and sign them on paper. The result is slow sign-off, a different card and signer chain at every operation, cards that cannot fit a worker's certificates, and expiries that are missed. The exposure that matters most is someone operating equipment on an expired record or an out-of-date card. Release one proves, at one pilot operation, that a single work profile, a routed electronic authorisation and a standard printed card with a QR status check can remove that exposure.

## What Changes

- Introduce an internal application (working name: Operating Licence System) built on one idea: the **work profile** is what a person holds (identity, fitness, induction, training, licences), and the **authorisation** is the employer's signed decision that the person may operate one equipment type in one area.
- Hold one work profile per person per operation, fed read-only from the LMS, the medical system and employee records, plus admin-verified manual evidence (for example contractor licences).
- Define requirement sets per area and equipment type, owned and versioned by each operation.
- Compute eligibility (never typed in) with a "valid until" date equal to the earliest required expiry.
- Provide an authorisation workflow: request, evidence check, routing to signers, e-signature, delegation, return with reason, revocation, suspension, withdrawal, automatic lapse.
- Send expiry reminders to the admin and supervisor and offer an expiring-items list.
- Print a standard card with an opaque QR token. The printer chooses which held items appear, with a fit indicator and an audit record. The QR opens a status page with no personal or medical detail.
- Load existing licences from Excel at cutover, with validation, dry run and legacy markers. Match people on the South African ID number, stored encrypted. Treat expiry as end of day in South African time. Produce the card as a PDF with an on-screen preview at a configurable card size. Send email through SMTP, with SMS as a manual, logged step. Maintain appointments in the application.
- Provide an admin view, an integration API with webhook events and issued client credentials, an immutable audit trail with configurable retention, and multi-operation (multi-tenant) support, shipped first at one operation.
- Guardrails: the tool is **not** a verification-of-competency tool. It never marks anyone competent, holds no assessment forms, and card and status wording says "authorised for [equipment type] in [area] until [date]", never "competent" or "certified".
- Out of release one: supervisor scanner and offline checks, phone wallet passes, permit to work, the temporary site-access permit before a medical is back, portable cross-mine profiles.

## Capabilities

### New Capabilities
- `work-profile`: one profile per person per operation; evidence from source systems and manual evidence; fit/unfit-only medical data; contractor records; photograph.
- `requirement-sets`: areas, equipment types and versioned requirement sets owned by each operation.
- `eligibility`: computed eligible / expiring / not eligible result, missing items and valid-until date, recomputed on change and over time.
- `authorisation-workflow`: request, evidence check, routing, e-signature, delegation, return, activation, expiry, revocation, suspension, withdrawal, waiting-on visibility.
- `expiry-reminders`: reminders to admin and supervisor at per-operation intervals, expiring-window list, optional person notification.
- `card-and-status-page`: standard printed card, print-time item selection, fit warning, print log, opaque QR token, token replacement, status page.
- `admin-view`: per-person view, signing tracker, card hand-over, optional training booking via the LMS.
- `integration-api`: read profile, eligibility and authorisations; write evidence; publish change events; per-operation authentication and access.
- `tenancy-and-audit`: several operations in one tenant, role-based access per operation, immutable audit trail with configurable tamper evidence, configurable retention, access logging.
- `legacy-import`: Excel template, dry-run validation, repeatable import of existing people, evidence and authorisations, with imported authorisations marked as legacy.

### Modified Capabilities
<!-- None. Greenfield project: openspec/specs is empty. -->

## Impact

- **New system**: greenfield, no existing code in the repository.
- **Integrations (read-only in)**: LMS, medical system, employee records. Products and interfaces are unconfirmed (Q22, Q23). SAP candidates are SuccessFactors Employee Central and Learning, SAP EHS, and BTP.
- **Integrations (to confirm)**: e-signing platform (Q24) and card printer artwork output.
- **Compliance**: POPIA (medical data limited to fit/unfit and dates; QR token opaque); Mine Health and Safety Act record-keeping and "authorised in writing" duties for lifting equipment (to verify with the mine's legal and safety team before pilot).
- **Data residency**: assume South African hosting until confirmed (Q25).
- **Disaster recovery**: not yet decided. Interim assumption is Azure SQL geo-replication to South Africa West, with recovery targets agreed with the sponsor.
- **Assumptions to confirm**: passport number as the fallback identifier for people without an ID number; CR80 as the default card size; "manual SMS" meaning a person sends the text outside the system; whether a legacy authorisation counts as authorised in writing.
- **Open questions that gate build or pilot**: Q8, Q15, Q20, Q21, Q22, Q24, Q25, Q27, Q28, Q29, Q30. The specs mark behaviour that depends on them as assumptions.
- **Buy-versus-build** (Q26) should be settled in a demo of Tutis, Cloud Assess, MyPass Global or an SAP partner extension before committing to build.

# Tasks

Everything below can be built and verified on a developer machine with no Azure resources and no outside parties. Azure delivery, real sign-in, production ports and pilot activities are in the change `azure-delivery-and-pilot-readiness`. Order follows `claude.md`: Pass 1 is the database and .NET API (groups 1 to 7), Pass 2 is OpenAPI client sync (group 8), Pass 3 is the UI5 views (group 9), then integration (group 10). Verification commands from `claude.md` apply to every task: `dotnet build` with zero warnings, `dotnet test`, and for UI work `npm run lint` and `npm run build`.

## 1. Foundations

- [ ] 1.1 Scaffold the solution with `src/Api`, `src/Application`, `src/Domain`, `src/Infrastructure`, `tests/` projects and the UI5 TypeScript app at `src/WebUi`; verify `dotnet build` has zero warnings and `npm run lint` and `npm run build` pass on the empty app
- [ ] 1.2 Define the ports for secrets and signing keys, blobs, email, clock and people sign-in, each with a local implementation chosen by configuration; verify the API starts and its health check passes with no Azure resources and no internet
- [ ] 1.3 Write the local database setup (create, migrate, seed) for the local SQL Server with Windows authentication and the connection string in user secrets, with ledger off locally and synthetic seed data (generated ID numbers that pass the check digit); verify one command on a clean database produces a migrated, seeded database and a second run changes nothing
- [ ] 1.4 Implement operation scoping (EF Core global query filter and API policy) and the role model; verify tests show cross-operation reads and writes are denied
- [ ] 1.5 Implement the append-only audit table with insert-only database grants, access logging, an audit search endpoint, and the deployment setting for ledger tamper evidence with a verify endpoint; verify tests show update and delete fail at database level, a profile view creates an entry, and on a Testcontainers SQL Server 2022 verification reports intact (setting on) and is unavailable (setting off)
- [ ] 1.6 Implement the single-run job claim (SQL lease) used by the background jobs; verify a test with two concurrent workers shows each job runs once
- [ ] 1.7 Implement the development-only login (seeded users per role and operation) and the time-control endpoint; verify tests that both are unavailable outside Development and that startup refuses to run them there
- [ ] 1.8 Add a Docker Compose file for Azurite and smtp4dev, a sample webhook receiver that verifies signatures, and a stub source for the agent; verify locally that an email appears in smtp4dev, a photograph is stored in Azurite, a webhook is received and verified, and the agent pushes stub evidence
- [ ] 1.9 Set up the test strategy: unit tests with no external dependencies, integration tests on Testcontainers SQL Server 2022 (ledger on and off), OPA5 and QUnit against the local stack; verify the same test commands pass locally and are documented for the pipeline
- [ ] 1.10 Write the one-command start script (API, background worker, UI5 dev server with API proxy); verify on a clean checkout that one command brings up the app and a seeded user can sign in with the development login

## 2. Work profile and evidence (API)

- [ ] 2.1 Create entities, configurations, a reversible migration and read endpoints for `Operation`, `Company`, `Person` (encrypted identifier with keyed hash, ID number format and check-digit validation, masked output) and `WorkProfile`; verify the migration applies and reverts on Testcontainers SQL Server, the duplicate-profile test passes, the database holds no plain ID number, and logs and webhook payloads contain none
- [ ] 2.2 Create `Evidence` with status derivation and the fit/unfit restriction; verify tests for valid, expiring, expired and a rejected clinical field
- [ ] 2.3 Implement the adapter port, canonical mapping and a stub adapter per source; verify the medical stub drops non-whitelisted fields
- [ ] 2.4 Implement stale-feed detection, unverified treatment and the `UnmatchedRecord` queue with resolve endpoints; verify a test for each
- [ ] 2.5 Implement manual evidence with verifier, contractor profile creation and photograph storage through the blob port; verify unverified manual evidence is excluded from eligibility and a photograph round trip works against Azurite
- [ ] 2.6 Choose the Excel library with a short licence and capability comparison, then implement the versioned import template, upload parsing, staging, validation and dry-run report endpoints; verify tests for invalid ID number, unknown area and a dry run that changes nothing
- [ ] 2.7 Implement confirmed import: repeatable matching, legacy authorisation markers, the dedicated permission, audit of each batch and deletion of the uploaded file after retention; verify a second import creates nothing, the permission-missing test, and a lapsed-evidence legacy authorisation is not eligible

## 3. Requirement sets and eligibility (API)

- [ ] 3.1 Create areas, equipment types and versioned requirement sets with effective dates, endpoints and migration; verify a future-dated change test and an operation-isolation test
- [ ] 3.2 Implement the pure eligibility function with valid-until as earliest expiry and the eligibility endpoint; verify unit tests for missing, lapsed, stale and expiring cases
- [ ] 3.3 Implement the background worker for expiry instants and recomputation using Africa/Johannesburg end-of-day expiry and the job claim; verify clock-controlled tests that an item is valid at 23:30 on its expiry date, expired at the next local midnight, and flips eligibility and writes an outbox event once
- [ ] 3.4 Create the wording catalogue and a test forbidding "competent" and "certified"; verify the test fails on a planted bad string

## 4. Authorisation workflow (API)

- [ ] 4.1 Implement the authorisation state machine with valid transitions and migration; verify invalid-transition tests
- [ ] 4.2 Implement request creation, the evidence check, routing rules and appointments with dated holders and endpoints; verify tests for missing evidence, unrouted keys, an ended holder not being routed to and an appointment with no current holder blocking submission
- [ ] 4.3 Implement the signature port, built-in signer, sign, delegate (same appointment only) and return (reason required); verify a test for each scenario
- [ ] 4.4 Implement activation rules, automatic lapse, revocation with display-as-expired, suspension and withdrawal; verify the audit holds the real state while the display shows expired
- [ ] 4.5 Implement parallel signing by appointment, the separation-of-duties rule and escalation to the operation owner after the configured days; verify tests for any-holder signing, out-of-order signing, self-sign refusal and escalation without reassignment
- [ ] 4.6 Implement reinstatement after evidence renewal (lapse reason, grace period setting, audited transition, never for revoked); verify tests for renewal inside the grace period, after it, after the authorisation's own expiry and after revocation
- [ ] 4.7 Implement waiting-on and waiting-time data; verify against a controlled clock

## 5. Reminders and notifications (API)

- [ ] 5.1 Implement per-operation reminder rules with de-duplication via the unique index and cancellation on renewal; verify tests for each
- [ ] 5.2 Implement the expiring-window list endpoint; verify a window-filter test
- [ ] 5.3 Implement the notification table and SMTP sender with retry, used for reminders and signer assignment and waiting emails, with person notification off by default; verify tests against smtp4dev for delivery, SMTP failure retry and the assignment email content
- [ ] 5.4 Implement the manual SMS action that prepares the text and records the admin's confirmation; verify a test that nothing is sent automatically and the confirmation is stored and audited

## 6. Card and status page (API and server-rendered page)

- [ ] 6.1 Implement opaque tokens, hashed storage and replacement; verify the old token returns the not-valid page
- [ ] 6.2 Implement the public status endpoint with rate limiting and no personal or medical detail, returning a server-rendered script-free page with text from the i18n catalogue; verify a response-content test, a check-logged test, an automated test that the response has no script tags and stays within the size budget, and that the result is clear as text without colour
- [ ] 6.3 Implement print-time selection from held items only, pre-filled, with the capacity indicator and overflow block; verify add-not-held and overflow tests
- [ ] 6.4 Choose the PDF library with a short licence and capability comparison, then implement one layout used for both the preview and the PDF at the tenant's configurable card size (default CR80), with print record, re-issue log, print date and hand-over; verify the preview and PDF match, a size change affects later prints only, and a regeneration test reproduces a printed card from its record

## 7. Integration API, credentials, webhooks and administration (API)

- [ ] 7.1 Implement read endpoints and the evidence write endpoint with scope attributes; verify tests for scope-too-narrow and wrong-operation
- [ ] 7.2 Implement the API client registry (entities, migration, admin endpoints), with secrets shown once and stored as salted hashes; verify a database check finds no plain secret
- [ ] 7.3 Integrate OpenIddict for client credentials using the signing key port with the local development key, the JWKS endpoint and lockout on repeated failures; verify token issue, bad-secret and lockout tests
- [ ] 7.4 Implement secret rotation with overlap, signing key rotation and disabling; verify tests that old and new secrets both work in the overlap and old tokens stay valid until expiry
- [ ] 7.5 Make the API accept tokens from the people sign-in port (the development login locally) and issued client tokens through one authorisation policy; verify the same scope tests pass for both
- [ ] 7.6 Implement webhook subscriptions, the transactional outbox, HMAC signing using the secrets port, retries with backoff, delivery log and manual redelivery; verify tests for retry, duplicate event id and persistent failure
- [ ] 7.7 Expose an outbox-age metric and a health check that degrades when events are stuck; verify a test with a deliberately failing subscriber turns the health check to degraded
- [ ] 7.8 Implement the polling events feed (sequence, cursor, retention, expired-cursor error, `events.read` scope); verify tests for ordered polling, only-newer events and an expired cursor
- [ ] 7.9 Make evidence writes idempotent on source system and source record id, and add the sync heartbeat endpoint; verify a retried write creates one item and a heartbeat with no changes keeps the source from being marked stale
- [ ] 7.10 Build the on-premises source agent as a .NET worker service with outbound-only HTTPS, retry and catch-up, packaged and documented for installation; verify with the stub source that it catches up after an outage and that its credentials cannot read profiles or manage webhooks
- [ ] 7.11 Implement the LMS training-booking request through a port with a stub LMS, and card hand-over recording; verify the request and its status are returned from the stub and hand-over attaches to the print record
- [ ] 7.12 Implement configurable retention (`RetentionPolicy`, no hard-coded periods) with a controlled, audited disposal process, and the tenant settings endpoints; verify tests that nothing is disposed before its configured period and disposal is audited

## 8. OpenAPI contract and client sync (Pass 2)

- [ ] 8.1 Complete OpenAPI metadata on every endpoint (`.WithName()`, `.WithSummary()`, typed `Produces`, `ProblemDetails` for 4xx and 5xx) and export the document; verify a test fails when any endpoint lacks metadata
- [ ] 8.2 Generate the strongly-typed TypeScript client into `src/WebUi` with an `npm run api:generate` script and commit it; verify a local drift check fails after a DTO change and passes after regeneration
- [ ] 8.3 Build the service layer that binds responses into `sap.ui.model.json.JSONModel`; verify QUnit tests with a mocked server for success and ProblemDetails errors

## 9. UI5 views (Pass 3)

All views use declarative XML, `sap_horizon`, no custom CSS, and every string in `i18n/i18n.properties`. Each task names its floorplan and is verified by `npm run lint`, `npm run build`, an OPA5 test and an audit of the views for hardcoded text and custom CSS.

- [ ] 9.1 Build the app shell (`sap.f.FlexibleColumnLayout`, routing, development login screen) and a lint rule rejecting hardcoded strings in views; verify the shell loads and the rule fails on a planted string
- [ ] 9.2 Build the People List Report and Object Page (`DynamicPage`, `ObjectPageLayout`) with evidence, authorisations, eligibility and cards; verify an OPA5 test opens a person and shows source and last-synced time
- [ ] 9.3 Build the Excel import screens (List Report, Object Page, `FileUploader`); verify an OPA5 test runs a dry run then confirms
- [ ] 9.4 Build requirement sets, routing and appointments (List Report and Object Page); verify OPA5 create and edit tests
- [ ] 9.5 Build requests, the signer inbox and the signing tracker (List Report and Object Page); verify OPA5 tests for sign, delegate and return, including at an emulated phone-sized viewport
- [ ] 9.6 Build the expiring items list (List Report) and reminder settings; verify an OPA5 window-filter test
- [ ] 9.7 Build the card print dialog (`Dialog`, item `Table`, `ProgressIndicator`, `PDFViewer` preview); verify an OPA5 test selects items, sees the capacity warning and opens the preview
- [ ] 9.8 Build the unmatched records queue, LMS booking request and card hand-over screens (List Report and Object Page); verify OPA5 tests for resolving a record and recording hand-over
- [ ] 9.9 Build API clients, webhooks and delivery log screens (List Report and Object Page); verify OPA5 tests that a secret is shown once and a failed delivery can be redelivered
- [ ] 9.10 Build the audit trail search (List Report), tamper verification action and tenant settings screens; verify OPA5 tests for search and for the setting being read-only after provisioning

## 10. Integration and verification

- [ ] 10.1 Write an automated end-to-end scenario on the local stack (request, sign with development users, activate, print, check the status page, revoke, recheck, receive and verify the webhook, advance time past an expiry); verify it passes from a clean database
- [ ] 10.2 Write the README covering setup, one-command start, seeding, every test suite and simulating the passage of time, and follow it from a clean clone in a temporary folder; verify the app starts and the end-to-end scenario passes
- [ ] 10.3 Run the full verification set: `dotnet build` with zero warnings, `dotnet test`, `npm run lint`, `npm run build`, `npm test`, and the idempotent migration script; verify all pass and the output is saved with the change

## Workflow follow-up

- Azure delivery, real sign-in, production ports, compliance answers and pilot activities continue in the change `azure-delivery-and-pilot-readiness`.
- Archive this change after review, before starting that change.

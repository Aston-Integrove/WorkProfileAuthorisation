# Tasks

## 1. Foundations, Azure and Azure DevOps

- [ ] 1.1 Confirm the proposed Azure services in `project.md` with the sponsor and record any changes; verify `project.md` shows no "to confirm" items left for release one
- [ ] 1.2 Scaffold the solution (`Api`, `Application`, `Domain`, `Infrastructure`), the UI5 TypeScript app and test projects; verify `dotnet build` and the UI5 build succeed locally
- [ ] 1.3 Create Azure DevOps YAML pipelines for build, test, lint and artifact publish, with branch policies on main; verify a pull request triggers a green run and a failing test blocks the merge
- [ ] 1.4 Add the OpenAPI stage: publish the spec and fail when the generated TypeScript service layer differs from the committed one; verify by changing an endpoint DTO and seeing the build fail
- [ ] 1.5 Write Bicep parameterised per client (own resource group or subscription, App Service, Azure SQL, Key Vault, Storage, Application Insights, Entra ID registrations) in South Africa North; verify a deployment for two test clients creates fully separate resources and a second run makes no changes
- [ ] 1.6 Add test and production environments with an approval gate on production and a separate migration step; verify a production deployment waits for approval
- [ ] 1.7 Add dependency and secret scanning to the pipeline; verify a planted dummy secret fails the build
- [ ] 1.8 Implement operation scoping (EF Core global query filter and API policy) and the role model; verify tests show cross-operation reads and writes are denied
- [ ] 1.9 Implement the append-only audit table with insert-only database grants, access logging, and the deployment setting for ledger tamper evidence with a verify action; verify tests show update and delete fail at database level, a profile view creates an entry, and verification reports intact (setting on) and is unavailable (setting off)
- [ ] 1.11 Implement the single-run job claim (SQL lease) used by the background jobs; verify a test with two concurrent workers shows each job runs once
- [ ] 1.13 Define the ports for secrets and signing keys, blobs, email, clock and people sign-in, each with a local implementation chosen by configuration; verify the API starts and its health check passes with no Azure resources and no internet
- [ ] 1.14 Write the local database setup (create, migrate, seed) for the local SQL Server with Windows authentication and the connection string in user secrets, with ledger off locally and synthetic seed data (generated ID numbers that pass the check digit); verify one command on a clean database produces a migrated, seeded database and a second run changes nothing
- [ ] 1.15 Implement the development-only login (seeded users per role and operation) and the time-control endpoint; verify tests that both are unavailable outside Development and that startup refuses to run them there, and a pipeline check fails if they are enabled in a deployed configuration
- [ ] 1.16 Add a Docker Compose file for Azurite and smtp4dev, a sample webhook receiver that verifies signatures, and a stub source for the agent; verify locally that an email appears in smtp4dev, a photograph is stored in Azurite, a webhook is received and verified, and the agent pushes stub evidence
- [ ] 1.17 Write the one-command start script (API, background worker, UI5 dev server with API proxy); verify on a clean checkout that one command brings up the app and a seeded user can sign in with the development login
- [ ] 1.18 Set up the test strategy: unit tests with no external dependencies, integration tests on Testcontainers SQL Server 2022 (ledger on and off), OPA5 and QUnit against the local stack; verify the same test commands pass on a developer machine and in the pipeline
- [ ] 1.19 Write the README covering setup, one-command start, seeding, running every test suite and simulating the passage of time; verify a second developer follows it from a clean clone without help
- [ ] 1.12 Configure Entra sign-in for people with B2B guests mapped to roles and operations; verify a guest contractor manager can sign in, reaches only requester functions for their operation, and loses access when the invitation is removed
- [ ] 1.10 Set up i18n in the UI5 app and a lint rule rejecting hardcoded strings in views; verify the rule fails on a planted string

## 2. Work profile and evidence

- [ ] 2.1 Create entities, configurations and a reversible migration for `Operation`, `Company`, `Person` (encrypted identifier with keyed hash, ID number format and check-digit validation, masked display) and `WorkProfile`; verify the migration applies, reverts and applies on a Testcontainers SQL Server, the duplicate-profile test passes, the database holds no plain ID number, and logs and webhook payloads contain none
- [ ] 2.2 Create `Evidence` with status derivation and the fit/unfit restriction; verify tests for valid, expiring, expired and a rejected clinical field
- [ ] 2.3 Implement the adapter port, canonical mapping and a stub adapter per source; verify the medical stub drops non-whitelisted fields
- [ ] 2.4 Implement stale-feed detection, unverified treatment and the `UnmatchedRecord` queue; verify a test for each
- [ ] 2.5 Implement manual evidence with verifier, contractor profile creation and photograph storage in Blob Storage; verify unverified manual evidence is excluded from eligibility and a photograph round trip works
- [ ] 2.6 Build the People list and Object Page (profile, evidence); verify OPA5 tests open a person and show source and last-synced time
- [ ] 2.7 Choose the Excel library with a short licence and capability comparison, then implement the versioned import template, upload parsing, staging, validation and dry-run report; verify tests for invalid ID number, unknown area and a dry run that changes nothing
- [ ] 2.8 Implement confirmed import: repeatable matching, legacy authorisation markers, the dedicated permission, audit of each batch and deletion of the uploaded file after retention; verify a second import creates nothing, the permission-missing test, and a lapsed-evidence legacy authorisation is not eligible
- [ ] 2.9 Build the import screens (List Report and Object Page, FileUploader); verify an OPA5 test runs a dry run then confirms

## 3. Requirement sets and eligibility

- [ ] 3.1 Create areas, equipment types and versioned requirement sets with effective dates, API and migration; verify a future-dated change test and operation-isolation test
- [ ] 3.2 Implement the pure eligibility function with valid-until as earliest expiry and the eligibility endpoint; verify unit tests for missing, lapsed, stale and expiring cases
- [ ] 3.3 Implement the background worker for expiry instants and recomputation using Africa/Johannesburg end-of-day expiry and the job claim; verify clock-controlled tests that an item is valid at 23:30 on its expiry date, expired at the next local midnight, and flips eligibility and writes an outbox event once
- [ ] 3.4 Create the wording catalogue and a test forbidding "competent" and "certified"; verify the test fails on a planted bad string
- [ ] 3.5 Build the requirement sets and routing UI (List Report and Object Page); verify OPA5 create and edit tests

## 4. Authorisation workflow

- [ ] 4.1 Implement the authorisation state machine with valid transitions and migration; verify invalid-transition tests
- [ ] 4.2 Implement request creation, the evidence check, routing rules and appointments with dated holders (API and Appointments UI); verify tests for missing evidence, unrouted keys, an ended holder not being routed to and an appointment with no current holder blocking submission
- [ ] 4.3 Implement the signature port, built-in signer, sign, delegate (same appointment only) and return (reason required); verify a test for each scenario
- [ ] 4.4 Implement activation rules, automatic lapse, revocation with display-as-expired, suspension and withdrawal; verify the audit holds the real state while the display shows expired
- [ ] 4.7 Implement reinstatement after evidence renewal (lapse reason, grace period setting, audited transition, never for revoked); verify tests for renewal inside the grace period, after it, after the authorisation's own expiry and after revocation
- [ ] 4.8 Implement parallel signing by appointment, the separation-of-duties rule and escalation to the operation owner after the configured days; verify tests for any-holder signing, out-of-order signing, self-sign refusal and escalation without reassignment
- [ ] 4.5 Implement waiting-on and waiting-time data; verify against a controlled clock
- [ ] 4.6 Build the request, signer inbox and signing tracker UI, usable on a phone; verify OPA5 tests and a manual run at phone width

## 5. Reminders

- [ ] 5.1 Implement per-operation reminder rules with de-duplication via the unique index and cancellation on renewal; verify tests for each
- [ ] 5.2 Implement the expiring-window list and UI; verify a window-filter test
- [ ] 5.3 Implement the notification table and SMTP sender with retry and credentials from Key Vault, used for reminders and signer assignment and waiting emails, with person notification off by default; verify tests with a mail stub for delivery, SMTP failure retry and the assignment email content
- [ ] 5.4 Implement the manual SMS action that prepares the text and records the admin's confirmation; verify a test that nothing is sent automatically and the confirmation is stored and audited

## 6. Card and status page

- [ ] 6.1 Implement opaque tokens, hashed storage and replacement; verify the old token returns the not-valid page
- [ ] 6.2 Implement the public status endpoint with rate limiting and no personal or medical detail, returning a server-rendered script-free page with text from the i18n catalogue; verify a response-content test, a check-logged test, a size-budget check in the pipeline, and a manual check on a throttled connection and without colour
- [ ] 6.3 Implement print-time selection from held items only, pre-filled, with the capacity indicator and overflow block; verify add-not-held and overflow tests
- [ ] 6.4 Choose the PDF library with a short licence and capability comparison, then implement one layout used for both the on-screen preview and the PDF at the tenant's configurable card size (default CR80), with print record, re-issue log, print date and hand-over; verify the preview and PDF match, a size change affects later prints only, and a regeneration test reproduces a printed card from its record

## 7. Integration API, credentials and webhooks

- [ ] 7.1 Implement read endpoints and evidence write endpoint with scope attributes; verify tests for scope-too-narrow and wrong-operation
- [ ] 7.2 Implement the API client registry (entities, migration, admin endpoints and UI), with secrets shown once and stored as salted hashes; verify a database check finds no plain secret
- [ ] 7.3 Integrate OpenIddict for client credentials with the signing key in Key Vault, the JWKS endpoint and lockout on repeated failures; verify token issue, bad-secret and lockout tests
- [ ] 7.4 Implement secret rotation with overlap, signing key rotation and disabling; verify tests that old and new secrets both work in the overlap and old tokens stay valid until expiry
- [ ] 7.5 Make the API accept both Entra ID tokens and issued tokens through one authorisation policy; verify the same scope test passes for both
- [ ] 7.6 Implement webhook subscriptions, the transactional outbox, HMAC signing with Key Vault secrets, retries with backoff, delivery log and manual redelivery; verify tests for retry, duplicate event id and persistent failure
- [ ] 7.7 Add an alert on outbox age and failed deliveries in Application Insights; verify an alert fires in dev with a deliberately failing subscriber
- [ ] 7.8 Implement the polling events feed (sequence, cursor, retention, expired-cursor error, `events.read` scope); verify tests for ordered polling, only-newer events and an expired cursor
- [ ] 7.9 Make evidence writes idempotent on source system and source record id, and add the sync heartbeat endpoint; verify a retried write creates one item and a heartbeat with no changes keeps the source from being marked stale
- [ ] 7.10 Build the on-premises source agent as a .NET worker service with outbound-only HTTPS, retry and catch-up, packaged and documented for installation; verify with a stub source that it catches up after an outage and that its credentials cannot read profiles or manage webhooks
- [ ] 7.11 Publish API documentation including scopes, token flow, webhook signature verification, the events feed and agent installation; verify the documented sample calls, signature check and agent install steps run as written

## 8. Pilot readiness

- [ ] 8.1 Replace stub adapters with the pilot operation's real interfaces once confirmed (Q22, Q23); verify a sync on non-production data
- [ ] 8.2 Configure retention per record type once compliance confirms minimums; verify the values are applied in a test
- [ ] 8.3 Complete a security review including a penetration test of the token endpoint and status page; verify findings are closed or accepted in writing
- [ ] 8.4 Run an end-to-end dry run (request, sign, activate, print, scan, revoke, recheck, webhook received); verify each step in the status page and delivery log
- [ ] 8.5 Measure baselines (lapsed-but-active, sign-off time, renewals in time, admin effort) at the pilot operation; verify they are recorded for sponsor review

## Workflow follow-up

- Ask each client to allow B2B guest invitations and to provide a mine server for the source agent.
- Set default values with the pilot operation for the reinstatement grace period, escalation days and event retention.
- Decide disaster recovery targets with the sponsor (interim plan: geo-replication to South Africa West).
- Confirm with compliance whether a legacy authorisation counts as authorised in writing, and confirm the passport fallback identifier, the default card size and the meaning of manual SMS.
- Close or record answers for Q8, Q15, Q20, Q21, Q22, Q24, Q25, Q26, Q27, Q28, Q29 and Q30 before the tasks that depend on them.
- Confirm legal acceptance of electronic signatures before the pilot.
- Archive the change after review.

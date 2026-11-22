# Design

## Context

Greenfield. See proposal.md for motivation and the specs for behaviour. The stack is fixed in `openspec/project.md`: SAPUI5 (TypeScript) with Fiori floorplans and no custom CSS, a .NET 10 Web API with OpenAPI, SQL Server through EF Core 10, hosted on Azure (South Africa regions) and delivered through Azure DevOps. Mines are likely SAP shops, but no source system has been confirmed (Q22, Q23). Site connectivity is limited, so signing and the status page need a connection while the printed card remains the credential.

## Goals / Non-Goals

**Goals:**
- Keep the domain stable while source systems are unconfirmed, by isolating them behind adapters.
- Make "active on lapsed evidence" structurally hard: eligibility derived, time-driven, audited.
- Operation isolation from day one, shipped at one operation.
- Issue and control credentials for integrating systems without a message broker.

**Non-Goals:**
- Competence assessment, LMS or HR functions, permit to work, scanner, wallet passes, offline checks, cross-mine portability, a message broker.

## Decisions

1. **Eligibility is a pure function, not stored state.** Computed from evidence, authorisations and the effective requirement-set version at a given instant, and always evaluated at read time for the status page and API. Alternative: stored flag kept current by triggers; rejected because it can drift.
2. **Time-driven work by a background worker** in the API host (hosted service). It finds the next expiry instants, recomputes affected people, writes outbox events and creates reminders. Alternative: Azure Functions timers; deferred to keep one deployable.
3. **Source adapters with a canonical evidence model.** One adapter per source maps into evidence with source, last-synced and verified flag. The medical adapter whitelists fit/unfit and dates at the boundary.
4. **Authorisation state machine** (draft, awaiting signature, returned, active, suspended, revoked, expired) with a separate display state: revoked and suspended display as expired or not authorised while the audit trail keeps the real state.
5. **Append-only audit table** written in the same transaction as the change. The application database user has insert and select only on it.
6. **E-signature behind a port.** Built-in signer first (authenticated action, timestamp, content hash) with a wet-ink fallback; a platform such as DocuSign can replace it after Q24 and legal review.
7. **Card and QR.** Random opaque token mapped server-side, replaceable. The print record stores the selected items so a card can be regenerated.
8. **Webhooks through a transactional outbox.** Events are inserted in the same transaction as the change. A background worker delivers them as HMAC-signed HTTPS requests with an event id, retries with backoff and logs each attempt. No Service Bus or Event Grid. Alternative: a broker; rejected to keep the architecture simple, and the outbox keeps the option open.
9. **Own credential issuing for integrating systems** with OpenIddict (client credentials). Secrets are shown once and stored as salted hashes. The token signing key is a non-exportable Key Vault key. Scopes are enforced per endpoint and intersected with the client's allowed operations. People use Microsoft Entra ID. Both token types pass through the same authorisation policy layer. Alternative: Entra app registrations only; simpler, and revisit if every consumer is in the Entra tenant.
10. **Tenancy by operation key** on every operation-owned row, enforced by a global EF Core query filter and checked in the API policy layer.
11. **Wording catalogue** is the single source of status strings, with a test that forbids "competent" and "certified". Strings reach the UI through `i18n.properties`.
12. **Person identifier.** The SA ID number is stored with application-level encryption (key in Key Vault), plus a keyed hash column for lookup and uniqueness. Screens show it masked unless the role allows. Logging and telemetry scrub it. Format and check-digit validation happen at every entry point. Alternative: storing in plain text with column encryption only; rejected because DBAs could read it.
13. **Dates and time zone.** Expiry columns are `date`. The eligibility function takes "today" in Africa/Johannesburg, so an item is valid through its expiry date and expires at the next local midnight. The scheduler computes next-change instants in that zone.
14. **Single-run background jobs.** Each job claims work in SQL (a claim row with owner and lease expiry, or `sp_getapplock`) so that several App Service instances never run the same job twice. Alternative: a separate single-instance worker; kept as a later option.
15. **Configurable audit tamper evidence.** A deployment setting creates the audit table as an Azure SQL ledger table when on. A verification endpoint runs ledger verification and records the result. Switching it on later needs a migration step (ledger tables cannot be converted in place), so the choice is made at provisioning and documented. Alternative: hash-chaining in the application; rejected as it duplicates what the database offers.
16. **Card as PDF.** One layout component produces both the HTML preview and the PDF so they cannot drift. The PDF is generated on the server with a .NET PDF library (to select in the build, with licence checked), sized to the configured card size. The capacity calculation uses the same layout measurements.
17. **Email and manual SMS.** Notifications go through an outbox-style table and an SMTP sender with retry, with credentials in Key Vault. There is no SMS gateway. A "prepare SMS" action renders the text and records the admin's confirmation that it was sent.
18. **Legacy Excel import.** The upload is parsed on the server with an open-source Excel library, validated into a staging table, shown as a dry-run report, then applied in one transaction on confirmation. Records are matched on identifier, evidence type and reference, and area and equipment type, so a re-import changes nothing. Legacy authorisations carry `IsLegacy` and the source reference. The uploaded file is deleted after a retention period.
19. **Appointments in the application.** `Appointment` and `AppointmentHolder` (person, start date, end date) are administered by the admin or operation owner. Routing resolves holders current on the day.
20. **Delivery (in the change `azure-delivery-and-pilot-readiness`):** the GitHub repository with Azure Pipelines YAML pipelines, Bicep, environments with approvals, EF migrations as a separate approved step, OpenAPI client generation checked in the build (see `project.md`).

21. **On-premises agent pushes evidence.** A small .NET worker service runs at the mine, reads the source (database view, file export or the source's API) and posts to `/api/v1/operations/{op}/evidence` over outbound HTTPS using its own API client limited to `evidence.write`. Writes are idempotent on (source system, source record id). The agent calls a heartbeat endpoint after every successful run so a quiet source is not marked stale. No VPN or inbound firewall rule is needed. Alternative: Azure pulls over VPN or ExpressRoute; rejected for lead time and per-mine network approvals, though the adapter port still allows it.
22. **Polling events feed beside webhooks.** `OutboxEvent` has a monotonically increasing sequence. `GET /api/v1/events?after={cursor}` returns events in sequence for the caller's allowed operations and types, with a next cursor. Retention is configurable; an expired cursor returns a "resynchronise" error. Both delivery styles read the same outbox.
23. **One deployment per client.** The Bicep templates are parameterised per client: own resource group (preferably own subscription), SQL database, Key Vault, signing keys, Storage and Entra registrations. Nothing is shared between clients. The pipeline deploys the same artifacts to each client through its own environment set with its own approvals. Alternative: shared multi-client deployment; rejected for confidentiality risk.
24. **People sign in through the client's Entra tenant.** Staff of other companies are invited as B2B guests and mapped to application roles and operations. Removing the guest ends access. Alternative: Entra External ID (second identity store, rejected) and no contractor sign-in (rejected as it removes the Requester role).
25. **Reinstatement after evidence renewal.** Lapsing because of evidence sets `State = Expired` with `LapseReason = EvidenceExpired` and `LapsedAt`. The worker, or the evidence-change handler, reinstates when the evidence is valid again, `LapsedAt` is within the tenant's `ReinstateGraceDays`, and the authorisation's own `ValidUntil` has not passed. Reinstatement is a distinct audited transition. Revoked authorisations are never reinstated. Eligibility remains the authority: reinstatement requires the full requirement set to pass.
26. **Signing rules.** Routing yields one signature task group per appointment, all created at submission (parallel). A task group is satisfied by any current holder signing. The workflow refuses a signature from the requester or the subject of the request. A scheduled check escalates by notifying the operation owner after `EscalationDays` and records it without changing the assignment.
27. **Status page is server-rendered.** The `/status/{token}` endpoint returns a small HTML page rendered by the API (no scripts, no UI5, no stylesheet), with text from the same i18n catalogue. Result is conveyed in text so it does not rely on colour. This is a documented exception to the UI5 standard in `project.md`. A build check enforces the size budget (under 50 KB, to validate on a throttled connection).

28. **Everything runs and tests locally with no Azure resources.** Each Azure or external dependency sits behind a port with a local implementation chosen by configuration:
    - Secrets and keys: user secrets and a locally generated development signing key, instead of Key Vault.
    - Blobs: Azurite (Docker) or a local folder, instead of Blob Storage.
    - Email: smtp4dev (Docker) as a local SMTP server that captures messages.
    - People sign-in: a development-only login offering seeded users per role and operation, instead of Entra ID.
    - Time: a clock port with a development time-control endpoint, so expiry, reinstatement, escalation and reminders can be tested without waiting.
    - Source systems: a stub source plus the real on-premises agent run against the local API, and a sample webhook receiver that verifies signatures.
    - Database: the developer's local SQL Server (Windows authentication, connection string in user secrets) for daily work. Integration tests use Testcontainers with a SQL Server 2022 image so they match Azure SQL, including ledger tables.
    Development-only features (dev login, time control) are compiled out or refuse to start outside the Development environment, and a test proves it. Seed data is synthetic, with generated ID numbers that pass the check digit, and no real client data is ever used. One script starts the API, the background worker and the UI5 dev server (with a proxy to the API); the pipeline runs the same build and test commands. Alternative: a shared cloud dev environment only; rejected because developers could not test offline or reproduce failures.

## Data model (SQL Server, EF Core 10)

All string columns have explicit `MaxLength`. Mutable aggregate roots carry a `rowversion`. All operation-owned tables include `OperationId` and index on it. Names are indicative and may change during build.

| Table | Key columns | Indexes and constraints |
|---|---|---|
| `Operation` | Id, Name, Code | Unique Code |
| `Person` | Id, IdType (SA ID or passport), IdentifierEncrypted, IdentifierHash, FullName, EmployerType, CompanyId, Occupation, PhotoBlobRef, RowVersion | Unique (IdType, IdentifierHash); FullName index |
| `Company` | Id, Name, IsContractor | Unique Name |
| `WorkProfile` | Id, OperationId, PersonId, Status, RowVersion | Unique (OperationId, PersonId) |
| `Evidence` | Id, WorkProfileId, Type, Issuer, IssueDate, ExpiryDate, Status, Reference, SourceSystem, LastSyncedAt, IsManual, VerifiedBy, VerifiedAt | (WorkProfileId, Type); (ExpiryDate) for expiry scans; FitnessResult limited to fit/unfit |
| `UnmatchedRecord` | Id, SourceSystem, PayloadHash, ReceivedAt, ResolvedProfileId | (ResolvedProfileId) filtered unresolved |
| `Area`, `EquipmentType` | Id, OperationId, Name | Unique (OperationId, Name) |
| `RequirementSet`, `RequirementSetVersion`, `RequirementItem` | Set: OperationId, AreaId, EquipmentTypeId; Version: EffectiveFrom, Number; Item: EvidenceType or AuthorisationRequired | Unique (OperationId, AreaId, EquipmentTypeId); (SetId, EffectiveFrom) |
| `RoutingRule`, `Appointment`, `AppointmentHolder` | RoutingRule: OperationId, AreaId, EquipmentTypeId, Sequence, AppointmentId | Unique (OperationId, AreaId, EquipmentTypeId, Sequence) |
| `Authorisation` | Id, OperationId, WorkProfileId, AreaId, EquipmentTypeId, State, RequestedBy, ValidUntil, RevokedReason, LapseReason, LapsedAt, ReinstatedAt, IsLegacy, LegacyReference, ImportBatchId, RowVersion | (WorkProfileId, State); (State, ValidUntil) |
| `SignatureTask` | Id, AuthorisationId, AppointmentId, AssignedTo, State, ReturnReason, SignedAt, ContentHash | (AssignedTo, State); (AuthorisationId) |
| `Card`, `CardPrint`, `CardPrintItem` | Card: PersonId, TokenHash; Print: PrintedBy, PrintedAt, IsReissue, HandedOverAt; Item: refers to Evidence or Authorisation | Unique TokenHash; (PersonId) |
| `ReminderRule`, `ReminderLog` | Rule: OperationId, DaysBefore; Log: ItemType, ItemId, ExpiryDate, DaysBefore | Unique (ItemType, ItemId, ExpiryDate, DaysBefore) for de-duplication |
| `AuditEntry` | Id, OperationId, OccurredAt, ActorType, ActorId, Action, EntityType, EntityId, RealState, Detail | (EntityType, EntityId); (OperationId, OccurredAt); insert-only grants |
| `ApiClient`, `ApiClientOperation`, `ApiClientScope` | Client: ClientId, Name, Owner, SecretHash, PreviousSecretHash, PreviousValidUntil, IsEnabled | Unique ClientId |
| `WebhookSubscription`, `OutboxEvent`, `WebhookDelivery` | Subscription: ClientId, Url, EventTypes, SecretKeyRef; Outbox: EventId, Type, Payload, CreatedAt, State; Delivery: Attempt, StatusCode, At | Unique EventId; (State, NextAttemptAt) |
| `RetentionPolicy` | RecordType, Months | Unique RecordType |
| `TenantSetting` | CardWidthMm, CardHeightMm, AuditLedgerEnabled, ReminderIntervals, ReinstateGraceDays, EscalationDays, EventRetentionDays | Single row per tenant |
| `ImportBatch`, `ImportRow` | Batch: FileName, RunBy, RunAt, Mode (dry run or applied), Created, Updated, Rejected; Row: RowNumber, Action, Errors | (BatchId, RowNumber) |
| `Notification`, `ManualSmsRecord` | Notification: Channel, Recipient, Subject, State, Attempts, NextAttemptAt; Sms: Text, Recipient, ConfirmedBy, ConfirmedAt | (State, NextAttemptAt) |
| `JobClaim` | JobName, OwnerInstance, LeaseUntil | Unique JobName |

## API surface (OpenAPI, .NET 10)

All endpoints use explicit `.WithName()`, `.WithSummary()`, typed `.Produces<T>()` and `ProblemDetails` for 4xx and 5xx. Request and response bodies are immutable `record` DTOs. Scopes apply to integrating systems; people use roles.

| Route | Method | Purpose | Scope or role |
|---|---|---|---|
| `/api/v1/operations/{op}/profiles/{personId}` | GET | Profile with evidence and authorisations | `profile.read` / admin, supervisor |
| `/api/v1/operations/{op}/profiles/{personId}/eligibility?area=&equipmentType=` | GET | Eligibility, missing items, valid-until | `eligibility.read` |
| `/api/v1/operations/{op}/profiles/{personId}/authorisations` | GET | Authorisations | `authorisation.read` |
| `/api/v1/operations/{op}/evidence` | POST | Write evidence from a source | `evidence.write` |
| `/api/v1/operations/{op}/requirement-sets` | GET, POST, PUT | Manage sets and versions | operation owner |
| `/api/v1/operations/{op}/authorisation-requests` | POST, GET | Request, list | requester, admin |
| `/api/v1/authorisation-requests/{id}/sign` / `delegate` / `return` / `withdraw` | POST | Signer actions | signer, requester |
| `/api/v1/authorisations/{id}/revoke` / `suspend` | POST | Revoke or suspend with reason | compliance officer |
| `/api/v1/profiles/{id}/cards/preview` / `print` | POST | Capacity check and preview; print returns a PDF (`application/pdf`) | admin |
| `/api/v1/operations/{op}/imports/template` | GET | Download the Excel template | admin |
| `/api/v1/operations/{op}/imports` | POST | Upload workbook with `dryRun` flag; returns report | admin, `legacy-import` permission for authorisations |
| `/api/v1/imports/{id}` / `confirm` | GET, POST | Review report, confirm apply | admin |
| `/api/v1/operations/{op}/appointments` | GET, POST, PUT | Manage appointments and holders | admin, operation owner |
| `/api/v1/notifications/sms-text` / `confirm-sent` | POST | Prepare manual SMS and record the send | admin |
| `/api/v1/events?after=&limit=` | GET | Polling events feed with cursor | `events.read` |
| `/api/v1/operations/{op}/sync-heartbeat` | POST | Source agent reports a successful run | `evidence.write` |
| `/api/v1/audit/verify` | POST | Run tamper verification (when enabled) | platform admin |
| `/api/v1/tenant-settings` | GET, PUT | Card size, ledger setting (read-only after provisioning), reminders | platform admin |
| `/api/v1/cards/{id}/replace-token` | POST | Replace token | admin |
| `/status/{token}` | GET | Public status page data, no personal detail | anonymous, rate limited |
| `/api/v1/reminders/expiring?from=&to=` | GET | Expiring items list | admin, supervisor |
| `/api/v1/api-clients` | GET, POST | Register, list clients | platform admin |
| `/api/v1/api-clients/{id}/rotate-secret` / `disable` | POST | Credential lifecycle | platform admin |
| `/connect/token` | POST | OAuth client credentials token issue | client credentials |
| `/.well-known/jwks.json` | GET | Public signing keys | anonymous |
| `/api/v1/webhooks` | GET, POST, DELETE | Manage subscriptions and view delivery log | `webhooks.manage` |
| `/api/v1/audit` | GET | Audit trail search | compliance officer |

## UI (SAPUI5, Fiori, no custom CSS)

| App or view | Floorplan | Control hierarchy |
|---|---|---|
| People list and person view | List Report and Object Page in `sap.f.FlexibleColumnLayout` | `sap.f.DynamicPage` with `FilterBar` and `sap.m.Table`; `sap.uxap.ObjectPageLayout` with sections for Evidence, Authorisations, Eligibility, Cards |
| Authorisation requests and signing tracker | List Report, Object Page | `DynamicPage` with `sap.m.Table` (waiting-on, waiting time); Object Page for request detail with sign, delegate, return actions |
| Signer inbox (phone friendly) | List Report | Responsive `sap.m.Table` or `sap.m.List` with action buttons |
| Requirement sets and routing | List Report, Object Page | `DynamicPage`; Object Page sections for items, versions, routing |
| Card print | Object Page section with dialog | `sap.m.Dialog` with item `sap.m.Table` (checkboxes), `sap.m.ProgressIndicator` for capacity, and a preview of the same PDF (`sap.m.PDFViewer`) |
| Excel import | List Report, Object Page | `DynamicPage` listing batches; Object Page for the dry-run report; `sap.ui.unified.FileUploader` |
| Appointments | List Report, Object Page | `DynamicPage` with `sap.m.Table`; Object Page for holders with start and end dates |
| Expiring items | List Report | `DynamicPage` with window filter and `sap.m.Table` |
| Unmatched records queue | List Report | `DynamicPage`, `sap.m.Table` |
| API clients and webhooks | List Report, Object Page | `DynamicPage`; Object Page for scopes, operations, secret rotation, delivery log |
| Audit trail | List Report | `DynamicPage` with `FilterBar` |

The public status page is the one exception to the UI5 standard: a server-rendered, script-free HTML page (see Decision 27) with all text from the i18n catalogue.

## Risks / Trade-offs

- [Stale or lagging feeds make eligibility wrong] → per-item last-synced, staleness threshold, stale evidence unverified, admin alert.
- [Identity matching errors] → match on a verified identifier only; unmatched queue.
- [E-signatures may not count as "in writing"] → legal confirmation before pilot; wet-ink fallback; signature port.
- [Printed card goes stale] → card shows print date; QR is the current answer; scanner in release three.
- [A lent card passes the QR check] → Q27 decided before build; hook for an optional photo.
- [Own credential issuing adds security surface] → OpenIddict, hashed secrets, Key Vault signing key, rotation, lockout, audit, penetration test before the pilot.
- [Webhook receivers down or slow] → outbox, retries, delivery log, manual redelivery, alerts.
- [Outbox worker in the API host stops] → health check and alert on outbox age.
- [Buy-versus-build (Q26)] → vendor demos before committing.
- [Open questions Q15, Q20, Q21, Q29, Q30] → specs state interim behaviour; each is configuration or isolated.
- [Azure service choices are proposed] → confirm with the sponsor before infrastructure work.
- [Local SQL Server 2019 has no ledger tables, so tamper evidence cannot run on the daily dev database] → ledger setting is off on the local database; ledger on and off are both covered by Testcontainers SQL Server 2022 tests; optionally upgrade the local instance to 2022.
- [Local behaviour drifts from Azure behaviour] → ports with contract tests run against both the local and the Azure implementations where possible; test and production deploys use the real services.
- [Development-only features leak into a deployed environment] → startup refuses to run them outside Development, and a test and a pipeline check enforce it.

## Open Questions

- Disaster recovery. Not decided. The interim plan is Azure SQL geo-replication to South Africa West, with recovery time and point targets agreed with the sponsor. This does not change the specs or tasks, but sets the infrastructure cost.
- Selection of the .NET PDF and Excel libraries, including licence terms. Decided during task 6.4 and 2.7 with a short comparison.
- Whether any consumer sits outside the Entra tenant. This does not change the build, because own issuing is chosen, but affects hardening effort.
- Default values for the grace period, escalation days and event retention. Set with the pilot operation; they are tenant settings so the answer does not change the build.
- Whether each client allows B2B guest invitations in its Entra tenant, and which mine server hosts the agent. Both are pilot preparation, not design changes.
- Webhook payload content: identifiers only, or the full change. Default is identifiers and state, with no personal detail.

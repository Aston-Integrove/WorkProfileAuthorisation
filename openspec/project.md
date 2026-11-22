# Project Architecture & Standards

## Overview
Enterprise web application built with a SAPUI5 (TypeScript) frontend adhering strictly to SAP Fiori design guidelines, backed by a .NET 10 Web API with OpenAPI specifications and SQL Server persistence.

## Tech Stack
- **Frontend:** SAPUI5 / OpenUI5 (TypeScript), XML Views, SAP Horizon theme (`sap_horizon`)
- **Backend:** .NET 10 (ASP.NET Core Web API), C# 13, native `Microsoft.AspNetCore.OpenApi`
- **Database:** Microsoft SQL Server, Entity Framework Core 10
- **Testing:** xUnit + Testcontainers (Backend), OPA5 / QUnit (Frontend)
- **Hosting:** Microsoft Azure, South Africa regions only (South Africa North, with South Africa West for disaster recovery) until the client confirms otherwise
- **CI/CD:** Azure DevOps (Repos, Pipelines, Boards, Artifacts)

## Deployment & Delivery (Azure & Azure DevOps)
- **Source control:** Azure DevOps Repos. Changes reach the main branch only through pull requests with a passing build and at least one reviewer, enforced by branch policy.
- **Pipelines:** Azure Pipelines defined as YAML in the repository (`azure-pipelines.yml` plus shared templates). No classic release pipelines.
  - **Build and test stage:** restore, build, run xUnit and Testcontainers tests, run UI5 lint, build and QUnit/OPA5 tests, and publish test and coverage results.
  - **OpenAPI stage:** publish the generated OpenAPI document as a build artifact. Fail the build if the UI5 TypeScript service layer generated from it differs from what is committed.
  - **Deploy stages:** dev, test, then production, each an Azure DevOps Environment. Production requires manual approval.
- **Azure services (proposed, to confirm with the sponsor):**
  - Azure App Service for the .NET API and the UI5 static assets (or Azure Static Web Apps for the UI5 build).
  - Azure SQL Database as the managed SQL Server, accessed through EF Core 10. Use the same SQL Server dialect in Testcontainers.
  - Azure Key Vault for secrets and certificates, accessed through managed identity. No secrets in the repository or pipeline variables.
  - Microsoft Entra ID for sign-in and API authorisation, with roles scoped per operation.
  - Azure Blob Storage for photographs and manually attached evidence documents.
  - Azure Monitor with Application Insights for logs, metrics and alerts. Personal data must not be written to telemetry.
  - Integration events are delivered to subscribers by signed HTTPS webhooks. No message broker (no Service Bus or Event Grid) in release one.
  - Webhook delivery: events are first written to an outbox table in the same SQL transaction as the change, then sent by a background worker in the API host. Failed deliveries retry with backoff and are recorded in a delivery log. Each request is signed (HMAC with a per-subscriber secret held in Key Vault) and carries an event id so receivers can de-duplicate.
- **API client credentials (integrating systems):** the application issues its own credentials.
  - Registered API clients authenticate with the OAuth 2.0 client credentials flow. Use an established library (OpenIddict) instead of hand-written token code.
  - Client secrets are shown once and stored only as a salted hash in SQL Server. They are never stored in Key Vault.
  - Token signing keys are held in Key Vault as non-exportable keys and rotated on a schedule. Public keys are published for verification.
  - Scopes are enforced per endpoint and combined with a per-client list of allowed operations. The same scope checks apply however the caller authenticated.
  - People authenticate through Microsoft Entra ID. The API accepts both Entra ID tokens (people) and its own tokens (integrating systems).
  - Webhook signing secrets are per subscriber, held in Key Vault and read back to sign each delivery.
- **Conventions decided:**
  - Person match identifier is the South African ID number (passport number as fallback, to confirm). Store it encrypted with a keyed hash for lookup. It never appears in logs, telemetry, webhooks, QR tokens or the status page.
  - Expiry dates are date-only and run to the end of the day in Africa/Johannesburg.
  - Background jobs that must run once (expiry, reminders, outbox) take a SQL-based claim or application lock, so scaling out App Service does not duplicate work.
  - Audit tamper evidence is a deployment setting that uses Azure SQL ledger tables when on. The audit table is insert-only either way.
  - The card is rendered as a PDF at a configurable size (default CR80, to confirm) with an on-screen preview from the same layout.
  - Email goes through the organisation's SMTP server, with credentials in Key Vault. There is no automated SMS. SMS is prepared by the system and sent by a person, and the send is logged.
  - Existing licences are loaded from Excel with validation, dry run and a legacy marker.
  - Appointments and holders are maintained in the application.
  - One deployment per client, with its own Azure resources, data, secrets and keys, deployed from the same parameterised Bicep.
  - People sign in through the client's Entra tenant. Staff of other companies are invited as B2B guests.
  - Mine systems that cannot be reached from Azure are read by an on-premises agent (a .NET worker service) that pushes evidence over outbound HTTPS using its own API client. Evidence writes are idempotent on source system and source record id.
  - Integration events are available both as signed webhooks and through a polling events feed with a cursor, both from the same outbox.
  - Reinstatement: an authorisation that lapsed only on evidence reinstates automatically if the evidence is renewed within a configured grace period. Revoked authorisations never reinstate.
  - Signing is parallel, one signature per appointment, no self-signing, with escalation to the operation owner after a configured wait.
  - **Exception to the UI5 standard:** the public status page (`/status/{token}`) is a server-rendered, script-free page so it loads quickly on weak mobile signal. It uses text from the i18n catalogue, no scripts and no stylesheet, and stays within a size budget. All other screens follow the UI5 and Fiori standards.
- **Infrastructure as code:** Bicep, stored in the repository and deployed by the pipeline. No manual changes in the Azure portal.
- **Database migrations:** EF Core migrations are applied by the pipeline as a separate, approved step before the application is deployed. Every migration is reversible.
- **Environments:** Dev, test and production in separate subscriptions or resource groups, with configuration kept per environment. Production data is never copied to dev or test unless it is masked.
- **Security and compliance:**
  - Encrypt data in transit and at rest. Use private endpoints for Azure SQL, Key Vault and Storage where the client network allows.
  - Run dependency and secret scanning in the pipeline.
  - POPIA applies to all environments. Medical data is limited to fit or unfit and dates.
- **Work tracking:** Azure Boards. Each OpenSpec change is linked to a feature work item, and each task group to child work items.

## Frontend Standards (UI5 & SAP Fiori)
- **Floorplans:** Strictly follow official SAP Fiori floorplans:
  - **List Report:** Use `sap.f.DynamicPage` with `sap.ui.comp.filterbar.FilterBar` (or `sap.m.OverflowToolbar` filters) and a responsive `sap.m.Table` or `sap.ui.table.Table`.
  - **Object Page:** Use `sap.uxap.ObjectPageLayout` with `ObjectPageHeaderTitle`, `ObjectPageDynamicHeaderContent`, and `ObjectPageSection` / `ObjectPageSubSection`.
  - **Master-Detail:** Use `sap.f.FlexibleColumnLayout` when combining List Report and Object Page views.
- **Views & Styling:**
  - Use declarative XML views (`.view.xml`) and XML fragments (`.fragment.xml`).
  - **NO custom CSS.** Rely exclusively on standard SAPUI5 controls and SAP utility margin/padding classes (e.g., `sapUiSmallMargin`, `sapUiResponsiveContentPadding`).
- **Localization:** Every user-facing label, title, button, and message MUST live in `i18n/i18n.properties`. Hardcoded strings in XML or controllers are forbidden.
- **API Integration:** Consume .NET REST endpoints via a strongly-typed TypeScript service layer generated from the backend OpenAPI spec, binding responses into `sap.ui.model.json.JSONModel`.

## Backend Standards (.NET 10 & OpenAPI)
- **Architecture:** Clean vertical slice or layered architecture (`Api`, `Application`, `Domain`, `Infrastructure`).
- **Endpoints & OpenAPI:**
  - Use Minimal APIs or `[ApiController]` with explicit `.WithName()`, `.WithSummary()`, and `[ProducesResponseType]` / `.Produces<T>()` metadata so OpenAPI generation is 100% accurate.
  - Never expose EF Core entities directly over HTTP; always map to immutable `record` DTOs.
- **Error Handling:** Return RFC 9457 `ProblemDetails` for all `4xx` and `5xx` responses.
- **Async:** Use `async`/`await` with `CancellationToken` propagation on all I/O operations.

## Database Standards (SQL Server & EF Core 10)
- **Configuration:** Use `IEntityTypeConfiguration<T>` classes (Fluent API) rather than cluttering domain models with data annotations.
- **Data Types & Integrity:**
  - Explicitly define `decimal(18,2)` (or appropriate precision) for all monetary/numeric columns.
  - Explicitly define `MaxLength` on all `string` properties to prevent `nvarchar(max)` defaults.
  - Include `[Timestamp]` / `IsRowVersion()` on mutable aggregate roots for optimistic concurrency.
- **Migrations:** Every schema change requires a clean, reversible EF Core migration.

## Local Development & Testing
The whole application MUST run and be testable on a developer machine with no Azure resources.
- **Prerequisites:** .NET SDK 10, Node 22 LTS, Docker, and a local SQL Server (the developer's default instance, Windows authentication). SQL Server 2019 works for daily development but has no ledger tables, so the audit tamper-evidence setting is off locally.
- **Database:** connection string in user secrets, never in the repository. One command creates, migrates and seeds the local database. Seed data is synthetic only, with generated ID numbers that pass the check digit. Never load real client data locally.
- **Ports with local implementations:**
  - Secrets and signing keys: user secrets and a generated development key.
  - Blobs: Azurite in Docker or a local folder.
  - Email: smtp4dev in Docker.
  - People sign-in: a development-only login with seeded users per role and operation.
  - Time: a clock port with a development-only time-control endpoint.
  - Source systems: a stub source and the real on-premises agent against the local API, and a sample webhook receiver that verifies signatures.
- **Development-only features** (dev login, time control) refuse to run outside the Development environment. A test and a pipeline check enforce this.
- **One command** (a script in the repository root) starts the API, the background worker and the UI5 dev server with a proxy to the API.
- **Tests:** xUnit unit tests need nothing external. Integration tests use Testcontainers with a SQL Server 2022 image, which matches Azure SQL including ledger tables. OPA5 and QUnit run against the local stack. The pipeline runs the same commands as a developer.
- **Documentation:** the README explains setup, one-command start, seeding, running every test suite, and how to simulate time passing.

## Spec-Driven Rules
- All feature proposals (`design.md`) MUST explicitly map:
  1. SQL Server table columns & indexes
  2. OpenAPI routes, HTTP methods, and DTO schemas
  3. SAP Fiori floorplan & UI5 control hierarchy

# Claude Code Execution & Verification Rules

## Core Workflow
1. Always read `openspec/project.md` before running `/opsx:propose` or `/opsx:apply`.
2. Never skip the human review gate between `/opsx:propose` and `/opsx:apply`.
3. Implement large features in three distinct passes: (1) Database & .NET API, (2) OpenAPI TypeScript Client Sync, (3) UI5 Fiori Views & Controllers.

## Mandatory Verification Commands
Before marking any task complete in `tasks.md`, execute the relevant verification steps:

### Backend (.NET 10 & SQL Server)
- Run `dotnet build` to verify zero compiler warnings or errors.
- Run `dotnet test` to verify all unit and integration tests pass.
- If EF Core entities or configurations changed:
  - Generate a migration: `dotnet ef migrations add <MigrationName> --project src/Infrastructure --startup-project src/Api`
  - Verify SQL generation: `dotnet ef migrations script --idempotent --project src/Infrastructure --startup-project src/Api -o /tmp/verify.sql`

### OpenAPI Contract Sync
- Whenever API routes or DTOs change, export the updated OpenAPI JSON document and regenerate the frontend TypeScript client/types so UI5 never references stale endpoints.

### Frontend (SAPUI5)
- Run `npm run lint` (ESLint + UI5 linter) to ensure SAPUI5 best practices and TypeScript strictness.
- Run `npm run build` (`ui5 build`) to verify XML views, manifest routing, and TypeScript compilation succeed.
- Audit modified `.view.xml` files to ensure zero hardcoded UI text (must use `{i18n>...}`) and zero custom CSS classes.

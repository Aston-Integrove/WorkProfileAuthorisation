# Design

## Context

See `work-profile-authorisation-release-one` for the architecture. That change defines ports with local implementations (secrets and signing keys, blobs, email, clock, people sign-in). This change adds the Azure implementations and the delivery pipeline. The repository is on GitHub and pipelines run in Azure Pipelines through a service connection, as set out in `openspec/project.md`.

## Goals / Non-Goals

**Goals:**
- Deploy the same artifacts to each client's Azure environment through reviewed, repeatable pipelines.
- Prove each Azure implementation of a port passes the same contract tests as its local implementation.

**Non-Goals:**
- New features, or changes to behaviour defined in release one.

## Decisions

1. **One parameterised Bicep set per client.** Each client gets its own resource group or subscription, SQL database, Key Vault, Storage and registrations. Nothing is shared.
2. **Port contract tests run against both implementations.** The Azure implementations are accepted only when they pass the same tests as the local ones, run in the test environment.
3. **GitHub is the source of truth, Azure Pipelines builds and deploys.** Branch protection and secret scanning live in GitHub. Production deployment waits for a manual approval on the Azure DevOps Environment.
4. **Migrations are a separate approved step** before the application deploys, and every migration is reversible.
5. **Telemetry excludes personal data.** The ID number, medical information and names are scrubbed before logs or traces leave the application.

## Risks / Trade-offs

- [Service connection or tenant access is delayed] → request access at the start; the local build is not blocked.
- [Azure behaviour differs from the local substitutes] → contract tests on both, and a full end-to-end run in the test environment before production.
- [Penetration test findings] → budget time to fix before the pilot gate.

# Proposal

## Why

The change `work-profile-authorisation-release-one` is built so that everything can be implemented and verified on a developer machine with no Azure resources, no external parties and no sign-offs. That leaves the work that cannot be done that way: the Azure delivery pipeline and infrastructure, the production implementations of the local ports, real sign-in, and the pilot-readiness activities that need people, real systems or compliance decisions. This change holds that work so the first change can run to completion without stopping.

## What Changes

- Confirm the proposed Azure services, the disaster recovery targets and the compliance and legal assumptions with the sponsor and the client.
- Connect the GitHub repository to Azure Pipelines and set up build, test, lint, OpenAPI drift, dependency and secret scanning, and GitHub branch protection.
- Provision per-client Azure infrastructure with Bicep, with dev, test and production environments, approvals and a separate migration step.
- Add the production implementations of the ports defined in release one: Key Vault for secrets, the token signing key and webhook secrets; Blob Storage for photographs; Application Insights telemetry without personal data, with alerts.
- Configure Entra ID sign-in with B2B guests mapped to roles and operations.
- Prepare the pilot: install the source agent against the real systems, set retention values, run a penetration test, check the status page on real devices and connections, dry-run the real Excel import, and measure baselines.

No application behaviour changes here, so this change declares no spec deltas.

Out of scope: any new product feature; anything already covered by `work-profile-authorisation-release-one`.

## Capabilities

### New Capabilities
<!-- None. Delivery, infrastructure and pilot activities only. -->

### Modified Capabilities
<!-- None. -->

## Impact

- **Depends on**: `work-profile-authorisation-release-one` being implemented, because it needs the ports, the API and the local test suite to exist.
- **Needs from others**: an Azure subscription per client, an Azure DevOps organisation and a service connection to GitHub, a client Entra tenant that allows B2B guests, a mine server for the source agent, compliance answers, and pilot access.
- **Open questions that gate this change**: Q8, Q22, Q23, Q24, Q25, Q26, Q28 and the disaster recovery decision.

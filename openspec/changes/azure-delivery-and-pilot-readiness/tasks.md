# Tasks

## 1. Confirmations and decisions

- [ ] 1.1 Confirm the proposed Azure services in `project.md` with the sponsor and record any changes; verify `project.md` lists no unconfirmed Azure choices
- [ ] 1.2 Decide disaster recovery targets (recovery time and point) and the geo-replication plan with the sponsor; verify the decision is recorded in `design.md`
- [ ] 1.3 Obtain compliance answers: electronic signature as "in writing", legacy authorisation as authorised in writing, retention minimums, passport fallback identifier, default card size and the meaning of manual SMS; verify each answer is recorded and the affected specs updated
- [ ] 1.4 Close or record answers for Q8, Q15, Q20, Q21, Q22, Q23, Q24, Q25, Q26, Q27, Q28, Q29 and Q30; verify each has a written answer or an accepted interim assumption

## 2. Source control and pipelines

- [ ] 2.1 Create the Azure DevOps project and a service connection to the GitHub repository, and write Azure Pipelines YAML for build, test, lint and artifact publish; verify a pull request triggers a green run and a failing test blocks the merge
- [ ] 2.2 Configure GitHub branch protection on `main` (pull request, passing pipeline, one reviewer); verify a direct push to `main` is refused
- [ ] 2.3 Add the OpenAPI stage that fails the build when the generated TypeScript client differs from the committed one; verify by changing an endpoint DTO and seeing the build fail
- [ ] 2.4 Enable dependency scanning, GitHub secret scanning and push protection, and a secret scan in the pipeline; verify a planted dummy secret is rejected

## 3. Azure infrastructure and production ports

- [ ] 3.1 Write Bicep parameterised per client (resource group or subscription, App Service, Azure SQL, Key Vault, Storage, Application Insights) in South Africa North; verify a deployment for two test clients creates fully separate resources and a second run changes nothing
- [ ] 3.2 Create dev, test and production environments with a production approval gate and a separate migration step; verify a production deployment waits for approval and migrations apply before the app starts
- [ ] 3.3 Implement the Key Vault versions of the secrets, token signing key and webhook secret ports using managed identity; verify the port contract tests pass against Key Vault and the signing key is non-exportable
- [ ] 3.4 Implement the Blob Storage version of the blob port; verify the contract tests pass against Azure
- [ ] 3.5 Configure Application Insights with personal-data scrubbing and alerts on outbox age, failed webhook deliveries and SMTP failures; verify a deliberately failing subscriber raises an alert and no ID number appears in telemetry
- [ ] 3.6 Configure production SMTP settings and credentials in Key Vault; verify a test email is delivered through the client's SMTP server
- [ ] 3.7 Provision the audit ledger setting per client at deployment and document that it cannot be changed in place; verify ledger verification passes on a deployed environment where it is enabled

## 4. Identity

- [ ] 4.1 Create the Entra app registrations and configure sign-in with B2B guests mapped to roles and operations in the client's tenant; verify a guest contractor manager signs in and reaches only requester functions for their operation
- [ ] 4.2 Verify access ends when a guest's invitation is removed; verify the guest cannot sign in and open sessions stop working

## 5. Pilot readiness

- [ ] 5.1 Install the source agent at the pilot operation against the real LMS, medical and employee-records interfaces (Q22, Q23); verify a sync on non-production data and the heartbeat showing in the admin view
- [ ] 5.2 Set retention values per record type from the compliance answers; verify the configured values are applied
- [ ] 5.3 Commission a penetration test of the token endpoint, status page, webhook endpoints and agent; verify findings are closed or accepted in writing
- [ ] 5.4 Check the status page on real phones and on a throttled mobile connection, and check signing on phones; verify the page loads within the agreed time and signing works on the pilot devices
- [ ] 5.5 Run a dry run of the real Excel import at the pilot operation; verify the report is reviewed with the admin and no data changes before confirmation
- [ ] 5.6 Measure baselines (lapsed-but-active, sign-off time, renewals booked in time, admin effort) at the pilot operation; verify the baselines are recorded for sponsor review

## Workflow follow-up

- Archive `work-profile-authorisation-release-one` before starting this change, so the specs it defines are the main specs.
- Archive this change after the pilot gate in the PRD is met.

## ADDED Requirements

### Requirement: Several operations in one tenant
The system SHALL support several operations in one tenant. An authorisation SHALL belong to one operation and SHALL NOT be portable to another in release one.

#### Scenario: Authorisation scoped to operation
- **WHEN** a person works at two operations
- **THEN** each operation has its own profile and authorisations, and one does not grant the other

### Requirement: Role-based access per operation
Access SHALL be role-based and scoped per operation across the roles: admin, requester, signer, site supervisor, operation owner, compliance or safety officer and integrating system. Workers SHALL NOT log in.

#### Scenario: Role limits
- **WHEN** a requester attempts to sign a request or edit a requirement set
- **THEN** the action is denied

#### Scenario: Mobile-friendly signing
- **WHEN** a signer opens a pending request on a phone browser
- **THEN** they can review and sign, delegate or return it

### Requirement: Immutable audit trail
The system SHALL keep an immutable audit trail of evidence changes, signatures, delegations, returns, revocations, prints and status page checks, recording real states even where the display differs.

#### Scenario: Entries cannot be altered
- **WHEN** any user attempts to edit or delete an audit entry
- **THEN** the system prevents it

### Requirement: Access logging
Every access to a profile SHALL be logged with who and when.

#### Scenario: Profile view logged
- **WHEN** a user opens a profile
- **THEN** an access entry is recorded

### Requirement: Configurable retention
The retention period SHALL be configurable per record type. The legal minimum is to be confirmed with compliance.

#### Scenario: Retention applied
- **WHEN** a record passes its configured retention period
- **THEN** it becomes eligible for disposal only through a controlled, audited process

### Requirement: Privacy and data residency
The system SHALL collect only data the requirements need, SHALL meet POPIA, SHALL encrypt data in transit and at rest, and SHALL be hosted in South Africa until the client confirms otherwise.

#### Scenario: Minimal status data
- **WHEN** the status page is returned
- **THEN** it contains no ID number, medical or other personal detail

### Requirement: Configurable tamper evidence
The system SHALL provide a deployment setting that enables database-level tamper evidence on the audit trail. With the setting on, an administrator SHALL be able to run a verification and see whether the audit trail is intact. With the setting off, the audit trail SHALL still be insert-only for the application.

#### Scenario: Verification with setting on
- **WHEN** tamper evidence is enabled and an administrator runs verification
- **THEN** the result states intact or identifies where tampering was detected, and the run is audited

#### Scenario: Setting off
- **WHEN** tamper evidence is disabled
- **THEN** the application still cannot update or delete audit entries and the verification action is unavailable

### Requirement: One deployment per client
Each client SHALL have its own deployment with its own data store, secrets and signing keys. No data, credentials or users of one client SHALL be reachable from another client's deployment. Operations are the isolation unit inside a deployment.

#### Scenario: Client separation
- **WHEN** a user or API client of client A presents credentials to client B's deployment
- **THEN** the call is refused

### Requirement: Sign-in for people
People SHALL sign in through the client's identity provider. Staff of other companies, such as contractor managers, SHALL be able to sign in as invited guests and SHALL be limited to the operations and roles they are given.

#### Scenario: Contractor manager signs in
- **WHEN** an invited contractor manager signs in with their own organisation's account
- **THEN** they can reach only the requester functions for their operations

#### Scenario: Guest removed
- **WHEN** a guest's invitation is removed or their access ends
- **THEN** they can no longer sign in and their open sessions stop working

## ADDED Requirements

### Requirement: Read API
The system SHALL expose an API to read a person's profile, evidence, authorisations and eligibility, by operation.

#### Scenario: Read eligibility
- **WHEN** an authorised integrating system requests eligibility for a person, area and equipment type
- **THEN** it receives eligible, expiring or not eligible, missing items and valid-until

### Requirement: Write evidence
The system SHALL accept evidence written by a source system, with source and timestamp.

#### Scenario: Evidence written
- **WHEN** the LMS posts a training record with source and timestamp
- **THEN** it is stored as synced evidence and eligibility is recomputed

#### Scenario: Missing provenance rejected
- **WHEN** a write has no source or timestamp
- **THEN** it is rejected with a validation error

### Requirement: API client registry
The system SHALL let a platform administrator register an integrating system as an API client with a name, an owner, the operations it may access, its scopes and the webhook events it may subscribe to. The administrator SHALL be able to disable a client at any time.

#### Scenario: Register a client
- **WHEN** an administrator registers the LMS with operation A, scope `evidence.write`
- **THEN** a client id is created and a secret is shown once and never shown again

#### Scenario: Disable a client
- **WHEN** an administrator disables a client
- **THEN** its existing tokens stop working and further token requests are refused

### Requirement: Scopes
The system SHALL define at least these scopes: `profile.read`, `eligibility.read`, `authorisation.read`, `evidence.write` and `webhooks.manage`. Each API endpoint SHALL require a named scope, and a client's effective access SHALL be limited to its granted scopes and its allowed operations.

#### Scenario: Scope too narrow
- **WHEN** a client holding only `eligibility.read` calls the evidence write endpoint
- **THEN** the call is refused as forbidden and no data changes

#### Scenario: Wrong operation
- **WHEN** a client permitted for operation A requests data for operation B
- **THEN** the call is refused

### Requirement: Credential issuing
The system SHALL issue access tokens to registered clients using the OAuth 2.0 client credentials flow. Tokens SHALL be short-lived and carry the client id and granted scopes. Client secrets SHALL be stored only as a salted hash. Token signing keys SHALL be held in a managed key vault and SHALL be rotatable without downtime, with public keys published for token verification.

#### Scenario: Token issued
- **WHEN** an enabled client presents a valid client id and secret
- **THEN** it receives a short-lived access token limited to its granted scopes

#### Scenario: Bad secret
- **WHEN** a client presents a wrong secret
- **THEN** no token is issued, the failure is audited, and repeated failures cause temporary lockout

#### Scenario: Key rotation
- **WHEN** the signing key is rotated
- **THEN** tokens signed with the previous key stay valid until they expire and new tokens use the new key

### Requirement: Credential lifecycle
The system SHALL let an administrator rotate a client's secret with a configurable overlap during which old and new secrets both work, and SHALL record registration, scope changes, rotation, disabling and every token issue in the audit trail.

#### Scenario: Rotate with overlap
- **WHEN** an administrator rotates a secret with a 24-hour overlap
- **THEN** both secrets work for 24 hours and then only the new one works

### Requirement: Authentication on every call
Every API call SHALL be authenticated. People SHALL authenticate through the organisation's identity provider and integrating systems through issued client credentials. Calls without valid credentials SHALL be refused and return no data.

#### Scenario: Unauthenticated call
- **WHEN** a call has no valid credentials
- **THEN** it is rejected and no data is returned

### Requirement: Webhook subscriptions
The system SHALL let a client with `webhooks.manage` register an HTTPS endpoint to receive events: eligibility changed, authorisation signed, revoked or expired, and evidence expiring. A client SHALL receive only events for operations and event types it is allowed.

#### Scenario: Register endpoint
- **WHEN** a client registers an HTTPS endpoint for authorisation-revoked events
- **THEN** a signing secret is issued once and the endpoint receives matching events

#### Scenario: Non-HTTPS endpoint refused
- **WHEN** a client registers an endpoint that is not HTTPS
- **THEN** registration is rejected

### Requirement: Reliable signed webhook delivery
Webhook requests SHALL be signed using the subscriber's secret and SHALL carry a unique event id and timestamp. An event SHALL be recorded in the same transaction as the change that caused it, so that no event is lost. Failed deliveries SHALL be retried with increasing delay, and each attempt SHALL be recorded in a delivery log visible to administrators.

#### Scenario: Retry after failure
- **WHEN** a subscriber endpoint returns an error or times out
- **THEN** delivery is retried with increasing delay and each attempt is logged

#### Scenario: Duplicate delivery
- **WHEN** an event is delivered more than once
- **THEN** every delivery carries the same event id so the receiver can discard duplicates

#### Scenario: Persistent failure
- **WHEN** retries are exhausted
- **THEN** the event is marked failed, administrators are alerted, and the event can be redelivered manually

### Requirement: Polling events feed
The system SHALL offer a feed that returns the same events as webhooks in order, using a cursor, to clients holding the `events.read` scope. A client SHALL see only events for operations and event types it is allowed. Events SHALL stay available for a configurable number of days.

#### Scenario: Poll from a cursor
- **WHEN** a client requests events after a cursor
- **THEN** it receives later events in order with a new cursor, and requesting again with the new cursor returns only newer events

#### Scenario: Cursor too old
- **WHEN** a client presents a cursor older than the retention period
- **THEN** the system says so and the client must resynchronise from the read API

### Requirement: Idempotent evidence writes
An evidence write that carries a source system and a source record id SHALL update the existing item for that pair instead of creating a duplicate, so that repeated or retried writes are safe.

#### Scenario: Retried write
- **WHEN** the same write is sent twice
- **THEN** one evidence item exists and eligibility is unchanged by the second write

### Requirement: On-premises source agent
The system SHALL provide an agent that runs inside a mine's network, reads from a source system, and pushes evidence to the API over outbound HTTPS using its own client credentials limited to `evidence.write` and its operations. The agent SHALL report a successful sync even when no records changed, so that a quiet source is not treated as a stale feed.

#### Scenario: Agent offline then back
- **WHEN** the agent cannot reach the API and later reconnects
- **THEN** it sends the records it missed and the feed's last-synced time updates

#### Scenario: Quiet source
- **WHEN** a sync completes and nothing changed
- **THEN** the agent reports success and the source is not marked stale

#### Scenario: Agent credentials limited
- **WHEN** the agent's credentials are used to read profiles or manage webhooks
- **THEN** the calls are refused

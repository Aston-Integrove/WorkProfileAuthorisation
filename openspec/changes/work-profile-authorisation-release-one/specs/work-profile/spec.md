## ADDED Requirements

### Requirement: One work profile per person per operation
The system SHALL hold one work profile per person per operation, containing identity, employer, occupation and company. The employer MAY be a contractor.

#### Scenario: Profile created for an employee
- **WHEN** an employee from employee records is first needed at an operation
- **THEN** one profile exists for that person at that operation with identity, employer, occupation and company populated

#### Scenario: Duplicate profile prevented
- **WHEN** a second profile is requested for the same person at the same operation
- **THEN** the system rejects it and returns the existing profile

### Requirement: Evidence held on the profile
The system SHALL hold evidence items on the profile, each with type (fitness, induction, training or licence), issuer, issue date, expiry date, status (valid, expiring or expired) and reference.

#### Scenario: Evidence status derives from expiry
- **WHEN** an evidence item's expiry date has passed
- **THEN** its status is expired

#### Scenario: Evidence nearing expiry
- **WHEN** an evidence item's expiry falls inside the operation's configured warning window
- **THEN** its status is expiring

### Requirement: Read-only synchronisation from source systems
The system SHALL receive evidence from the LMS, the medical system and employee records by read-only synchronisation. Each item SHALL show its source system and last-synced time. The source systems SHALL remain the record of truth for the evidence they own.

#### Scenario: Synced evidence shows provenance
- **WHEN** an admin opens a synced evidence item
- **THEN** the source system and last-synced time are displayed and the item cannot be edited in this system

#### Scenario: Stale feed treated as unverified
- **WHEN** a source feed has not synced within its configured staleness threshold
- **THEN** evidence from that source is marked unverified, a stale-feed alert is raised, and eligibility does not treat the evidence as verified

#### Scenario: Unmatched record queued
- **WHEN** an incoming record cannot be matched to a person on a verified identifier
- **THEN** the record is placed in an admin queue for unmatched records and is not attached to any profile

### Requirement: Manual evidence verified by the admin
The system SHALL let the admin attach evidence that has no system source, such as a contractor licence, and verify it. Manual evidence SHALL be marked as manual and SHALL record who verified it and when.

#### Scenario: Admin verifies a contractor licence
- **WHEN** an admin attaches a licence and marks it verified
- **THEN** the item is stored as manual evidence with the verifying admin and timestamp recorded

#### Scenario: Unverified manual evidence does not count
- **WHEN** manual evidence has been attached but not verified
- **THEN** it is not counted towards eligibility

### Requirement: Medical evidence limited to fit or unfit
The system SHALL hold medical evidence only as fit or unfit with dates. It SHALL NOT store or display clinical detail.

#### Scenario: Clinical detail rejected
- **WHEN** an inbound medical record contains clinical fields beyond fit or unfit and dates
- **THEN** those fields are discarded and not stored

### Requirement: Contractor staff records
The system SHALL create and maintain a record for contractor staff who are not in employee records. Who owns contractor data is to be confirmed (Q25).

#### Scenario: Contractor without HR record
- **WHEN** an admin creates a profile for a contractor with no employee record
- **THEN** the profile is created with the contractor company as employer and is flagged as a locally held record

### Requirement: Photograph for the card
The system SHOULD store one photograph per profile for use on the card.

#### Scenario: Photograph attached
- **WHEN** an admin uploads a photograph
- **THEN** it is stored on the profile and available at card print

### Requirement: Competence is evidence only
The system SHALL NOT mark any person competent, SHALL NOT provide assessment checklists or observation forms, and SHALL accept competence outcomes only as evidence from a named source with assessor, date, result, expiry and reference.

#### Scenario: Assessment outcome received as evidence
- **WHEN** a verification-of-competency outcome is supplied by a named source
- **THEN** it is stored as an evidence item with its source details and the system makes no competence judgement of its own

### Requirement: Person match identifier
The system SHALL use the South African ID number as the verified identifier for matching a person across the LMS, medical system, employee records, imports and manual entry. Where a person has no South African ID number, the passport number SHALL be used instead (to confirm). The identifier SHALL be stored encrypted, SHALL be searchable without being decrypted, and SHALL be masked in screens except for authorised roles. It SHALL NOT appear on the card, the QR token, the status page, webhook payloads, logs or telemetry.

#### Scenario: Matching on ID number
- **WHEN** an inbound record carries an ID number that matches an existing person
- **THEN** the record is attached to that person's profile

#### Scenario: Identifier never exposed
- **WHEN** a status page, webhook payload or log entry is produced
- **THEN** it contains no ID number or passport number

#### Scenario: Invalid identifier
- **WHEN** an ID number fails the format and check-digit validation
- **THEN** the record is placed in the unmatched queue and flagged as invalid

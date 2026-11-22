## ADDED Requirements

### Requirement: One view per person
The admin view SHALL show, for one person, their evidence, authorisations, eligibility and what is missing.

#### Scenario: Person view
- **WHEN** an admin opens a person
- **THEN** evidence with source and last-synced time, authorisations with state, the eligibility result and missing items are shown together

### Requirement: Signing status tracking
The admin view SHALL track the signing status of every request, including who it awaits and for how long.

#### Scenario: Open requests list
- **WHEN** an admin opens the signing tracker
- **THEN** every request not yet active, returned or closed is listed with its current signer and waiting time

### Requirement: Card hand-over
The admin view SHALL support hand-over of the printed card by recording that it was handed over, to whom and when.

#### Scenario: Hand-over recorded
- **WHEN** an admin marks a printed card as handed over
- **THEN** the hand-over is stored against the print record

### Requirement: Training booking through the LMS
The admin view COULD request a booking for missing training through the LMS. The system SHALL NOT run the course. The extent of booking via the LMS is an open question.

#### Scenario: Booking request
- **WHEN** an admin requests a booking for a missing course
- **THEN** a request is passed to the LMS and its status is shown, if the LMS interface supports it

### Requirement: Unmatched records queue
The admin view SHALL provide a queue of inbound records that could not be matched to a person.

#### Scenario: Resolve unmatched record
- **WHEN** an admin links a queued record to a person
- **THEN** the record is attached to that profile and the action is audited

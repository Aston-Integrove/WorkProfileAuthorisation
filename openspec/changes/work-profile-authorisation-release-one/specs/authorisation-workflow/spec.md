## ADDED Requirements

### Requirement: Request an authorisation
A requester SHALL be able to request an authorisation for one person, for one equipment type in one area.

#### Scenario: Request created
- **WHEN** a requester submits a request for a worker, an area and an equipment type
- **THEN** a request is created in the draft state linked to that person, area and equipment type

### Requirement: Evidence check at request time
The system SHALL check the required evidence when a request is made and SHALL show what is missing.

#### Scenario: Missing evidence shown
- **WHEN** a request is made for a worker lacking a required induction
- **THEN** the induction is shown as missing and the request is flagged as not currently eligible

### Requirement: Routing to signers
The system SHALL route a submitted request to the signers defined by a routing table keyed on area and equipment type. Who may request and who may sign for each combination is to be confirmed with the pilot operation (Q21).

#### Scenario: Routed by table
- **WHEN** a request for "Pit 2" and "Dump truck" is submitted
- **THEN** it is sent to the signers listed for that key and moves to awaiting signature

#### Scenario: No routing defined
- **WHEN** no routing entry exists for the area and equipment type
- **THEN** the request cannot be submitted and the requester is told routing is not configured

### Requirement: Sign, delegate or return
A signer SHALL be able to sign electronically, delegate to a person holding the same appointment, or return the request with a reason. The e-signature mechanism and legal validity are to be confirmed (Q24); a wet-ink fallback SHALL remain possible.

#### Scenario: Delegation to same appointment
- **WHEN** a signer delegates to a person who holds the same appointment
- **THEN** the delegate becomes the signer and the delegation is recorded

#### Scenario: Delegation to a different appointment refused
- **WHEN** a signer delegates to a person who does not hold the same appointment
- **THEN** the system refuses the delegation

#### Scenario: Return requires a reason
- **WHEN** a signer returns a request without a reason
- **THEN** the system rejects the return until a reason is entered

#### Scenario: Returned request
- **WHEN** a signer returns a request with a reason
- **THEN** the request moves to returned and the requester sees the reason

### Requirement: Visibility of who a request awaits
The system SHALL show who a record is waiting on and for how long.

#### Scenario: Waiting display
- **WHEN** a request has been awaiting a signer for three working days
- **THEN** the request shows that signer's name and a waiting time of three working days

### Requirement: Activation conditions
An authorisation SHALL become active only when every required signature is in and eligibility passes.

#### Scenario: All signatures but eligibility fails
- **WHEN** all signatures are collected but required evidence has lapsed
- **THEN** the authorisation does not become active

#### Scenario: Activation
- **WHEN** the final required signature is received and eligibility passes
- **THEN** the authorisation becomes active

### Requirement: Expiry and automatic lapse
Every authorisation SHALL have an expiry date and SHALL lapse automatically when it passes. An authorisation SHALL also lapse when its required evidence expires.

#### Scenario: Lapse on date
- **WHEN** an authorisation's expiry date passes
- **THEN** its state becomes expired without user action

### Requirement: Revocation
An authorised user SHALL be able to revoke an authorisation with a reason. A revoked authorisation SHALL display as expired on the card and status page, and the system SHALL keep the real state in the audit trail.

#### Scenario: Revoke
- **WHEN** a compliance officer revokes an active authorisation with a reason
- **THEN** the status page shows it as expired, the audit trail records revoked with the reason and user, and an authorisation-revoked event is published

#### Scenario: Revocation without reason
- **WHEN** a revocation is submitted without a reason
- **THEN** it is rejected

### Requirement: Suspension
The system SHOULD allow an authorisation to be suspended. How suspension displays on the card and status page is to be confirmed (Q15); until then it SHALL display as not authorised.

#### Scenario: Suspend
- **WHEN** an authorised user suspends an active authorisation
- **THEN** its state becomes suspended and the status page shows the person as not authorised for it

### Requirement: Withdraw a request
The system SHOULD let a request be withdrawn before it is fully signed.

#### Scenario: Withdraw
- **WHEN** a requester withdraws a request that is awaiting signature
- **THEN** it is closed, outstanding signature tasks are cancelled, and the withdrawal is recorded

### Requirement: Authorisation states
The authorisation lifecycle SHALL use the states draft, awaiting signature, returned, active, suspended, revoked and expired, and SHALL permit only valid transitions between them.

#### Scenario: Invalid transition blocked
- **WHEN** an action tries to move an expired authorisation directly to active, other than a reinstatement under the evidence-renewal rule
- **THEN** the system rejects it and requires a new request

### Requirement: Appointments maintained in the application
Appointments and the people who hold them SHALL be maintained in the application by the admin or operation owner, each holder with a start date and an optional end date. Routing and delegation SHALL use only holders who are current on the day.

#### Scenario: Holder ended
- **WHEN** an appointment holder's end date has passed
- **THEN** they are no longer offered as a delegate and new requests are not routed to them

#### Scenario: Appointment with no current holder
- **WHEN** a request would route to an appointment with no current holder
- **THEN** the request cannot be submitted and the admin is alerted

### Requirement: Signer notifications
The system SHALL email a signer when a request is assigned or delegated to them and SHALL send a reminder when it has waited longer than the configured time. SMS to signers SHALL follow the manual SMS rule in expiry reminders.

#### Scenario: Assignment email
- **WHEN** a request is routed to a signer
- **THEN** the signer receives an email with a link to the request and no personal detail beyond what is needed to identify it

### Requirement: Reinstatement after evidence renewal
An authorisation that lapsed only because required evidence expired SHALL be recorded as lapsed with the reason evidence expired, SHALL display as expired while lapsed, and SHALL remain reinstatable. If the evidence is renewed within the tenant's configured grace period (in days from the lapse) and the authorisation's own expiry date has not passed, the system SHALL return the authorisation to active without new signatures and SHALL record the reinstatement. Otherwise a new request and signatures SHALL be required. A revoked authorisation SHALL NOT be reinstated.

#### Scenario: Renewed within grace period
- **WHEN** a medical lapses on day 0, is renewed on day 5, and the grace period is 14 days
- **THEN** the authorisation returns to active automatically and the audit trail records the reinstatement

#### Scenario: Renewed after grace period
- **WHEN** the evidence is renewed on day 20 with a grace period of 14 days
- **THEN** the authorisation stays expired and a new request is required

#### Scenario: Revoked is final
- **WHEN** an authorisation was revoked and its evidence is later renewed
- **THEN** it is not reinstated

### Requirement: Parallel signing by appointment
A request SHALL require one signature from each appointment on its route. Any current holder of an appointment MAY sign for it. The order of signing SHALL NOT matter, and the request SHALL be fully signed when every appointment has signed.

#### Scenario: Any holder signs
- **WHEN** an appointment has three current holders and one of them signs
- **THEN** that appointment is satisfied and the other holders' tasks are closed

#### Scenario: Out of order
- **WHEN** the second appointment signs before the first
- **THEN** the signature is accepted

### Requirement: Separation of duties
A person SHALL NOT sign a request they raised or a request for themselves.

#### Scenario: Self-sign refused
- **WHEN** a requester who also holds a signing appointment tries to sign their own request
- **THEN** the system refuses and routes the request to another holder

### Requirement: Escalation of overdue requests
When a request has waited on a signer longer than the tenant's configured number of days, the system SHALL notify the operation owner and record the escalation. Escalation SHALL NOT sign or reassign the request by itself.

#### Scenario: Escalate after waiting
- **WHEN** a request has awaited a signer for longer than the configured days
- **THEN** the operation owner is notified, the escalation is recorded, and the signer remains responsible

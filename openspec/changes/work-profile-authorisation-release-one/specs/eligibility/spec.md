## ADDED Requirements

### Requirement: Eligibility is computed, never entered
The system SHALL compute eligibility for a person against a requirement set. No user or API caller SHALL be able to set eligibility directly.

#### Scenario: Direct write rejected
- **WHEN** a caller attempts to write an eligibility value through the UI or API
- **THEN** the request is rejected

### Requirement: Eligibility result content
The system SHALL return eligible, expiring or not eligible, the missing or lapsed items, and a valid-until date equal to the earliest expiry among the required items. Only valid, verified evidence and active authorisations SHALL count.

#### Scenario: Valid-until is earliest expiry
- **WHEN** a person holds a medical expiring 2027-03-01, an induction expiring 2027-06-01 and an active authorisation expiring 2027-09-01, all required
- **THEN** the result is eligible with valid-until 2027-03-01

#### Scenario: Missing item reported
- **WHEN** a required training certificate is absent
- **THEN** the result is not eligible and lists that certificate as missing

#### Scenario: Stale evidence not counted
- **WHEN** required evidence is marked unverified because its feed is stale
- **THEN** it is listed as an issue and the result is not eligible

### Requirement: Recomputation
The system SHALL recompute eligibility when evidence or an authorisation changes and when time passes, so that an expiry takes effect without user action.

#### Scenario: Expiry passes
- **WHEN** the earliest required expiry date is reached
- **THEN** eligibility changes to not eligible and an eligibility-changed event is published

### Requirement: Eligibility wording
Screens, the card and the status page SHALL state authorisation for an equipment type in an area until a date, and SHALL NOT describe a person as competent or certified.

#### Scenario: Status wording
- **WHEN** eligibility is displayed
- **THEN** no text states or implies that the person is competent or certified

### Requirement: Expiry dates and time zone
Expiry dates SHALL be held as dates without a time. An item SHALL be valid through the end of its expiry date in the time zone Africa/Johannesburg and SHALL become expired at the start of the next day in that time zone. The valid-until date shown SHALL be that expiry date.

#### Scenario: Valid on expiry day
- **WHEN** an item's expiry date is 2027-03-01 and the time is 2027-03-01 23:30 in South African time
- **THEN** the item is still valid

#### Scenario: Expired the next day
- **WHEN** the time reaches 2027-03-02 00:00 in South African time
- **THEN** the item is expired and eligibility is recomputed

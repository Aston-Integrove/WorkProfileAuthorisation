## ADDED Requirements

### Requirement: Excel import template
The system SHALL provide a versioned Excel template and accept an Excel upload for loading existing people, evidence and authorisations at an operation.

#### Scenario: Download template
- **WHEN** an admin requests the import template
- **THEN** a versioned workbook is provided with the required columns and instructions

### Requirement: Validate before import
The system SHALL validate an uploaded workbook before changing any data and SHALL produce a report listing each row with errors, warnings and the action it would take. The admin SHALL be able to run this as a dry run that changes nothing.

#### Scenario: Dry run
- **WHEN** an admin uploads a workbook as a dry run
- **THEN** a report shows rows that would be created, updated or rejected and no data is changed

#### Scenario: Row errors
- **WHEN** a row has an invalid ID number or an unknown area or equipment type
- **THEN** the row is rejected with the reason and the other valid rows can still be imported on confirmation

### Requirement: Repeatable import
Importing the same workbook again SHALL update matching records and SHALL NOT create duplicates. Matching SHALL use the person identifier, the evidence type and reference, and the area and equipment type.

#### Scenario: Re-import
- **WHEN** the same workbook is imported twice
- **THEN** the second import changes nothing and reports zero creates

### Requirement: Legacy authorisations are marked
An imported authorisation SHALL be marked as legacy, SHALL reference the existing paper or Excel record, SHALL carry no system signature, and SHALL remain subject to eligibility and to its expiry date. Importing legacy authorisations SHALL require a dedicated permission and an explicit confirmation by the admin. Whether a legacy authorisation counts as authorised in writing is to be confirmed with compliance.

#### Scenario: Legacy marker
- **WHEN** a legacy authorisation is imported and confirmed
- **THEN** it displays as authorised until its expiry date with a legacy marker in the admin view, and the audit trail names the importer and the file

#### Scenario: Lapsed evidence
- **WHEN** a legacy authorisation's required evidence is expired
- **THEN** it is not eligible, like any other authorisation

#### Scenario: Permission missing
- **WHEN** a user without the legacy import permission attempts to import authorisations
- **THEN** the import is refused

### Requirement: Import audit and privacy
Every import SHALL be recorded with the file name, the person who ran it and the counts. The uploaded file SHALL be kept only as long as needed to run and review the import, and identifiers in reports SHALL be masked.

#### Scenario: Import recorded
- **WHEN** an import is confirmed
- **THEN** an audit entry records the file name, user, time and counts of created, updated and rejected rows

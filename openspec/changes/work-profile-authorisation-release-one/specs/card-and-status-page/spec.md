## ADDED Requirements

### Requirement: Standard card layout
The system SHALL produce one standard card layout for all operations in the tenant.

#### Scenario: Same layout across operations
- **WHEN** cards are printed for workers at two operations
- **THEN** both use the same layout structure

### Requirement: Print and re-issue logging
The system SHALL print the card for hand-over and SHALL log every print and re-issue. The printed card SHALL state its print date.

#### Scenario: Print logged
- **WHEN** an admin prints a card
- **THEN** a print record exists with the person, admin, timestamp and whether it is a first issue or re-issue

### Requirement: Opaque QR token
The card SHALL carry a QR code holding an opaque token and no personal details.

#### Scenario: Token content
- **WHEN** the QR code is decoded
- **THEN** it yields only a token that cannot be resolved to a person without calling the system

### Requirement: Status page
The QR code SHALL open a status page showing whether the person can operate the equipment type and the valid-until date. It SHALL show no personal or medical detail and SHALL require a connection. Whether it shows only printed items or all current authorisations (Q29) and whether it shows a photograph or name check (Q27) are to be confirmed before build.

#### Scenario: Eligible person
- **WHEN** a supervisor opens the status page for a person with an active authorisation
- **THEN** it shows authorised for the equipment type in the area until the valid-until date

#### Scenario: Revoked shown as expired
- **WHEN** the authorisation was revoked after the card was printed
- **THEN** the status page shows it as expired

#### Scenario: Unknown token
- **WHEN** a token is unknown or replaced
- **THEN** the page states the card is not valid and shows no personal detail

#### Scenario: Check logged
- **WHEN** the status page is opened
- **THEN** the check is recorded in the audit trail

### Requirement: Token replacement
The system SHOULD replace a card's token when a card is lost so the old QR stops working.

#### Scenario: Replace token
- **WHEN** an admin reports a card lost
- **THEN** a new token is issued and the old token returns the not-valid page

### Requirement: Print-time item selection
At print, the person printing SHALL choose which items appear in the allowed list on the card. The choice SHALL be made only from items the person currently holds (active authorisations and valid evidence), SHALL be pre-filled with all of them, and the printer SHALL be able to deselect. No free text SHALL be accepted and an item not held SHALL NOT be addable. Who may choose (Q30) is to be confirmed before build.

#### Scenario: Pre-filled selection
- **WHEN** an admin opens the print screen
- **THEN** all held items are selected by default

#### Scenario: Cannot add item not held
- **WHEN** an admin attempts to add an expired or absent item
- **THEN** the system does not allow it

### Requirement: Card capacity indicator
The system SHALL show how much of the card is used and SHALL warn before printing when the selection does not fit.

#### Scenario: Overflow warning
- **WHEN** the selected items exceed the card's capacity
- **THEN** a warning is shown and printing is blocked until items are deselected

### Requirement: Print record of selection
The print record SHALL store the items selected, who selected them and when, so the printed card can be reproduced and audited.

#### Scenario: Reproduce card
- **WHEN** an auditor opens a print record
- **THEN** the card can be regenerated exactly as printed

### Requirement: Item validity dates on the card
The card SHOULD show each item's own valid-until date next to it.

#### Scenario: Dates shown
- **WHEN** a card is printed with three items
- **THEN** each item appears with its own valid-until date

### Requirement: Card size, preview and PDF output
The card size SHALL be configurable per tenant, with the CR80 size (85.6 mm by 54 mm) as the default (to confirm). The system SHALL show an on-screen preview of the card exactly as it will print, and printing SHALL produce a PDF sized to the card. The capacity indicator SHALL be calculated for the configured size.

#### Scenario: Preview before print
- **WHEN** an admin selects items and opens the preview
- **THEN** the preview matches the PDF that will be produced, including the print date and item validity dates

#### Scenario: PDF generated
- **WHEN** an admin confirms the print
- **THEN** a PDF at the configured card size is produced and the print is logged with the selected items

#### Scenario: Size change
- **WHEN** the tenant's card size is changed
- **THEN** later previews and PDFs use the new size and earlier print records can still be regenerated at the size they were printed

### Requirement: Lightweight status page
The status page SHALL be a lightweight page that loads quickly on a weak mobile connection. It SHALL need no script libraries, SHALL show the result as text that does not rely on colour alone, and SHALL stay within a small size budget (target under 50 KB, to validate on a throttled connection).

#### Scenario: Weak connection
- **WHEN** the status page is loaded on a throttled mobile connection
- **THEN** the result is readable within the agreed time and the response is within the size budget

#### Scenario: Meaning without colour
- **WHEN** the page is viewed without colour
- **THEN** the authorised or not authorised result is still clear from the text

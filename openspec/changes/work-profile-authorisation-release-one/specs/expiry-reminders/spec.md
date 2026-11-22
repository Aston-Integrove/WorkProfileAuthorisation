## ADDED Requirements

### Requirement: Expiry notifications to admin and supervisor
The system SHALL notify the admin and the site supervisor ahead of the expiry of any evidence or authorisation, at intervals configured per operation.

#### Scenario: Reminder at configured interval
- **WHEN** an operation is configured with reminders at 60, 30 and 7 days and a medical expires in 30 days
- **THEN** the admin and the supervisor each receive a reminder naming the person, the item and the expiry date

#### Scenario: No duplicate reminders
- **WHEN** a reminder for an item and interval has already been sent
- **THEN** it is not sent again for the same expiry date

#### Scenario: Renewed item cancels pending reminders
- **WHEN** evidence is renewed with a later expiry before a reminder is due
- **THEN** reminders for the old expiry are not sent

### Requirement: Expiring items list
The system SHALL show a list of people and items expiring in a chosen window.

#### Scenario: Window filter
- **WHEN** an admin selects a 45-day window
- **THEN** every item at that operation expiring within 45 days is listed with person, item, expiry date and any booked renewal

### Requirement: Notification to the person
The system COULD notify the person by email or SMS. Channel and consent are to be confirmed and the feature SHALL be off by default.

#### Scenario: Off by default
- **WHEN** an operation has not enabled person notifications
- **THEN** no message is sent to workers

### Requirement: Email delivery
The system SHALL deliver notifications by email through the organisation's SMTP server. Delivery failures SHALL be recorded and visible to administrators, and repeated failures SHALL raise an alert.

#### Scenario: SMTP failure
- **WHEN** the SMTP server rejects a notification or cannot be reached
- **THEN** the failure is logged against the notification and retried with increasing delay

### Requirement: SMS is manual
The system SHALL NOT send SMS automatically. For a notification that an administrator wants to send by SMS, the system SHALL prepare the message text and let the administrator record that it was sent, to whom and when. Manual here means a person sends it outside the system (assumption to confirm).

#### Scenario: Prepare SMS text
- **WHEN** an admin chooses to send an expiry reminder by SMS
- **THEN** the system shows the message text for the person to send and records the send once the admin confirms

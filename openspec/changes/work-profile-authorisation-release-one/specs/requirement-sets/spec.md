## ADDED Requirements

### Requirement: Requirement sets per area and equipment type
The system SHALL let an operation define requirement sets keyed on area and equipment type, listing the evidence types and authorisations required.

#### Scenario: Requirement set defined
- **WHEN** an operation owner defines a set for "Pit 2" and "Dump truck" listing medical fitness, induction and one training certificate
- **THEN** the set is stored against that area and equipment type for that operation

### Requirement: Operation owns its requirement sets
Each operation SHALL own and edit its own requirement sets using a structure shared across operations. One operation SHALL NOT be able to view or change another operation's sets.

#### Scenario: Cross-operation edit denied
- **WHEN** an operation owner for operation A attempts to edit a set belonging to operation B
- **THEN** the request is denied

### Requirement: Areas and equipment types per operation
The system SHALL let each operation define its own areas and equipment types. Granularity is to be confirmed with the pilot operation (Q20).

#### Scenario: Operation-specific list
- **WHEN** an operation defines areas and equipment types
- **THEN** only those values are offered when creating requirement sets and authorisation requests at that operation

### Requirement: Versioned requirement sets
The system SHOULD version requirement sets. A change SHALL apply from a stated effective date and SHALL be logged.

#### Scenario: Future-dated change
- **WHEN** an operation owner saves a change with an effective date in the future
- **THEN** the current version stays in force until that date and the change is recorded in the audit trail

#### Scenario: Existing authorisations after change
- **WHEN** a new version takes effect
- **THEN** eligibility for affected people is recomputed against the new version

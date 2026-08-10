## Purpose

Lets a customer fund their wallet from a linked bank account by initiating an ACH debit against fresh, Plaid-verified account numbers, with the resulting funds attributed correctly in an omnibus FBO account.

## ADDED Requirements

### Requirement: Deposits require an idempotency key
The system SHALL require a client-supplied idempotency key on every deposit initiation request and SHALL NOT create a second deposit for a key that has already been used.

#### Scenario: First request with a key
- **WHEN** a deposit is initiated with an idempotency key that has not been seen before
- **THEN** the system creates a new deposit and associates it with that key

#### Scenario: Repeated request with the same key
- **WHEN** a deposit is initiated with an idempotency key that was already used for a prior deposit
- **THEN** the system does not create a second deposit
- **THEN** the system returns the same deposit id and status as the original request

#### Scenario: Missing idempotency key
- **WHEN** a deposit initiation request is submitted without an idempotency key
- **THEN** the system rejects the request without attempting to initiate a debit

### Requirement: Live account numbers at transaction time
The system SHALL fetch current account and routing numbers from Plaid at the moment a deposit is initiated rather than relying on any previously stored value.

#### Scenario: Initiating a deposit
- **WHEN** a deposit is initiated against a funding source
- **THEN** the system retrieves the current account and routing numbers from Plaid for that funding source before constructing the ACH debit instruction

### Requirement: Deposits blocked on funding sources needing re-authentication
The system SHALL refuse to initiate a deposit against a funding source that is marked as requiring re-authentication.

#### Scenario: Attempting a deposit on an expired funding source
- **WHEN** a deposit is initiated against a funding source marked `requires_reauth`
- **THEN** the system does not attempt to fetch account numbers or submit an ACH debit
- **THEN** the client receives a clear error identifying that the funding source needs to be re-linked

### Requirement: Customer-attributed subledger recording
The system SHALL record every deposit in a subledger keyed on an identifier the system controls, and SHALL NOT rely on the ACH memo or description field for attribution.

#### Scenario: Recording a deposit
- **WHEN** an ACH debit instruction is submitted for a deposit
- **THEN** the system records a pending subledger entry for that deposit, keyed to the customer's internal id
- **THEN** the subledger entry does not depend on any freeform memo or description field surviving the ACH network round trip

### Requirement: ACH origination is a defined but unimplemented boundary
The system SHALL expose deposit initiation through a stable interface for submitting the ACH debit to the sponsor bank, with the actual submission behavior explicitly stubbed pending confirmation of the origination path.

#### Scenario: Submitting a deposit today
- **WHEN** a deposit passes idempotency, funding-source-status, and account-number-retrieval checks
- **THEN** the system calls the origination interface with the debit routing number, debit account number, amount, and customer reference
- **THEN** the stub records that a submission was attempted without completing a real ACH origination, and this behavior is clearly identifiable as a stub rather than a production path

## Purpose

Surfaces consent expiration explicitly and immediately, since a lapsed Plaid item is a silent failure that otherwise surfaces months later as a failed deposit or a support ticket instead of at the moment it actually breaks.

## ADDED Requirements

### Requirement: Consent expiration detection
The system SHALL mark a funding source as requiring re-authentication when either `/auth/get` fails with an item error for it, or its stored `consent_expiration_time` has passed.

#### Scenario: Item error detected outside a webhook
- **WHEN** the system calls `/auth/get` for a funding source and receives an item error
- **THEN** the system marks that funding source as requiring re-authentication

#### Scenario: Consent expiration time has passed
- **WHEN** the system evaluates a funding source whose stored `consent_expiration_time` is in the past
- **THEN** the system marks that funding source as requiring re-authentication

### Requirement: Update-mode re-link
The system SHALL provide a way to mint a Plaid Link token in update mode for a specific funding source that requires re-authentication.

#### Scenario: Requesting a re-link token
- **WHEN** a customer requests to re-authenticate a funding source marked as requiring re-authentication
- **THEN** the system mints a Link token in update mode scoped to that funding source's existing item
- **THEN** successfully completing that update-mode Link flow clears the requires-re-authentication state for the funding source

### Requirement: Deposits blocked while re-authentication is required
The system SHALL prevent deposit initiation against any funding source currently marked as requiring re-authentication, and SHALL communicate the reason clearly rather than failing silently.

#### Scenario: Deposit attempt while blocked
- **WHEN** a deposit is initiated against a funding source marked as requiring re-authentication
- **THEN** the deposit is rejected before any ACH numbers are fetched or any debit is attempted
- **THEN** the error returned to the caller explicitly states that the funding source needs to be re-linked, rather than a generic failure

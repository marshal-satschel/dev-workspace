## Purpose

Lets a customer connect an external bank account for wallet funding by verifying its routing/account numbers through Plaid, without ever routing money through Plaid itself.

## ADDED Requirements

### Requirement: Link token issuance
The system SHALL create a Plaid Link token for an authenticated customer on demand, scoped to the `auth` product and `US` country code, using a `client_user_id` that is our stable internal customer id.

#### Scenario: Requesting a link token
- **WHEN** an authenticated customer requests a link token
- **THEN** the system calls Plaid's link token creation with `products: ["auth"]`, `country_codes: ["US"]`, the customer's stable internal id as `client_user_id`, and a webhook URL pointing at this service
- **THEN** the response to the client contains only the `link_token` — no other Plaid response fields are exposed

#### Scenario: Link tokens are never cached
- **WHEN** a customer requests a link token twice in the same session
- **THEN** the system mints a new token each time rather than reusing a previously issued one, since link tokens expire after 4 hours

### Requirement: Funding source verification before persistence
The system SHALL confirm that a linked item actually has retrievable ACH account and routing numbers before persisting it as a funding source.

#### Scenario: Exchanging a public token
- **WHEN** a customer completes Plaid Link and the client submits the resulting `public_token`
- **THEN** the system exchanges it for an access token and item id
- **THEN** the system verifies ACH numbers are retrievable for that item before creating a funding source record
- **THEN** the response to the client contains only a funding source id, the institution name, and the account mask

#### Scenario: Item has no ACH numbers
- **WHEN** a public token is exchanged but the linked item cannot return ACH account/routing numbers
- **THEN** the system does not persist a funding source
- **THEN** the client receives an error indicating the account cannot be used for ACH funding

### Requirement: Sensitive data never persisted in the clear
The system SHALL encrypt the Plaid access token at rest and SHALL NOT persist full account or routing numbers under any circumstance.

#### Scenario: Funding source storage
- **WHEN** a funding source is created
- **THEN** the stored record contains an encrypted access token, item id, account id, institution name, account mask, and consent expiration time
- **THEN** the stored record does NOT contain a plaintext access token, a full account number, or a full routing number

#### Scenario: Retrieving numbers when needed
- **WHEN** account or routing numbers are needed for an operation
- **THEN** the system retrieves them fresh from Plaid at that time rather than reading them from storage, because they were never stored

### Requirement: Access token confidentiality
The system SHALL NOT expose a decrypted Plaid access token outside the backend process handling it.

#### Scenario: API responses
- **WHEN** any API response is constructed for a funding source
- **THEN** the response body does not contain the access token in any form

#### Scenario: Logging and error handling
- **WHEN** a log line or error message is produced anywhere in the funding-source lifecycle
- **THEN** the access token does not appear in that log line or error message, whether the operation succeeded or failed

#### Scenario: Account number logging
- **WHEN** an account reference is written to a log
- **THEN** at most the last four digits of the account number appear — never the full account or routing number

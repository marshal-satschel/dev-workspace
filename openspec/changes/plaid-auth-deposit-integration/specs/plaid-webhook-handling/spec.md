## Purpose

Keeps funding source state accurate over time by reacting to Plaid's asynchronous notifications about item health and consent, instead of only discovering problems the next time a customer tries to deposit.

## ADDED Requirements

### Requirement: Webhook signature verification
The system SHALL verify the signature of every incoming Plaid webhook before processing its contents, and SHALL reject any webhook whose signature does not verify.

#### Scenario: Valid webhook
- **WHEN** a webhook arrives with a valid Plaid signature
- **THEN** the system processes the webhook payload according to its type

#### Scenario: Invalid or missing signature
- **WHEN** a webhook arrives with a missing or invalid signature
- **THEN** the system does not process the payload
- **THEN** the system does not treat any funding source as having a state change based on that payload

### Requirement: Item error handling
The system SHALL flag the affected funding source as requiring re-authentication when it receives an `ITEM_ERROR` webhook for that item.

#### Scenario: Receiving an item error webhook
- **WHEN** a verified webhook of type `ITEM_ERROR` is received for a linked item
- **THEN** the system marks the corresponding funding source as requiring re-authentication

### Requirement: Pending expiration handling
The system SHALL flag the affected funding source as requiring re-authentication when it receives a `PENDING_EXPIRATION` webhook for that item.

#### Scenario: Receiving a pending expiration webhook
- **WHEN** a verified webhook of type `PENDING_EXPIRATION` is received for a linked item
- **THEN** the system marks the corresponding funding source as requiring re-authentication before the consent actually lapses

### Requirement: Hosted Link session completion handling
The system SHALL process `SESSION_FINISHED` webhooks for flows using Plaid Hosted Link.

#### Scenario: Hosted Link session completes
- **WHEN** a verified `SESSION_FINISHED` webhook is received
- **THEN** the system records the outcome of that Hosted Link session for the associated customer

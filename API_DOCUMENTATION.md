# Beckn Registry API Documentation

## Table of Contents
1. [Subscriber Management APIs](#subscriber-management-apis)
2. [Document Management APIs](#document-management-apis)
3. [Network Management APIs](#network-management-apis)
4. [Geographic Management APIs](#geographic-management-apis)
5. [Verification Services APIs](#verification-services-apis)

## Subscriber Management APIs

### Register Subscriber
- **Endpoint**: `/subscribers/register`
- **Method**: POST
- **Authentication**: No login required
- **Request Body**: Subscriber details
- **Response**: Registered subscriber details
- **Functionality**: 
  - Registers new subscribers to the network
  - Creates NetworkParticipant and NetworkRole records
  - Associates domains and regions with the subscriber
  - Registers public keys for encryption and signing

### Subscribe
- **Endpoint**: `/subscribers/subscribe`
- **Method**: POST
- **Authentication**: Signature verification required
- **Request Body**: Subscriber details
- **Response**: Subscriber status
- **Functionality**:
  - Subscribes an existing registered subscriber
  - Verifies the subscriber's signature
  - Updates subscriber status
  - Can handle multiple domains
  - Triggers async subscription tasks

#### Detailed Subscribe Endpoint Documentation

##### Request Flow and Checks

1. **Initial Request Processing**
   - Reads raw request body
   - Captures complete request payload for processing

2. **Authorization Header Processing**
   - Validates presence of Authorization header
   - Extracts parameters:
     - `pub_key_id`
     - `subscriber_id`
   - Failure Condition: Empty authorization parameters
   - Error Message: "Signature Verification failed"

3. **Key Verification**
   - Verifies key exists and is verified
   - Database Operation: Queries `ParticipantKey` table
   - Failure Condition: Key not found or not verified
   - Error Message: "Your signing key is not verified by the registrar!"

4. **Subscriber Identity Verification**
   - Verifies key belongs to the subscriber
   - Database Operation: Queries `NetworkRole` table
   - Failure Condition: Key not associated with subscriber
   - Error Message: "Key signed with is not registered against you"

5. **Signature Verification**
   - Validates request signature
   - Parameters:
     - Header name: "Authorization"
     - Headers: All request headers
     - Strict mode: true
   - Failure Condition: Invalid signature
   - Error Message: "Signature Verification failed"

6. **Payload Processing**
   - Parses and validates JSON payload
   - Handles:
     - Single subscriber requests
     - Multiple subscriber requests
   - Data Structure: Converts to `Subscribers` object

7. **Empty Payload Handling**
   - Handles empty subscriber list
   - Actions:
     - Triggers async subscription if needed
     - Returns current status

8. **Subscriber Processing**
   For each subscriber in the request:

   a. **Domain Validation**
      - Normalizes domain information
      - Handles:
        - Single domain
        - Multiple domains
        - No domains

   b. **Subscriber ID Verification**
      - Validates subscriber identity
      - Failure Condition: Attempt to modify different subscriber
      - Error Message: "Cannot sign for a different subscriber!"

   c. **Key Management**
      - Checks:
        - Key existence
        - Key verification status
        - Key modification permissions
      - Updates:
        - Key validity period
        - Key associations
      - Failure Conditions:
        - Attempt to modify verified key
        - Invalid key parameters

   d. **Domain Processing**
      - Checks:
        - Domain validity
        - Subscription status
        - Key creation restrictions
      - Updates:
        - Network role
        - Subscription status
        - Region information
      - Failure Conditions:
        - Invalid domain
        - Concurrent key creation and subscription modification

9. **Response Generation**
   - Format: JSON response
   - Content: Updated subscriber information
   - Variations:
     - Single subscriber response
     - Multiple subscriber response

##### Error Handling Summary

1. **Authentication Errors**
   - Missing/invalid authorization header
   - Invalid signature
   - Unverified key

2. **Authorization Errors**
   - Key-subscriber mismatch
   - Attempt to modify other subscriber's data

3. **Validation Errors**
   - Invalid domain
   - Invalid key modifications
   - Concurrent operations

4. **Business Logic Errors**
   - Invalid subscription status changes
   - Invalid key-subscription combinations

##### Security Considerations

1. **Signature Verification**
   - All requests must be signed
   - Signatures must be valid
   - Keys must be verified

2. **Access Control**
   - Subscribers can only modify their own data
   - Keys must be properly associated
   - Domain ownership must be verified

3. **Data Integrity**
   - Key modifications are restricted
   - Status changes are validated
   - Domain associations are verified

## Subscribers Subscribe Endpoint Documentation

## Endpoint Details
- **URL**: `/subscribers/subscribe`
- **Method**: POST
- **Authentication**: Signature verification required
- **Content-Type**: application/json

## Request Format

### Headers
```
Authorization: <signature>
```

### Request Body
```json
{
    "subscriber_id": "string",
    "pub_key_id": "string",
    "subscriber_url": "string",
    "type": "string",
    "domain": "string",
    "domains": ["string"],
    "signing_public_key": "string",
    "encr_public_key": "string",
    "valid_from": "timestamp",
    "valid_to": "timestamp"
}
```

## Functionality

### 1. Authentication & Authorization
1. Extracts authorization parameters from the "Authorization" header
2. Verifies the signature using the provided public key
3. Validates that:
   - The signing key exists and is verified
   - The key belongs to the subscriber
   - The signature is valid

### 2. Request Processing
1. Parses the JSON payload into a `Subscribers` object
2. Handles both single subscriber and multiple subscriber requests
3. For each subscriber:
   - Validates domain information
   - Processes key information
   - Updates or creates network roles

### 3. Key Management
1. If a new public key is provided:
   - Creates or updates the key record
   - Sets validity period
   - Associates with the network participant
2. Validates key modifications:
   - Prevents modification of verified keys
   - Allows creation of new keys

### 4. Domain Management
1. Processes domain information:
   - Handles both single domain and multiple domains
   - Creates or updates network roles for each domain
2. Updates subscriber URL if provided

### 5. Status Management
1. Updates subscriber status:
   - Sets status to "INITIATED" if changes are made
   - Triggers async subscription tasks
2. Loads region information for the subscriber

## Response Format

### Success Response (200 OK)
```json
{
    "subscriber_id": "string",
    "status": "string",
    "type": "string",
    "domain": "string",
    "domains": ["string"],
    "subscriber_url": "string"
}
```

### Error Responses

1. **Signature Verification Failed (400)**
```json
{
    "error": "Signature Verification failed"
}
```

2. **Key Not Verified (400)**
```json
{
    "error": "Your signing key is not verified by the registrar! Please contact registrar or sign with a verified key."
}
```

3. **Invalid Subscriber (400)**
```json
{
    "error": "Cannot sign for a different subscriber!"
}
```

4. **Key Modification Error (400)**
```json
{
    "error": "Cannot modify a verified registered key. Please create a new key."
}
```

5. **Subscription Modification Error (400)**
```json
{
    "error": "Cannot create a new key and modify your subscription in the same call."
}
```

## Status Codes
- `SUBSCRIBER_STATUS_INITIATED`: Initial state after subscription
- `SUBSCRIBER_STATUS_SUBSCRIBED`: Successfully subscribed
- `SUBSCRIBER_STATUS_UNSUBSCRIBED`: Unsubscribed state

## Asynchronous Processing
- Triggers `OnSubscribe` task asynchronously for status changes
- Handles background processing of subscription tasks

## Security Considerations
1. Signature verification is mandatory
2. Key modifications are restricted
3. Subscriber identity is verified
4. Domain ownership is validated

## Example Usage

### Request
```http
POST /subscribers/subscribe
Authorization: <signature>
Content-Type: application/json

{
    "subscriber_id": "example.com",
    "pub_key_id": "key-123",
    "subscriber_url": "https://example.com",
    "type": "BG",
    "domain": "example.com",
    "signing_public_key": "public-key-string",
    "encr_public_key": "encryption-key-string",
    "valid_from": "2024-01-01T00:00:00Z",
    "valid_to": "2025-01-01T00:00:00Z"
}
```

### Response
```json
{
    "subscriber_id": "example.com",
    "status": "INITIATED",
    "type": "BG",
    "domain": "example.com",
    "subscriber_url": "https://example.com"
}
```

This endpoint is crucial for managing subscriber registrations in the Beckn network, handling both new subscriptions and updates to existing ones while maintaining security through signature verification and key management.

### Disable Subscriber
- **Endpoint**: `/subscribers/disable`
- **Method**: POST
- **Authentication**: Requires signature verification
- **Request Body**: Subscriber details
- **Response**: Updated subscriber status
- **Functionality**:
  - Disables/unsubscribes a subscriber
  - Verifies the subscriber's signature
  - Updates subscriber status to UNSUBSCRIBED

### Lookup Subscribers
- **Endpoint**: `/subscribers/lookup`
- **Method**: GET
- **Authentication**: No login required
- **Query Parameters**: Search criteria
- **Response**: Matching subscribers
- **Functionality**:
  - Searches for subscribers based on criteria
  - Supports pagination
  - Returns matching subscribers with their details

### Key Generation
- **Generate Signature Keys**: `/subscribers/generateSignatureKeys`
- **Generate Encryption Keys**: `/subscribers/generateEncryptionKeys`
- **Method**: GET
- **Authentication**: No login required
- **Response**: Generated keys
- **Functionality**: Generates new cryptographic keys for subscribers

## Document Management APIs

### Submitted Documents
- **Base Endpoint**: `/submitted_documents`
- **Methods**:
  - GET `/submitted_documents` - List all submitted documents
  - GET `/submitted_documents/{id}` - Get specific document
  - POST `/submitted_documents` - Submit new document
  - PUT `/submitted_documents/{id}` - Update document
  - DELETE `/submitted_documents/{id}` - Delete document
  - POST `/submitted_documents/{id}/approve` - Approve document
  - POST `/submitted_documents/{id}/reject` - Reject document
- **Authentication**: Required
- **Functionality**: Manages document submission and verification

### Verifiable Documents
- **Base Endpoint**: `/verifiable_documents`
- **Methods**:
  - POST `/verifiable_documents/{id}/approve` - Approve document
  - POST `/verifiable_documents/{id}/reject` - Reject document
- **Authentication**: Required
- **Functionality**: Handles document verification and approval/rejection

## Network Management APIs

### Network Domains
- **Base Endpoint**: `/network_domains`
- **Methods**:
  - GET `/network_domains` - List all network domains
  - GET `/network_domains/{id}` - Get specific domain
  - POST `/network_domains` - Create domain
  - PUT `/network_domains/{id}` - Update domain
  - DELETE `/network_domains/{id}` - Delete domain
- **Authentication**: Required
- **Functionality**: Manages network domains

### Network Participants
- **Base Endpoint**: `/network_participants`
- **Methods**:
  - GET `/network_participants` - List all participants
  - GET `/network_participants/{id}` - Get specific participant
  - POST `/network_participants` - Create participant
  - PUT `/network_participants/{id}` - Update participant
  - DELETE `/network_participants/{id}` - Delete participant
  - POST `/network_participants/{id}/claim` - Initiate claim request
- **Authentication**: Required
- **Functionality**: Manages network participants and their claims

### Claim Requests
- **Base Endpoint**: `/claim_requests`
- **Methods**:
  - POST `/claim_requests/{id}/verify_domain` - Verify domain ownership
- **Authentication**: Required
- **Functionality**: Handles domain verification for claims

## Geographic Management APIs

### Countries
- **Base Endpoint**: `/countries`
- **Methods**:
  - GET `/countries` - List all countries
  - GET `/countries/{id}` - Get specific country
  - POST `/countries` - Create country
  - PUT `/countries/{id}` - Update country
  - DELETE `/countries/{id}` - Delete country
- **Authentication**: Required
- **Functionality**: Manages country information

### Cities
- **Base Endpoint**: `/cities`
- **Methods**:
  - GET `/cities` - List all cities
  - GET `/cities/{id}` - Get specific city
  - POST `/cities` - Create city
  - PUT `/cities/{id}` - Update city
  - DELETE `/cities/{id}` - Delete city
- **Authentication**: Required
- **Functionality**: Manages city information

## Verification Services APIs

### Domain Verification
- **Endpoint**: `/claim_requests/{id}/verify_domain`
- **Method**: POST
- **Authentication**: Required
- **Functionality**:
  - Verifies domain ownership
  - Checks domain TXT records
  - Updates verification status

### Document Verification
- **Endpoints**:
  - `/submitted_documents/{id}/approve`
  - `/submitted_documents/{id}/reject`
  - `/verifiable_documents/{id}/approve`
  - `/verifiable_documents/{id}/reject`
- **Method**: POST
- **Authentication**: Required
- **Functionality**: Handles document verification and approval/rejection

### Participant Verification
- **Endpoints**:
  - `/network_participants/{id}/claim`
  - `/claim_requests/{id}/verify_domain`
- **Method**: POST
- **Authentication**: Required
- **Functionality**: Manages participant verification and claims

## Authentication and Authorization

Most endpoints require authentication and authorization. The system uses:
1. Signature verification for subscriber-related operations
2. Login-based authentication for administrative operations
3. Role-based access control for different types of operations

## Error Handling

The API follows standard HTTP status codes:
- 200: Success
- 400: Bad Request
- 401: Unauthorized
- 403: Forbidden
- 404: Not Found
- 500: Internal Server Error

## Response Format

All responses are in JSON format and include:
- Status code
- Message (for errors)
- Data (for successful responses)

## Rate Limiting

The API implements rate limiting to prevent abuse. Specific limits are configured based on the endpoint and user role.


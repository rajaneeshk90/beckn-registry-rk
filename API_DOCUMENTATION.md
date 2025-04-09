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
- **Authentication**: Requires signature verification
- **Request Body**: Subscriber details
- **Response**: Subscriber status
- **Functionality**:
  - Subscribes an existing registered subscriber
  - Verifies the subscriber's signature
  - Updates subscriber status
  - Can handle multiple domains
  - Triggers async subscription tasks

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
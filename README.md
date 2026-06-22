# Shopr Wallet Service API Documentation

Generated from OpenAPI 3.1.0 specification.

## Overview

The Shopr Wallet Service is a **standalone, multi-tenant wallet service** designed to be consumed by multiple applications within the organization. It exposes two groups of endpoints:

| Namespace | Base path | Audience |
|---|---|---|
| **Server** | `/v1/api/server/shopr/wallet` | Internal services (service-to-service) |
| **Client** | `/v1/api/client/shopr/wallet` | End-user clients (frontend / mobile) |

---

## Authentication

The service implements a **decentralized, token-based authentication** pattern using **JSON Web Keys (JWK)**. Each consuming service acts as an independent **resource server** — there is no central auth gateway that all traffic must pass through.

### How it works

```
Calling Service                Wallet Service
─────────────                  ──────────────
1. Generate/load private key
2. Sign JWT (RS256 / ES256)
3. POST /v1/api/server/...  ──► 4. Receive request + Bearer token
   Authorization: Bearer <jwt>  5. Fetch JWKs from caller's JWKS endpoint
                                 6. Validate signature, issuer, audience, expiry
                                 7. Process request
```

- Each calling service **publishes its own JWKS endpoint** (e.g. `https://service-a.internal/.well-known/jwks.json`).
- The Wallet Service fetches the public key from that endpoint to **verify the token signature** — no shared secret is needed.
- Token claims should include at minimum: `iss` (issuer), `aud` (must match the wallet service identifier), `sub` (service identity), `exp` (expiry).

### Client endpoints

Client-facing routes (`/v1/api/client/...`) should use your standard **user-scoped JWT** issued by the platform's identity provider (e.g. the same token the frontend uses for other services). The user's identity is resolved from this token — no `userId` is passed in the request body for client calls.

---

## Base URL

```
http://127.0.0.1:8080
```

---

## Table of Contents

- [Server Endpoints (Service-to-Service)](#server-endpoints-service-to-service)
  - [Create Wallet](#create-wallet)
  - [Withdraw Funds](#withdraw-funds)
  - [Admin Refund User](#admin-refund-user)
  - [Purchase Order](#purchase-order)
- [Client Endpoints (User-Facing)](#client-endpoints-user-facing)
  - [Get Wallet](#get-wallet)
  - [Verify Payout Account](#verify-payout-account)
  - [Get Transaction History](#get-transaction-history)
  - [Get Transaction by ID](#get-transaction-by-id)
- [Data Models](#data-models)
- [Enumerations](#enumerations)

---

## Server Endpoints (Service-to-Service)

These endpoints are intended for **machine-to-machine** calls. All requests must carry a service-signed JWT in the `Authorization` header.

```
Authorization: Bearer <service-signed-jwt>
```

---

### Create Wallet

Provision a new wallet for a customer. Called by the onboarding or identity service after a user is verified.

| | |
|---|---|
| **Method** | `POST` |
| **Path** | `/v1/api/server/shopr/wallet` |
| **Operation ID** | `createWallet` |
| **Auth** | Service JWT (JWK-verified) |

**Request body:** [`ShoprCustomerDetails`](#shoprcustomerdetails) *(required)*

```json
{
  "userId": "string",
  "firstName": "string",
  "lastName": "string",
  "email": "user@example.com",
  "bvn": "12345678901"
}
```

| Field | Type | Required | Constraints |
|---|---|---|---|
| `userId` | string | Yes | minLength: 1 |
| `firstName` | string | Yes | minLength: 1 |
| `lastName` | string | Yes | minLength: 1 |
| `email` | string | No | — |
| `bvn` | string | Yes | exactly 11 characters |

**Response `200 OK`** — [`WalletDto`](#walletdto)

---

### Withdraw Funds

Initiate a withdrawal from a user's wallet to a registered payout account.

| | |
|---|---|
| **Method** | `POST` |
| **Path** | `/v1/api/server/shopr/wallet/withdraw` |
| **Operation ID** | `withdraw` |
| **Auth** | Service JWT (JWK-verified) |

**Request body:** [`WithdrawalDto`](#withdrawaldto) *(required)*

```json
{
  "payoutAccountId": "string",
  "amount": 0
}
```

**Response `200 OK`** — [`WalletTransactionDto`](#wallettransactiondto)

> **Resulting transaction type:** `WITHDRAWAL`

---

### Admin Refund User

Reverse a charge and credit a user's wallet. Typically triggered by an order service after a cancellation or dispute.

| | |
|---|---|
| **Method** | `POST` |
| **Path** | `/v1/api/server/shopr/wallet/refund` |
| **Operation ID** | `adminRefundUser` |
| **Auth** | Service JWT (JWK-verified) |

**Request body:** [`RefundDto`](#refunddto) *(required)*

```json
{
  "userId": "string",
  "orderId": "string",
  "amount": 0,
  "narration": "string"
}
```

| Field | Type | Required | Constraints |
|---|---|---|---|
| `userId` | string | Yes | minLength: 1 |
| `orderId` | string | Yes | minLength: 1 |
| `amount` | number | Yes | — |
| `narration` | string | No | — |

**Response `200 OK`** — [`WalletTransactionDto`](#wallettransactiondto)

> **Resulting transaction type:** `REFUND`

---

### Purchase Order

Deduct funds from a user's wallet to settle a purchase. Called by the order/checkout service.

| | |
|---|---|
| **Method** | `POST` |
| **Path** | `/v1/api/server/shopr/wallet/purchase` |
| **Operation ID** | `purchaseOrder` |
| **Auth** | Service JWT (JWK-verified) |

**Request body:** [`OrderDto`](#orderdto) *(required)*

```json
{
  "orderId": "string",
  "userId": "string",
  "amount": 0
}
```

| Field | Type | Required | Constraints |
|---|---|---|---|
| `orderId` | string | Yes | minLength: 1 |
| `userId` | string | Yes | minLength: 1 |
| `amount` | number | Yes | — |

**Response `200 OK`** — [`WalletTransactionDto`](#wallettransactiondto)

> **Resulting transaction type:** `PURCHASE`

---

## Client Endpoints (User-Facing)

These endpoints are called directly by frontend or mobile clients on behalf of an authenticated user. The user's identity is resolved from the bearer token — no `userId` field is present in request bodies.

```
Authorization: Bearer <user-jwt>
```

---

### Get Wallet

Retrieve the wallet details for the currently authenticated user.

| | |
|---|---|
| **Method** | `GET` |
| **Path** | `/v1/api/client/shopr/wallet` |
| **Operation ID** | `getWallet` |
| **Auth** | User JWT |

**Request body:** None

**Response `200 OK`** — [`WalletDto`](#walletdto)

```json
{
  "walletId": "string",
  "balance": 0,
  "bankAccountDetails": [
    {
      "accountDetailsId": "string",
      "accountNumber": "string",
      "bankName": "string",
      "provider": "string"
    }
  ]
}
```

---

### Verify Payout Account

Verify a bank account before adding it as a payout destination (name enquiry).

| | |
|---|---|
| **Method** | `POST` |
| **Path** | `/v1/api/client/shopr/wallet/verify` |
| **Operation ID** | `verifyPayoutAccount` |
| **Auth** | User JWT |

**Request body:** [`PayoutAccountDto`](#payoutaccountdto) *(required)*

```json
{
  "bankName": "string",
  "accountNumber": "string",
  "accountName": "string"
}
```

**Response `200 OK`** — [`PayoutAccountDto`](#payoutaccountdto)

The response echoes back the account details with the resolved `accountName` populated by the payment provider.

---

### Get Transaction History

Retrieve a paginated list of wallet transactions for the authenticated user.

| | |
|---|---|
| **Method** | `GET` |
| **Path** | `/v1/api/client/shopr/wallet/transactions` |
| **Operation ID** | `getTransactionHistory` |
| **Auth** | User JWT |

**Query Parameters**

| Name | Type | Required | Default | Description |
|---|---|---|---|---|
| `page` | integer (int32) | No | `0` | Zero-indexed page number |
| `size` | integer (int32) | No | `20` | Records per page |

**Example request**

```
GET /v1/api/client/shopr/wallet/transactions?page=0&size=20
```

**Response `200 OK`** — [`PagedModelWalletTransactionDto`](#pagedmodelwallettransactiondto)

```json
{
  "content": [
    {
      "transactionId": "string",
      "wallet": { "walletId": "string", "balance": 0, "bankAccountDetails": [] },
      "type": "DEPOSIT",   // set internally via payment provider webhook
      "status": "SUCCESSFUL",
      "reference": "string",
      "createdAt": "2026-01-01T00:00:00Z"
    }
  ],
  "page": {
    "size": 20,
    "number": 0,
    "totalElements": 100,
    "totalPages": 5
  }
}
```

---

### Get Transaction by ID

Retrieve a single transaction by its unique ID.

| | |
|---|---|
| **Method** | `GET` |
| **Path** | `/v1/api/client/shopr/wallet/transactions/{transactionId}` |
| **Operation ID** | `getTransactionHistoryById` |
| **Auth** | User JWT |

**Path Parameters**

| Name | Type | Required | Description |
|---|---|---|---|
| `transactionId` | string | Yes | The unique transaction ID |

**Response `200 OK`** — [`WalletTransactionDto`](#wallettransactiondto)

---

## Data Models

### ShoprCustomerDetails

| Field | Type | Required | Constraints |
|---|---|---|---|
| `userId` | string | Yes | minLength: 1 |
| `firstName` | string | Yes | minLength: 1 |
| `lastName` | string | Yes | minLength: 1 |
| `email` | string | No | — |
| `bvn` | string | Yes | exactly 11 characters |

### BankAccountDetailsDto

| Field | Type | Description |
|---|---|---|
| `accountDetailsId` | string | Unique ID for this linked account |
| `accountNumber` | string | Bank account number |
| `bankName` | string | Name of the bank |
| `provider` | string | Payment provider powering this account |

### WalletDto

| Field | Type | Description |
|---|---|---|
| `walletId` | string | Unique wallet identifier |
| `balance` | number | Current wallet balance |
| `bankAccountDetails` | array of [`BankAccountDetailsDto`](#bankaccountdetailsdto) | Linked payout accounts |

### WithdrawalDto

| Field | Type | Description |
|---|---|---|
| `payoutAccountId` | string | ID of the registered payout account to withdraw to |
| `amount` | number | Amount to withdraw |

### WalletTransactionDto

| Field | Type | Description |
|---|---|---|
| `transactionId` | string | Unique transaction identifier |
| `wallet` | [`WalletDto`](#walletdto) | Wallet snapshot at the time of the transaction |
| `type` | enum string | One of: `DEPOSIT`, `WITHDRAWAL`, `REFUND`, `PURCHASE` |
| `status` | enum string | One of: `SUCCESSFUL`, `REVERSED`, `FAILED`, `PENDING` |
| `reference` | string | External payment provider reference |
| `createdAt` | string (date-time) | ISO 8601 timestamp |

### RefundDto

| Field | Type | Required | Description |
|---|---|---|---|
| `userId` | string | Yes | ID of the user to refund |
| `orderId` | string | Yes | Order being refunded |
| `amount` | number | Yes | Amount to credit back |
| `narration` | string | No | Reason/description for the refund |

### OrderDto

| Field | Type | Required | Description |
|---|---|---|---|
| `orderId` | string | Yes | ID of the order being paid |
| `userId` | string | Yes | ID of the user making the purchase |
| `amount` | number | Yes | Amount to deduct from the wallet |

### PayoutAccountDto

| Field | Type | Description |
|---|---|---|
| `bankName` | string | Name of the bank |
| `accountNumber` | string | Bank account number |
| `accountName` | string | Registered account holder name |

### PageMetadata

| Field | Type | Description |
|---|---|---|
| `size` | integer (int64) | Page size requested |
| `number` | integer (int64) | Current page index (zero-based) |
| `totalElements` | integer (int64) | Total number of transactions |
| `totalPages` | integer (int64) | Total number of pages |

### PagedModelWalletTransactionDto

| Field | Type | Description |
|---|---|---|
| `content` | array of [`WalletTransactionDto`](#wallettransactiondto) | Transactions on this page |
| `page` | [`PageMetadata`](#pagemetadata) | Pagination metadata |

---

## Enumerations

**Transaction Type** (`WalletTransactionDto.type`)

| Value | Produced by |
|---|---|
| `DEPOSIT` | Triggered internally when the payment provider webhook notifies the wallet service of a successful inbound payment — not callable directly via this API |
| `WITHDRAWAL` | `POST /v1/api/server/shopr/wallet/withdraw` |
| `REFUND` | `POST /v1/api/server/shopr/wallet/refund` |
| `PURCHASE` | `POST /v1/api/server/shopr/wallet/purchase` |

**Transaction Status** (`WalletTransactionDto.status`)

| Value | Meaning |
|---|---|
| `SUCCESSFUL` | Transaction completed and settled |
| `PENDING` | Awaiting confirmation from payment provider |
| `REVERSED` | Transaction was reversed after success |
| `FAILED` | Transaction did not complete |

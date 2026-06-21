# Shopr Wallet API Documentation

Generated from OpenAPI 3.1.0 specification.

## Overview

The Shopr Wallet API provides endpoints for managing a user's digital wallet, including creating wallets, processing withdrawals, refunds, and purchases, viewing transaction history, and verifying payout (bank) accounts.

## Base URL

```
http://127.0.0.1:8080
```

## Authentication

The specification does not define an authentication scheme. Confirm with the backend team whether requests require a bearer token, API key, or session-based auth before integrating.

## Table of Contents

- [Wallet](#wallet)
  - [Get Wallet](#get-wallet)
  - [Create Wallet](#create-wallet)
  - [Withdraw Funds](#withdraw-funds)
  - [Admin Refund User](#admin-refund-user)
  - [Purchase Order](#purchase-order)
  - [Get Transaction History](#get-transaction-history)
  - [Get Transaction by ID](#get-transaction-by-id)
- [Payout Verification](#payout-verification)
  - [Verify Payout Account](#verify-payout-account)
- [Data Models](#data-models)

---

## Wallet

### Get Wallet

Retrieve the current user's wallet details.

| | |
|---|---|
| **Method** | `GET` |
| **Path** | `/v1/api/shopr/wallet` |
| **Operation ID** | `getWallet` |

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

### Create Wallet

Create a new wallet for a customer, supplying KYC details (name and BVN).

| | |
|---|---|
| **Method** | `POST` |
| **Path** | `/v1/api/shopr/wallet` |
| **Operation ID** | `createWallet` |

**Request body:** [`ShoprCustomerDetails`](#shoprcustomerdetails) *(required)*

```json
{
  "firsName": "string",
  "lastName": "string",
  "bvn": "12345678901"
}
```

> **Note:** The `firsName` field is spelled this way in the source spec (missing a "t"). This is preserved here for accuracy — flag with the API maintainer if this is unintentional.

| Field | Type | Required | Constraints |
|---|---|---|---|
| `firsName` | string | Yes | minLength: 1 |
| `lastName` | string | Yes | minLength: 1 |
| `bvn` | string | Yes | exactly 11 characters |

**Response `200 OK`** — [`WalletDto`](#walletdto)

---

### Withdraw Funds

Withdraw an amount from the wallet to a registered payout account.

| | |
|---|---|
| **Method** | `POST` |
| **Path** | `/v1/api/shopr/wallet/withdraw` |
| **Operation ID** | `withdraw` |

**Request body:** [`WithdrawalDto`](#withdrawaldto) *(required)*

```json
{
  "payoutAccountId": "string",
  "amount": 0
}
```

**Response `200 OK`** — [`WalletTransactionDto`](#wallettransactiondto)

---

### Admin Refund User

Issue a refund to a user's wallet. Intended for administrative use.

| | |
|---|---|
| **Method** | `POST` |
| **Path** | `/v1/api/shopr/wallet/refund` |
| **Operation ID** | `adminRefundUser` |

**Request body:** [`RefundDto`](#refunddto) *(required)*

```json
{
  "userId": "string",
  "amount": 0,
  "narration": "string"
}
```

**Response `200 OK`** — [`WalletTransactionDto`](#wallettransactiondto)

---

### Purchase Order

Deduct funds from the wallet to pay for an order.

| | |
|---|---|
| **Method** | `POST` |
| **Path** | `/v1/api/shopr/wallet/purchase` |
| **Operation ID** | `purchaseOrder` |

**Request body:** [`OrderDto`](#orderdto) *(required)*

```json
{
  "orderId": "string",
  "amount": 0
}
```

**Response `200 OK`** — [`WalletTransactionDto`](#wallettransactiondto)

---

### Get Transaction History

Retrieve a paginated list of wallet transactions.

| | |
|---|---|
| **Method** | `GET` |
| **Path** | `/v1/api/shopr/wallet/transactions` |
| **Operation ID** | `getTransactionHistory` |

**Query Parameters**

| Name | Type | Required | Default | Description |
|---|---|---|---|---|
| `page` | integer (int32) | No | `0` | Zero-indexed page number |
| `size` | integer (int32) | No | `20` | Number of records per page |

**Example request**

```
GET /v1/api/shopr/wallet/transactions?page=0&size=20
```

**Response `200 OK`** — [`PagedModelWalletTransactionDto`](#pagedmodelwallettransactiondto)

```json
{
  "content": [
    {
      "transactionId": "string",
      "wallet": {
        "walletId": "string",
        "balance": 0,
        "bankAccountDetails": []
      },
      "type": "DEPOSIT",
      "status": "SUCCESSFUL",
      "reference": "string",
      "createdAt": "2026-01-01T00:00:00Z"
    }
  ],
  "page": {
    "size": 0,
    "number": 0,
    "totalElements": 0,
    "totalPages": 0
  }
}
```

---

### Get Transaction by ID

Retrieve a single transaction by its ID.

| | |
|---|---|
| **Method** | `GET` |
| **Path** | `/v1/api/shopr/wallet/transactions/{transactionId}` |
| **Operation ID** | `getTransactionHistoryById` |

**Path Parameters**

| Name | Type | Required | Description |
|---|---|---|---|
| `transactionId` | string | Yes | ID of the transaction to retrieve |

**Response `200 OK`** — [`WalletTransactionDto`](#wallettransactiondto)

---

## Payout Verification

### Verify Payout Account

Verify a payout (bank) account's details, such as confirming the account name matches the account number/bank.

| | |
|---|---|
| **Method** | `GET` |
| **Path** | `/v1/api/shopr/verify/payout/account` |
| **Operation ID** | `verifyPayoutAccount` |

**Request body:** [`PayoutAccountDto`](#payoutaccountdto) *(required)*

> **Note:** The spec defines a request body on a `GET` request, which is unconventional and not supported by all HTTP clients/proxies. Verify with the backend whether this should be a `POST`, or whether these fields are actually expected as query parameters.

```json
{
  "bankName": "string",
  "accountNumber": "string",
  "accountName": "string"
}
```

**Response `200 OK`** — [`PayoutAccountDto`](#payoutaccountdto)

---

## Data Models

### ShoprCustomerDetails

| Field | Type | Required | Constraints |
|---|---|---|---|
| `firsName` | string | Yes | minLength: 1 |
| `lastName` | string | Yes | minLength: 1 |
| `bvn` | string | Yes | length: 11 |

### BankAccountDetailsDto

| Field | Type |
|---|---|
| `accountDetailsId` | string |
| `accountNumber` | string |
| `bankName` | string |
| `provider` | string |

### WalletDto

| Field | Type | Description |
|---|---|---|
| `walletId` | string | Unique wallet identifier |
| `balance` | number | Current wallet balance |
| `bankAccountDetails` | array of [`BankAccountDetailsDto`](#bankaccountdetailsdto) | Linked bank accounts |

### WithdrawalDto

| Field | Type |
|---|---|
| `payoutAccountId` | string |
| `amount` | number |

### WalletTransactionDto

| Field | Type | Description |
|---|---|---|
| `transactionId` | string | Unique transaction identifier |
| `wallet` | [`WalletDto`](#walletdto) | The wallet associated with this transaction |
| `type` | enum string | One of: `DEPOSIT`, `WITHDRAWAL`, `REFUND`, `PURCHASE` |
| `status` | enum string | One of: `SUCCESSFUL`, `REVERSED`, `FAILED`, `PENDING` |
| `reference` | string | Transaction reference code |
| `createdAt` | string (date-time) | Timestamp the transaction was created |

### RefundDto

| Field | Type |
|---|---|
| `userId` | string |
| `amount` | number |
| `narration` | string |

### OrderDto

| Field | Type |
|---|---|
| `orderId` | string |
| `amount` | number |

### PageMetadata

| Field | Type |
|---|---|
| `size` | integer (int64) |
| `number` | integer (int64) |
| `totalElements` | integer (int64) |
| `totalPages` | integer (int64) |

### PagedModelWalletTransactionDto

| Field | Type | Description |
|---|---|---|
| `content` | array of [`WalletTransactionDto`](#wallettransactiondto) | The transactions on this page |
| `page` | [`PageMetadata`](#pagemetadata) | Pagination metadata |

### PayoutAccountDto

| Field | Type |
|---|---|
| `bankName` | string |
| `accountNumber` | string |
| `accountName` | string |

---

## Enumerations

**Transaction Type** (`WalletTransactionDto.type`)
- `DEPOSIT`
- `WITHDRAWAL`
- `REFUND`
- `PURCHASE`

**Transaction Status** (`WalletTransactionDto.status`)
- `SUCCESSFUL`
- `REVERSED`
- `FAILED`
- `PENDING`

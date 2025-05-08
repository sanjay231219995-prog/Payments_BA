# 📘 Functional Specification Document (FSD)  
**Project:** Global Payments Cockpit – Cross-Border & Instant Payments  
**Date:** June 2025  

---

## 1. 🔍 Functional Overview

The Global Payments Cockpit enables the simulation of cross-border and instant payment flows through standardized API endpoints. It validates message integrity using ISO 20022 elements (focused on `pacs.008`) and simulates realistic operational behaviors including exceptions, holds, and manual releases.

All flows are modeled using Postman Mock Servers, with validation logic tied to business rules and mapped ISO elements.

---

## 2. API Endpoints & Operations

### 2.1 `POST /payments` – Initiate Payment

| Attribute         | Type     | Required | Rules / Mapping                  |
|------------------|----------|----------|----------------------------------|
| `payer_account`   | string   | ✅        | Must be 10-16 digit numeric       |
| `payee_account`   | string   | ✅        | Must be 10-16 digit numeric       |
| `amount`          | decimal  | ✅        | Must be > 0 (`IntrBkSttlmAmt`)   |
| `currency`        | string   | ✅        | Allowed: INR, USD, EUR           |
| `method`          | string   | ✅        | NEFT, RTP, SWIFT                 |
| `debtor_bic`      | string   | ✅        | Valid BIC (`DbtrAgt/.../BICFI`)  |
| `creditor_bic`    | string   | ✅        | Valid BIC (`CdtrAgt/.../BICFI`)  |
| `value_date`      | string   | ✅        | YYYY-MM-DD; today to today+2     |
| `remarks`         | string   | ⛔        | Optional free text               |

**Response (200):**
```json
{
  "transaction_id": "TXN12345678",
  "status": "Pending",
  "message": "Payment initiated successfully"
}
```

---

### 2.2 `GET /payments/{id}` – Check Payment Status

Returns the current status of a previously initiated transaction.

**Response:**
```json
{
  "transaction_id": "TXN12345678",
  "status": "Completed",
  "value_date": "2025-08-06",
  "method": "NEFT"
}
```

---

### 2.3 `POST /payments/{id}/cancel` – Cancel Payment

| Condition                          | Outcome           |
|-----------------------------------|-------------------|
| Status is `Pending` or `On Hold`  | Cancel allowed    |
| Status is `Completed`             | 409 Conflict      |

**Response (200):**
```json
{
  "transaction_id": "TXN12345678",
  "status": "Cancelled",
  "message": "Transaction cancelled"
}
```

---

### 2.4 `POST /payments/{id}/release` – Manual Release (Sanctions Hold)

| Condition             | Outcome           |
|----------------------|-------------------|
| Status is `On Hold`  | Released          |
| Other statuses       | 409 Conflict      |

---

## 3. 🚦 Status Codes & Transitions

| Trigger                     | New Status   |
|----------------------------|--------------|
| Payment created            | Pending      |
| Validation fails           | Rejected     |
| Sanction match             | On Hold      |
| Manual release             | Completed    |
| Cancel request (valid)     | Cancelled    |

---

## 4. Error Taxonomy

| Code                | Description                         |
|---------------------|-------------------------------------|
| `ERR_INVALID_BIC`   | BIC is missing or malformed         |
| `ERR_NEG_AMT`       | Amount is zero or negative          |
| `ERR_UNSUPPORTED_CURRENCY` | Currency not in INR/USD/EUR   |
| `ERR_INVALID_DATE`  | Value date outside allowed range    |
| `ERR_SANCTION_HIT`  | Name matched sanction list (mocked) |
| `409`               | Operation not allowed in current state |

---

## 5. Sample Workflows

### A. Happy Path
1. `POST /payments` → 200 (Pending)
2. Auto-validation passes → `Completed`

### B. Sanctions Exception
1. `POST /payments` → `status: On Hold`
2. `POST /payments/{id}/release` → `Completed`

### C. Error Path
1. `POST /payments` with `amount = -50` → `ERR_NEG_AMT`

---

## 6. ISO 20022 Mapping Summary

| Field         | ISO Element                      |
|---------------|----------------------------------|
| amount        | `IntrBkSttlmAmt/@Ccy`            |
| debtor_bic    | `DbtrAgt/FinInstnId/BICFI`       |
| creditor_bic  | `CdtrAgt/FinInstnId/BICFI`       |
| value_date    | `ReqdExctnDt`                    |
| payer/payee   | `Dbtr/Nm`, `Cdtr/Nm`             |

---

## 7. Mock Server Logic

- All validations and logic are simulated through **Postman examples**
- Status progression hardcoded for illustration
- Newman test suite executes multiple permutations

---

## 8. Linked Artifacts

- BRD: `docs/BRD.md`
- Business Rules Mapping: `mappings/business_rules_mapping.xlsx`
- Test Matrix: `qa/test_matrix.csv`

# # 📄 Business Requirements Document (BRD)  
**Project:** Global Payments – Cross-Border & Instant Payments  
 
**Date:** June 2025  

---

## 1. Purpose & Scope

This document outlines the business requirements for designing and validating a payments cockpit that standardizes **cross-border (SWIFT/ISO 20022)** and **instant domestic payment flows (NEFT/RTP)** using SwiftRef API and Postman mock APIs.

The initiative aims to simulate key operational workflows, enforce ISO 20022 semantic rules (especially pacs.008 structure), and demonstrate exception handling and operational readiness KPIs such as STP% and error taxonomies.

### ✅ In Scope
- Payment lifecycle functions: **initiate**, **check status**, **cancel**, **manual release**
- Exception scenarios:  
  - Invalid BIC  
  - Negative amount  
  - Unsupported currency  
  - Invalid value date  
  - Sanction name hit
- Business Analyst deliverables:
  - BRD, FSD, user journeys, business rules mapping  
  - Regulatory config (mock)  
  - Test plan & results  
  - Operational dashboards (mock)
  - Live bank or SWIFT network integration
  - Real sanctions screening or ledger booking
  - Full ISO 20022 pacs.008 envelope (focus is on core high-value fields)

---

## 2. Objectives

- Capture **field-level business rules** and align them to ISO 20022 message components  
- Define a clear **error taxonomy** for common failure points  
- Simulate happy and negative test paths using **Postman Mock Server**  
- Derive **STP metrics** and error patterns for operational insights

---

## 3. Stakeholders (Simulated Roles)

| Role               | Responsibilities                                  |
|--------------------|---------------------------------------------------|
| Treasury Lead      | Defines value-date policy and high-value thresholds |
| Operations Manager | Oversees exception queue, manual review SLAs     |
| QA Lead            | Drives test scenario design and defect resolution |
| Compliance Analyst | Ensures sanctions adherence and audit readiness   |

---

## 4. Assumptions & Constraints

- All flows are simulated using SwiftRef Sandbox API  
- No production or personally identifiable data used  
- ISO 20022 focus is limited to fields relevant to **pacs.008** payment initiation  
- Test evidence and metrics are based on **mock responses only**

---

## 5. Business Rules Summary

> Full rules mapping available in: `mappings/business_rules_mapping.xlsx`

| Rule ID | Rule Description                                  | Outcome        |
|---------|--------------------------------------------------|----------------|
| R01     | Amount must be greater than 0                    | Reject         |
| R02     | Debtor/Creditor BIC must be a valid 11-char code | Reject         |
| R03     | Sanctions name match triggers hold               | Hold + Review  |
| R04     | Supported currencies: INR, USD, EUR              | Reject         |
| R05     | Value date must be today or within 2 days        | Reject         |
| R06     | Cancellation allowed only for Pending/Hold       | Reject (409)   |
| R07     | Manual release allowed only for On Hold status   | Reject (409)   |

---

## 6. ISO 20022 Field Mapping (Extract – pacs.008)

| Business Field     | ISO 20022 Mapping                  |
|--------------------|-------------------------------------|
| `amount`           | `IntrBkSttlmAmt` (`value + @Ccy`)  |
| `debtor_bic`       | `DbtrAgt/FinInstnId/BICFI`         |
| `creditor_bic`     | `CdtrAgt/FinInstnId/BICFI`         |
| `debtor_name`      | `Dbtr/Nm`                          |
| `creditor_name`    | `Cdtr/Nm`                          |
| `value_date`       | `ReqdExctnDt`                      |

---

## 7. User Journeys (High-Level)

1. **Happy Path**:  
   Initiate Payment → Pass validations → Status: `Completed`  

2. **Invalid BIC**:  
   Initiate Payment → BIC validation fails → Status: `Rejected (ERR_INVALID_BIC)`  

3. **Sanctions Exception**:  
   Initiate Payment → Name match hit → Status: `On Hold`  
   → Manual release by analyst → Status: `Completed`

---

## 8. Success Criteria / Acceptance Conditions

- BRD and FSD approved by relevant stakeholders  
- Business rule mapping aligned with ISO 20022 semantics  
- ≥10 test cases executed across happy and exception flows  
- Evidence of test runs and mock results captured via Newman  
- Visual insights generated: STP % line chart, error frequency bar chart  
- Go-live checklist and mock regulatory config prepared

---

## 9. Operational Metrics 

- **STP (Straight-Through Processing) %** = `Successful (200)` / `Total Requests`  
- **Top 3 Error Codes** by volume:  
  - `ERR_INVALID_BIC`  
  - `ERR_NEG_AMT`  
  - `ERR_UNSUPPORTED_CURRENCY`  
- **Hold Release SLA** (mock) = 15 min 

---

## 10. Deliverables

| Artifact                       | Location                                 |
|-------------------------------|------------------------------------------|
| BRD & FSD                     | `docs/BRD.md`, `docs/FSD.md`             |
| Business Rule Mapping         | `mappings/business_rules_mapping.xlsx`   |
| Test Matrix                   | `qa/test_matrix.csv`                     |
| Mock API + Test Results       | `postman/*.json`, `qa/newman_results.json` |
| Analytics Dashboards          | `analytics/stp_trend.png`, `analytics/error_breakdown.png` |
| Regulatory & Go-live Docs     | `docs/Regulatory_Config.md`, `docs/GoLive_Checklist.md` |

---

## 11. Project Timeline (7-Day PoC)

| Phase              | Duration | Description                              |
|--------------------|----------|------------------------------------------|
| Discovery          | 1 day    | Business process understanding            |
| Design             | 2 days   | BRD, FSD, user flows, mappings            |
| Mock Build & Testing | 2 days | Create Postman mocks, test matrix, execution |
| Insights & Review  | 1 day    | Generate KPIs and dashboards              |
| Final Packaging    | 1 day    | Compile documentation, export evidence    |

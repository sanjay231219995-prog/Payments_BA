# Regulatory Workflow Configuration — Global Payments Cockpit (BA PoC)

> Purpose: Define how sanctions screening & regulatory controls are applied in the PoC, including
> parameters, matching strategy, statuses, SLAs, and audit fields. This is simulated on a Postman Mock Server.

---

## 1. Scope & Principles
- **Flows covered:** Cross-border (ISO 20022 semantics), RTP/NEFT-style instant payments.
- **Controls modeled:** Sanctions/name screening, watchlist hygiene, false-positive reduction, manual review & release.
- **Out of scope:** Live list ingestion, adverse media, real-time KYC/AML engines, production alerting.

---

## 2. Screening Inputs (ISO 20022 mapping)
| Data Point             | ISO 20022 Element                          | Notes                                  |
|------------------------|---------------------------------------------|----------------------------------------|
| Debtor Name            | `Dbtr/Nm`                                   | Required                                |
| Creditor Name         | `Cdtr/Nm`                                   | Required                                |
| Debtor Agent BIC       | `DbtrAgt/FinInstnId/BICFI`                  | Used for corridor risk context          |
| Creditor Agent BIC     | `CdtrAgt/FinInstnId/BICFI`                  |                                          |
| Amount & Currency      | `IntrBkSttlmAmt` (+ `@Ccy`)                 | For corridor/threshold rules            |
| Value Date             | `ReqdExctnDt`                               | Cut-off & regional policy               |
| Country (if available) | `Dbtr/PstlAdr/Ctry` / `Cdtr/PstlAdr/Ctry`   | Optional; improves precision            |

---

## 3. Watchlist & Matching Strategy (PoC)
- **Watchlist source (mock):** `watchlist/mock_watchlist.csv` (subset resembling OFAC/EU names)
- **Refresh cadence:** Manual (PoC); target daily in production
- **Matching algorithm (PoC):**  
  - Primary: **Exact & case-insensitive** match (simulated)  
  - Optional: Fuzzy (Jaro/Winkler) ≥ **0.92** (simulated toggle)
- **Normalization:** strip punctuation, multiple spaces, diacritics
- **List hygiene rules:**
  - Merge duplicates by canonical name
  - Expire entries with stale provenance > 24 months (flag)

---

## 4. Decisioning Logic & Statuses
| Rule ID | Condition/Check                                 | Outcome              | HTTP | Error Code         |
|--------:|--------------------------------------------------|----------------------|-----:|--------------------|
| AML-01  | Exact/fuzzy match ≥ threshold                    | `ON_HOLD` (manual)   | 202  | `WARN_SANCTION`    |
| AML-02  | No match                                         | Continue processing  | 200  | —                  |
| AML-03  | Analyst release (holds only)                     | `COMPLETED`          | 200  | —                  |
| AML-04  | Release attempted when not `ON_HOLD`             | `CONFLICT`           | 409  | `ERR_NOT_ON_HOLD`  |

**Status transitions:**
`PENDING → ON_HOLD → COMPLETED` (release)  
`PENDING → COMPLETED` (no hit)

---

## 5. Thresholds & Tuning (v1 → v3)
| Version | Match Threshold | Expected FP Rate | Notes                                              |
|--------:|------------------|------------------|----------------------------------------------------|
| v1      | Exact only       | ~90% of alerts   | Baseline; higher false positives                   |
| v2      | Exact + 0.94     | ~88–90%          | Minor improvement; emphasis on field validation    |
| v3      | Exact + 0.92 + allow-list for known counterparties | ~92% FP but **fewer alerts overall** | Lower alert volume; faster review SLA |

---

## 6. Allow/Deny Lists (PoC)
- **Allow-list:** Known recurring counterparties (hash of name+BIC) → skip fuzzy escalation, still log check result.
- **Deny-list:** If introduced, forces immediate `REJECTED` (not modeled in this PoC to keep focus on `ON_HOLD`).

---

## 7. SLAs & Escalation
| Item                         | Target (PoC) |
|------------------------------|--------------|
| Analyst median review time   | **≤ 5–7 min** |
| p95 review time              | **≤ 20–25 min** |
| On-hold breach (>24h)        | **0 cases**   |
| Escalation                   | If not reviewed in **4 hours**, alert Ops Manager |

---

## 8. Audit & Evidence (fields to capture)
| Field                     | Example                               |
|---------------------------|----------------------------------------|
| `payment_id`              | `pmt_1002`                             |
| `hit_name`                | `BLOCKED CORP LTD`                     |
| `match_score`             | `0.95`                                 |
| `screened_fields`         | `Dbtr/Nm`, `Cdtr/Nm`                   |
| `decision`                | `ON_HOLD` / `RELEASED` / `REJECTED`    |
| `reviewer`                | `ops_user@bank.example`                |
| `review_start/end_ts`     | ISO timestamps                         |
| `notes`                   | free text / justification              |
| `evidence_link`           | path to case note / screenshot         |

---

## 9. Data Retention & Privacy (PoC posture)
- Mask PII in shared artifacts/screenshots.
- Limit PoC datasets to synthetic names.
- In production: align to local data-retention & GDPR/DPDP mandates.

---

## 10. Testing Checklist (maps to Test Matrix)
- Positive: happy path, release when on hold
- Negative: invalid BIC (separate from AML), negative amount, invalid date
- AML: at least 2 × `ON_HOLD` scenarios; one quick release, one conflict
- Regression: v2 introduces `ERR_BIC_FORMAT` without breaking AML flow

---

## 11. Change Log
- **v1:** Baseline thresholds; high alert volume; SLA 7/22 (median/p95)  
- **v2:** Minor tuning; still high volume; priority on BIC pre-validation  
- **v3:** Allow-list + threshold refinement → fewer holds, faster reviews


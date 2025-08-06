# 🌐 Global Payments – Business Analyst Project

## 🧭 Project Overview

This project simulates the functional design and validation of a modern **Global Payments System**, supporting both:

- **Cross-border transactions** using **ISO 20022/SWIFT gpi**
- **Instant domestic payments** via **RTP/NEFT**

The goal is to demonstrate and walk through payments domain analysis, solution design, and stakeholder-ready documentation.

---

## 📦 Deliverables

| Category      | Artifacts |
|---------------|-----------|
| **Requirements** | - [Business Requirements Document (BRD)](docs/BRD.md)  
- Business rule mapping: `mappings/business_rules_mapping.xlsx` |
| **Functional Design** | - [Functional Specification Document (FSD)](docs/FSD.md)  
- Linked diagrams (payment flows, integration points) |
| **Test Assets** | - Test case matrix: `qa/test_matrix.xlsx`   
| **Insights & Analysis** | - Analytics dashboards (e.g., error rate, SLA breach): `analytics/*.png`  
- Executive summary included below |

---

## How to Use

1. **Understand Business Context**  
   Start with [`BRD.md`](docs/BRD.md) to understand scope, user journeys, and actors.

2. **Explore Functional Specs**  
   Go through [`FSD.md`](docs/FSD.md) and related diagrams to explore:
   - API schemas
   - ISO 20022 message mapping (pain.001, pacs.008, etc.)
   - Exception handling & routing logic

3. **Run/Test the APIs**  
   - Import the SwiftRef API Sandbox Collection
   - Use mock endpoints to simulate payments, status checks, and reversals
   - View test results in `qa/newman_results.html`

4. **Review Analysis Outputs**  
   - Check `analytics/` for dashboards (e.g., response time, error rates)
   - Summary insights are below
---

## Executive Summary (Key Insights)

- **96%** of simulated payments completed within SLA thresholds
- Failure cases correctly routed to exception queues with audit trail
- ISO 20022 messages mapped and validated against business rules
- Payments dynamically routed based on method, amount, and destination

---

## Project Context

This repo was designed as a **domain case walk through** of Business/Functional Analyst's responsibilities in:
- Digital payments (B2B/B2C)
- Core banking & transaction banking
- Payment orchestration platforms

> Technologies used: SwiftRef API, Postman (Mocks + Newman), Excel, Mermaid, Markdown

---

## 📁 Repository Structure

├── docs/ # BRD, FSD, flows, user journeys
├── mappings/ # Business rules, ISO 20022 field mapping
├── SwiftRef/postman/ # Postman collection and environment
├── qa/ # Test matrix and Newman test reports
├── analytics/ # PNG dashboards and visual insights
└── README.md # You're here

*Email:* sanjay231219995@email.com  


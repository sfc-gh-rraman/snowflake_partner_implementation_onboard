# Well Test Report Intelligence Platform — Partner Implementation Brief & Request for Cost Estimate

**From:** Snowflake (Account & Solutions Team)
**To:** [Implementation Partner / SI Name]
**Prospective customer:** NESR (National Energy Services Reunited)
**Re:** Production implementation of the Well Test Report Intelligence Platform on Snowflake
**Date:** [Date] · **Status:** Partner scoping / cost request

---

## 1. Purpose of this document

Snowflake is engaged with **NESR**, a prospective customer, on a platform to ingest, standardize, and
analyze end-of-job **well test reports**. To validate the approach, **Snowflake built a working proof of
concept** on our platform (data pipeline, unified schema, data-quality validation, ML, and a natural-language
analytics copilot).

We would now like **[Partner]** to **scope, price, and deliver the production implementation** on Snowflake.
This brief translates the customer's original dashboard SOW into a **concrete, Snowflake-native target
architecture** and asks you to return an **implementation cost estimate** (fixed-price and/or T&M) broken
down by the workstreams in Section 6.

Two capabilities we want explicitly scoped and priced:
1. **Automated Excel/report ingestion from Microsoft OneDrive / SharePoint via Snowflake Openflow.**
2. **Robust multi-format Excel parsing** into a unified schema.

Note on platform: the customer's original SOW referenced generic technologies (PostgreSQL/MongoDB,
Tableau/Power BI). For this engagement the platform is **Snowflake** (data, ML, Cortex AI) with a
**React + FastAPI application on Snowpark Container Services (SPCS)** — please price against Snowflake-native
services rather than substituting third-party equivalents.

---

## 2. Background — what Snowflake has already proven in the POC

Snowflake stood up a reference implementation (`NESR_WELL_TEST` database) against the five real NESR report
templates, demonstrating the full pipeline. This de-risks the build and gives your team a concrete blueprint:

- **5 source formats** landed into per-format raw tables, then unified:
  D&WO Well Test, SAWCOD SMS Flowback (With/Without Separator), UR SMS Flowback With Separator, UR & LSTK Milling.
- **Unified analytics layer** normalizing measures (gas/oil/water rates, wellhead & separator pressures,
  temperatures, H2S, solids) across formats, with **unit-aware** canonical fields.
- **Data-quality validation** (out-of-range values, BS&W bounds, choke-vs-wellhead pressure consistency).
- **Cortex Analyst semantic view** for natural-language querying.
- **Cortex Search** over report narratives and a **Cortex Agent copilot** (text-to-SQL + search + charting).
- **4 ML models** (gas deliverability/AOF forecaster, sand/solids breakthrough classifier, H2S sour-well
  safety classifier, reading anomaly detector), registered to the Snowflake Model Registry with explainability.
- **React + FastAPI web app** (command center, fields map, report deep-dive, forensics, safety, data quality,
  ML model cards, AI copilot) intended for SPCS deployment.

**The POC used synthetic-but-schema-accurate data.** The production engagement hardens this into a governed,
automated, secure product operating on **live reports flowing from the customer's OneDrive/SharePoint.**
Snowflake can share the POC assets (SQL, semantic model, agent spec, app source) with the partner under NDA.

---

## 3. Target architecture (to be implemented / hardened by partner)

```
Microsoft OneDrive / SharePoint (report libraries)
        │   Openflow Connector for SharePoint (M365) — Simple Ingest + ACLs
        ▼
Snowflake internal stage (raw files: XLSX / PDF / CSV)
        │   Parsing & extraction (Snowpark Python / AI_PARSE_DOCUMENT for PDFs+scans)
        ▼
RAW  (per-format tables + ingestion metadata)         ── Bronze
        │   dbt / SQL transforms, unit conversion, mapping
        ▼
ANALYTICS (unified readings, reports dimension, solids)── Silver/Gold
        │   validation views + Data Metric Functions
        ▼
SEMANTIC (Cortex Analyst semantic view)  •  DOCS (Cortex Search)  •  ML (models + predictions)
        │
        ▼
Cortex Agent copilot  +  React/FastAPI app on SPCS  (RBAC, SSO, audit)
```

Platform decisions (please price against these Snowflake-native services — no substitutions):
- **Storage/compute/AI:** Snowflake (Cortex Analyst, Cortex Search, Cortex Agents, Model Registry, SPCS).
- **Ingestion:** Snowflake **Openflow Connector for SharePoint** (Microsoft 365) — covers OneDrive for
  Business and SharePoint document libraries; incremental sync; optional document ACL propagation.
- **Transformation:** dbt models on Snowflake (Bronze→Silver→Gold), Snowpark Python for complex logic.
- **Application:** React + TypeScript frontend, FastAPI backend, deployed as a single SPCS service.
- **Auth:** Snowflake OAuth / SSO (Active Directory) with role-based access (uploader / analyst / admin).

---

## 4. Ingestion detail — OneDrive/SharePoint via Openflow (price explicitly)

- Deploy and configure the **Openflow Connector for SharePoint** (GA) against NESR's Microsoft 365 tenant.
  - Variant: **Simple Ingest with document ACLs** to land files into a Snowflake stage (and optionally the
    Cortex Search variant for narrative documents).
  - Entra ID (Azure AD) app registration, permissions/consent, service credentials (NESR IT to provide).
- Monitor designated **report libraries / OneDrive folders**; ingest **new and updated** files incrementally.
- Capture file metadata (path, owner, modified time, operation/country/customer tags) into ingestion metadata.
- Handle rate-limit/backoff per SharePoint API limits; alerting on sync failures.
- In scope for the partner: connector deployment, folder-to-schema routing, incremental scheduling,
  error handling/retries, and ACL mapping to Snowflake roles where required.

## 5. Excel/report parsing detail (price explicitly)

- **Structured Excel (XLSX/XLS):** template-aware parsers for the 5 known formats (samples available from the
  POC), extracting report metadata + timestamped readings into the correct RAW table per format.
- **Format variation handling:** header/section detection, sheet mapping, and a configurable mapping layer so
  new customer/country variants can be added without code rewrites.
- **Unit normalization** to canonical units (e.g., psi↔bar, bbl/day↔m³/day) at the Silver layer.
- **PDF / scanned reports (if in scope):** use **Snowflake AI_PARSE_DOCUMENT** (Cortex) for OCR/extraction
  rather than a third-party OCR stack — please price as an optional line item.
- Idempotent loads with source-file provenance and data versioning/audit.

---

## 6. Workstreams to price (please return a cost per line)

| # | Workstream | Scope highlights | Est. effort* | Cost |
|---|------------|------------------|-------------|------|
| 1 | OneDrive/SharePoint ingestion (Openflow) | Connector deploy, incremental sync, ACLs, metadata, monitoring | | |
| 2 | Multi-format Excel parsing & schema mapping | 5 templates → RAW, mapping layer, unit conversion, provenance | | |
| 3 | PDF/scanned report extraction (optional) | AI_PARSE_DOCUMENT pipeline for non-Excel reports | | |
| 4 | Unified analytics layer (dbt Bronze→Silver→Gold) | Canonical readings, reports dimension, incremental models | | |
| 5 | Data quality & validation | Range/consistency/cross-report checks, DMFs, validation reports, alerts | | |
| 6 | ML models (4) + Model Registry | Deliverability/AOF, solids, H2S, anomaly; retraining; explainability | | |
| 7 | Cortex AI layer | Semantic view (Analyst), Cortex Search, Cortex Agent copilot w/ charting | | |
| 8 | Web application (React + FastAPI on SPCS) | Upload UI, dashboards, trending, search/slicing, exports | | |
| 9 | Security, RBAC, SSO, audit logging | AD/SSO integration, roles (uploader/analyst/admin), encryption, audit | | |
| 10 | Testing, deployment, handover, training | Unit/integration/UAT, go-live, docs, user training | | |

*Please provide effort in person-days and a blended/role-based rate card.

---

## 7. Mapping to the customer's original SOW (traceability)

| Customer SOW section | Covered by workstream(s) |
|----------------------|--------------------------|
| Report Upload Interface + batch + metadata | 1, 8 |
| Data Parsing & Extraction (multi-format, AI/OCR, unit conversion) | 2, 3 |
| Data Validation & Consistency Checks | 5 |
| Data Storage (unified schema, versioning, audit) | 4, 9 |
| Analysis & Visualization (search, slicing, trending, exports, BI) | 7, 8 |
| **Advanced predictive modeling** (was out-of-scope in the SOW) | **6 — now IN scope** |
| User Auth & Access Control (SSO/AD, roles) | 9 |
| Security & Compliance (encryption, audit, standards) | 9 |
| Testing & Deployment, User Guide, Training | 10 |

The customer's SOW listed "advanced AI predictive modeling" as out-of-scope; this engagement **includes it**
(Workstream 6). Delivery is on **Snowflake/SPCS**, not PostgreSQL/MongoDB or Tableau/Power BI.

---

## 8. Assumptions, volumes & environments (to be confirmed with NESR)

- Report volume: **[N] reports/month**, **5 source formats initially**, growth to **[X]/year**.
- Sources: OneDrive for Business + SharePoint document libraries in NESR's M365 tenant.
- Environments: Snowflake account(s) for **[Dev / Test / Prod]**; NESR provides Snowflake + M365 access.
- NESR provides: sample reports per format, M365/Entra app registration, SSO/AD integration details, and a
  data steward for UAT and mapping sign-off.
- Snowflake provides: POC assets and architecture guidance, and solution-architect support during delivery.
- Compliance: encryption in transit/at rest, audit logging; adherence to **[GDPR / API standards / NESR IT policy]**.

---

## 9. Deliverables expected from the partner (for costing)

1. Design docs: data-flow diagrams, RAW→Gold schema, mapping specs, API/SPCS spec.
2. Working ingestion (Openflow) + parsing pipeline for the 5 formats.
3. dbt project (Bronze→Silver→Gold) + data-quality suite.
4. 4 ML models registered with retraining/monitoring runbook.
5. Cortex semantic view, search service, and agent copilot.
6. React + FastAPI application deployed to SPCS with SSO and RBAC.
7. Test reports (unit/integration/UAT), user guide, admin guide, and training session(s).
8. Source code handover + deployment scripts (CI/CD).

---

## 10. Commercial request

Please return, within **[10 business days]**:
- Cost per workstream (Section 6) and a **total fixed price**, plus a **T&M alternative** with a role-based rate card.
- Proposed **timeline/milestones** (indicative range ~15–22 weeks; please confirm against this scope).
- Team composition, Snowflake competencies/certifications, and relevant prior delivery references.
- Any assumptions/dependencies affecting price.
- Optional line items: PDF/OCR extraction (WS3) and a post-go-live support/maintenance agreement.

**Snowflake contacts:** [Account Executive] · [Solutions Engineer / SA] · [Partner Manager]

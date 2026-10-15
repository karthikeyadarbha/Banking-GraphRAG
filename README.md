# Enterprise Banking GraphRAG Architecture

## Executive Overview

This repository contains the enterprise architecture, semantic ontologies, SHACL validation shapes, 3NF conformed staging data models, Data Vault 2.0 designs, and automated CI/CD governance pipelines for the **Semantic-Graph Ontological Mortgage & Retail Banking System**.

By unifying **BIAN Service Domains**, **FIBO (Financial Industry Business Ontology)** semantics, **Data Vault 2.0**, **Neo4j Knowledge Graphs**, and **dbt MetricFlow**, this system replaces unreliable vector-only RAG with **100% deterministic, audit-traceable, zero-hallucination Ontological GraphRAG** for agentic AI in enterprise banking.

---

## Checkpoint Status: CHECKPOINT-01
* **Milestone Name:** CHECKPOINT-01: SEMANTIC_ARCHITECTURE_&_GOVERNANCE_FROZEN
* **Version Tag:** v2.0.0-design-approved
* **Status:** All semantic ontologies, business rule axioms, 3-Gate ingestion fences, data privacy tokenization controls, and architecture decision logs are formally approved and frozen for implementation.

---

## Repository Directory Layout

```text
banking-graphrag-architecture/
├── docs/
│   ├── adr/
│   │   ├── ADR-LOG-v2.md                       # Formal Decision Log (ADRs 001–007)
│   │   └── ADR-LOG-v1.md                       # Initial Decision Log (ADRs 001–005)
│   ├── digests/
│   │   └── medium-digest-banking-graphrag.md   # Publication-ready executive summary digest
│   └── diagrams/
│       ├── system-dependency-map-v2.png       # End-to-End System & Security Map (v2)
│       ├── strategy-traceability-map-v2.png   # Strategic Lineage Matrix (v2)
│       ├── system-dependency-map-v1.png       # Baseline System Map
│       └── strategy-traceability-map-v1.png   # Baseline Strategy Map
│
├── ontology/
│   ├── core/
│   │   └── fibo_banking_core.ttl               # W3C OWL 2 Ontology with Property Chain Axioms
│   ├── shapes/
│   │   └── shapes-lending.ttl                  # SHACL Data Quality Shapes
│   └── queries/
│       └── business-rules-test-suite.sparql    # SPARQL Regression Query Suite (Rules 1–7)
│
├── data-models/
│   ├── staging-3nf/
│   │   ├── staging-data-model-erd-v2.png       # Conformed 3NF ERD v2 (FPE & ABAC Attributes)
│   │   └── staging-data-model-erd-v1.png       # Baseline 3NF ERD
│   └── raw-vault/
│       └── data-vault-architecture.md          # Hubs, Links, Satellites, PIT & Bridge Tables
│
└── ci-cd/
    ├── scripts/
    │   └── validate_ontology.py                # pySHACL & Turtle Quality Linter Script
    └── workflows/
        └── ontology-governance-ci.yml          # GitHub Actions CI/CD Pipeline
```

---

## Key Updates & Latest Architectural Enhancements (v2 Release)

### 1. Data Privacy & Security Overhaul (GDPR / CCPA / NPI)
* **Format-Preserving Encryption (FPE):** Replaced cleartext SSN/EIN with `TOKENIZED_TAX_ID` in `STG_PARTY`.
* **KMS AES-256 Column Encryption:** Encrypted sensitive names (`ENCRYPTED_FIRST_NAME`, `ENCRYPTED_LAST_NAME`).
* **Device & Cyber Anonymization:** Replaced raw identifiers with `HASHED_DEVICE_ID` (SHA-256) and `ANONYMIZED_IP_SUBNET` in `STG_DIGITAL_IDENTITY`.
* **Pre-LLM Anonymizer Gateway:** Enforced prompt sanitization before payloads reach FastMCP tools or LLM context windows.

### 2. Attribute-Based Access Control (ABAC) & BCBS 239 Lineage
* **Data Classification Tags:** Added explicit `DATA_CLASSIFICATION` metadata (`RESTRICTED_PII`, `CONFIDENTIAL`) to all 10 staging entities for dynamic row/column-level masking in Databricks/Snowflake.
* **Cryptographic Lineage:** Standardized `RECORD_SOURCE` (operational system tracking) and `LOAD_DTS` (WORM immutable timestamps) across all 3NF tables and Data Vault Satellites.

### 3. Fair Lending & Non-Bias Guardrails (FCRA / ECOA / NIST AI RMF 1.0)
* **Demographic Exclusion Verification:** Explicitly tagged `STG_RISK_ASSESSMENT` with `NON_DEMOGRAPHIC_BIAS_FLAG` ensuring race, gender, age, and protected class attributes are excluded from automated credit scoring and Probability of Default (PD) models.

### 4. Updated Architecture Decision Log (`ADR-LOG-v2.md`)
* **ADR-001:** W3C OWL 2 / FIBO / BIAN Modular Monorepo Architecture.
* **ADR-002:** 3-Gate Control Ingestion Architecture.
* **ADR-003:** Hybrid Storage Layer (Data Vault 2.0 Lakehouse + Neo4j Graph).
* **ADR-004:** AI Decoupling via dbt MetricFlow & FastMCP Protocol Wrappers.
* **ADR-005:** Conformed 3NF Staging Layer as Intermediate Pipeline Bridge.
* **ADR-006 (New):** Format-Preserving Encryption (FPE) PII Tokenization & ABAC Access Controls.
* **ADR-007 (New):** Cryptographic Data Lineage Tracking & Immutable WORM Audit Storage.

---

## Core Architectural Layers & 3-Gate Control Fence

```text
 [ OPERATIONAL SOURCE SYSTEMS ]
 (KYC/AML, Raw Data Lake, Mortgage POS/CRM, LOS Encompass, Collateral Admin, Servicing MSP, Capital Markets)
                                │
                                ▼
 [ GATE 1: INGESTION BOUNDARY ]
 └── Canonical Data Contracts (Kafka / Avro Schemas with FPE PII Tokenization)
                                │
                                ▼
 [ GATE 2: STAGING & SHACL QUALITY GATE ]
 ├── Conformed 3NF Staging Layer (STG_*)
 ├── pySHACL Quality Linter (shapes-lending.ttl)
 └── DLQ Automated Quarantine & Alerting Engine
                                │
                                ▼
 [ GATE 3: CORE STORAGE & KNOWLEDGE GRAPH ]
 ├── Data Vault 2.0 (Hubs, Links, Satellites, PIT & Bridge Tables)
 └── Neo4j Knowledge Graph (fibo_banking_core.ttl + OWL Property Chains)
                                │
                                ▼
 [ AGENTIC AI EXECUTION LAYER ]
 ├── dbt MetricFlow Semantic Layer (Certified DTI, LTV, RAROC, RWA Metrics)
 └── FastMCP Read-Only Tool Wrappers (NIST AI RMF 1.0 Compliant)
```

---

## Governance-as-Code & SPARQL Test Suite

Every pull request updating `.ttl` or shape files automatically triggers the `ontology-governance-ci.yml` pipeline executing `validate-ontology.py`.

### 7 Refined Business Rules Tested (`business-rules-test-suite.sparql`):
1. **Rule 1 (Household Exposure):** Transitive property chain inferring indirect household liability (`bank-sem:hasIndirectRiskExposure`).
2. **Rule 2 (Cross-Collateralization):** Symmetric property chain detecting shared collateral across facilities (`bank-sem:isCrossCollateralizedWith`).
3. **Rule 3 (Multi-Hop UBO):** Transitive corporate ownership traversal (`bank-sem:hasIndirectUBOExposure`).
4. **Rule 4 (Synthetic Fraud):** Detection of shared digital footprints without household bonds (`bank-sem:SyntheticFraudRiskFlag`).
5. **Rule 5 (RAROC AI Firewall):** Intersection class requiring GL revenue and Risk assessment before profitability calculation (`bank-sem:ValidRAROCEvaluation`).
6. **Rule 6 (Capital Reserve Trigger):** Automated Basel III RWA penalty flag on collateral downgrade (`bank-sem:RequiresCapitalBufferIncrease`).
7. **Rule 7 (Household Credit Guardrail):** Mandatory income verification for all primary household members (`bank-sem:FullyVerifiedHousehold`).

---

## Summary of Project Artifacts

| Artifact File | Description / Purpose |
| :--- | :--- |
| **`project-readme-v2.md`** | Primary project documentation, architectural overview, and directory guide. |
| **`staging-data-model-erd-v2.png`** | Conformed 3NF Staging ERD v2 with FPE encryption, ABAC metadata, and lineage columns. |
| **`system-dependency-map-v2.png`** | End-to-end 3-Gate system architecture and security overlay diagram. |
| **`strategy-traceability-map-v2.png`** | Strategic alignment matrix linking business goals to technical control outcomes. |
| **`architectural-decision-records-v2.md`** | Formal decision log documenting ADRs 001–007. |
| **`medium-digest-banking-graphrag.md`** | Publication-ready Medium article digesting the strategic architecture. |
| **`shapes-lending.ttl`** | SHACL data quality shape constraints for loan ingestion. |
| **`business-rules-test-suite.sparql`** | SPARQL regression query suite validating 7 business rule axioms. |
| **`validate-ontology.py`** | Automated Python linter for RDF/SHACL validation. |
| **`ontology-governance-ci.yml`** | GitHub Actions workflow for automated PR governance gates. |

---

## Next Steps: Phase 2 Implementation Roadmap

1. **Materialize Data Vault 2.0 Pipelines:** Build dbt staging transformation models and SHA-256 hash key generation scripts.
2. **Stream CDC to Neo4j:** Implement PySpark graph projection pipelines streaming Data Vault changes into Neo4j nodes and edges.
3. **Deploy FastMCP Protocol Wrappers:** Wrap certified dbt metrics and Cypher queries in read-only FastMCP tools for LLM integration.

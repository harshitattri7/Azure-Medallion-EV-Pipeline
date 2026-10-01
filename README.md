# Azure Medallion EV Pipeline

A production-grade end-to-end Azure data engineering project built for **VoltGrid Mobility Solutions** — an EV charging franchise network operating across Australia.

Built a complete Medallion Lakehouse (Bronze → Silver → Gold) on Azure to centralise visibility across 120+ charging stations, eliminate billing mismatches, and enable real-time charger health monitoring.

---

## Project Overview

| Detail | Value |
|---|---|
| **Domain** | Electric Vehicle (EV) Charging Intelligence |
| **Scale** | ~50K records across all source systems |
| **Architecture** | Medallion Lakehouse — Bronze → Silver → Gold |
| **Cloud** | Microsoft Azure |
| **Processing** | Batch + Structured Streaming (PySpark on Databricks) |
| **Serving** | Power BI Dashboards + Synapse Analytics |
| **Region** | Central India (`centralindia`) |

---

## Business Problems Solved

- No centralised visibility across franchise stations
- Revenue leakage from billing mismatches between payments and invoices
- No real-time charger health or battery monitoring
- Invoice tracking scattered across email and PDFs
- No predictive maintenance signals from IoT telemetry

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      SOURCE SYSTEMS                          │
│  VoltGrid REST API  |  Blob Storage  |  Azure Event Hub     │
│  (payments, fleet)  | (CSV/PDF/XML)  |  (IoT telemetry)     │
└──────────────┬──────────────┬────────────────┬──────────────┘
               │              │                │
               ▼              ▼                ▼
┌─────────────────────────────────────────────────────────────┐
│                     INGESTION LAYER                          │
│    Azure Data Factory (batch)  |  Databricks Auto Loader    │
│    (metadata-driven pipelines) |  (streaming ingestion)     │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│              ADLS Gen2 — Medallion Layers                    │
│                                                              │
│  Bronze  ← raw, append-only, all source formats             │
│  Silver  ← cleaned, validated, MERGE upsert (Delta)         │
│  Gold    ← aggregated, star schema, analytics-ready          │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                    SERVING LAYER                             │
│    Power BI (14 dashboards)  |  Synapse Analytics (SQL)     │
└─────────────────────────────────────────────────────────────┘
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| **Ingestion** | Azure Data Factory, Databricks Auto Loader |
| **Storage** | ADLS Gen2 (Delta Lake format) |
| **Processing** | Azure Databricks (PySpark) |
| **Orchestration** | ADF Metadata-Driven Pipelines |
| **Security** | Azure Key Vault, Service Principal OAuth |
| **Governance** | Unity Catalog, External Locations, RBAC |
| **Streaming** | Azure Event Hubs + Databricks Structured Streaming |
| **Serving** | Power BI, Azure Synapse Analytics |

---

## Repository Structure

```
Azure-Medallion-EV-Pipeline/
│
├── README.md
├── ARCHITECTURE.md                         ← full system design doc
│
├── 01_azure_architecture_setup/            ← Azure provisioning, ADLS Gen2, Key Vault
├── 02_bronze_ingestion_adf/                ← ADF linked services, initial ingestion pipeline
├── 03_bronze_cdc_and_audit_logging/        ← watermark-based CDC, pipeline audit logging
├── 04_unity_catalog_governance/            ← Unity Catalog, managed vs external tables
├── 05_metadata_driven_framework/           ← config-table-driven ADF ingestion framework
├── 06_blob_to_bronze_migration/            ← CSV, XML, PDF, JSON multi-format ingestion
├── 07_adf_databricks_integration/          ← ADF orchestrating Databricks notebooks
├── 08_silver_transformation_layer/         ← Silver MERGE upsert, data quality validation
├── 09_silver_blob_transformations/         ← Silver layer for invoices, reports, realtime
├── 10_gold_dimensional_model/              ← Gold star schema dims, facts, incremental load
├── 11_gold_adf_orchestration/              ← ADF master pipeline for Gold transformations
├── 12_streaming_eventhub_databricks/       ← Event Hub → Bronze structured streaming
├── 13_silver_streaming_transformation/     ← Silver streaming for vehicle battery telemetry
└── 14_gold_streaming_mart/                ← Live Gold mart for charging progress monitoring
```

---

## Key Patterns Implemented

- **Medallion Architecture** — Bronze (raw) → Silver (cleaned) → Gold (aggregated)
- **Delta Lake** — ACID transactions, time travel, schema evolution, MERGE upsert
- **Service Principal OAuth** — secure, least-privilege ADLS access (no hardcoded keys)
- **Azure Key Vault** — centralized secret management with Databricks secret scope
- **CDC & Watermarking** — incremental load pattern for large API-sourced tables
- **Metadata-Driven Pipelines** — single ADF pipeline driven by a JSON config table
- **Unity Catalog** — three-level namespace (catalog.schema.table), external locations
- **Structured Streaming** — Event Hub → Bronze → Silver → Gold live mart
- **Auto Loader** — file discovery with schema inference for blob-sourced data

---

## Star Schema — Gold Layer

```
                    ┌─────────────────┐
                    │  DimTime        │
                    │  DimStation     │
                    │  DimCharger     │
  FactChargingSession ──► DimVehicle  │
  FactPayments         ──► DimCustomer │
  FactEnergyConsumption──► DimEmployee │
  FactComplaints       ──► DimFranchise│
  FactInvoice                          │
                    └─────────────────┘
```

---

## Running the Project

### Prerequisites
- Azure subscription with Contributor access
- Databricks workspace (Premium tier for Unity Catalog)
- ADLS Gen2 storage account
- Azure Key Vault + registered Service Principal

### Setup Steps
1. Provision Azure resources following `01_azure_architecture_setup/`
2. Configure Key Vault secrets and Databricks secret scope
3. Set up Unity Catalog external locations for Bronze/Silver/Gold
4. Run ADF pipelines in order: Bronze → Silver → Gold
5. Deploy Databricks jobs for streaming pipelines (Event Hub)

---

## Cost Reference

| Resource | Estimated Cost |
|---|---|
| ADLS Gen2 (~10–15 GB) | ~₹20–30/month |
| Azure Key Vault | ~₹5 total |
| Databricks (per session) | ~₹40–45 |
| **Total across project** | **~₹850–900** |

> New Azure accounts get ₹13,370 (~$200 USD) free credits — covers this entire project.

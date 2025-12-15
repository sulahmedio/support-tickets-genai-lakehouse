<h1> Support Tickets GenAI Lakehouse Pipeline (Databricks) </h1>
Overview

This project demonstrates an end-to-end Databricks Lakehouse pipeline using a real-world customer support tickets dataset. The workflow follows Bronze → Silver → Gold layering and is designed to mirror common GenAI pipeline preparation steps: data ingestion, quality validation, salvage/quarantine handling, and creation of LLM-ready text chunks.

# Why this project
Support tickets are a high-value GenAI use case because they contain unstructured text that can be used for:
- Ticket classification / routing
- Summarization
- Knowledge base retrieval (RAG)
- Embeddings + semantic search
- ustomer sentiment and satisfaction insights

This project focuses on preparing high-quality, traceable data for those downstream use cases.

# Dataset
Source: Kaggle (Customer Support Tickets)
File: customer_support_tickets.csv
 *Note: The raw dataset is not included in this repository. Download it from Kaggle and upload to Databricks (instructions below).*

# Architecture (Bronze → Silver → Gold)

```
 Raw CSV (Kaggle)
   ↓
Bronze (Delta) — raw ingestion + schema normalization
   ↓
Silver (Delta) — data profiling, salvage logic, quarantine output
   ↓
Gold (Delta) — LLM-ready text chunks + metadata for embeddings / RAG
```
# Tables created
```
genai_support_pipeline.bronze_support_tickets
genai_support_pipeline.silver_support_tickets_clean
genai_support_pipeline.silver_support_tickets_quarantine
genai_support_pipeline.gold_llm_chunks
```
# Key Features
1) Secure ingestion pattern (no public DBFS)
This project ingests the CSV into Databricks and persists it as a Bronze Delta table under a project database/schema.

2) Data profiling + schema contamination detection
During Silver development, the dataset was profiled and found to contain:
- High null rates in certain fields
- Values appearing in the wrong columns (schema contamination)

3) Salvage + Quarantine approach (Silver)
Instead of dropping large amounts of data, the Silver layer implements:
- Salvage logic (recover known categorical values when misfiled)
- Quarantine output with reject_reason for auditability

Example outcomes observed during development:
Bronze record count: 18,762
Silver clean: 10,019
Silver quarantine: 8,743

4) GenAI-ready text preparation (Gold)
Silver outputs a consolidated text field (text_for_llm) and Gold converts records into chunked text rows suitable for:
- LLM processing
- Eembeddings generation
- Downstream retrieval workflows

Chunking was implemented to be adaptive (no unnecessary fragmentation):
- CHUNK_SIZE = 500
- Observed text lengths: max_len = 437, avg_len ≈ 178.6
- Result: most records naturally produced one chunk, but the pipeline scales automatically for longer text.

# Notebooks
Exported Databricks notebooks are available in /notebooks:
- ```01_ingest_bronze```
- ingest CSV → Bronze Delta table
- ```02_silver_clean_validate_v2```
- profiling, salvage logic, quarantine output, LLM-ready text field
- ```03_gold_llm_chunks```
- chunking + gold table creation for GenAI workflows

# How to Run (Databricks)
1) Upload data
- Download the dataset from Kaggle
- Upload customer_support_tickets.csv into Databricks using the UI upload flow

2) Create/attach compute
- Use an all-purpose cluster for notebooks

3) Run notebooks in order
   Run in sequence:
- ```01_ingest_bronze```
- ```02_silver_clean_validate_v2```
- ```03_gold_llm_chunks```

# Future Enhancements
- Add automated expectations (e.g., Delta Live Tables expectations)
- Add CI/CD (Databricks Repos + GitHub Actions)
- Add embeddings generation (Databricks model endpoint) and vector search
- Add dashboard (Databricks SQL or Power BI) for operational reporting

# Author
Sulaiman Ahmed
- LinkedIn: https://www.linkedin.com/in/sulahmedio/
- GitHub: https://github.com/sulahmedio

---
title: "SQL as Universal Data Fabric"
date: 2026-03-27 09:00:00 -0500
categories: [Essays]
tags: [technology]
---

Maybe one of the most consequential decisions in modern data engineering isn't which database to choose — it's which *wire protocol* to standardize on. PostgreSQL's wire protocol has become the lingua franca of the data world, and a new generation of tools has emerged that exploits this convergence to dramatic effect.

The architecture described in this paper composes three independently valuable tools — Steampipe, DuckDB, and PostgreSQL — into a pipeline where every stage speaks the same dialect. A SQL query against a directory of CSV files looks identical to a query against a live API, which looks identical to a query against the final warehouse. The cognitive load collapses. The tooling unifies. The debugging surface shrinks to one language an analyst already knows.

This isn't a minor ergonomic improvement. It represents a fundamental shift in how we think about data integration: rather than writing bespoke connectors, format-specific parsers, and translation layers between incompatible systems, we project every data source into a single query surface and let SQL — the most widely understood data language in the world — do the heavy lifting.

## Steampipe and the *Pipe Ecosystem: SQL Over Everything

Steampipe, developed by Turbot, began as a tool for querying cloud infrastructure via SQL. Its genius was recognizing that the Foreign Data Wrapper (FDW) pattern — PostgreSQL's mechanism for querying external data sources as if they were native tables — could be generalized far beyond its original intent. Steampipe embeds a PostgreSQL instance (on port 9193 by default) and exposes plugins that translate SQL queries into API calls, file reads, or service interactions.

The ecosystem has expanded considerably. Flowpipe adds workflow orchestration. Powerpipe provides dashboards and compliance benchmarks. Tailpipe handles log ingestion. Together, the *pipe family covers a remarkable breadth of the data lifecycle, all unified by a single principle: present a PostgreSQL interface and let SQL be the query language.

### What This Means in Practice

Consider a healthcare data platform that needs to integrate provider data from the CMS National Plan and Provider Enumeration System (NPPES), geographic health indicators from the CDC PLACES dataset, Health Professional Shortage Area designations, and dental insurance marketplace data from QHP Landscape files. Without Steampipe, each of these requires a dedicated ingestion pathway — a Python script for the NPPES bulk download, another for the CDC API, a third for the HPSA shapefiles, a fourth for the QHP CSVs.

With Steampipe's CSV plugin, every flat file becomes a queryable table. A single `.spc` configuration file maps directory globs to connection names, and the data is immediately available via standard SQL:

```sql
SELECT npi, provider_first_name, provider_last_name, 
       healthcare_provider_taxonomy_code_1
FROM nppes_npi.nppes_npi_full
WHERE provider_business_practice_location_address_state_name = 'Missouri'
  AND entity_type_code = '1';
```

This query runs against a 9GB CSV file. No ETL has occurred. No data has been loaded into any database. Steampipe reads the file, applies predicate pushdown where possible, and returns results through the PostgreSQL wire protocol. Any tool that can connect to PostgreSQL — psql, DBeaver, DataGrip, a .NET application using Npgsql — can execute this query without modification.

### The Plugin Architecture

Steampipe plugins are authored in Go and compiled as independent binaries. Each plugin implements a schema definition (which columns exist, their types) and a hydration function (how to fetch the data). The community has produced over 150 plugins spanning AWS, Azure, GCP, GitHub, Jira, Salesforce, Kubernetes, and dozens more. When no community plugin exists for a proprietary API, the Go SDK makes authoring one straightforward — and the result is indistinguishable from a native PostgreSQL table.

This extensibility is the key architectural lever. A custom Steampipe plugin wrapping an internal REST API gives that API a SQL interface permanently. Every downstream consumer — BI tools, data pipelines, ad-hoc analysts — gets access through the same mechanism. The API's authentication, pagination, rate limiting, and error handling are encapsulated once in the plugin and never reimplemented.

## DuckDB: The Understated Staging Engine

In the reference architecture, DuckDB occupies the staging layer: raw data lands in DuckDB as all-TEXT columns, transforms are applied, and clean data is promoted to PostgreSQL. This is a perfectly valid use of DuckDB, but it significantly undersells the tool's capabilities and the optionality it creates.

### What DuckDB Actually Is

DuckDB is an in-process columnar analytical database — often described as "SQLite for analytics." Like SQLite, it requires no server, no configuration, and no separate process. Unlike SQLite, it's built from the ground up for analytical (OLAP) workloads: columnar storage, vectorized execution, and a query optimizer designed for aggregations, joins, and scans over large datasets.

DuckDB reads Parquet, CSV, JSON, Excel, and Arrow IPC files natively, without requiring any data to be loaded first. It can query files directly from S3. It implements a substantial subset of PostgreSQL's SQL dialect, including window functions, CTEs, UNNEST, and lateral joins. Recent versions add support for spatial types and functions, full-text search, and an extension ecosystem that continues to grow.

### Unlocking DuckDB's Full Potential

In the staging role described in the reference architecture, DuckDB receives extracted data, holds it in TEXT columns for type-safe transformation, and serves as a checkpoint between extraction and loading. But the architecture could exploit DuckDB far more aggressively:

**Exploratory Data Analysis Before Schema Design.** Before committing to a PostgreSQL schema, an engineer can point DuckDB at the raw files and run analytical queries instantly. What's the cardinality of this column? How many NULLs? What's the distribution of values? Are there encoding inconsistencies? DuckDB answers these questions in seconds against multi-gigabyte files, without any loading step. This turns schema design from a speculative exercise into an empirically grounded one.

**Data Quality Validation at Wire Speed.** DuckDB's columnar engine excels at the exact queries that data validation requires: full-column scans, distinct counts, pattern matching, and cross-file joins. A validation pass that checks referential integrity across staging tables, flags orphaned foreign keys, and computes completeness metrics runs orders of magnitude faster in DuckDB's columnar engine than it would in a row-oriented staging database.

**Intermediate Analytical Queries Without PostgreSQL.** For many analytical questions, the answer lives entirely within the staged data and never needs to touch PostgreSQL at all. DuckDB can serve as a lightweight analytical engine for development, testing, or even production workloads that don't require PostgreSQL's concurrency or persistence guarantees. A developer iterating on a complex analytical query can run it against DuckDB locally in milliseconds, then deploy the validated query against PostgreSQL.

**Dead Letter Analysis.** The reference architecture includes a dead-letter table for rows that fail transformation. In DuckDB, analyzing these failures is trivial — the dead-letter table sits alongside the staging tables, and a single query can correlate failures with source file characteristics, identify systematic patterns, and quantify data quality issues before they propagate.

**Parquet as a First-Class Interchange Format.** DuckDB reads and writes Parquet natively and efficiently. This means the staging layer can persist its outputs as Parquet files — compressed, columnar, and self-describing — which can be consumed by any tool in the modern data ecosystem (Spark, Polars, pandas, Arrow). The staging layer becomes not just a waypoint to PostgreSQL but a hub that can feed multiple downstream consumers.

### The In-Process Advantage

Because DuckDB runs in-process, it eliminates an entire category of infrastructure complexity. There is no DuckDB server to provision, secure, monitor, or maintain. The C# ETL process opens a DuckDB connection the same way it opens a SQLite connection — a file path and a NuGet package (`DuckDB.NET.Data.Full`). The staging database is a single file on disk that can be inspected, copied, versioned, or discarded. If the ETL fails halfway through, the staging file captures exactly how far it got.

This also means the staging layer scales with the compute, not the infrastructure. Running the ETL on a larger EC2 instance or a developer's workstation with more RAM automatically gives DuckDB more resources. There is no cluster to resize, no connection pool to tune, no shared resource to contend for.

## PostgreSQL as Warehouse: The Gravity Well

PostgreSQL is the destination for a reason that goes beyond its technical merits as a database engine (though those are considerable). PostgreSQL is the center of gravity for the modern data ecosystem. More tools connect to PostgreSQL natively than to any other database. More engineers know PostgreSQL's SQL dialect than any other. More extensions exist for PostgreSQL than for any other relational database.

### Bulk Loading at Scale

Getting data into PostgreSQL efficiently is a solved problem, though the solution is less widely known than it should be. The `COPY` command (and its programmatic equivalent via Npgsql's `BeginBinaryImport`) loads data at wire speed — hundreds of thousands of rows per second on modest hardware. For bulk loads, this is the only mechanism worth considering. Row-by-row INSERT statements, even batched, cannot compete.

The pattern is straightforward:

```csharp
await using var writer = conn.BeginBinaryImport(
    "COPY core.provider (npi, first_name, last_name, credential, entity_type) " +
    "FROM STDIN (FORMAT BINARY)");

foreach (var row in stagedRows)
{
    writer.StartRow();
    writer.Write(row.Npi, NpgsqlDbType.Bigint);
    writer.Write(row.FirstName, NpgsqlDbType.Text);
    writer.Write(row.LastName, NpgsqlDbType.Text);
    writer.Write(row.Credential, NpgsqlDbType.Text);
    writer.Write(row.EntityType, NpgsqlDbType.Text);
}

await writer.CompleteAsync();
```

For a 7-million-row provider table, this completes in under a minute. Combined with PostgreSQL's transactional DDL (even `CREATE INDEX` and `ALTER TABLE` are transactional), a bulk load can be wrapped in a transaction that either fully succeeds or fully rolls back, leaving the warehouse in a consistent state.

### UPSERT and Incremental Loading

PostgreSQL's `INSERT ... ON CONFLICT` (UPSERT) is essential for the enrichment pattern that dominates real-world data integration. A canonical provider record from NPPES gets enriched by CMS Doctors and Clinicians data (adding medical school, graduation year), then by Open Payments data (adding payment relationships), then by PECOS enrollment data (adding Medicare participation). Each source contributes different columns to the same entity. UPSERT handles this cleanly:

```sql
INSERT INTO core.provider (npi, med_school, grad_year, data_source_id)
VALUES ($1, $2, $3, $4)
ON CONFLICT (npi) DO UPDATE SET
    med_school = COALESCE(EXCLUDED.med_school, core.provider.med_school),
    grad_year = COALESCE(EXCLUDED.grad_year, core.provider.grad_year),
    data_source_id = EXCLUDED.data_source_id,
    updated_at = NOW();
```

The `COALESCE` pattern ensures that enrichment sources add data without overwriting values already present from higher-priority sources. The `data_source_id` and `updated_at` columns provide full lineage tracking: which source last touched each row, and when.

### The Extension Ecosystem

PostgreSQL's extension model turns it from a database into a platform. Apache AGE adds labeled property graph capabilities — Cypher queries running inside PostgreSQL, with graph nodes referencing relational primary keys. PostGIS adds spatial types and functions, enabling geographic queries (e.g., "find all providers within 30 miles of a Health Professional Shortage Area"). `pg_trgm` adds trigram-based fuzzy matching for entity resolution when clean keys don't exist.

The graph capability deserves special attention. In the reference architecture, Apache AGE is an optional layer for relationship traversal queries: "which physicians share organizational affiliations?", "what referral patterns connect specialists?", "which pharmaceutical companies have payment relationships spanning multiple practice groups?" These queries are awkward to express in relational SQL (recursive CTEs, multiple self-joins) but natural in Cypher. AGE runs these graph queries inside the same PostgreSQL process, against data that references the same primary keys, with no synchronization layer between a separate graph database and the relational warehouse.

## The C# Transform Layer: ELT with Type Safety

The choice of C# for the transform layer is deliberate and carries several architectural advantages over the more common Python-based ETL tooling.

### Why C# Over Python for Transforms

The transforms in this architecture are not ad-hoc scripts — they are a structured pipeline with well-defined stages: column mapping, type casting, normalization (pivoting repeating groups), geographic resolution, and key linking. Each stage has a clear contract (an `ITransform` interface), is independently testable, and composes into a pipeline orchestrated by a central coordinator.

C# brings static typing to this pipeline. When a `TypeCaster` converts a TEXT column to `NUMERIC(15,2)`, the compiler enforces that the downstream code treats the result as a decimal, not a string. When a `ColumnMapper` renames source headers to clean PostgreSQL column names, the mapping is a strongly-typed dictionary, not a runtime-interpreted YAML file. Errors that would surface at runtime in Python surface at compile time in C#.

Npgsql, the .NET PostgreSQL driver, provides binary COPY support, parameterized queries, and connection pooling out of the box. `DuckDB.NET.Data.Full` provides a full ADO.NET provider for DuckDB. The C# project links both databases through a common ADO.NET abstraction, which means swapping staging backends or adding loader targets requires implementing an interface, not rewriting a pipeline.

### The ELT Pattern

The architecture follows an ELT pattern rather than traditional ETL. Data is *extracted* from source files into the staging layer (DuckDB) with minimal transformation — primarily schema inference and TEXT-column projection. The *load* into DuckDB staging is fast and forgiving. The *transforms* then operate on data that's already queryable, which means transform logic can be validated with SQL queries against the staging tables before the final load into PostgreSQL.

This inversion — load first, transform second — is particularly valuable during development. An engineer can load all source data into DuckDB staging, explore it interactively, discover data quality issues, iterate on transform logic, and only push to PostgreSQL when the transforms are validated. The staging layer acts as a checkpoint: if a transform is wrong, the engineer fixes the C# code and re-runs from staging, without re-extracting from source files.

## Architectural Properties

Several properties emerge from this composition that wouldn't be available from any single tool:

**Uniform Query Surface.** Every stage of the pipeline — raw files via Steampipe, staged data in DuckDB, warehouse data in PostgreSQL — is queryable via SQL through a PostgreSQL-compatible interface. Debugging a data quality issue means writing SQL, not reading log files or inspecting intermediate file formats.

**Provenance by Construction.** The architecture requires that every column in the warehouse traces back to a source file and source column header, via both DDL comments and a queryable `column_provenance` table. This isn't an afterthought bolted on to satisfy compliance — it's structural. The C# `ColumnMapper` transform populates provenance as it renames columns. The `data_source_registry` tracks every file load. The result is a warehouse where any analyst can answer "where did this number come from?" with a SQL query.

**Progressive Fidelity.** The pipeline supports exploration at every stage. Steampipe lets an analyst query raw files without loading anything. DuckDB lets an engineer validate transforms without touching the warehouse. PostgreSQL serves the final, clean, indexed data. Each stage is independently useful, and the pipeline can stop at any stage that meets the current need.

**Infrastructure Minimalism.** Steampipe and DuckDB are both single-binary, zero-configuration tools. PostgreSQL is the only component that requires traditional infrastructure management. The pipeline can run entirely on a developer's laptop for datasets up to tens of gigabytes, and the transition to server deployment changes connection strings, not architecture.

## Limitations and Tradeoffs

This architecture is not universal. It is optimized for analytical workloads where data arrives in batches (files, bulk API pulls) and is queried by humans or BI tools. It is not designed for sub-second transactional writes, real-time streaming, or workloads that require horizontal write scaling.

Steampipe's CSV plugin, while remarkably capable, doesn't perform predicate pushdown the way a true columnar store does — it reads the full file and filters in memory. For very large files (tens of gigabytes), the initial Steampipe query can be slow, and direct DuckDB file reads may be more appropriate for extraction.

The C# transform layer requires more upfront engineering than a dbt model or a Python notebook. This is the correct tradeoff when the pipeline will run in production for months or years, but it's overhead for a one-time analysis.

DuckDB's concurrency model is single-writer — fine for a staging layer, but not suitable for workloads where multiple processes write simultaneously. PostgreSQL handles that role.

## Conclusion

The convergence on PostgreSQL's wire protocol is one of the most consequential shifts in data infrastructure of the past decade. It means that learning one query language, one connection model, and one set of tooling idioms gives an engineer access to flat files, live APIs, columnar analytics, relational warehousing, and graph traversal. The architecture described here — Steampipe for universal extraction, DuckDB for staging and exploration, C# for type-safe transforms, PostgreSQL for warehousing — exploits this convergence fully.

The result is a pipeline that is simple to reason about, straightforward to debug, and powerful enough to handle datasets spanning millions of rows across dozens of sources. It requires no proprietary cloud services, no orchestration platforms, and no specialized query languages beyond SQL. For organizations building analytical platforms — particularly those in regulated domains like healthcare, finance, or government contracting where provenance and lineage are non-negotiable — this pattern deserves serious consideration.

---

*Tom Winans is a Principal Technology and Private Equity Advisor and startup co-founder. He can be reached at concentrum.com.*

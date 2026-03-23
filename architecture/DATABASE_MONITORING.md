# Database Monitoring Architecture

## Overview

MonLite supports 7 database engines with a unified monitoring framework. Each engine has a dedicated Python driver implementing a standard 5-method interface, while the backend handles scheduling, threshold evaluation, and alarm management uniformly.

## Supported Engines

| Engine | Driver | Connection Library | Special Handling |
|--------|--------|-------------------|-----------------|
| Microsoft SQL Server | `mssql.py` | pymssql | Backup age/status, autogrow-aware storage |
| Oracle | `oracle.py` | oracledb | System view queries, tablespace-level thresholds |
| SAP HANA | `hana.py` | hdbcli | 3-strategy NAT cascade, SYSTEMDB discovery |
| IBM DB2 | `db2.py` | ibm_db | Community + Enterprise, bufferpool decomposition |
| SAP/Sybase ASE | `sybase.py` | pyodbc + FreeTDS | No native Python driver (SAP discontinued) |
| SAP MaxDB | `maxdb.py` | jpype1 + sapdbc.jar | JVM bridge — no ODBC driver ships with MaxDB |
| PostgreSQL | `postgresql.py` | psycopg2 | Connection counts, bloat detection, replication lag |

## Driver Interface

Every driver implements:
```
connect(config, auth, timeout) → connection
collect_metrics(connection) → dict
collect_backups(connection) → dict
collect_storage_details(connection) → dict
close(connection) → None
```

## Metric Groups

Each database type has 4 metric groups with independent collection intervals:

| Group | Default Interval | Metrics |
|-------|-----------------|---------|
| **connectivity** | 60s | Availability, response time |
| **performance** | 300s | Sessions, deadlocks, cache hit ratio, connections |
| **storage** | 600s | Disk usage, log space, tablespace fill |
| **backups** | 1800s | Full/log backup age, backup status |

## Smart Storage Features

- **Autogrow awareness** (MSSQL) — Calculates actual available space considering autogrow limits
- **Per-tablespace thresholds** — Oracle and HANA support individual thresholds per tablespace
- **Predictive projections** — Linear regression on 30-day daily-max storage data, with R²≥0.5 quality gates and 14-day minimum data requirements

## SAP HANA Connection Cascade

HANA environments often involve Docker, NAT, and multi-tenant architectures. MonLite implements a 3-strategy cascade:

1. **Direct TCP** — Standard host:port connection
2. **Docker port-mapped** — Handles port translation for containerized HANA
3. **SYSTEMDB routing** — Connect to SYSTEMDB, then route to tenant database

This ensures connectivity across bare-metal, Docker, cloud, and multi-tenant HANA deployments.

## MaxDB JVM Bridge

SAP MaxDB has no native Python driver and no ODBC driver in the standard installation. MonLite uses:
- **jpype1** — Python → JVM bridge
- **sapdbc.jar** — SAP's JDBC driver for MaxDB

The collector dynamically starts a JVM, loads the JDBC driver, and executes queries through the bridge. This approach avoids requiring any additional MaxDB client installation.

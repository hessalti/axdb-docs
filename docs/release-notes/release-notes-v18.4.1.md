# AXDB 18.4.1 ({{date.18_4_1}})

--8<-- "release-notes-intro.md"

This release of AXDB is based on AXDB Server for PostgreSQL 18.4 - a binary compatible, drop in replacement of [PostgreSQL Community 18.4](https://www.postgresql.org/docs/18/release-18-4.html).

## Release Highlights

This release includes `pg_tde` 2.2.0 for Transparent Data Encryption and more. See the full component list below for details.

!!! note
    Starting with this release, `shared_preload_libraries` is empty by default. Extensions such as `pg_tde` must be added manually.

## Supplied third-party extensions

Review each extension’s release notes for What’s new, improvements, or bug fixes.

The following is the list of extensions available in AXDB.

| Extension | Version | Description |
| --- | --- | --- |
| [etcd :octicons-link-external-16:](https://etcd.io/) | 3.5.30 | A distributed, reliable key-value store for setting up highly available Patroni clusters |
| [python-etcd :octicons-link-external-16:](https://python-etcd.readthedocs.io/en/latest/) | 0.4.5 | A Python client library for interacting with etcd |
| [HAProxy :octicons-link-external-16:](https://www.haproxy.org/) | 2.8.23 | A high-availability and load-balancing solution |
| [Patroni :octicons-link-external-16:](https://patroni.readthedocs.io/en/latest/) | 4.1.3 | A HA (High Availability) solution for PostgreSQL |
| [PgAudit :octicons-link-external-16:](https://www.pgaudit.org/) | 18 | A detailed session or object audit logging via the standard logging facility provided by PostgreSQL |
| [pgAudit set_user :octicons-link-external-16:](https://github.com/pgaudit/set_user) | 4.2.0 | Provides an additional layer of logging and control when unprivileged users must escalate roles for maintenance |
| [pgBackRest :octicons-link-external-16:](https://pgbackrest.org/) | 2.58.0 | A backup and restore solution for PostgreSQL |
| [pgBadger :octicons-link-external-16:](https://github.com/darold/pgbadger) | 13.2 | A fast PostgreSQL log analyzer |
| [PgBouncer :octicons-link-external-16:](https://www.pgbouncer.org/) | 1.25.2 | A lightweight connection pooler for PostgreSQL |
| [pg_cron :octicons-link-external-16:](https://github.com/citusdata/pg_cron) | 1.6.7 | A simple cron-based job scheduler for PostgreSQL |
| [pg_gather :octicons-link-external-16:](https://github.com/jobinau/pg_gather) | v33 | An SQL script for running diagnostics on the health of a PostgreSQL cluster |
| [pg_oidc_validator](https://github.com/Percona-Lab/pg_oidc_validator) | 1.0 | OAuth validator library for PostgreSQL 18 |
| [pg_repack :octicons-link-external-16:](https://github.com/reorg/pg_repack) | 1.5.3 | Rebuilds PostgreSQL database objects |
| [pg_stat_monitor](https://github.com/percona/pg_stat_monitor) | 2.3.2 | Collects and aggregates statistics for PostgreSQL and provides histogram information |
| [pg_tde :octicons-link-external-16:](https://github.com/percona/pg_tde) | v2.2.0 | A PostgreSQL extension that provides Transparent Data Encryption (TDE) to protect data at rest |
| [pg_vector](https://github.com/pgvector/pgvector) | v0.8.2 | A vector similarity search extension for PostgreSQL |
| [pgpool2 :octicons-link-external-16:](https://git.postgresql.org/gitweb/?p=pgpool2.git;a=summary) | 4.7.1 | A middleware between PostgreSQL server and client for high availability, connection pooling, and load balancing |
| [PostGIS :octicons-link-external-16:](https://github.com/postgis/postgis) | 3.5.6 | A spatial extension for PostgreSQL |
| [PostgreSQL Commons :octicons-link-external-16:](https://salsa.debian.org/postgresql/postgresql-common) | 290 | PostgreSQL database-cluster manager. Supports multiple PostgreSQL versions and clusters simultaneously |
| [wal2json :octicons-link-external-16:](https://github.com/eulerto/wal2json) | 2.6 | A PostgreSQL logical decoding JSON output plugin |

For Red Hat Enterprise Linux 8 and compatible derivatives, AXDB also includes the  supplemental `python3-etcd` 0.4.5 packages, which are used for setting up Patroni clusters.

AXDB is also shipped with the [libpq](https://www.postgresql.org/docs/18/libpq.html) library. It contains "a set of library functions that allow client programs to pass queries to the PostgreSQL backend server and to receive the results of these queries."

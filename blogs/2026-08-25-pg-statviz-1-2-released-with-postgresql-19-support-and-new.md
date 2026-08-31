---
title: "pg_statviz 1.2 released with PostgreSQL 19 support and new features"
url: "https://www.postgresql.org/about/news/pg_statviz-12-released-with-postgresql-19-support-and-new-features-3369/"
date: "2026-08-25"
feed_url: "https://www.postgresql.org/news.rss"
---
Just in time for the PostgreSQL 19 betas, I'm excited to announce release 1.2 of pg_statviz , the minimalist extension and utility pair for time series analysis and visualization of PostgreSQL internal statistics. This release adds support for the upcoming PostgreSQL 19 : pg_statviz now captures the new wal_fpi_bytes counter from pg_stat_wal . The PG18/19 I/O worker, effective WAL level, and autovacuum scoring settings are captured in snapshot_conf .

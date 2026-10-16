---
title: "pg_ivm 1.16 released"
url: "https://www.postgresql.org/about/news/pg_ivm-116-released-3391/"
date: "2026-10-05"
feed_url: "https://www.postgresql.org/news.rss"
---
IVM Development Group is pleased to announce the release of pg_ivm 1.16 . Changes since the v1.15 release include: What's Changed New features Add support for PostgreSQL 19 ( Devrim Gündüz ) Bug fixes Fix a segfault when an IMMV is maintained and dropped in the same transaction (Yugo Nagata) Fix IVM maintenance for tables without default equality operators (Yugo Nagata) When a base table contained a type without a default equality operator, maintenance of the views failed when the table was updated, even if the column was not referenced by the view definition. Fix OID handling when OID exceeds

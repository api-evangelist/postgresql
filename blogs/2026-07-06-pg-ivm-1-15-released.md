---
title: "pg_ivm 1.15 released"
url: "https://www.postgresql.org/about/news/pg_ivm-115-released-3338/"
date: "2026-07-06"
feed_url: "https://www.postgresql.org/news.rss"
---
IVM Development Group is pleased to announce the release of pg_ivm 1.15 . Changes since the v1.14 release include: What's Changed New features Add IMMV metadata restore support for pg_dump and pg_upgrade (Yugo Nagata) After restoring a database from pg_dump or upgrading PostgreSQL using pg_upgrade, IMMVs no longer need to be dropped and recreated manually. The new pg_ivm_dump_metadata utility and restore_immv() function restore the metadata required by pg_ivm, allowing incremental maintenance to continue without recreating the IMMVs.

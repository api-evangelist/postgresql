---
title: "pg_vault_tde v1.7.2 : Critical crash fixes, new on-disk format, and stability improvements"
url: "https://www.postgresql.org/about/news/pg_vault_tde-v172-critical-crash-fixes-new-on-disk-format-and-stability-improvements-3393/"
date: "2026-10-05"
feed_url: "https://www.postgresql.org/news.rss"
---
Version 1.7.2 of pg_vault_tde is now available. This is a binary patch release where extension version stays at 1.7, but users can distinguish the build at runtime using pg_vault_tde_build_version() . This update focuses on critical stability, security, and correctness fixes: New On-Disk Tuple Layout (v5) : Introduces a walkable structure to fix a critical segmentation fault on UPDATE operations for tables with indexed variable-length columns (PSQLE-165).

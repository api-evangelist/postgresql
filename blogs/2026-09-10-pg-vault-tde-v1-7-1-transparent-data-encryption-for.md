---
title: "pg_vault_tde v1.7.1 : Transparent Data Encryption for PostgreSQL 17 and 18"
url: "https://www.postgresql.org/about/news/pg_vault_tde-v171-transparent-data-encryption-for-postgresql-17-and-18-3376/"
date: "2026-09-10"
feed_url: "https://www.postgresql.org/news.rss"
---
pg_vault_tde provides Transparent Data Encryption for PostgreSQL 17 and 18. A table access method, encrypted_heap , encrypts every tuple with AES-256-GCM before it reaches the storage manager and decrypts it after it leaves, so applications require no changes. Keys are held outside the database: HashiCorp Vault or OpenBao through the Transit engine, a PKCS#11 token or HSM, or a local PKCS#12 wallet.

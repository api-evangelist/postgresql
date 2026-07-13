---
title: "PostgreSQL JDBC 42.7.12 Security Release"
url: "https://www.postgresql.org/about/news/postgresql-jdbc-42712-security-release-3340/"
date: "2026-07-06"
feed_url: "https://www.postgresql.org/news.rss"
---
Silent channel-binding authentication downgrade (CVE-2026-54291) channelBinding=require connections can be silently downgraded from SCRAM-SHA-256-PLUS (with channel binding) to plain SCRAM-SHA-256 (without it), losing the man-in-the-middle protection the setting is meant to guarantee. An attacker who can intercept the TLS connection triggers the downgrade with a certificate whose signature algorithm has no tls-server-end-point channel-binding hash. Examples are Ed25519, Ed448, and post-quantum algorithms.

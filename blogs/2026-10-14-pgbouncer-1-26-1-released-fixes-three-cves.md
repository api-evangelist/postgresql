---
title: "PgBouncer 1.26.1 released - Fixes three CVEs"
url: "https://www.postgresql.org/about/news/pgbouncer-1261-released-fixes-three-cves-3403/"
date: "2026-10-14"
feed_url: "https://www.postgresql.org/news.rss"
---
PgBouncer 1.26.1 has been released. This release fixes three CVEs: CVE-2026-107350: Authentication bypass, allowing a client that knows the password of one SCRAM user to log in as any other SCRAM user, including admins and backend superusers. Caused by a second StartupMessage being processed during SCRAM authentication when connecting to a non-existent database.

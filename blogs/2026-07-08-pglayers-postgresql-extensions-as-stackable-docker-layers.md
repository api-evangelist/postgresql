---
title: "pglayers: PostgreSQL extensions as stackable Docker layers"
url: "https://www.postgresql.org/about/news/pglayers-postgresql-extensions-as-stackable-docker-layers-3344/"
date: "2026-07-08"
feed_url: "https://www.postgresql.org/news.rss"
---
🚀 Announcing pglayers Pre‑built PostgreSQL extensions as composable Docker image layers Project: https://github.com/pglayers/pglayers 📌 What It Does pglayers publishes 53 PostgreSQL extensions as minimal Docker images ( FROM scratch ). Each image contains only: Shared libraries Control files SQL scripts Correct filesystem paths You compose them onto the official postgres Docker image using COPY --from : dockerfile FROM postgres:17 COPY --from=ghcr.io/pglayers/pgx-pgvector:17 / / COPY --from=ghcr.io/pglayers/pgx-postgis:17 / / COPY --from=ghcr.io/pglayers/pgx-pg_cron:17 / / No compilation. No a

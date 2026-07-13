---
title: "pg_dbms_errlog v2.4 released"
url: "https://www.postgresql.org/about/news/pg_dbms_errlog-v24-released-3331/"
date: "2026-07-05"
feed_url: "https://www.postgresql.org/news.rss"
---
Bangkok, Thailand - June 23, 2026 PostgreSQL DBMS_ERRLOG compatibility extension The pg_dbms_errlog extension provides the infrastructure that enables you to create an error logging table so that DML operations can continue after encountering errors rather than abort and roll back. It requires the use of the pg_statement_rollback extension or to fully manage the SAVEPOINT in the DML script. Logging in the corresponding error table is done using dynamic shared memory for error queuing and a background worker to write the errors queued into the corresponding error log tables.

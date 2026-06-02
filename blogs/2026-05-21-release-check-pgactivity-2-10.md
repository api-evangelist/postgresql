---
title: 'Release: check_pgactivity 2.10'
url: https://www.postgresql.org/about/news/release-check_pgactivity-210-3302/
date: '2026-05-21'
author: ''
feed_url: https://postgresql.com/news/rss
---
check_pgactivity version 2.10 released check_pgactivity is a PostgreSQL plugin for Nagios. This plugin is written with a focus on a rich perfdata set. Every new features of PostgreSQL can be easily monitored with check_pgactivity. Changelog : add: new unused_indexes service add: new relation_size service change: pgss_dealloc service now handles --dbservice change: invalid_indexes now also gets TOAST invalid indexes change: add the database name to a query error message change: set application_name to the program and service names change: add application_name of the offending query in the msg change: various enhancements to check_last_maintenance change: exclude partitioned table for last_analyze service change: add exclude option for last_vauum and last_analyze services fix: keep only three integers after the comma fix: pgss_dealloc service is only PG14+ (doc) fix: slru_hit_ratio is only PG13+ (doc) Here are some useful links: github repo: https://github.com/OPMDG/check_pgactivity reporting issues: https://github.com/OPMDG/check_pgactivity/issues latest release: https://github.com/OPMDG/check_pgactivity/releases/latest contributors: https://github.com/OPMDG/check_pgactivity/blob/master/contributors Thanks to all the contributors!

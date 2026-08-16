---
title: "plx : Write PostgreSQL functions in the language you already know."
url: "https://www.postgresql.org/about/news/plx-write-postgresql-functions-in-the-language-you-already-know-3358/"
date: "2026-08-12"
feed_url: "https://www.postgresql.org/news.rss"
---
What plx is plx is a PostgreSQL extension that lets you write stored functions and triggers in the dialect you already know (the current set is listed below). When you run CREATE FUNCTION , plx transpiles the body to plpgsql and stores that plpgsql in pg_proc.prosrc . At run time the function is executed by PostgreSQL's own plpgsql interpreter.

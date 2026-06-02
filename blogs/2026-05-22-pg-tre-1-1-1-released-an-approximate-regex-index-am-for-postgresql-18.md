---
title: pg_tre 1.1.1 released -- an approximate-REGEX index AM for PostgreSQL 18+
url: https://www.postgresql.org/about/news/pg_tre-111-released-an-approximate-regex-index-am-for-postgresql-18-3305/
date: '2026-05-22'
author: ''
feed_url: https://postgresql.com/news/rss
---
I am pleased to announce the first public release of [pg_tre] (https://codeberg.org/gregburd/pg_tre), a native PostgreSQL 18+ index access method for approximate-regex matching. pg_tre indexes text columns through a three-tier filter funnel (BRIN-style range bloom -> sparsemap trigram postings -> per-tuple bloom) backed by Ville Laurikari's TRE library for the heap recheck. The result is genuine Levenshtein-distance regex matching ("find text within k edits of this pattern") driven through a real IndexAmRoutine, with WAL coverage, VACUUM awareness, and REINDEX CONCURRENTLY support. Highlights Custom IndexAmRoutine registered as USING tre, with custom
    rmgr (id 140) and full crash-recovery / streaming-replication
    coverage validated by TAP tests. Edit-distance regex with per-subexpression budgets, e.g.
    body %~~ tre_pattern('(error){~1}.*(42[0-9]){~0}', 1)
    runs as a single indexed bitmap heap scan. UTF-8 codepoint trigrams: CJK, accented characters, and
    emoji are indexed correctly; ASCII pays zero overhead. DoS protection via configurable caps on NFA states, compile
    time, and per-match runtime. Three-tier funnel cuts heap I/O dramatically: queries that
    return a handful of rows out of millions typically finish in
    sub-millisecond time. Backward-compatible tre_amatch* UDFs from 0.1.0 are
    preserved. How pg_tre fits alongside what you already have PostgreSQL already ships strong text-search primitives. pg_tre is meant to complement them, not replace them. pg_trgm (GIN/GiST):  exact regex, LIKE, and trigram-set
    similarity. Battle-tested. pg_trgm % is Jaccard similarity
    over trigram sets, which is not the same as edit distance:
    two strings with overlapping trigrams can score "similar"
    even when their Levenshtein distance is huge. Use pg_trgm
    when you need exact-substring or LIKE acceleration. tsvector / tsquery (built-in FTS):  word-level linguistic
    search with stemming, stopwords, ranking, and language
    configuration. Use FTS for natural-language prose. pg_tre
    is language-agnostic, which is a feature for identifiers,
    SKUs, error codes, and log lines, and a non-feature for
    "running" matching "run". pgvector / pgvectorscale:  semantic similarity over float
    embeddings. Orthogonal to pg_tre -- meaning vs. lexical
    structure. The two compose naturally as a hybrid filter
    (lexical pre-filter with pg_tre, semantic rank with
    pgvector). pg_tre:  approximate regex with explicit edit budgets and
    full regex semantics (character classes, alternation,
    anchors, {m,n} repetition) composable with the {~k} edit
    operator. No other in-tree or out-of-tree PostgreSQL
    extension answers "is this text within N edits of this
    regex?" through an index. A few things only pg_tre does well Typo-tolerant log and trace search:
    body %~~ tre_pattern('(timeout){~1}.*(connection){~1}', 1)
    finds "timeoutt" and "conection" in the same query. Catalog and SKU lookup with edit tolerance:
    sku %~~ tre_pattern('AB-9?[0-9]{4}', 1)
    catches dropped digits or transposed characters without
    expensive post-filtering. Per-phrase edit budgets in a single index query:
    body %~~ tre_pattern('(postgres){~2}.*(system){~0}', 0)
    expresses "postgres-ish" with strict "system" -- one round
    trip, no application-level reranking. Hybrid retrieval for agent / RAG pipelines: pg_tre is the
    fuzzy-lexical leg that pg_trgm, FTS, and pgvector cannot
    cover on their own. An LLM-generated identifier with a
    typo, an OCR error in a scanned document, a near-duplicate
    error code -- pg_tre catches them all without sacrificing
    regex expressivity. Installation -- Requires shared_preload_libraries = 'pg_tre'
CREATE EXTENSION pg_tre;

CREATE INDEX docs_body_tre ON docs USING tre (body);

SELECT id FROM docs
 WHERE body %~~ tre_pattern('database', 1); A note on stability and feedback pg_tre is new software. The on-disk format is stable from 1.0.0 forward (1.1.x is byte-compatible with 1.0.0), the SQL surface is fixed, the WAL records are versioned, and the test suite covers the storage, query, recovery, and replication paths -- but the project is young and almost certainly has bugs that the existing tests do not yet provoke. It is not a beta and not a research toy; it is real software released early, and it should be used with appropriate caution. Bug reports, feature requests, and pull requests are very welcome. If you can attach a reproduction case (a minimal schema and a query that misbehaves), that is the most useful form, but anything is better than nothing. Links Repository: https://codeberg.org/gregburd/pg_tre Issues: https://codeberg.org/gregburd/pg_tre/issues TRE library: https://github.com/laurikari/tre

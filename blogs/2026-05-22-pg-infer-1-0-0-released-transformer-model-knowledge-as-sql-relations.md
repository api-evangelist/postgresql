---
title: pg_infer 1.0.0 released -- transformer model knowledge as SQL relations
url: https://www.postgresql.org/about/news/pg_infer-100-released-transformer-model-knowledge-as-sql-relations-3307/
date: '2026-05-22'
author: ''
feed_url: https://postgresql.com/news/rss
---
I am pleased to announce the first public release of pg_infer , a PostgreSQL 18+
extension that exposes the internals of small transformer language models -- gate activations, feature labels, learned associations, embeddings -- as SQL-queryable relations and a custom index access method. pg_infer is not "natural language to SQL." It is not "SQL to natural language." There is no chat interface, no agent loop, no prompt template generating queries. pg_infer brings model inference into the query plan as an operator the planner can cost, schedule, parallelize, and combine with ordinary predicates and joins. The model becomes a first-class data source -- a set of relations the planner can scan, filter, and join -- not an external service the database talks to. Quick example -- Register a vindex (extracted model knowledge):
`SELECT infer_create_model('qwen05b', '/data/qwen-0.5b.vindex');`

-- What does the model know about France? SELECT * FROM describe('France');
    --  relation  |  target   | confidence | layer
    -- -----------+-----------+------------+-------
    --  capital   | Paris     |       42.7 |    18
    --  language  | French    |       38.1 |    17
    --  continent | Europe    |       35.4 |    16
    -- ... -- `ORDER BY` model-knowledge similarity: ```
    CREATE INDEX ON documents USING infer (title)
       WITH (model = 'qwen05b'); SELECT * FROM documents
 ORDER BY title <~> 'artificial intelligence'
 LIMIT 5; ``` The <~> operator is index-backed, supports
EXPLAIN (ANALYZE, BUFFERS), and composes with WHERE, JOIN,
aggregation, and partitioning the way any other operator does. What pg_infer does that other extensions do not pgvector / pgvectorscale stores user-supplied embedding
    vectors and answers nearest-neighbour distance queries.
    pg_infer goes a layer deeper: it stores the model itself
    (gate vectors, feature activations, label metadata) in
    WAL-logged 8KB pages, and answers questions like "does
    the model treat A and B as related?" -- not "are these
    two embeddings close?" pg_search / RAG-style integrations turn user queries into
    embedding lookups against external vector stores. pg_infer
    exposes the model's internal structure to SQL: walk(prompt)
    returns per-layer feature activations; describe(entity)
    returns the relations the model has learned about an
    entity; implies(a, b) tests directional support. pg_infer's index AM ships in two modes: "model" indexes WAL-log the entire vindex inside
    PostgreSQL pages, so backup, replication, and
    point-in-time recovery cover the model the same way
    they cover your tables. "column" indexes attach a model to a text column and
    make ORDER BY <~> on that column index-driven. The mmap-backed local backend lets multiple PG backends
    share decoded model pages through the OS page cache; the
    remote backend (larql-server / larql-router over HTTP/2 or
    a Unix socket) shares one copy of the model across a host
    and supports layer-sharded routing. In-flight remote calls
    respond to pg_cancel_backend(...) within roughly 100ms. CPU inference, BitNet, and idle-cluster compute Database servers almost never have GPUs. They have a lot of fast cores, a lot of RAM, and -- on most production deployments -- standby replicas, read-only physical replicas, logical subscribers, and DR hosts that spend most of the day at single-digit CPU utilization while the primary takes the write traffic. pg_infer and the underlying larql crates target this hardware profile directly: The default execution paths run efficiently on CPU, with
    BLAS-backed linear algebra (OpenBLAS) and f16 gate vectors
    that decode to f32 lazily. pg_infer / larql support models in the Microsoft BitNet
    b1.58 family ("two-bit / 1.58-bit" ternary-weight
    transformers, https://arxiv.org/abs/2402.17764), which
    were specifically designed to run on commodity CPUs at
    competitive quality and dramatically lower memory and power
    cost than f16 baselines. Combined with f16 gate
    activations, this brings useful inference inside a
    PostgreSQL backend without any specialized accelerator. The cluster model is the point. A typical PostgreSQL HA /
    DR / read-scale deployment has one busy primary and one or
    more largely idle physical replicas, plus, increasingly, a
    fleet of logical subscribers. Those replicas already pay
    for themselves in availability, but their CPUs are idle the
    vast majority of the time. With pg_infer's remote backend,
    larql-server runs on the replica hosts and serves model
    operators back to the primary's query plans -- the model is
    materialized once per host, the activation cache is shared,
    and the work happens on capacity you have already paid for.
    No GPU, no separate inference cluster, no extra network
    egress. A few queries that are uniquely pg_infer Model-aware document ranking that does not depend on
    keyword overlap or pre-computed embeddings: SELECT id, title
        FROM papers
       ORDER BY title <~> 'neural architecture search'
       LIMIT 10; This finds "AutoML for Deep Networks" because the model
learned that relationship -- pg_trgm cannot, FTS cannot,
and pgvector can only do so if you computed and stored
embeddings ahead of time with a model whose semantics
happen to agree with your query. Joining model knowledge with relational data: SELECT c.id, c.name, p.title,
             p.title <~> c.research_interest AS dist
        FROM candidates c
        JOIN papers     p
          ON p.title <~> c.research_interest < 0.2
       WHERE c.country = 'DE'; Standard SQL semantics, standard PostgreSQL planner, plus
a model-driven join condition. Probing what a model "knows" without running it: SELECT relation, target, confidence
        FROM describe('PostgreSQL')
       WHERE confidence > 30; Auditing model behaviour over time. Because the vindex is
    stored in WAL-logged pages, point-in-time recovery on a
    pg_infer-using cluster gives you the model state at any
    historical moment alongside the data state. "What was the
    model saying about this entity at 03:14 UTC last Tuesday?"
    is a literal PITR + describe(...) question. Acknowledgements pg_infer would not exist without the LARQL project by Chris Hayuk (https://github.com/chrishayuk). LARQL pioneered the idea of making transformer model internals queryable -- extracting gate vectors, feature activations, and learned associations into a format ("vindex") that can be explored interactively and expressed as a query language. The vindex format, the gate KNN algorithm, and the feature-labeling pipeline all originate from LARQL; pg_infer adapts them into a PostgreSQL access method, a WAL-logged storage layer, and a planner-visible operator. If the larql ideas resonate, please look at the original work and at Chris's video walkthroughs explaining the vindex format, the gate-KNN algorithm, and the LARQL query language: Chris Hayuk on YouTube:    https://www.youtube.com/@chrishayuk Original LARQL repo:       https://github.com/chrishayuk/larql larql-rs (Rust port):      https://github.com/chrishayuk/chuk-larql-rs Thank you, Chris, for the foundational work and for being open with the design. A note on stability and feedback pg_infer is new software. The SQL surface, the index AM, the remote backend protocol, and the test suite are stable enough to release at 1.0.0, and the vindex on-disk format is stable forward; but the project is young, and the combination of PostgreSQL + pgrx + transformer internals is unusual enough that there are certainly bugs that the existing tests do not yet provoke. It is not a beta and not a research toy; it is real software released early, and it should be used with appropriate caution. Bug reports, feature requests, and pull requests are very welcome -- especially reproductions, vindex compatibility issues, planner-cost regressions, and integration suggestions for other PostgreSQL extensions. Links Repository: https://codeberg.org/gregburd/pg_infer Issues: https://codeberg.org/gregburd/pg_infer/issues LARQL: https://github.com/chrishayuk/larql larql-rs: https://github.com/chrishayuk/chuk-larql-rs BitNet b1.58: https://arxiv.org/abs/2402.17764

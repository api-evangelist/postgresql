---
title: pg_mentat 1.3.0 released -- Datomic-compatible Datalog inside PostgreSQL
url: https://www.postgresql.org/about/news/pg_mentat-130-released-datomic-compatible-datalog-inside-postgresql-3306/
date: '2026-05-22'
author: ''
feed_url: https://postgresql.com/news/rss
---
I am pleased to announce the first public release of [pg_mentat] (https://github.com/gburd/pg_mentat), a PostgreSQL extension that
implements Datomic's data model -- immutable facts (datoms), schema-first attributes, a full Datalog query compiler, the pull API, time travel, and ACID transactions -- entirely inside PostgreSQL. pg_mentat is built with pgrx 0.17 in Rust and supports PostgreSQL 13 through 18. The current release is 1.3.0, the "Postgres Extension Family" release, which adds Datalog where-fns that bridge into rum, pg_trgm, fuzzystrmatch, pgvector, pg_infer, PostGIS, and several other extensions as soft dependencies (nothing pg_mentat ships requires any of them, but where-fns light up automatically when they are present). An optional companion daemon, mentatd, speaks the Datomic client wire protocol (EDN, Transit+JSON, Transit+MsgPack) over HTTP for
applications that already expect it. Highlights Full Datalog query language: relation / collection / tuple /
    scalar find specifications; pattern matching, predicates,
    function expressions; not / not-join / or / or-join; named
    rules and recursive rules with cycle detection. Pull API with wildcards, reverse refs, nested / recursive
    pulls, defaults, rename, and limit clauses -- arbitrarily
    shaped JSON results in a single round trip. Time travel: as-of, since, history, tx-range. Every datom
    is timestamped by transaction; the database is an append-
    only log of immutable facts, and any past state is queryable. GDPR-compliant excision: permanent deletion of datoms by
    entity, attribute, or value, with the audit trail removed. Reactive subscriptions via PostgreSQL LISTEN / NOTIFY. Storage: nine narrow per-type datom tables (ref, long,
    string, bool, double, inst, kw, uuid, bytes) with the four
    canonical Datomic indexes (EAVT, AEVT, AVET, VAET) covered
    by ordinary PostgreSQL btrees. Soft integrations new in 1.3.0: rum-fulltext, similar-to
    (pg_trgm), levenshtein / soundex / metaphone / daitch-
    mokotoff (fuzzystrmatch), vector-near (pgvector), infer-near
    / infer-similar / infer-implies / infer-walk / infer-
    describe / infer-predict (pg_infer), geom-near / geom-
    within / geom-contains / geom-intersects (PostGIS), and
    detection helpers (mentat.has_<ext>()) for capability
    discovery. Why have EDN and Datalog next to relational tables? Datalog and SQL are not redundant; they answer different questions well. Putting them in the same database, sharing the same MVCC, the same WAL, the same backup, and the same connection pool unlocks workloads that are awkward in either alone. Knowledge graphs that join your business data.
    Encode an ontology -- categories, hierarchies, taxonomies,
    typed relationships -- as datoms with refs, then query with
    Datalog rules. Recursive rules express transitive closure
    in a few lines (no recursive CTE, no cycle-detection
    boilerplate). Then expose that graph as a PostgreSQL VIEW
    via mentat.mentat_query_view(...) and JOIN it directly with
    your relational tables in ordinary SQL. One database, one
    transaction, two query languages. Schema as data -- queryable like everything else.
    Schema attributes (:db/ident, :db/valueType, :db/cardinality,
    :db/unique, :db/index, :db/fulltext, :db/isComponent,
    :db/noHistory) live in the datom store. You query the
    schema with the same Datalog you use for application data,
    instead of grovelling in pg_catalog or information_schema. Audit and time travel without triggers.
    Because datoms are immutable, "what did this entity look
    like at tx 1000005?" and "what changed between tx 1000003
    and 1000007?" are first-class queries. No audit tables,
    no triggers, no temporal-extension contortions. Compliance
    workloads that take weeks to retrofit on a relational schema
    are a one-line query in pg_mentat. Speculative transactions.
    mentat.mentat_with('[ ... ]') runs a transaction in memory,
    returns the full report (tempid resolution, new datoms,
    constraint outcomes), and writes nothing. Validate
    imports, preview merges, or detect conflicts before
    committing. EDN as the data interchange format.
    Schema, transactions, and queries are all EDN. Tagged values
    (#inst, #uuid, keywords, sets, vectors) survive a round trip
    without ad hoc JSON conventions. Existing Clojure / Datomic
    code, including pull patterns and Datalog expressions, runs
    against pg_mentat with minor adjustments. A short example: a graph traversal that would be 20+ lines of recursive CTE in plain SQL: SELECT mentat.q('
      [:find ?name
       :in $ ?start
       :where [?start :person/name ?start-name]
              (reachable ?start ?friend)
              [?friend :person/name ?name]
       :rules [[(reachable ?a ?b) [?a :person/friends ?b]]
               [(reachable ?a ?b) [?a :person/friends ?c]
                                  (reachable ?c ?b)]]]
    ', '["Alice"]'); Datalog says what to find. The compiler decides how. The plan is ordinary SQL, executed by ordinary PostgreSQL, against ordinary PostgreSQL pages. How pg_mentat fits among the alternatives Datomic / XTDB / Crux:  hosted Datalog stores, often with
    excellent query languages and weaker operational stories
    for shops that have already standardized on PostgreSQL.
    pg_mentat brings their data model into the database you
    already run, back up, monitor, and replicate. pg_graphql / Apache AGE:  graph layers over PostgreSQL with
    Cypher / GraphQL respectively. Different query languages,
    different data models, no time-travel semantics, no pull
    API, no built-in immutable-facts model. hstore / jsonb:  schema-flexible storage without a query
    language designed for joins across attributes. pg_mentat
    gives you Datalog, refs, recursive rules, and the pull API. A note for the Datomic / Mentat / Datalog community If you have been writing Clojure against Datomic, Datascript, or the original Mentat (RIP), pg_mentat will feel familiar.  EDN schema, transactions, and queries; the pull API; as-of / since / history; uniqueness via :db.unique/identity (which makes upserts trivial -- no INSERT ON CONFLICT, no MERGE) -- the surface is intentionally close. The differences are deliberate and pragmatic: storage is PostgreSQL, transactions are PG transactions, the fulltext attribute uses tsvector, and operational tooling is whatever you already use for PostgreSQL (pg_basebackup, pg_dump, pgBackRest, logical replication, WAL-G, and so on). I would love feedback from anyone who has production experience with Datomic-style stores and wants a Datalog-on-Postgres path -- specifically: which Datalog forms do you rely on that pg_mentat does not yet implement, and which Datomic-isms do you NOT want carried over? File issues or send mail. Quick start ```
    CREATE EXTENSION pg_mentat; SELECT mentat.t('[
  {:db/ident :person/name :db/valueType :db.type/string
   :db/cardinality :db.cardinality/one
   :db/unique :db.unique/identity}
  {:db/ident :person/age  :db/valueType :db.type/long
   :db/cardinality :db.cardinality/one}
]');

SELECT mentat.t('[{:person/name "Alice" :person/age 30}]');

SELECT mentat.q('
  [:find ?name ?age
   :where [?e :person/name ?name]
          [?e :person/age ?age]
          [(> ?age 18)]]'); ``` A note on stability and feedback pg_mentat is new software. The on-disk schema, the function surface, the EDN encoding, and the Datalog dialect are documented and covered by tests, and 1.3.0 ships ten extension integrations gated behind capability detection -- but the project is young, and bugs you can reach are very likely bugs nobody has reached yet. It is not a beta and not a research toy; it is real software released early, and it should be used with appropriate caution. Bug reports, feature requests, and pull requests are very welcome -- particularly from people who already speak Datalog and can spot semantic gaps quickly. Links Repository: https://github.com/gburd/pg_mentat Issues: https://github.com/gburd/pg_mentat/issues Integrations: https://github.com/gburd/pg_mentat/blob/main/docs/INTEGRATIONS.md

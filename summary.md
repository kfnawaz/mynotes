If the choice of language and stack is completely open, choose SQLGlot (Python) for extracting data lineage across diverse databases and data warehouses.
While Java has strong enterprise frameworks like Apache Calcite, SQLGlot is widely considered the industry standard for modern, multi-dialect data lineage extraction—even inside organizations whose core backend runs on the JVM.

Why SQLGlot Wins for Multi-Database Lineage


Evaluation Criteria	SQLGlot (Python)	Java Stack (Calcite / JSqlParser)
Multi-Dialect Support	30+ dialects (Snowflake, BigQuery, Databricks, Redshift, Oracle, Postgres, T-SQL, Trino, etc.) maintained actively.	Fragmented. Calcite handles ~15–20 well; JSqlParser breaks frequently on proprietary cloud warehouse syntax.
Built-in Lineage API	First-class feature. sqlglot.lineage traces column-level lineage through CTEs, subqueries, and unions out of the box.	DIY. Calcite requires complex RelNode traversal; JSqlParser requires writing custom Visitor logic.
Schema Handling	Flexible. Works without schemas, or accepts a simple Python dictionary ({"db": {"table": {"col": "type"}}}) to resolve SELECT *.	Rigid (Calcite) or Missing (JSqlParser). Calcite forces heavy catalog wiring before it can parse reliably.
Maintenance Effort	Low. Cloud data warehouses change syntax monthly; SQLGlot updates weekly to match them.	High. You will spend significant time patching grammar edge cases yourself.


Key Trade-Offs to Consider

• Dialect Coverage vs. Strict Type Checking. Because your goal is extracting lineage from all sorts of data storage and databases, dialect coverage is your biggest risk. SQLGlot handles legacy systems (Oracle, Teradata, SQL Server) alongside modern lakehouses (Spark, Trino, DuckDB, BigQuery) in a single tool. Apache Calcite offers stricter relational validation, but it will fail to parse many real-world vendor-specific functions.
• Non-SQL Data Storage. Neither SQLGlot nor Java SQL parsers will handle NoSQL, object storage (S3/GCS), or streaming pipelines (Kafka) by themselves. However, Python gives you direct access to the OpenLineage ecosystem, SQLLineage (which wraps SQLGlot), and metadata crawlers (like DataHub or OpenMetadata) that combine SQL parsing with storage-level metadata.
• Architecture Integration. If your broader platform is built in Java, running SQLGlot does not force you to rewrite your stack. Many teams deploy SQLGlot as a lightweight Python microservice (e.g., via FastAPI) that accepts a SQL string, dialect, and optional schema JSON, and returns a clean lineage graph to the Java application.

Recommended Architecture

1. Use SQLGlot as your core SQL lineage engine. Pass in the dialect name and a lightweight schema dictionary (fetched from your database catalogs) so it can accurately expand SELECT * and trace column-level dependencies.
2. Pair it with OpenLineage (optional) for non-SQL storage. If you also need to track files moving between S3 buckets, Kafka topics, or Spark jobs outside of SQL queries, adopt the OpenLineage standard so your SQLGlot output merges cleanly with storage-level events.
3. Wrap it in a small REST API if your main app is Java. Keep your core orchestration in Java if that is your team's strength, and call the Python lineage service over HTTP or gRPC.
To help you map out the exact architecture, could you share:
• Which databases or storage systems (e.g., Snowflake, Postgres, S3, Kafka, Oracle) are highest priority
• Whether your team's primary backend is Java or Python
• If you are building an internal tool or a customer-facing product?

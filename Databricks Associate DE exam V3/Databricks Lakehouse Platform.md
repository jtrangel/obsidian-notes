&copy [Rangel](https://github.com/jtrangel)

## Lakehouse Architecture

1. Describe the relationship between the data lakehouse and the data warehouse. [[1](https://docs.databricks.com/en/lakehouse/)] 
	- Lakehouse = ACID and Governance of warehouses + flexibility (schema on read) and cost efficiency of lakes
	- Same data for ETL/ELT/processing and for training ML models and BI use cases
	<br />
2. Identify the improvement in data quality in the data lakehouse over the data lake. [[1](https://www.databricks.com/discover/pages/data-quality-management#what-is-data-quality)]
	- ACID + Data governance/auditing 
	- Usual features like `CONSTRAINT`
	- New features like `VALIDATE` and Expectations with DLT
	- Delta time travel, for being safe with data changes
	<br />
3. Compare and contrast silver and gold tables, which workloads will use a bronze table as a source, which workloads will use a gold table as a source. [[1](https://docs.databricks.com/en/lakehouse/medallion.html#silver)] 
	- Bronze = raw data. Also known as **Information**
	- Silver = enrich tables via validation and dedup. Can also join columns/tables together for better info.
	- Gold = business level aggregates that were refined from silver. Also known as **Knowledge**
	<br />
4. Identify elements of the Databricks Platform Architecture, such as what is located in the data plane versus the control plane and what resides in the customer’s cloud account. [[1](https://docs.databricks.com/en/getting-started/overview.html)] 
	- Control plane - backend of Databricks consisting of Notebooks, workspace config, clusters, web GUI, VM management
	- Data plane - where data is processed (i.e. EC2, Azure/GCP VMs) and stored (i.e. S3, Azure Blob), VMs
	<br />
5. Differentiate between all-purpose clusters and jobs clusters. [[1](https://docs.databricks.com/en/clusters/index.html)]
	- All purpose clusters - Continuously running, best for dev sessions
	- Job clusters - spins up and runs specifically for a workflow job. Turns off afterwards. Di pwede irestart, and is best for jobs that need high compute (para saglitan lang yung high cost)
	<br />
6. Identify how cluster software is versioned using the Databricks Runtime. [[1](https://docs.databricks.com/en/runtime/index.html)]
	- Runtime is specified for each cluster, and each type will be optimized for different purposes (Standard, ML, Uncategorized)
	- ML packages (Tensorflow, Keras, Pytorch, XGBoost)
	- Photon (optimized for SQL workloads via vectorized query engine)
	<br />
7. Identify how clusters can be filtered to view those that are accessible by the user. [[1](https://docs.databricks.com/en/security/auth-authz/access-control/cluster-acl.html)]
	- Can set permissions on clusters
	<br />
8. Describe how clusters are terminated and the impact of terminating a cluster. [[1](https://docs.databricks.com/en/clusters/clusters-manage.html#terminate-a-cluster)]
	- Terminating a cluster when a NB is running can impact other users using that cluster
	<br />
9. Identify a scenario in which restarting the cluster will be useful. [[1](https://docs.databricks.com/en/clusters/clusters-manage.html#restart-a-cluster)]
	- To update a long running cluster with the latest images
	<br />
10. Describe how to use multiple languages within the same notebook. [[1](https://docs.databricks.com/en/notebooks/notebooks-code.html)]
	- `%sh`, `%md`, `%fs`, `%python`, `%scala`, `%sql` keywords on top of notebook cell
	<br />
11. Identify how to run one notebook from within another notebook. [[1](https://docs.databricks.com/en/notebooks/notebooks-code.html#run-selected-text)] 
	- `%run`, for referencing another NB, can only be used once per cell
	<br />
12. Identify how notebooks can be shared with others.
	- Share via permissions settings on top right of NB
	
	![](https://notejoy.s3.amazonaws.com/note_images/3085854.1.Image%202023-09-08%20at%2012.31.01%20PM.png)

13. Describe how Databricks Repos enables CI/CD workflows in Databricks. [[1](https://docs.databricks.com/en/dev-tools/index-ci-cd.html#steps-for-cicd-on-databricks), [2](https://docs.databricks.com/en/repos/ci-cd-techniques-with-repos.html)]
	- Repos acts similar to Github Desktop, where importing a remote repo can be edited within Databricks
	- Several CI/CD functionality is available by integrating with a tool like Github Actions (Call Repos API to automate)
	<br />
14. Identify Git operations available via Databricks Repos. [[1](https://docs.databricks.com/en/repos/git-operations-with-repos.html)]
	- Commit, Push, Pull, Merge, Rebase, Reset, and create branches
	- **NOTE**: As of Oct 2023, Rebase and Reset are currently experimental/preview features in the exam context. They are considered as not part of available git operations when asked in the exam.
	<br />
15. Identify limitations in Databricks Notebooks version control functionality relative to Repos. [[1](https://docs.databricks.com/en/repos/git-version-control-legacy.html), [2](https://www.examtopics.com/discussions/databricks/view/104744-exam-certified-data-engineer-associate-topic-1-question-10/#:~:text=An%20advantage%20of%20using%20Databricks%20Repos%20over%20the%20built%2Din,Databricks%20Repos%20is%20built%20upon.)]

	- NBs have version control via Revisions and you can link it with a git repository as well.
	- However NBs cannot be linked to multiple branches, which makes it impractical for developing in parallel

## Data Management with Delta Lake

This section on Delta Lake overlaps with [[ELT With Spark SQL and Python]].

1. Identify where Delta Lake provides ACID transactions. [[1](https://docs.databricks.com/en/lakehouse/acid.html)]
	- Transactions are at the table level, one table at a time
	- [Optimistic concurrency control](https://en.wikipedia.org/wiki/Optimistic_concurrency_control) for concurrent transactions
		- BEGIN -> Modify -> Validate -> Commit/Rollback
	- Databricks has no BEGIN/END syntax like TSQL. Changes are made in a serial manner (1 at a time ata meaning neto)
	<br />
2. Identify the benefits of ACID transactions. [[1](https://www.databricks.com/glossary/acid-transactions#:~:text=Why%20are%20ACID%20transactions%20a,operation%20that%20only%20partially%20completes.)]
	- 'Highest possible data reliability and integrity'
	<br />
3. Identify whether a transaction is ACID-compliant.
	- Atomic - each txn statement completes or fails ONLY
		- BEGIN/END statements and/or Stored procedures 
	- Consistency - data must be predictable before and after txn 
		- i.e. row counts consistent when moving rows from one table to another
		- i.e. when moving money from one acc to another, total money must be same
	- Isolation - no other process can change the data/table during a transaction
	- Durability - changes from txn persist, even if servers die (hand in hand with Atomic)
	<br />
4. Compare and contrast data and metadata.
	- metadata - data about data. used for management, support, and context

	```SQL
	# Describe statements for showing metadata
	DESCRIBE SCHEMA EXTENDED ${schema_name};
	DESCRIBE DETAIL <table-name>;
	DESCRIBE TABLE EXTENDED <table-name>;
	```

5. Compare and contrast managed and external tables. [[1](https://docs.databricks.com/en/data-governance/unity-catalog/create-tables.html)]
	- managed tables - made within databricks via DDL
	- external tables - any tables with external data, regardless of where it is stored (dbfs, abfss, adls, s3).
	- when dropping managed, data and metadata is lost. when dropping external, only metadata is lost.
	<br />
6. Identify a scenario to use an external table. [[1](https://learn.microsoft.com/en-us/azure/databricks/data-governance/unity-catalog/create-tables)]
	- when you need direct access to data outside of Databricks clusters/SQL warehouses (avoid data egress from external source)
	```SQL
	# Sample syntax
	CREATE TABLE <catalog>.<schema>.<table-name>
	(
	  <column-specification>
	)
	LOCATION 's3://<bucket-path>/<table-directory>';
	```
7. Create a managed table.
	```SQL
	CREATE TABLE <catalog-name>.<schema-name>.<table-name>
	(
	  <column-specification>
	);
	```
8. Identify the location of a table.
	```SQL
	# Either command works
	DESCRIBE EXTENDED <table-name>
	DESCRIBE DETAIL <table-name>
	```
9. Inspect the directory structure of Delta Lake files.
	- path contains `/_delta_log/` and `*.snappy.parquet` files which form the delta table
	- delta log contains transactions in the form of `*.crc` and `*.json` files
	```Python
	display(dbutils.fs.ls(f"{path}/table"))
	```
	
![[Pasted image 20230924224438.png]]
10. Identify who has written previous versions of a table.
11. Review a history of table transactions. 
	```SQL
	DESCRIBE HISTORY <table-name>
	```

	![[Pasted image 20230924224721.png]]

12.  Roll back a table to a previous version.
13. Identify that a table can be rolled back to a previous version.
14. Query a specific version of a table.
	```SQL
	# Query what previous version looks like (time travel)
	SELECT * FROM students VERSION AS OF 3;
	
	# Rollback 
	RESTORE TABLE students TO VERSION AS OF 8
	```
15. Identify why Zordering is beneficial to Delta Lake tables.
	- z-ordering = indexing for delta tables (afaik, only works on delta tables). For parquet, you can convert them first to `DELTA` format.
	```SQL
	OPTIMIZE students
	ZORDER BY id
	```
16. Identify how vacuum commits deletes.
	- VACUUM deletes old versions of a table (the snappy parquet files)
	- does not delete the delta log, so we can still see the history of the table via `DESCRIBE HISTORY`
	```SQL
	# By default you cannot delete table versions that are less than 7 days old, we change this for demonstration
	SET spark.databricks.delta.retentionDurationCheck.enabled = false;
	SET spark.databricks.delta.vacuum.logging.enabled = true;
	
	# DRY RUN first to see which files will be deleted (`*.snappy.parquet files`)
	VACUUM students RETAIN 0 HOURS DRY RUN
	```
	```SQL
	VACUUM students RETAIN 0 HOURS
	```
17. Identify the kind of files Optimize compacts. [[1](https://docs.gcp.databricks.com/sql/language-manual/delta-optimize.html)]
	- small files are compacted and balanced out (combined towards an optimal size, determined by table size)
	- idempotent process
	<br />
18. Identify CTAS (`CREATE TABLE AS SELECT`) as a solution.
	- autoinfer schema from input (only works for sources with well defined schema, i.e. parquet/existing tables)
	- does not support `OPTIONS`  for csvs
		- you will need to use temp views first with options, then reference the view in CTAS
		- `CREATE OR REPLACE TEMP VIEW.... USING CSV ... OPTIONS...`
	<br />
19. Create a generated column.
	- column made from another column (lateral referencing)
	- when inserting into table w/ generated column via values, the value needs to be correct/consistent with the generation equation
	<br />
20. Add a table comment.
	- either in column, or for whole table
	```SQL
	CREATE OR REPLACE TABLE purchase_dates (
		id STRING,
		transaction_timestamp STRING,
		price STRING,
		date DATE GENERATED ALWAYS AS (
		cast(cast(transaction_timestamp/1e6 AS TIMESTAMP) AS DATE))
		COMMENT "generated based on `transactions_timestamp` column")
	```

	```SQL
	CREATE OR REPLACE TABLE users_pii
	COMMENT "Contains PII"
	LOCATION "${da.paths.working_dir}/tmp/users_pii"
	PARTITIONED BY (first_touch_date)
	AS
		SELECT *,
		cast(cast(user_first_touch_timestamp/1e6 AS TIMESTAMP) AS DATE) first_touch_date,
		current_timestamp() updated,
		input_file_name() source_file
		FROM parquet.`${da.paths.datasets}/ecommerce/raw/users-historical/`;
	```
21. Use CREATE OR REPLACE TABLE and INSERT OVERWRITE
	- CRAS options:
		- COMMENT `comment`
		- LOCATION `location`
		- PARTITION BY `column\s`
		- DEEP CLONE - full copy of data and metadata
		- SHALLOW CLONE - metadata only (delta log)
		- CRAS old table still exists via delta time travel
	```SQL
	CREATE OR REPLACE TABLE events AS
	SELECT * FROM parquet.`${da.paths.datasets}/ecommerce/raw/events-historical`
	
	
	INSERT OVERWRITE sales
	SELECT * FROM parquet.`${da.paths.datasets}/ecommerce/raw/sales-historical/`
	```

22. Compare and contrast CREATE OR REPLACE TABLE and INSERT OVERWRITE
	- INSERT OVERWRITE 
		- can only work on existing tables
		- only works when schema to insert is correct (matches target table). this is due to delta's schema on write policy
		- can overwrite individual partitions
	<br />
23. Identify a scenario in which MERGE should be used.
	- when you need to upsert in a single transaction
24. Identify MERGE as a command to deduplicate data upon writing.
	- only INSERT data, WHEN NOT MATCHED
25. Describe the benefits of the MERGE command.
	- custom logic
	- combine update, insert, and delete as single transaction
	<br />
26. Identify why a COPY INTO statement is not duplicating data in the target table.
	- COPY INTO is idempotent as compared to INSERT INTO, which just appends per use
	- COPY INTO uses a directory/file path, while INSERT INTO uses a query
27. Identify a scenario in which COPY INTO should be used.
	- for incrementally loading data into a table FROM a source that continuously receives data
28. Use COPY INTO to insert data.
	```SQL
	COPY INTO sales
	FROM "${da.paths.datasets}/ecommerce/raw/sales-30m"
	FILEFORMAT = PARQUET
	```

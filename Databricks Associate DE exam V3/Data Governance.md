&copy [Rangel](https://github.com/jtrangel)

## Data Management with Delta Lake

This section on Delta Lake overlaps with [[ELT With Spark SQL and Python]].

1. Identify where Delta Lake provides ACID transactions. [1](https://docs.databricks.com/en/lakehouse/acid.html)
	- Transactions are at the table level, one table at a time
	- (Optimistic concurrency control)[https://en.wikipedia.org/wiki/Optimistic_concurrency_control] for concurrent transactions
		- BEGIN -> Modify -> Validate -> Commit/Rollback
	- Databricks has no BEGIN/END syntax like TSQL. Changes are made in a serial manner (1 at a time ata meaning neto)
	<br />
2. Identify the benefits of ACID transactions. [1](https://www.databricks.com/glossary/acid-transactions#:~:text=Why%20are%20ACID%20transactions%20a,operation%20that%20only%20partially%20completes.)
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

5. Compare and contrast managed and external tables. [1](https://docs.databricks.com/en/data-governance/unity-catalog/create-tables.html)
	- managed tables - made within databricks via DDL
	- external tables - any tables with external data, regardless of where it is stored (dbfs, abfss, adls, s3).
	- when dropping managed, data and metadata is lost. when dropping external, only metadata is lost.
	<br />
6. Identify a scenario to use an external table. [1](https://learn.microsoft.com/en-us/azure/databricks/data-governance/unity-catalog/create-tables)
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
	- z-ordering = indexing
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
17. Identify the kind of files Optimize compacts. [1](https://docs.gcp.databricks.com/sql/language-manual/delta-optimize.html)
	- small files are compacted and balanced out (combined towards an optimal size, determined by table size)
	- idempotent process
	<br />
18. Identify CTAS (`CREATE TABLE AS SELECT`) as a solution.
	- autoinfer schema from input
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

## Data Access with Unity Catalog

1. Identify one of the four areas of data governance. [1](https://docs.gcp.databricks.com/data-governance/index.html)
	- Data Access Control
	- Data Access Audit
	- Data Lineage
	- Data Discovery
	
	![[Pasted image 20231002144943.png]]

2. Compare and contrast metastores and catalogs. [1](https://docs.gcp.databricks.com/data-governance/unity-catalog/index.html#the-unity-catalog-object-model)
	- Metastores are the top level container which has catalogs. By default, we have hive metastore. 
	- Metastores include **storage credentials**, **external locations**, **shares**, and **recipients**.

3. Identify Unity Catalog securables. [1](https://docs.databricks.com/en/data-governance/unity-catalog/manage-privileges/privileges.html#securable-objects-in-unity-catalog)
	- Securables are the things we can grant or revoke privileges onto.
	
	![[Pasted image 20231002150143.png]]
		
4. Define a service principal.
	- these are like accounts that can be used for connecting to databricks via API on an external IDE/platform. Similar to service accounts which will have credentials usable by several users.
	<br />
5. Identify the cluster security modes compatible with Unity Catalog.
	- Single user
	- User isolation
	
	![[Pasted image 20231002153033.png]]
6. Create a UC-enabled all-purpose cluster.
	- use access mode/security mode to specify the cluster type. Only **single user** and **user isolation** modes are supported by UC.
	
	![[Pasted image 20231002152925.png]]
	
7. Create a DBSQL warehouse.
	![[Pasted image 20231002152745.png]]
	
8. Identify how to query a three-layer namespace.
	```SQL
	SELECT * FROM <catalog>.<schema>.<table>
	```
	
9. Implement data object access control
	- Go to the **Catalog** tab, and under the catalogs/metastores, you can grant/revoke privileges to the data objects
	- For tables/views, use `is_accont_group_member()` function to redact columns or rows using dynamic views 

	```SQL
	# Redact Columns
	CREATE OR REPLACE VIEW agg_heartrate AS
	SELECT
		CASE WHEN
			is_account_group_member('analysts') THEN 'REDACTED'
		ELSE mrn
		END AS mrn,
		CASE WHEN
			is_account_group_member('analysts') THEN 'REDACTED'
		ELSE name
		END AS name,
		MEAN(heartrate) avg_heartrate,
		DATE_TRUNC("DD", time) date
		FROM heartrate_device
		GROUP BY mrn, name, DATE_TRUNC("DD", time)
		
	# Redact rows
	CREATE OR REPLACE VIEW agg_heartrate AS
	SELECT
		mrn,
		time,
		device_id,
		heartrate
	FROM heartrate_device
	WHERE
		CASE WHEN
			is_account_group_member('developers') THEN device_id < 30
			ELSE TRUE
		END
	```
10. Identify colocating metastores with a workspace as best practice.
	-  Add the metastore to the same region as the workspace, to enable low latency. Not sure with what the key point is actually referring to lol.

11.  Identify using service principals for connections as best practice.
	- "Databricks recommends using a service principal and its OAuth token or personal access token instead of your Azure Databricks user account and personal access token. Benefits include: **Granting and restricting access to resources independently of a user**. Enabling users to better protect their own access tokens." [1](https://learn.microsoft.com/en-us/azure/databricks/dev-tools/service-principals)

12. Identify the segregation of business units across catalog as best practice
	- You can do all governance for several workspaces using Unity Catalog. 
	
	 ![[Pasted image 20231002151809.png]]
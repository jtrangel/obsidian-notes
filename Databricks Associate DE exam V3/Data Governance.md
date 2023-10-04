&copy [Rangel](https://github.com/jtrangel)

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
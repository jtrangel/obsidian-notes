&copy [Rangel](https://github.com/jtrangel)

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
	- Silver = enrich tables via validation and dedup
	- Gold = business level aggregates that were refined from silver. Also known as **Knowledge**
	<br />
4. Identify elements of the Databricks Platform Architecture, such as what is located in the data plane versus the control plane and what resides in the customer’s cloud account. [[1](https://docs.databricks.com/en/getting-started/overview.html)] 
	- Control plane - backend of Databricks consisting of Notebooks, workspace config, clusters
	- Data plane - where data is processed (i.e. EC2, Azure/GCP VMs) and stored (i.e. S3, Azure Blob)
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
	- Commit, Push, Pull, Merge, Rebase, and create branches
	<br />
15. Identify limitations in Databricks Notebooks version control functionality relative to Repos. [[1](https://docs.databricks.com/en/repos/git-version-control-legacy.html), [2](https://www.examtopics.com/discussions/databricks/view/104744-exam-certified-data-engineer-associate-topic-1-question-10/#:~:text=An%20advantage%20of%20using%20Databricks%20Repos%20over%20the%20built%2Din,Databricks%20Repos%20is%20built%20upon.)]

	- NBs have version control via Revisions and you can link it with a git repository as well.
	- However NBs cannot be linked to multiple branches, which makes it impractical for developing in parallel
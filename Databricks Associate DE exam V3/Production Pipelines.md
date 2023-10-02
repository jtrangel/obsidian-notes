&copy [Rangel](https://github.com/jtrangel)

## Workloads with Workflows

1.  Identify benefits of using multiple tasks in Jobs.
	- allows you to orchestrate processing/ingestion of data/tables
	- allows isolation of each task and setting of dependencies between tasks
	<br />
1. Set up a predecessor task in Jobs.
	- aka dependencies
	- use the **Depends on:** field to name a dependency.
	- additionally can add custom run behavior using **Run if** field.
		
	![[Pasted image 20231002140717.png]]
	![[Pasted image 20231002140901.png]]
	
3. Identify a scenario in which a predecessor task should be set up.
	- When you want to run another notebook first.
	
4. Review a task's execution history.
	- Go to **Jobs** under **Workflows**
	
	![[Pasted image 20231002141321.png]]
	 - For task specific details:
		- Go to **Job runs** tab under **Workflows**
		- Click on the task of interest
			
	![[Pasted image 20231002141217.png]]
	![[Pasted image 20231002141231.png]]

5. Identify CRON as a scheduling opportunity.
	- Under job details, we can schedule with CRON syntax
	
	![[Pasted image 20231002141953.png]]
	![[Pasted image 20231002141934.png]]
	
6. Debug a failed task.
	- Could not run due to GCP quota issues, but you can re-run the DAG at the task which failed, after debugging.
	<br />
7.  Set up a retry policy in case of failure.
	![[Pasted image 20231002142320.png]]
8. Create an alert in the case of a failed task.
9. Identify that an alert can be sent via email.
	![[Pasted image 20231002142401.png]]

## Additional Notes:
- You can share data between tasks using **task values**. [1](https://docs.databricks.com/en/workflows/jobs/share-task-context.html) This is similar to XCOMs in Airflow. You can set task values inside the notebook.

- Creating an alert for a Job Task is separate from the **Alerts** tab on the left, which only alerts for queries.
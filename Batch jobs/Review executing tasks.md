# Review currently running or executing batch tasks in D365 FinOps (Finance & Operations)/SCM (Supply Chain Management)
> You may encounter a scenario where you see a **batch job** in <code>"Executing"</code> status but it is never running and you probably need to know the reason behind this. If you click the batch job that's in "Executing" status, D365 FinOps (Finance & Operations) will open details page for that <code>"Executing"</code> batch job and you spot that its lines or batch tasks are not executing sitting in <code>"Ready"</code> state. This is because the system is currently servicing other batch tasks and the system does not have enough capacity to process the batch task you are wanting it to execute. **This is when this script** comes in handy which will give you a snapshot of currently <code>"Executing"</code> **batch tasks** which is going to tell you which batch tasks are currently executing. You can then decide the fate of it.
``` sql
/*
	BatchJob.[Status] & Batch.[Status] enumeration:
	--0--Withhold
	--1--Waiting
	--2--Executing
	--3--
	--4--
	--5--Ready
*/
SELECT 
CASE
	WHEN Batch.[Status] =0 THEN 'Withhold'
	WHEN Batch.[Status] =1 THEN 'Waiting'
	WHEN Batch.[Status] =2 THEN 'Executing'
	WHEN Batch.[Status] =5 THEN 'Ready'
ELSE CAST(Batch.[Status] AS NVARCHAR(5)) END AS BatchTaskStatus,
GroupId BatchGroupId, 
Batch.Caption BatchTask,
CASE
	WHEN Batch.Caption = '' THEN BatchJob.Caption
ELSE Batch.Caption END AS BatchJob,
Batch.Company, BatchJobId,
CASE
	WHEN BatchJob.[Status] =0 THEN 'Withhold'
	WHEN BatchJob.[Status] =1 THEN 'Waiting'
	WHEN BatchJob.[Status] =2 THEN 'Executing'
	WHEN BatchJob.[Status] =5 THEN 'Ready'
ELSE CAST(BatchJob.[Status] AS NVARCHAR(5)) END AS BatchJobStatus	
FROM Batch
JOIN BatchJob ON
	BatchJob.RecId = Batch.BatchJobId
WHERE 
	Batch.[Status] = 2
```

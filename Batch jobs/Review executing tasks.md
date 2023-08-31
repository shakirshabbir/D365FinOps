``` sql
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
	WHEN BatchJob.[Status] =2 THEN'Executing'
	WHEN BatchJob.[Status] =5 THEN 'Ready'
ELSE CAST(BatchJob.[Status] AS NVARCHAR(5)) END AS BatchJobStatus	
FROM Batch
JOIN BatchJob ON
	BatchJob.RecId = Batch.BatchJobId
WHERE 
	Batch.[Status] = 2

/*
	BatchJob.[Status] & Batch.[Status] enumeration:
	--0--Withhold
	--1--Waiting
	--2--Executing
	--3--
	--4--
	--5--Ready
*/
```

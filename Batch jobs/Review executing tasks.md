``` sql
SELECT 
	CASE WHEN Batch.[Status] =1 THEN 'Waiting' WHEN Batch.[Status] =2 THEN 'Executing' WHEN Batch.[Status] =5 THEN 'Ready'
	ELSE CAST(Batch.[Status] AS NVARCHAR(5)) END AS BatchTaskStatus,
	GroupId, 
	Batch.Caption BatchTask,
	CASE WHEN TRIM(Batch.Caption) = '' THEN BatchJob.Caption ELSE Batch.Caption END AS BatchJob,
	Batch.Company, BatchJobId,
	CASE WHEN BatchJob.[Status] =1 THEN 'Waiting' WHEN BatchJob.[Status] =2 THEN 'Executing' WHEN BatchJob.[Status] =5 THEN 'Ready'
	ELSE CAST(BatchJob.[Status] AS NVARCHAR(5)) END AS BatchJobStatus	
FROM BATCH 
JOIN BatchJob ON BatchJob.RecId = Batch.BatchJobId
WHERE 
	Batch.[Status] = 2

/*
	BatchJob.Status & Batch.Status values
	--1--Waiting
	--2--Executing
	--3--
	--
	--5--Ready
*/
```

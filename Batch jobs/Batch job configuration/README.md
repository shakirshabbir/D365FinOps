## Copy batch job to multiple/all legal entities in D365 FinOps (Finance & Operations)/SCM (Supply Chain Management)

Consider a scenario where you need to configure a batch job (example posting general journals) for one legal entity and then you need to replicate the same across all legal entities in your D365 FinOps or D365 Finance & Operations environment. Now, it wont be an issue, if its a handful of legal entities, simply repeat it manually. But, if number of legal entities exceeds 100, or say 200, in my case it was more than 700 legal entities, then you cant just do it manually. You will need an automation. One way to achive it, is write a job in D365 X++, but if this is not a recurring activity and you will need it to do only once/twice before go-live, the following script is handy with only one caveat that I will mention in the end with solution.

```sql
DECLARE @LegalEntityId VARCHAR(25)
DECLARE @SourceLegalEntityId VARCHAR(25)
SET @SourceLegalEntityId = '0000'
DECLARE @BatchJobDescription VARCHAR(255)
SET @BatchJobDescription = 'Customer Aging Snapshot'

	--SELECT COUNT(*) LEGALENTITYID FROM OMLEGALENTITY WHERE LEGALENTITYID NOT IN ('0000', 'DAT')

DECLARE Companies_Cursor CURSOR FOR 
WITH Companies 
	AS (
		SELECT TOP 5 LEGALENTITYID FROM OMLEGALENTITY --******Replace with actual count*******
		WHERE LEGALENTITYID NOT IN ('0000', 'DAT')
		ORDER BY LEGALENTITYID ASC
	)
SELECT * FROM Companies

OPEN Companies_Cursor
FETCH NEXT FROM Companies_Cursor into @LegalEntityId
WHILE @@FETCH_STATUS=0
BEGIN
	/*
		====================================
		Refer to D365 X++ class BatchJobCopy
		====================================
	*/
	
	/*
		Insert batch header
	*/
		INSERT INTO [dbo].[BATCHJOB] (
			 [CANCELEDBY]
			,[CAPTION]
			,[COMPANY]
			,[DATAPARTITION]
			,[ENDDATETIME]
			,[ENDDATETIMETZID]
			,[FINISHING]
			,[LOGLEVEL]
			,[ORIGSTARTDATETIME]
			,[ORIGSTARTDATETIMETZID]
			,[RECURRENCEDATA]
			,[RUNTIMEJOB]
			,[STARTDATETIME]
			,[STARTDATETIMETZID]
			,[STATUS]
			,[STARTDATE]
			,[STARTTIME]
			,[CRITICAL]
			,[MONITORINGCATEGORY]
			,[MANAGED]
			,[ACTIVEPERIOD]
			,[EXECUTINGBY]
			,[SCHEDULINGPRIORITY]
			,[SCHEDULINGPRIORITYISOVERRIDDEN]
			,[BATCHGROUP]
			,[EMITBUSINESSEVENT]
			,[RECID]
			,[RECVERSION]
			,[MODIFIEDDATETIME]
			,[CREATEDDATETIME]
			,[CREATEDBY]
		)
		SELECT '' --[CANCELEDBY] 
			  ,[CAPTION]
			  ,@LegalEntityId --[COMPANY]
			  ,[DATAPARTITION]
			  ,'1900-01-01 00:00:00.000' --[ENDDATETIME] --newJob.EndDateTime = DateTimeUtil::minValue();
			  ,0 --[ENDDATETIMETZID] --newJob.EndDateTime = DateTimeUtil::minValue();
			  ,0 --[FINISHING] -- newJob.Finishing = 0;
			  ,[LOGLEVEL]
			  ,[ORIGSTARTDATETIME]
			  ,[ORIGSTARTDATETIMETZID]
			  ,[RECURRENCEDATA]
			  ,[RUNTIMEJOB]
			  ,'1900-01-01 00:00:00.000' --[STARTDATETIME] --newJob.StartDateTime = DateTimeUtil::minValue();
			  ,0 --[STARTDATETIMETZID] --newJob.StartDateTime = DateTimeUtil::minValue();
			  ,0 --[STATUS] -- Withhold=0
			  ,'1900-01-01 00:00:00.000' --[STARTDATE] --newJob.StartDate = dateNull();
			  ,0 --[STARTTIME] -- newJob.StartTime = 0;
			  ,[CRITICAL]
			  ,[MONITORINGCATEGORY]
			  ,[MANAGED]
			  ,[ACTIVEPERIOD]
			  ,'Admin' --[EXECUTINGBY]
			  ,[SCHEDULINGPRIORITY]
			  ,[SCHEDULINGPRIORITYISOVERRIDDEN]
			  ,[BATCHGROUP]
			  ,[EMITBUSINESSEVENT]
			  ,(SELECT MAX(RECID)+1 FROM [dbo].[BATCHJOB]) -- RECID
			  ,[RECVERSION]
			  ,GETDATE() --[MODIFIEDDATETIME]
			  ,GETDATE() --[CREATEDDATETIME]
			  ,'Admin' --[CREATEDBY]
		FROM [dbo].[BATCHJOB]
		WHERE CAPTION = @BatchJobDescription
		AND [COMPANY] = @SourceLegalEntityId

	/*
		Insert batch task(s)
	*/
		INSERT INTO [dbo].[BATCH](
			 [AUTOMATICTRANSACTION]
			,[BATCHJOBID]
			,[CAPTION]
			,[CLASSNUMBER]
			,[COMPANY]
			,[CONSTRAINTTYPE]
			,[DATAPARTITION]
			,[ENDDATETIME]
			,[ENDDATETIMETZID]
			,[EXCEPTIONCODE]
			,[EXECUTEDBY]
			,[GROUPID]
			,[IGNOREONFAIL]
			,[INFO]
			,[PARAMETERS]
			,[PRIORITY]
			,[PRIVATETASK]
			,[RETRIESONFAILURE]
			,[RETRYCOUNT]
			,[RUNTIMETASK]
			,[RUNTYPE]
			,[SERVERID]
			,[SESSIONIDX]
			,[SESSIONLOGINDATETIME]
			,[SESSIONLOGINDATETIMETZID]
			,[STARTDATETIME]
			,[STARTDATETIMETZID]
			,[STATUS]
			,[RECID]
			,[RECVERSION]
			,[MODIFIEDDATETIME]
			,[CREATEDDATETIME]
			,[CREATEDBY]		
		)
		SELECT [AUTOMATICTRANSACTION]
			  ,(SELECT TOP 1 [RECID] FROM [dbo].[BATCHJOB] WHERE CAPTION = @BatchJobDescription AND [COMPANY] = @LegalEntityId) --[BATCHJOBID]
			  ,[CAPTION]
			  ,[CLASSNUMBER]
			  ,@LegalEntityId --[COMPANY]
			  ,[CONSTRAINTTYPE]
			  ,[DATAPARTITION]
			  ,'1900-01-01 00:00:00.000' --[ENDDATETIME] --newJob.EndDateTime = DateTimeUtil::minValue();
			  ,0 --[ENDDATETIMETZID] --newJob.EndDateTime = DateTimeUtil::minValue();
			  ,[EXCEPTIONCODE] -- Exception::Info
			  ,'' --[EXECUTEDBY] -- newTask.ExecutedBy = '';
			  ,[GROUPID]
			  ,[IGNOREONFAIL]
			  ,NULL --[INFO] --newTask.Info = conNull();
			  ,[PARAMETERS]
			  ,[PRIORITY]
			  ,[PRIVATETASK]
			  ,[RETRIESONFAILURE]
			  ,[RETRYCOUNT] --newTask.retryCount = 0;
			  ,[RUNTIMETASK]
			  ,[RUNTYPE]
			  ,'' --[SERVERID] --newTask.ServerId = '';
			  ,0 --[SESSIONIDX] --newTask.SessionIdx = 0;
			  ,'1900-01-01 00:00:00.000' --[SESSIONLOGINDATETIME] --newTask.SessionLoginDateTime = DateTimeUtil::minValue();
			  ,0 --[SESSIONLOGINDATETIMETZID] --newTask.SessionLoginDateTime = DateTimeUtil::minValue();
			  ,'1900-01-01 00:00:00.000' --[STARTDATETIME] --newTask.StartDateTime = DateTimeUtil::minValue();
			  ,0 --[STARTDATETIMETZID] --newTask.StartDateTime = DateTimeUtil::minValue();
			  ,0 --[STATUS] --Withhold=0
			  ,(SELECT MAX(RECID)+1 FROM [dbo].[BATCH])
			  ,[RECVERSION]
			  ,GETDATE() --[MODIFIEDDATETIME]
			  ,GETDATE() --[CREATEDDATETIME]
			  ,'Admin' --[CREATEDBY]
		FROM [dbo].[BATCH]
		WHERE BATCHJOBID = (
			SELECT TOP 1 [RECID] FROM [dbo].[BATCHJOB] 
			WHERE CAPTION = @BatchJobDescription
			AND [COMPANY] = @SourceLegalEntityId
		)
	
	/*
		Insert batch alerts(s)
	*/
		INSERT INTO [dbo].[BATCHJOBALERTS](
			 [BATCHJOBCANCELED]
			,[BATCHJOBENDED]
			,[BATCHJOBERROR]
			,[BATCHJOBID]
			,[DATAPARTITION]
			,[EMAIL]
			,[POPUP]
			,[USERID]
			,[RECID]
			,[RECVERSION]
			,[CREATEDBY]		
		)
		SELECT [BATCHJOBCANCELED]
			  ,[BATCHJOBENDED]
			  ,[BATCHJOBERROR]
			  ,(SELECT TOP 1 [RECID] FROM [dbo].[BATCHJOB] WHERE CAPTION = @BatchJobDescription AND [COMPANY] = @LegalEntityId) --[BATCHJOBID]
			  ,[DATAPARTITION]
			  ,[EMAIL]
			  ,[POPUP]
			  ,'Admin' --[USERID]
			  ,(SELECT MAX(RECID)+1 FROM [dbo].[BATCHJOBALERTS]) -- RECID
			  ,[RECVERSION]
			  ,'Admin' --[CREATEDBY]
		FROM [dbo].[BATCHJOBALERTS]
		WHERE BATCHJOBID = (
			SELECT TOP 1 [RECID] FROM [dbo].[BATCHJOB] 
			WHERE CAPTION = @BatchJobDescription
			AND [COMPANY] = @SourceLegalEntityId
		)

	PRINT 'Batch job "' + @BatchJobDescription + '" sucessfully copied into legal entity ' + @LegalEntityId

	FETCH NEXT FROM Companies_Cursor INTO @LegalEntityId
END
CLOSE Companies_Cursor
DEALLOCATE Companies_Cursor
  ```

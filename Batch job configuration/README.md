# D365 Finance & Operation

## Batch job configuration for multiple legal entities

### Insert header record


 DirPartyLocation
 -DirPartyTable
  -VendTable
 -LogisticsLocation
  -LogisticsPostalAddress

```sql
INSERT INTO [BATCHJOB]
SELECT [CANCELEDBY] 
      ,[CAPTION]
      ,'0003'
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
      ,(SELECT MAX(RECID)+1 FROM [AxDB].[dbo].[BATCHJOB])
      ,1
      ,[MODIFIEDDATETIME]
      ,[CREATEDDATETIME]
      ,[CREATEDBY]
  FROM [AxDB].[dbo].[BATCHJOB]
  WHERE CAPTION = 'O2C_AR0003DL_Customer Aging Snapshot'
  AND [COMPANY] = '0001'
  ```

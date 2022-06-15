# Database movement from Tier 2 to Tier 1 (Sandbox to DEV)
Reference : [Export a copy of the standard user acceptance testing (UAT) database](https://docs.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/database/dbmovement-scenario-exportuat)

```mermaid
stateDiagram-v2
  s1: Export copy of the sandbox database
  s2: Download the latest copy of SQLPackage
  s3: Extract .bacpac file using SQLPackage utility
  note right of s2
    This is a one time step only
  end note
  s4: Update the imported database

  state fork_state <<fork>>
    [*] --> fork_state : Take a backup of AxDB database on the target environment
    fork_state --> s1
    s1 --> s2
    s1 --> s3
    s2 --> s3
    s3 --> s4
```
## Step#1 (Export copy of the sandbox database)

<img src="https://user-images.githubusercontent.com/1909329/173754594-1b9b6722-c9bb-4db1-92a9-93199f97cc05.png" width="250">  <img src="https://user-images.githubusercontent.com/1909329/173754724-17f13e95-1720-4374-93a7-6ba6e55bdca5.png" width="250">   <img src="https://user-images.githubusercontent.com/1909329/173754910-b90aa1aa-351d-43ad-8fac-90a3e85fafce.png" width="250">
> - [ ] By clicking Submit, you agree that Export Database will make the environment temporarily unavailable

## Step#2 (Download the latest copy of SQLPackage)
On the target environment:
- Use the Windows (.NET Framework)/DacFramework installer to install the the latest SQLPackage
 - [Download and run the DacFramework.msi installer for Windows](https://aka.ms/dacfx-msi)
- SqlPackage is installed to the ```C:\Program Files\Microsoft SQL Server\160\DAC\bin``` folder

## Step#3 (Extract .bacpac file using SQLPackage utility)
> [!NOTE] Open command prompt in administrator mode

Navigate to the SQLPackage folder
```Console
cd C:\Program Files\Microsoft SQL Server\160\DAC\bin
```

Extract the .bacpac file
```Console
SqlPackage.exe /a:import /sf:<Location for .bacpac file> /tsn:localhost /tdn:<target database name> /p:CommandTimeout=1200
```
Example:
```Console
SqlPackage.exe /a:import /sf:"D:\Backup\SATbackup.bacpac" /tsn:localhost /tdn:AxDB_copiedFromSandbox_06152022 /p:CommandTimeout=1200
```
> [!NOTE] This is a long running process (takes aboout 30-60 minutes or more depending on the database size)

![image](https://user-images.githubusercontent.com/1909329/173769433-8c620db3-6908-4863-ba01-0b230c05fff3.png)

## Step#4 (Update the imported database)
```sql
USE [AxDB_copiedFromSandbox_06152022]
GO

CREATE USER axdeployuser FROM LOGIN axdeployuser
EXEC sp_addrolemember 'db_owner', 'axdeployuser'

CREATE USER axdbadmin FROM LOGIN axdbadmin
EXEC sp_addrolemember 'db_owner', 'axdbadmin'

CREATE USER axmrruntimeuser FROM LOGIN axmrruntimeuser
EXEC sp_addrolemember 'db_datareader', 'axmrruntimeuser'
EXEC sp_addrolemember 'db_datawriter', 'axmrruntimeuser'

CREATE USER axretaildatasyncuser FROM LOGIN axretaildatasyncuser
EXEC sp_addrolemember 'DataSyncUsersRole', 'axretaildatasyncuser'

CREATE USER axretailruntimeuser FROM LOGIN axretailruntimeuser
EXEC sp_addrolemember 'UsersRole', 'axretailruntimeuser'
EXEC sp_addrolemember 'ReportUsersRole', 'axretailruntimeuser'

CREATE USER axdeployextuser FROM LOGIN axdeployextuser
EXEC sp_addrolemember 'DeployExtensibilityRole', 'axdeployextuser'

CREATE USER [NT AUTHORITY\NETWORK SERVICE] FROM LOGIN [NT AUTHORITY\NETWORK SERVICE]
EXEC sp_addrolemember 'db_owner', 'NT AUTHORITY\NETWORK SERVICE'

UPDATE T1
SET T1.storageproviderid = 0
    , T1.accessinformation = ''
    , T1.modifiedby = 'Admin'
    , T1.modifieddatetime = getdate()
FROM docuvalue T1
WHERE T1.storageproviderid = 1 --Azure storage

DROP PROCEDURE IF EXISTS SP_ConfigureTablesForChangeTracking
DROP PROCEDURE IF EXISTS SP_ConfigureTablesForChangeTracking_V2
GO
-- Begin Refresh Retail FullText Catalogs
DECLARE @RFTXNAME NVARCHAR(MAX);
DECLARE @RFTXSQL NVARCHAR(MAX);
DECLARE retail_ftx CURSOR FOR
SELECT OBJECT_SCHEMA_NAME(object_id) + '.' + OBJECT_NAME(object_id) fullname FROM SYS.FULLTEXT_INDEXES
    WHERE FULLTEXT_CATALOG_ID = (SELECT TOP 1 FULLTEXT_CATALOG_ID FROM SYS.FULLTEXT_CATALOGS WHERE NAME = 'COMMERCEFULLTEXTCATALOG');
OPEN retail_ftx;
FETCH NEXT FROM retail_ftx INTO @RFTXNAME;

BEGIN TRY
    WHILE @@FETCH_STATUS = 0 
    BEGIN 
        PRINT 'Refreshing Full Text Index ' + @RFTXNAME;
        EXEC SP_FULLTEXT_TABLE @RFTXNAME, 'activate';
        SET @RFTXSQL = 'ALTER FULLTEXT INDEX ON ' + @RFTXNAME + ' START FULL POPULATION';
        EXEC SP_EXECUTESQL @RFTXSQL;
        FETCH NEXT FROM retail_ftx INTO @RFTXNAME;
    END
END TRY
BEGIN CATCH
    PRINT error_message()
END CATCH

CLOSE retail_ftx; 
DEALLOCATE retail_ftx; 
-- End Refresh Retail FullText Catalogs

```


## Step#5 (Reprovision admin user)

## Step#6 (Swap the database)

## Step#7 (Synchronize the database)

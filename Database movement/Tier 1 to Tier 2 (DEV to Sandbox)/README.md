# Database movement from Tier 1 to Tier 2 (DEV to Sandbox)
Reference : [Golden configuration promotion](https://docs.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/database/dbmovement-scenario-exportuat)

```mermaid
stateDiagram-v2
  s1: Download & install the latest copy of SQLPackage
  note right of s1
    This is one time setup only
  end note
  s2: Backup & Restore DEV database
  s3: Prepare the database
  s4: Export the database to .bacpac file
  s5: Upload the .bacpac file to LCS
  s6: Import the .bacpac file on Sandbox using DataALM operation
  s7: Enable the users

  state fork_state <<fork>>
    [*] --> fork_state : Take a backup of Sanbdox database
    fork_state --> s1
    fork_state --> s2
    s1 --> s2
    s2 --> s3
    s3 --> s4
    s4 --> s5
    s5 --> s6
    s6 --> s7
    s7 --> [*]
```
## Step#1 (Download the latest copy of SQLPackage)
On the target environment:
- Use the Windows (.NET Framework)/DacFramework installer to install the the latest SQLPackage
 - [Download and run the DacFramework.msi installer for Windows](https://aka.ms/dacfx-msi)
- SqlPackage is installed to the ```C:\Program Files\Microsoft SQL Server\160\DAC\bin``` folder

## Step#2 Backup & Restore DEV database
1. Backup AxDB database
    
    <img src="https://user-images.githubusercontent.com/1909329/173927519-51cf3fbf-b01f-4822-95a6-7cacd7960bfa.png" width="750
1. Restore as AxDB_CopyForExport_06152022

    <img src="https://user-images.githubusercontent.com/1909329/173933534-448a2f56-27e2-45e5-9949-34ba6765cb7e.png" width="700">
    
    - > [!NOTE] Change "Restore as" with new file names
      
      <img src="https://user-images.githubusercontent.com/1909329/173933729-07ac8b76-16e7-4b1e-aff4-a320a22291f0.png" width="600">

## Step#3 (Prepare the database)
```sql
USE [AxDB_CopyForExport_06152022]
GO

update sysglobalconfiguration
set value = 'SQLAZURE'
where name = 'BACKENDDB'

update sysglobalconfiguration
set value = 1
where name = 'TEMPTABLEINAXDB'

drop procedure if exists XU_DisableEnableNonClusteredIndexes
drop procedure if exists SP_ConfigureTablesForChangeTracking
drop procedure if exists SP_ConfigureTablesForChangeTracking_V2
drop schema [NT AUTHORITY\NETWORK SERVICE]
drop user [NT AUTHORITY\NETWORK SERVICE]
drop user axdbadmin
drop user axdeployuser
drop user axmrruntimeuser
drop user axretaildatasyncuser
drop user axretailruntimeuser
drop user axdeployextuser

--Tidy up the batch server config from the previous environment
DELETE FROM SYSSERVERCONFIG

--Tidy up server sessions from the previous environment
DELETE FROM SYSSERVERSESSIONS

--Tidy up printers from the previous environment
DELETE FROM SYSCORPNETPRINTERS

--Tidy up client sessions from the previous environment
DELETE FROM SYSCLIENTSESSIONS

--Tidy up batch sessions from the previous environment
DELETE FROM BATCHSERVERCONFIG

--Tidy up batch server to batch group relation table
DELETE FROM BATCHSERVERGROUP

-- Clear encrypted hardware profile merchant properties
update dbo.RETAILHARDWAREPROFILE set SECUREMERCHANTPROPERTIES = null where SECUREMERCHANTPROPERTIES is not null
```

## Step#4 (Export the database to .bacpac file)
> [!NOTE] Open command prompt in administrator mode

Navigate to the SQLPackage folder
```Console
cd C:\Program Files\Microsoft SQL Server\160\DAC\bin
```

Extract the .bacpac file
```Console
SqlPackage.exe /a:export /ssn:localhost /sdn:<database to export> /tf:D:\Exportedbacpac\my.bacpac /p:CommandTimeout=1200 /p:VerifyFullTextDocumentTypesSupported=false
```
Example:
```Console
SqlPackage.exe /a:export /ssn:localhost /sdn:AxDB_CopyForExport_06152022 /tf:"D:\Backup\AxDB_moveToSAT.bacpac" /p:CommandTimeout=1200 /p:VerifyFullTextDocumentTypesSupported=false
```
> [!NOTE] This is a long running process (takes aboout 30-60 minutes or more depending on the database size)

<img src="https://user-images.githubusercontent.com/1909329/173956740-a00f6b43-392b-4a33-8ada-c7edbcabb649.png" width="750">

![image](https://user-images.githubusercontent.com/1909329/173960969-9b864b63-ac5a-472d-bfdb-61400b14e6a7.png)
  
## Step#5 (Upload the .bacpac file to LCS)

## Step#6 (Import the .bacpac file on Sandbox using DataALM operation)
![image](https://user-images.githubusercontent.com/1909329/173960697-c578bc4b-4b5a-457a-b89c-8b37d1c29658.png)
  
![image](https://user-images.githubusercontent.com/1909329/173960747-b263c742-3027-4dd7-afc2-081ca33fed4a.png)
  
![image](https://user-images.githubusercontent.com/1909329/173960833-a85995ae-3454-4b10-a2d5-94d0827fd7fc.png)



## Step#7 (Enable the users)

# Database movement from Tier 2 to Tier 1 (Sandbox to DEV)
Reference : [Export a copy of the standard user acceptance testing (UAT) database](https://docs.microsoft.com/en-us/dynamics365/fin-ops-core/dev-itpro/database/dbmovement-scenario-exportuat)

```mermaid
stateDiagram-v2
  s1: Download the latest copy of SQLPackage
  s2: Export copy of the sandbox database
  s3: Extract .bacpac file using SQLPackage utility
  note right of s1
    This is a one time step only
  end note

  s1 --> s2
  s2 --> s3
```

## Step#1 (Download the latest copy of SQLPackage)
- Use the Windows (.NET Framework)/DacFramework installer to install the the latest SQLPackage
 - [Download and run the DacFramework.msi installer for Windows](https://aka.ms/dacfx-msi)
- SqlPackage is installed to the ```C:\Program Files\Microsoft SQL Server\160\DAC\bin``` folder

## Step#2 (Export copy of the sandbox database)

<img src="https://user-images.githubusercontent.com/1909329/173754594-1b9b6722-c9bb-4db1-92a9-93199f97cc05.png" width="250">  <img src="https://user-images.githubusercontent.com/1909329/173754724-17f13e95-1720-4374-93a7-6ba6e55bdca5.png" width="250">   <img src="https://user-images.githubusercontent.com/1909329/173754910-b90aa1aa-351d-43ad-8fac-90a3e85fafce.png" width="250">
> - [ ] By clicking Submit, you agree that Export Database will make the environment temporarily unavailable

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

![image](https://user-images.githubusercontent.com/1909329/173769433-8c620db3-6908-4863-ba01-0b230c05fff3.png)


## Step#4 (Update the imported database)

## Step#5 (Reprovision admin user)

## Step#6 (Swap the database)

## Step#7 (Synchronize the database)

### Cannot Login after Restore Database

```sql
DECLARE @SID NVARCHAR(124)
DECLARE @NetworkAlias NVARCHAR(255)

--Get the SID, NetworkAlias from the original database
SELECT
    @SID = [SID],
    @NetworkAlias = NetworkAlias
FROM [AxDB_Orig].[dbo].[UserInfo] 
WHERE Id = 'Admin'

--Restore the SID, NetworkAlias to the restored database
UPDATE [AxDB].[dbo].[UserInfo] SET
    [SID] = @SID,
    NetworkAlias = @NetworkAlias
WHERE Id = 'Admin'
```


DRAFT:
```sql
--SELECT * FROM UserInfo WHERE NAME LIKE 'Shakir%'
--SELECT * FROM SecurityUserRole WHERE [User_] = (
--	SELECT TOP 1 ID FROM UserInfo WHERE NAME LIKE 'Shakir%'
--)

--DELETE FROM UserInfo WHERE RECID IN ('5637172327')
--DELETE FROM SecurityUserRole WHERE RECID IN ('5637172363', '5637172364')



DECLARE @UserId VARCHAR(50)
DECLARE @UserName VARCHAR(50)
DECLARE @NetworkAlias VARCHAR(100)
DECLARE @SID VARCHAR(MAX)
--DECLARE @ObjectId VARCHAR(MAX)
DECLARE @IdentityProvider VARCHAR(50)
SET @UserId = 'shakirshabbir'
SET @UserName = 'Shakir Shabbir'
SET @NetworkAlias = 'shakirshabbir@yahoo.com'
SET @SID = 'S-1-XX-XXXXX............'
--SET @ObjectId = '0ACA6E33-3D05-48DB-85FE-C63A8D6276A9'
SET @IdentityProvider = 'https://sts.windows.net/'
UPDATE UserInfo 
SET 
	[SID]=@SID,
	[NETWORKALIAS] = @NetworkAlias,
	[IDENTITYPROVIDER] = @IdentityProvider,
	[COMPANY] = ''
WHERE [ID] = 'Admin'

--[RECVERSION], --If you dont specify, SQL will default 1 for all D365 tables

INSERT INTO UserInfo(
	[ID], [NAME], [NETWORKALIAS], [IDENTITYPROVIDER], [ENABLE], [ENABLEDONCE], [DEFAULTPARTITION],
	[SID], 
	--[OBJECTID],
	--[ACCOUNTTYPE], [INTERACTIVELOGON], --Try without defining these values
	--[EXTERNALIDTYPE], [EXTERNALID], --Try without defining these values
	[COMPANY],
	[NETWORKDOMAIN], 
    [LANGUAGE],
    [HELPLANGUAGE],
	[PREFERREDTIMEZONE],
    [PARTITION],
    [RECID]
    --,[SYSROWVERSIONNUMBER] -- Gets auto generated
) VALUES(
	@UserId, @UserName, @NetworkAlias, @IdentityProvider, 1, 1, 1,
	@SID,
	--@ObjectId,
	--2, 1, --[ACCOUNTTYPE], [INTERACTIVELOGON],
	--1, '', --[EXTERNALIDTYPE], [EXTERNALID],
	(SELECT [COMPANY] FROM UserInfo WHERE Id = 'Admin'), --[COMPANY]
	(SELECT [NETWORKDOMAIN] FROM UserInfo WHERE Id = 'Admin'), --[NETWORKDOMAIN]
	(SELECT [LANGUAGE] FROM UserInfo WHERE Id = 'Admin'), --[LANGUAGE]
	(SELECT [HELPLANGUAGE] FROM UserInfo WHERE Id = 'Admin'), --[HELPLANGUAGE]
	(SELECT [PREFERREDTIMEZONE] FROM UserInfo WHERE Id = 'Admin'), --[PREFERREDTIMEZONE]
	(SELECT [PARTITION] FROM UserInfo WHERE Id = 'Admin'), --[PARTITION]
	(SELECT MAX([RECID])+1 FROM UserInfo) --[RECID]
)

INSERT INTO SecurityUserRole(
	[USER_],
    [SECURITYROLE], [ASSIGNMENTSTATUS], [ASSIGNMENTMODE],
	[VALIDFROMTZID], [VALIDTOTZID],
	[VALIDFROM], [VALIDTO],
    [PARTITION],
    [RECID]
) VALUES(
	@UserId,
	8, 1, 1, --[SECURITYROLE], [ASSIGNMENTSTATUS], [ASSIGNMENTMODE],
	37001, 37001,
	'1900-01-01 00:00:00.000', '1900-01-01 00:00:00.000',
	(SELECT [PARTITION] FROM UserInfo WHERE Id = 'Admin'), --[PARTITION]
	(SELECT MAX([RECID])+1 FROM SecurityUserRole) --[RECID]
)
INSERT INTO SecurityUserRole(
	[USER_],
    [SECURITYROLE], [ASSIGNMENTSTATUS], [ASSIGNMENTMODE],
	[VALIDFROMTZID], [VALIDTOTZID],
	[VALIDFROM], [VALIDTO],
    [PARTITION],
    [RECID]
) VALUES(
	@UserId,
	177, 1, 1, --[SECURITYROLE], [ASSIGNMENTSTATUS], [ASSIGNMENTMODE],
	0, 0,
	'1900-01-01 00:00:00.000', '1900-01-01 00:00:00.000',
	(SELECT [PARTITION] FROM UserInfo WHERE Id = 'Admin'), --[PARTITION]
	(SELECT MAX([RECID])+1 FROM SecurityUserRole) --[RECID]
)

INSERT INTO SysUserInfo(
	[ID], [EMAIL],
    [PARTITION],
    [RECID]
) VALUES(
	@UserId, @NetworkAlias,
	(SELECT [PARTITION] FROM UserInfo WHERE Id = 'Admin'), --[PARTITION]
	--(SELECT MAX([RECID])+1 FROM SysUserInfo) --[RECID]
	'5637172363'
)
```

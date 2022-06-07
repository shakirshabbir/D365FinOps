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

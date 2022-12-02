# Add a user through SQL in D365 Finance & Operations

### Get a list of security roles that you need to assign the user to be added
```sql
--Gives you the list of all security roles related to the system
SELECT [Name], RecId FROM SecurityRole WHERE Name LIKE '%System%'
```

### Add the user to the system
We need the following information:
 - User ID or/ **ID**
 - User name or/ **Name**
 - Email address /or **NetworkAlias**
 
```sql
DECLARE @ExistingUserId NVARCHAR(20) 
DECLARE @UserId NVARCHAR(20) 
DECLARE @UserName NVARCHAR(81) 
DECLARE @NetworkAlias NVARCHAR(255)

SET @ExistingUserId = 'Admin'

SET @UserId = 'lp3sxs11'
SET @UserName = 'Shakir Shabbir'
SET @NetworkAlias = 'lp3sxs11@leggett.com'

INSERT INTO UserInfo(
	 ID
	,[Name]
	,NetworkAlias
	--The following fields will be defaulted from an existing user
	,[Enable]
	,Company
	,NetworkDomain
	,[Language]
	,RecId
) VALUES (
	 @UserId
	,@UserName
	,@NetworkAlias
	--Copy these from an existing user
	,1
	,(SELECT Company FROM UserInfo WHERE ID=@ExistingUserId)
	,(SELECT NetworkDomain FROM UserInfo WHERE ID=@ExistingUserId)
	,(SELECT [Language] FROM UserInfo WHERE ID=@ExistingUserId)
	,(SELECT MAX(RECID) FROM UserInfo)+1
)
```

### Assign roles to the user
I am adding the following roles to the user:
- System administrator
- System user

```sql
SELECT [Name], RecId FROM SecurityRole WHERE [Name] = 'System administrator'
SELECT [Name], RecId FROM SecurityRole WHERE [Name] = 'System user'
```

```sql
DECLARE @ExistingUserId NVARCHAR(20) 
DECLARE @UserId NVARCHAR(20) 

SET @ExistingUserId = 'Admin' -- Copied from previous

SET @UserId = 'lp3sxs11' -- Copied from previous

INSERT INTO SecurityUserRole(
	 User_
	,SecurityRole
	,AssignmentStatus
	,AssignmentMode
	,ValidFrom
	,ValidTo
	,[Partition]
	,RecId
) VALUES (
	 @UserId
	,(SELECT RecId FROM SecurityRole WHERE [Name] = 'System administrator')
	,(SELECT TOP 1 AssignmentStatus FROM SecurityUserRole WHERE User_=@ExistingUserId)
	,(SELECT TOP 1 AssignmentMode FROM SecurityUserRole WHERE User_=@ExistingUserId)
	,(SELECT TOP 1 ValidFrom FROM SecurityUserRole WHERE User_=@ExistingUserId)
	,(SELECT TOP 1 ValidTo FROM SecurityUserRole WHERE User_=@ExistingUserId)
	,(SELECT TOP 1 [Partition] FROM SecurityUserRole WHERE User_=@ExistingUserId)
	,(SELECT MAX(RECID) FROM SecurityUserRole)+1
)
```

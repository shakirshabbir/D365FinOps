# All about data entities

## DMFEntityBase
## DmfEntityWriter

Number sequence preallocation function in Microsoft Dynamics 365 for Finance and Operations
https://blog.mohamedaamer.net/microsoft-dynamics/number-sequence-preallocation-function-in-microsoft-dynamics-365-for-finance-and-operations/

### Problem statement
One of the execution had issues and couldnt remove the execution, so tried to do with SQL, but it couldnt resolve as well.
#### Resolution
Closed services and then tried
## T-SQL
```sql
SELECT * FROM DMFDEFINITIONGROUP
SELECT * FROM DMFDEFINITIONGROUPEXECUTION
SELECT * FROM DMFDEFINITIONGROUPEXECUTIONHISTORY
SELECT * FROM DMFEXECUTION
```

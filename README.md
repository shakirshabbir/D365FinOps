# D365FinOps

## Data entity auto-generate fields

When importing through the Data Management Framework in Dynamics 365 for Finance and Operations, you can use the Auto-generated feature to automatically assign values from the number sequence associated with the field. <b>However, it's worth noting that not all number sequence features are supported.,</b>

The logic is handled by the <b>generateAutoNumbers</b> method in the class <b>DMFGenerateSSISPackage</b>. When looking at the code, the numbers are basically assigned by getting NumberSequenceTable.NextRec and then adding the number of values required based on the number of records being imported. There are some validations, such as making sure the last value isn't higher than NumberSequenceTable.Highest, and it also seem to primarily use any free numbers from NumberSequenceList if any exist.

However, the number sequence feature Preallocation is not supported (see blog post [Number sequence preallocation function in Microsoft Dynamics 365 for Finance and Operations by Mohamed Aamer for more information on the Preallocation functionality](https://blog.mohamedaamer.net/microsoft-dynamics/number-sequence-preallocation-function-in-microsoft-dynamics-365-for-finance-and-operations/)). This means that even if it's enabled on the number sequence, the DMF import will still use the NextRec (plus any free numbers from the list). This is most likely for performance reason, as the Preallocation functionality would with all likelihood be slower when importing large volumes of records.

https://www.gangsta.se/blog/entry/auto-generate-values-during-dmf-import


## Delete vendor addressess(s) from SQL


 DirPartyLocation
 -DirPartyTable
  -VendTable
 -LogisticsLocation
  -LogisticsPostalAddress

```sql
--1) Delete all addresses except primary address
--2) Change Address description with Vendor Name (only for Primary Address)
--3) Add purpose "Delivery" & "RemitTo" to existing primary address

DECLARE @VendorId VARCHAR(50)
SET @VendorId = '3500 COFFEE LLC'
DECLARE @VendorName VARCHAR(50)
SET @VendorName = '3500 COFFEE LLC'
DECLARE @PrimaryPartyLocation VARCHAR(50)
DECLARE @LocationRoleDelivery VARCHAR(50)
DECLARE @LocationRoleRemitTo VARCHAR(50)
SET @LocationRoleDelivery = '5637144577'
SET @LocationRoleRemitTo = '5637144584'

DELETE FROM LogisticsPostalAddress
WHERE [Location] IN (
	SELECT LogisticsLocation.RECID
	FROM DirPartyLocation
		JOIN DirPartyTable ON 
			DirPartyTable.RECID = DirPartyLocation.Party
		JOIN LogisticsLocation ON 
			LogisticsLocation.RECID = DirPartyLocation.[Location]
		JOIN VendTable ON 
			VendTable.Party = DirPartyTable.RecId
	WHERE
		VendTable.AccountNum = @VendorId
		AND LogisticsLocation.ISPOSTALADDRESS = 1
		AND DirPartyLocation.ISPRIMARY <> 1
) PRINT 'Deleted LogisticsPostalAddress'

DELETE FROM LogisticsLocation
WHERE LogisticsLocation.RecId IN (
	SELECT [Location]
	FROM DirPartyLocation
		JOIN DirPartyTable ON 
			DirPartyTable.RECID = DirPartyLocation.Party
		JOIN VendTable ON 
			VendTable.Party = DirPartyTable.RecId
	WHERE
		VendTable.AccountNum = @VendorId
		AND DirPartyLocation.ISPRIMARY <> 1
)
AND LogisticsLocation.ISPOSTALADDRESS = 1
PRINT 'Deleted LogisticsLocation'

DELETE FROM DirPartyLocation
WHERE RecId IN (
	SELECT DirPartyLocation.RECID
	FROM DirPartyLocation
		JOIN DirPartyTable ON 
			DirPartyTable.RECID = DirPartyLocation.Party
		JOIN VendTable ON 
			VendTable.Party = DirPartyTable.RecId
	WHERE
		VendTable.AccountNum = @VendorId
)
AND DirPartyLocation.ISPRIMARY <> 1
PRINT 'Deleted DirPartyLocation'

SELECT @PrimaryPartyLocation = DirPartyLocation.RECID
FROM DirPartyLocation 
JOIN DirPartyTable ON 
	DirPartyTable.RECID = DirPartyLocation.Party
	JOIN VendTable ON 
		VendTable.Party = DirPartyTable.RecId
WHERE
	VendTable.AccountNum = @VendorId
	AND DirPartyLocation.ISPRIMARY = 1
PRINT ''
PRINT '@PrimaryPartyLocation = ' + @PrimaryPartyLocation

IF NOT EXISTS (
	SELECT * FROM LogisticsLocation WHERE [DESCRIPTION] = @VendorName AND RecId = @PrimaryPartyLocation
) 
BEGIN
	UPDATE LogisticsLocation
	SET [DESCRIPTION] = @VendorName
	WHERE RecId = @PrimaryPartyLocation
	
	PRINT 'Updated Address description'
END


IF NOT EXISTS (
	SELECT * FROM DirPartyLocationRole WHERE LOCATIONROLE = @LocationRoleDelivery --Delivery
		AND PARTYLOCATION = @PrimaryPartyLocation
)
BEGIN
    INSERT INTO DirPartyLocationRole (LOCATIONROLE, PartyLocation)
    VALUES (@LocationRoleDelivery, @PrimaryPartyLocation)--Delivery
	
	PRINT 'Address set to Delivery role'
END


IF NOT EXISTS (
	SELECT * FROM DirPartyLocationRole WHERE LOCATIONROLE = @LocationRoleRemitTo --RemitTo
		AND PARTYLOCATION = @PrimaryPartyLocation
)
BEGIN
    INSERT INTO DirPartyLocationRole (LOCATIONROLE, PartyLocation)
    VALUES (@LocationRoleRemitTo, @PrimaryPartyLocation)--RemitTo

	PRINT 'Address set to RemitTo role'
END
```

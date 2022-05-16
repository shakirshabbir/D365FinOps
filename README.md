# D365FinOps

DELETE ADDRESS


 DirPartyLocation
 >DirPartyTable
  >>VendTable
 >LogisticsLocation
  >>LogisticsPostalAddress

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

SELECT @PrimaryPartyLocation = DirPartyLocation.RECID
FROM DirPartyLocation 
JOIN DirPartyTable ON 
	DirPartyTable.RECID = DirPartyLocation.Party
	JOIN VendTable ON 
		VendTable.Party = DirPartyTable.RecId
WHERE
	VendTable.AccountNum = @VendorId
	AND DirPartyLocation.ISPRIMARY = 1

IF NOT EXISTS (
	SELECT * FROM LogisticsLocation WHERE [DESCRIPTION] = @VendorName AND RecId = @PrimaryPartyLocation
) 
BEGIN
	UPDATE LogisticsLocation
	SET [DESCRIPTION] = @VendorName
	WHERE RecId = @PrimaryPartyLocation
END

IF NOT EXISTS (
	SELECT * FROM DirPartyLocationRole WHERE LOCATIONROLE = @LocationRoleDelivery --Delivery
		AND PARTYLOCATION = @PrimaryPartyLocation
)
    INSERT INTO DirPartyLocationRole (LOCATIONROLE, PartyLocation)
    VALUES (@LocationRoleDelivery, @PrimaryPartyLocation)--Delivery

IF NOT EXISTS (
	SELECT * FROM DirPartyLocationRole WHERE LOCATIONROLE = @LocationRoleRemitTo --RemitTo
		AND PARTYLOCATION = @PrimaryPartyLocation
)
    INSERT INTO DirPartyLocationRole (LOCATIONROLE, PartyLocation)
    VALUES (@LocationRoleRemitTo, @PrimaryPartyLocation)--RemitTo

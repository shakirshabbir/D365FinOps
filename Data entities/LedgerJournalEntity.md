Some key points:
- On the entity, if you specify AccountType=Ledger on a line, then, it will ignore the DefaultDimensionDisplayValue, instead it will take into account the AccountDisplayValue
- If the AccountType!=Ledger i.e. Customer, Vendor, it will take the DefaultDimensionDisplayValue for ledger entry
- If Debit & Credit is specified on the same line, then it must have an offset account
- Voucher field cannot be left blank, it can be an arbitary number
- For all journals imported, if voucher field is blank, then setting "Voucher Number Allocation at Posting" must be set to Yes
- If you change the "Number allocation at posting" to yes after a journal has been imported, it will not override the previously created journal, you might have to re-create the journal.

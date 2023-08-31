## Some key points related to LedgerJournalEntity or General journal entity in D365 FinOps (Finance & Operations)/SCM (Supply Chain Management)
- On the entity, if you specify <code>AccountType=Ledger</code> on a line, then, it will ignore the ~~DefaultDimensionDisplayValue~~, instead it will take into account the **AccountDisplayValue**
- On the contrary, if the AccountType!=Ledger i.e. <code>Customer</code>, <code>Vendor</code> or <code>Bank</code>, then it will take into account the **DefaultDimensionDisplayValue** and will ignore the ~~DefaultDimensionDisplayValue~~
- If **Debit** & **Credit** is specified on the same line, then it must have an **offset account** defined.
- <code>Voucher</code> field cannot be left blank, it can be an temporary number.
- For all journals imported:
  - If voucher field is a temporary number, then it must be unique in the respective legal entity **OR**
  - **"Number Allocation at Posting"** (voucher setting for the journal ) must be set to **"Yes"**.
- If you modify the voucher setting **"Number allocation at posting"** after a journal has been imported, it will not override the "Number allocation at posting" setting for previously created journals. They will still fail at posting if the temporary voucher number was not unique in that respective legal entity

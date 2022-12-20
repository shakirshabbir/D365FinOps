Breakpint (When packing slip tries to post)
[s]    \Classes\FormletterService\postSalesOrderPackingSlip                                                   14

Breakpint (Updating physical issue)
[s]    \Classes\InventUpd_Physical\updatePhysicalIssue                                                        26
    if (movement.mustBePicked())
    {
        caseStatusIssueFrom     = StatusIssue::Picked;
        caseStatusIssueTo       = StatusIssue::Picked;
    }
    else
    {
        caseStatusIssueFrom     = StatusIssue::Picked;
        caseStatusIssueTo       = StatusIssue::OnOrder;
    }

Breakpint (Checking for negative inventory)
[s]    \Classes\InventOnHandQty\checkItemDraw                                                                 39
    if (! _negativePhysical)
    {
        // If WHS item, perform different check
        if (itemUsesWHS)
        {
            return this.whsCheckItemDraw(_inventQty, _addInfo);
        }

Breakpint (Custom check Item Draw method for WHS items - Checks against the item's physical availability if provided quantity can be picked)
[s]    \Classes\InventOnHandQty\whsCheckItemDraw                                                              20
    availQty = WHSInventOnHand::getPhysicalAvailQty(itemId,
                                                    inventDimCriteria,
                                                    !skipDelta);

    if (availQty < (-_inventQty))
    {
        if (_addInfo)
        {
            setPrefix("@SYS70390");

Breakpint (Checks for max physicalAvailable on WHSInventReserve)
[s]    \Classes\WHSInventOnHand\getPhysicalAvailQty                                                           10
[s]    \Classes\WHSInventOnHand\getPhysicalAvailQty                                                           36
                maxPhysicalAvail = WHSInventOnHand::getReservePhysicalAvailQty(_itemId, _inventDimCriteria, _isWork);

Breakpoint (Checks for max physical available inventory - Checks in InventDim table)
[s]    \Classes\WHSInventOnHand\getReserveAvailQty                                                            21
    // Get the single row for this InventDim
    if (_inventDimCriteria.InventDimId)
    {
        select firstonly AvailPhysical, AvailOrdered from inventReserve
            where inventReserve.ItemId      == _itemId &&
                  inventReserve.InventDimId == _inventDimCriteria.InventDimId;
    }

Breakpoint (Checks for max physical available inventory - Now Checks in WHSInventReserve table)    
[s]    \Classes\WHSInventOnHand\getReserveAvailQty                                                            96
        select minof(AvailPhysical), minof(AvailOrdered) from inventReserve
            where inventReserve.ItemId == _itemId
            #WHSInventDimExistsJoin(inventReserve.InventDimId, inventDim, _inventDimCriteria);
    }

    availPhysical = min(availPhysical, inventReserve.AvailPhysical);
    availOrdered = min(availOrdered, inventReserve.AvailOrdered);
    
Breakpoint (Ready to consume the physica issue)    
[s]    \Classes\InventUpd_Physical\updatePhysicalIssue                                                       142
            if (qtyNow)
            {
                if (qtyNow  > inventTrans.Qty)
                {
                    inventTrans.updateSplit(qtyNow, cwQtyNow);
                }    

Breakpoint (Inserts records in the WHSInventReserveDelta table based on the parameters)
[s]    \Classes\WHSInventOnHand\insertWHSInventReserveDeltaFromSumDelta                                       21
    [availPhysical, availOrdered, reservPhysical, reservOrdered] = WHSInventOnHand::calcInventReserveQtyFromInventSumDelta(_inventSumDelta);

Breakpoint (Inserts records in the WHSInventReserveDelta table based on the parameters)
[s]    \Classes\WHSInventOnHand\insertReserveDelta                                                            20
    if (inventDim.wmsLocationId || !_onlyLocationAndBelow)
    {
        inventReserveDelta.ItemId               = _itemId;
        inventReserveDelta.InventDimId          = _inventDimId;
        inventReserveDelta.ReservPhysical       = _reservPhysical;
        inventReserveDelta.ReservOrdered        = _reservOrdered;
        inventReserveDelta.AvailPhysical        = _availPhysical;
        inventReserveDelta.AvailOrdered         = _availOrdered;
        inventReserveDelta.TTSId                = _ttsId;

        levelsEnumerator = WHSInventReserveDeltaLevelsEnumerator::newFromDelta(inventReserveDelta, _onlyLocationAndBelow);
        while (levelsEnumerator.moveNext())
        {
            inventDim = levelsEnumerator.currentInventDim();

            inventReserveDelta.InventDimId = inventDim.inventDimId;
            inventReserveDelta.HierarchyLevel = levelsEnumerator.currentHierarchyLevel();

            recordInsertList = recordInsertList ? recordInsertList : new RecordInsertList(tableNum(WHSInventReserveDelta));
            recordInsertList.add(inventReserveDelta);    

Breakpoint (When output order is attached to a sales line TransChildRefId)
[s]    \Classes\InventUpd_Physical\updatePhysicalIssue                                                       246
        while select sum(Qty), TransChildType, TransChildRefId from inventTrans
            group by InventTransOrigin, StatusReceipt, StatusIssue, TransChildType, TransChildRefId
            where inventTrans.InventTransOrigin == movement.inventTransOriginId()
               && inventTrans.StatusReceipt     == StatusReceipt::None
               && inventTrans.StatusIssue       >= caseStatusIssueFrom
               && inventTrans.StatusIssue       <= caseStatusIssueTo
               && inventTrans.TransChildType    != movement.transChildType()
        {
            if (inventTrans.Qty != 0)
            {
                if (inventTrans.TransChildType == InventTransChildType::None)
                {
                    info(strFmt("@SYS66069", abs(inventTrans.Qty), movement.transType(), movement.transRefId()));
                }
                else
                {
                    info(strFmt("@SYS66069", abs(inventTrans.Qty), inventTrans.TransChildType, inventTrans.TransChildRefId));
                }
            }
        }


[s]    \Classes\InventUpdateOnhand\ttsNotifyPreCommit                                                         39
        // Process Insert for reservation records
        this.whsInsertInventReserve();

        this.lockInventSum();                   // serialize on InventSum for Onhand checks

        this.updateInventSum();                 // set based update of inventSum

        // Lock and process reservation records
        this.whsLockInventReserve();            // serialize on WHSInventReserve for Avail checks

        this.whsAdjustDeltasForNegativeWarehouses();

        this.whsUpdateInventReserve();          // set based update of WHSInventReserve

        if (!this.checkOnhand())                // combined Onhand check for all movements
        {
            throw error("@SYS18447");
        }


Breakpoint (When records are inserted into WHSInventReserve table - more records will be inserted if Reservation Heiracrhy is setup)
[s]    \Classes\InventUpdateOnhand\whsInsertInventReserve                                                     11
    if (inventReserveDeltaExist)
    {
        while select ItemId, InventDimId, HierarchyLevel from inventReserveDelta
            group by ItemId, InventDimId, HierarchyLevel
            where inventReserveDelta.ttsId  == this.ttsId()
         notexists join inventReserve
            where inventReserve.ItemId      == inventReserveDelta.ItemId    &&
                  inventReserve.InventDimId == inventReserveDelta.InventDimId
        {

Breakpoint (When system tries to check 0 quantity against available quantity to ensure negative invetory check)
[s]    \Classes\InventUpdateOnhand\checkOnhand                                                               223
      case InventOnhandCheckType::WHSPhysical         :
          if (!inventOnhand.checkItemDraw(0,false,true))
          {
              return false;
          }

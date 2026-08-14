ME_PROCESS_OUT_CUST

Implementation Short Text     PO Screen Enhancement
Definition Name               ME_GUI_PO_CUST


IF_EX_ME_PROCESS_PO_CUST~PROCESS_ITEM

    " BOC adde by Vamsi SHDK922152 | 12-06-2025 export item details to check method
    DATA: ls_item TYPE mepoitem.
    DATA: ls_item1 TYPE TABLE OF mepoitem.
    DATA: lt_item TYPE STANDARD TABLE OF mepoitem.
      ls_item = im_item->get_data( ).
      IMPORT lt_item TO lt_item FROM  MEMORY ID 'LV_ITEM'.  "Import Item Details From ProcessItem methods
      READ TABLE lt_item INTO DATA(wa_item) WITH KEY ebelp = ls_item-ebelp.
      if sy-subrc = 0.
      DELETE TABLE lt_item FROM wa_item.
      endif.
    IF ls_item-loekz NE 'D'.
      APPEND ls_item TO lt_item.
      CLEAR: ls_item.
      endif.
      EXPORT lt_item FROM lt_item TO MEMORY ID 'LV_ITEM'.
    " eoc




IF_EX_ME_PROCESS_PO_CUST~CHECK
  IF sy-tcode = 'ME21' OR sy-tcode = 'ME22' OR sy-tcode = 'ME21N' OR sy-tcode = 'ME22N'.
      DATA : ls_msg TYPE char100.
      DATA: ls_header TYPE mepoheader.
      DATA: ls_item TYPE mepoitem.
      DATA: lt_item TYPE STANDARD TABLE OF mepoitem.

      ls_header = im_header->get_data( ).

      IMPORT lt_item TO lt_item FROM  MEMORY ID 'LV_ITEM'.  "Import Item Details From ProcessItem methods
*      IMPORT ls_item TO ls_item FROM  MEMORY ID 'LV_ITEM'.  "Import Item Details From ProcessItem methods

      LOOP AT lt_item INTO ls_item.

        " Skip deleted or cancelled items
        IF ls_item-loekz = 'L' OR ls_item-loekz = 'D'.
          CONTINUE.
        ENDIF.

        IF ls_item-infnr IS NOT INITIAL AND ls_header-lifnr IS NOT INITIAL AND ls_item-matnr IS NOT INITIAL.
          SELECT SINGLE * FROM zpir_warranty INTO @DATA(ls_pirwar) WHERE  lifnr = @ls_header-lifnr AND pt_no = @ls_item-matnr.
          IF sy-subrc NE 0.
            CONCATENATE ls_item-matnr 'and Vendor'  ls_header-lifnr INTO ls_msg SEPARATED BY space.
            MESSAGE e899(m3) WITH 'Warranty data not available' '-Entry Not Found In ZPIR_WARRANTY' 'For The Specified Material' ls_msg.
            LEAVE LIST-PROCESSING.
            EXIT.
          ENDIF.
        ENDIF.
        CLEAR : ls_item.
      ENDLOOP.
* FREE MEMORY ID 'LV_ITEM'.
    ENDIF.

Implementation Short Text     Validation forTotal Bundle Qty. vs Goodsmvt. Qty thr 101 Mvt
Definition Name               WORKORDER_GOODSMVT

Implementation Short Text     order to order creation when order confirmation.
Definition Name               WORKORDER_CONFIRM

Implementation Short Text     Compare Issue Quantity with Reservation Quantity
Definition Name               MB_MIGO_BADI

Implementation Short Text     Badi to Restrict PO confirmation without std cost
Definition Name               MB_MIGO_BADI

Implementation Short Text     ERROR MESSAGE FOR 343/4 MOV TYPES
Definition Name               MB_MIGO_BADI

Implementation Short Text     Add Custom Tab In Migo Transaction
Definition Name               MB_MIGO_BADI

Implementation Short Text     Block 343/344 in MIGO and MB1B
Definition Name               MB_CHECK_LINE_BADI

Implementation Short Text     Enhancement for MB1C T Code
Definition Name               MB_CHECK_LINE_BADI

METHOD if_ex_mb_migo_badi~post_document.
*****************************************************************************
* Date              : 02/06/2016
* Requestor         : Satish Mallina
* Programmer        : Anil A Anchan
* Transport Request : IN1K906671,IN1K906848
*****************************************************************************
*Short Description  : Restrict PO confirmatn without standard  cost estimate
*****************************************************************************

  TYPES: BEGIN OF ty_keko,
          matnr   TYPE matnr,                     " Material
          kadat   TYPE ck_abdat,                  " Costing Date From
          bidat   TYPE ck_bidat,                  " Costing Date To
          feh_sta TYPE ck_feh_sta,                " Costing Status
        END OF ty_keko.

  DATA: lt_keko TYPE STANDARD TABLE OF ty_keko,
        ls_keko TYPE ty_keko.

  DATA: wa_mseg  TYPE mseg,
        lv_bsart TYPE ekko-bsart,        " Material Type
        ls_cust  TYPE zxx_generic_data.

  IF is_mkpf-blart EQ 'WE'.   " WE - Goods Receipt
    IF it_mseg[] IS NOT INITIAL.    " check item availability

      LOOP AT it_mseg INTO wa_mseg.

        IF wa_mseg-kzzug IS INITIAL.  " Receipt Indicator

          IF wa_mseg-ebeln IS NOT INITIAL.
*-- Purchasing Document Type
            SELECT SINGLE bsart
               FROM ekko
               INTO lv_bsart
               WHERE ebeln EQ wa_mseg-ebeln.

            IF lv_bsart IS NOT INITIAL.
*-- Custom data
              SELECT SINGLE *
                 FROM zxx_generic_data
                 INTO ls_cust
                 WHERE idnt1 EQ 'RESTRICT'
                 AND   idnt2 EQ 'MIGO'
                 AND   idnt3 EQ 'KEKO'
                 AND   activ EQ 'X'
                 AND   flval EQ lv_bsart.   "Purchasing Document Type
*If maintained
              IF sy-subrc EQ 0 AND ls_cust-flval IS NOT INITIAL.

                SELECT matnr                       " Material
                       kadat                       " Costing Date From
                       bidat                       " Costing Date To
                       feh_sta                     " Costing Status
                   FROM keko
                   INTO TABLE lt_keko
                   WHERE matnr   EQ wa_mseg-matnr
                   AND   werks   EQ wa_mseg-werks
                   AND   feh_sta EQ 'FR'.          " RF - Released Without Errors

                IF sy-subrc EQ 0.   "Check if costing done
                  DELETE lt_keko WHERE bidat LT is_mkpf-budat.
*                  DELETE lt_keko WHERE kadat GT sy-datum.
                  IF lt_keko[] IS NOT INITIAL.
                  ELSE.
                    MESSAGE e064(zpp_msg) WITH wa_mseg-matnr. " Std. Cost estimate not released for the material
                  ENDIF.
                ELSE.
                  MESSAGE e064(zpp_msg) WITH wa_mseg-matnr.   " Std. Cost estimate not released for the material
                ENDIF.
              ENDIF.
            ENDIF.
          ENDIF.
        ENDIF.
      ENDLOOP.
    ENDIF.
  ENDIF.
ENDMETHOD.

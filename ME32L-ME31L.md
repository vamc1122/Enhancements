Function Exit : EXIT_SAPMM06E_012
INCLUDE ZXM06U43


*&---------------------------------------------------------------------*
*& Include          ZXM06U43
*&---------------------------------------------------------------------*
************************************************************************
* Author: Amit R                            Create Date:04/07/2023     *
*----------------------------------------------------------------------*
* PURPOSE:                                                             *
* This include code is to validate the PO line item details while      *
* or modifying the PO line item                                        *
*----------------------------------------------------------------------*
* CHANGE LOG:                                                          *
* Date         Author      Correction    Description                   *
*-------       ------      ----------    -----------                   *
* 04/07/2023  APLUS02     SHDK912289    Initial Development            *
* 06/01/2024  APLUS05     SHDK918963    Added FirmZone and Netprice    *
*                                                   Validation         *
* 11-06-2025  APLUS05     SHDK922112    PIR Vendor Validation in       *
*                                       ZPIR_WARRANTY table            *
*----------------------------------------------------------------------*
DATA: lt_ekpo TYPE TABLE OF bekpo.

CONSTANTS : lc_tcode1 TYPE syst_tcode VALUE 'ME31L',
            lc_tcode2 TYPE syst_tcode VALUE 'ME32L'.


BREAK aplus02.
** I_TRTYP Values
"A   Display
"V   Change
"H   Create

IF ( i_trtyp = 'H' OR i_trtyp = 'V' ). "sy-tcode CS 'ME2' AND
*IF sy-tcode = 'ME21N' OR sy-tcode = 'ME22N' OR sy-tcode = 'ME23N'.
  IF i_ekko IS NOT INITIAL AND tekpo[] IS NOT INITIAL.
    IF i_ekko-bedat GE '20230805'.
      LOOP AT tekpo[] INTO DATA(wa_ekpo).

        IF i_ekko-bsart = 'ZCAP'. "AND wa_ekpo-werks <> '1000'.
*        MESSAGE 'Please enter the correct plant for PO Type ZCAP' TYPE 'E'.

        ELSEIF i_ekko-bsart = 'ZDOM'.
          IF wa_ekpo-pstyp = '0' OR wa_ekpo-pstyp = '' OR wa_ekpo-pstyp = '3'.
          ELSE.
            MESSAGE 'Item category should be blank or L' TYPE 'E'.
          ENDIF.

          IF wa_ekpo-knttp <> ''.
            MESSAGE 'Account Assignment Category should be blank' TYPE 'E'.
          ENDIF.
          IF wa_ekpo-infnr IS INITIAL AND wa_ekpo-matnr IS NOT INITIAL.
            MESSAGE 'Please maintain the info record' TYPE 'E'.
          ENDIF.

          SELECT SINGLE land1 FROM lfa1 INTO @DATA(lv_land1)
            WHERE lifnr = @i_ekko-lifnr.
          IF lv_land1 <> 'IN'.
            MESSAGE 'Vendor is not belonging to Country IN' TYPE 'E'.
          ENDIF.

        ELSEIF i_ekko-bsart = 'ZDRD'.

          SELECT SINGLE land1 FROM lfa1 INTO @lv_land1
            WHERE lifnr = @i_ekko-lifnr.
          IF lv_land1 <> 'IN'.
            MESSAGE 'Vendor is not belonging to Country IN' TYPE 'E'.
          ENDIF.

        ELSEIF i_ekko-bsart = 'ZFRM'.

          SELECT SINGLE land1 FROM lfa1 INTO @lv_land1
            WHERE lifnr = @i_ekko-lifnr.
          IF lv_land1 <> 'IN'.
            MESSAGE 'Vendor is not belonging to Country IN' TYPE 'E'.
          ENDIF.

        ELSEIF i_ekko-bsart = 'ZIMP'.

          SELECT SINGLE land1 FROM lfa1 INTO @lv_land1
            WHERE lifnr = @i_ekko-lifnr.
          IF lv_land1 = 'IN'.
            MESSAGE 'Vendor is belonging to Country IN' TYPE 'E'.
          ENDIF.

          IF wa_ekpo-infnr IS INITIAL.
            MESSAGE 'Please maintain the info record' TYPE 'E'.
          ENDIF.

        ELSEIF i_ekko-bsart = 'ZIRD'.

          SELECT SINGLE land1 FROM lfa1 INTO @lv_land1
            WHERE lifnr = @i_ekko-lifnr.
          IF lv_land1 = 'IN'.
            MESSAGE 'Vendor is belonging to Country IN' TYPE 'E'.
          ENDIF.

**--BOC for BSOFT-23584
        ELSEIF i_ekko-bsart = 'ZISR'.
          IF wa_ekpo-pstyp <> '9'.
            MESSAGE 'Item category should be D' TYPE 'E'.
          ENDIF.
          SELECT SINGLE land1 FROM lfa1 INTO @lv_land1
            WHERE lifnr = @i_ekko-lifnr.
          IF lv_land1 = 'IN'.
            MESSAGE 'Vendor is belonging to Country IN' TYPE 'E'.
          ENDIF.
**--EOC for BSOFT-23584

        ELSEIF i_ekko-bsart = 'ZSER'.

          IF wa_ekpo-pstyp <> '9'.
            MESSAGE 'Item category should be D' TYPE 'E'.
          ENDIF.

          IF wa_ekpo-knttp = ''.
            MESSAGE 'Account Assignment Category should not be blank' TYPE 'E'.
          ENDIF.

          SELECT SINGLE land1 FROM lfa1 INTO @lv_land1
            WHERE lifnr = @i_ekko-lifnr.
          IF lv_land1 <> 'IN'.
            MESSAGE 'Vendor is not belonging to Country IN' TYPE 'E'.
          ENDIF.
* BoC BSOFT-24730 SHDK914604
        ELSEIF i_ekko-bsart = 'ZCRD'.
          SELECT SINGLE land1 FROM lfa1 INTO @lv_land1
           WHERE lifnr = @i_ekko-lifnr.
          IF lv_land1 = 'IN'.
            MESSAGE 'Vendor is belonging to Country IN' TYPE 'E'.
          ENDIF.
* EoC BSOFT-24730
        ENDIF.

        CLEAR: wa_ekpo.
      ENDLOOP.
    ENDIF.
  ENDIF.

**--BOC for BSOFT-17346
********** Start BudgetID , CC & PC Validateion ********
*  IF i_ekko IS NOT INITIAL AND tekkn[] IS NOT INITIAL..
*
*    DATA e_fy   TYPE char4.
*
*    CALL FUNCTION 'GM_GET_FISCAL_YEAR'
*      EXPORTING
*        i_date                     = i_ekko-bedat
*        i_fyv                      = 'V3'
*      IMPORTING
*        e_fy                       = e_fy
*      EXCEPTIONS
*        fiscal_year_does_not_exist = 1
*        not_defined_for_date       = 2.
*
*    SELECT SINGLE * FROM zbudget_idlist INTO @DATA(wa_budid)
*      WHERE budgetid = @i_ekko-zbudgetid
*      AND gjahr = @e_fy.
*
*    LOOP AT tekkn INTO DATA(wa_ekkn).
*      IF wa_ekkn-kostl = wa_budid-budg_cc AND wa_ekkn-sakto = wa_budid-hkont.
*      ELSE.
*        CONCATENATE 'Maintain the budget relevant profit center and cost center for the line item'
*        wa_ekkn-ebelp INTO DATA(lv_msg) SEPARATED BY space.
*        MESSAGE lv_msg TYPE 'E'.
*      ENDIF.
*    ENDLOOP.
*
*  ENDIF.
**--EOC for BSOFT-17346
**********End  BudgetID , CC & PC Validation ********
ENDIF.

"" BOC SHDK918963    Vamsi   NC: 2000003041 Frim Zone Mandt
IF sy-tcode = lc_tcode1 OR sy-tcode = lc_tcode2.
  IF sy-ucomm = 'BU' OR sy-ucomm = 'YES'.
    REFRESH : lt_ekpo.
    MOVE tekpo[] TO lt_ekpo.
    LOOP AT lt_ekpo INTO DATA(ls_ekpo1).
      IF ls_ekpo1-loekz EQ 'L' OR ls_ekpo1-loekz EQ 'S'.
      ELSE.
        IF ls_ekpo1-bstae = '0001' AND ls_ekpo1-etfz1 LT 1.
          MESSAGE e899(m3) WITH 'FirmZone Cannot Be Less Than 1 For ConfContr.' '0001 In The Line Item' ls_ekpo1-ebelp.
*    FrimZone cannot be less than 1 against line item
          LEAVE TO LIST-PROCESSING.
          EXIT.
        ENDIF.

        IF ls_ekpo1-netpr IS NOT INITIAL.
          MESSAGE e000(e4) WITH 'Net Price Should Be Zero For the Item' ls_ekpo1-ebelp.
          LEAVE TO LIST-PROCESSING.
          EXIT.
        ENDIF.

      ENDIF.
      CLEAR : ls_ekpo1.
    ENDLOOP.
  ENDIF.
ENDIF.
""EOC SHDK918963  vamsi 06.01.2025



""BOC Validation for PIR check Vendor in ZPIR_WARRANTY table     SHDK922112 | 11-06-2025 | vamsi



"" eoc

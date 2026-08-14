BTE ZZFI_INTERFACE_00001011 -> FB*
MIRO,MIR7,MIR4 MRM_HEADER_CHECK
  
  METHOD if_ex_badi_fdcb_subbas01~put_data_to_screen_object.
* fill interface attributes from importing paramters
    me->if_ex_badi_fdcb_subbas01~invfo  = im_invfo.
** added by Vinoth Balaji on 30.08.2024 to restric user's invoice date selections in FB60, FB65, FB70, FB75
    DATA(lv_date) = sy-datum - 730.
    IF ( sy-tcode = 'MIRO' OR sy-tcode = 'FB60' OR sy-tcode = 'FB65' OR sy-tcode = 'FB70' OR sy-tcode = 'FB75' ) AND
       ( sy-ucomm = 'BU' OR sy-ucomm = 'PB' OR sy-ucomm = 'ABBR' OR sy-ucomm = 'BS' ).
      IF im_invfo-bldat > sy-datum.
        MESSAGE 'Check - Invoice date is in the future' TYPE 'W' DISPLAY LIKE 'E'.
      ELSEIF im_invfo-bldat < lv_date.
        MESSAGE 'Invoice date is in the past' TYPE 'W' DISPLAY LIKE 'E'.
      ENDIF.
    ENDIF.
    CLEAR : lv_date.
** added by Vinoth Balaji on 30.08.2024 to restric user's invoice date selections
  ENDMETHOD.

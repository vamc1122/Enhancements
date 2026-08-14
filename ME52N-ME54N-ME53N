Implementation Short Text     Modifications Account Assignment
Definition Name               ME_REQ_POSTED

Implementation Short Text     Implementing GOS_SRV_SELECT BADI
Definition Name               GOS_SRV_SELECT


METHOD if_ex_me_req_posted~posted.
*  BREAK-POINT.

  DATA : is_eban LIKE LINE OF im_eban.

  DATA : v_sakto TYPE c.

  IF sy-tcode = 'ME52N' OR
     sy-tcode = 'ME54N' .
    LOOP AT  im_eban INTO is_eban.

      IF  is_eban-banfn IS NOT INITIAL  AND
          is_eban-frgkz IS NOT INITIAL  AND
          is_eban-frgzu IS NOT INITIAL   .

        v_sakto = 'X'.

      ENDIF.

    ENDLOOP.


    IF v_sakto = 'X'.

      DATA : v_n TYPE c.

      LOOP AT  SCREEN.
        v_n = 1.
        IF screen-name = 'OK-CODE'.
          CLEAR v_n.
          v_n = 1.
          screen-active = 1.
*          screen-input = 0.
          MODIFY SCREEN  .
        ENDIF.

      ENDLOOP.
    ENDIF
        .

  ENDIF.
ENDMETHOD.

---
title: 'GitLab API via ABAP'
categories:
  - programming
tags:  
  - sap
  - abap
  - gitlab
summary:
  - image: sap.gif  
---

GitLab offers a nice API to work with Git without having the Git runtime/tools installed. This becomes handy when trying to access a Git repository from ABAP code.

In our [Weblate integration](/sap-translation-with-weblate) we needed the option to transfer generated translation files to/from GitLab. To achieve this, we implemented several API endpoints of the GitLab API in an ABAP class:

## Prerequisites

To access the GitLab API via HTTP you may need to setup the GitLab server via transaction `SM59` if you don't want to insert the server name via literals/constants in your ABAP code. We've decided to provide the options to work with multiple servers without `SM59` (hostnames only).

For an anonymous access to the GitLab API you need to setup a [personal access token](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html) which can be used with the HTTP call from ABAP.

## Class structure

We've created an ABAP class to handle all requests to the GitLab API. Inside the class we've the following global fields:

```abap
CONSTANTS: gc_api_prefix TYPE string VALUE '/api/v4/'.

DATA: gv_host_name       TYPE string,
      gv_repository_name TYPE string,
      gv_branch_name     TYPE string,
      gv_auth_token      TYPE string.
```

These fields get initialized by the constructor call. For example:

```abap
DATA(lo_gitlab) = NEW zcl_gitlab( iv_host_name = <GITLAB_HOST>
                                  iv_repository_name = <GITLAB_REPO>
                                  iv_branch_name = 'master'
                                  iv_auth_token = <GITLAB_AUTH_TOKEN> ).
```

## File listing

To get a list of all files in a repository the endpoint [List repository tree](https://docs.gitlab.com/ee/api/repositories.html#list-repository-tree) can be used. A method using this endpoint could look like:

```abap
METHOD get_file_listing.
  TYPES: BEGIN OF t_file,
           path TYPE string,
           name TYPE string,
           type TYPE string,
         END OF t_file.

  CONSTANTS:  lc_service_path TYPE string VALUE 'projects/{id}/repository/tree?ref={branch}'.

  DATA: lt_files TYPE TABLE OF t_file.

  " prepare the service path for the given project
  DATA(lv_service_path) = |{ gc_api_prefix }{ lc_service_path }|.
  REPLACE ALL OCCURRENCES OF '{id}'
    IN lv_service_path
    WITH escape( val = gv_repository_name format = cl_abap_format=>e_url_full ).
  REPLACE ALL OCCURRENCES OF '{branch}'
    IN lv_service_path
    WITH escape( val = gv_branch_name format = cl_abap_format=>e_url_full ).

  IF iv_subfolder IS NOT INITIAL.
    lv_service_path = |{ lv_service_path }&path={ escape( val = iv_subfolder format = cl_abap_format=>e_url_full ) }|.
  ENDIF.

  " create the http client
  cl_http_client=>create( EXPORTING host = gv_host_name scheme = cl_http_client=>schemetype_https
                          IMPORTING client = lo_http_client
                          EXCEPTIONS OTHERS = 4 ).
  IF sy-subrc NE 0. RETURN. ENDIF.

  cl_http_utility=>set_request_uri( request = lo_http_client->request uri = lv_service_path ).

  IF gv_auth_token IS NOT INITIAL.
    lo_http_client->request->set_header_field( name = 'PRIVATE-TOKEN' value = gv_auth_token ).
  ENDIF.

  lo_http_client->send( EXCEPTIONS  OTHERS = 4 ).
  IF sy-subrc NE 0.
    " get the error message
    lo_http_client->get_last_error( IMPORTING message = DATA(lv_send_message) ).

    RAISE EXCEPTION TYPE zcx_gitlab
      EXPORTING
        textid = VALUE #( msgid = '00' msgno = '001' attr1 = lv_send_message ).
  ENDIF.

  lo_http_client->receive( EXCEPTIONS OTHERS = 4 ).
  IF sy-subrc NE 0.
    " get the error message
    lo_http_client->get_last_error( IMPORTING message = DATA(lv_receive_message) ).

    RAISE EXCEPTION TYPE zcx_gitlab
      EXPORTING
        textid = VALUE #( msgid = '00' msgno = '001' attr1 = lv_receive_message ).
  ENDIF.

  DATA(lv_response) = lo_http_client->response->get_cdata( ).
  lo_http_client->close( ).

  " if the result contains "message": * there is a problem
  IF lv_response CS '"message"'.
    RAISE EXCEPTION TYPE zcx_gitlab
      EXPORTING
        textid = VALUE #( msgid = '00' msgno = '001' attr1 = lv_response ).
  ENDIF.

  " now unmarshall the JSON response
  /ui2/cl_json=>deserialize( EXPORTING json = lv_response
                              CHANGING data = lt_files ).

  LOOP AT lt_files ASSIGNING FIELD-SYMBOL(<file>) WHERE type NE 'tree'.
    APPEND INITIAL LINE TO rt_files
      ASSIGNING FIELD-SYMBOL(<return>).

    <return>-path_and_name = <file>-path.
    <return>-file_name = <file>-name.
  ENDLOOP.

ENDMETHOD.
```

## Receive file

To receive files via the API the endpoint [Get file from repository](https://docs.gitlab.com/ee/api/repository_files.html#get-file-from-repository) can be used. Our method implementing the endpoint looks like:

```abap
METHOD receive_file.
  TYPES: BEGIN OF t_file,
           file_name TYPE string,
           file_path TYPE string,
           size      TYPE i,
           encoding  TYPE string,
           content   TYPE string,
         END OF t_file.

  CONSTANTS: lc_service_path TYPE string VALUE 'projects/{id}/repository/files/{filename}?ref={branch}'.

  DATA: ls_file TYPE t_file.

  " prepare the service path for the given project
  DATA(lv_service_path) = |{ gc_api_prefix }{ lc_service_path }|.
  REPLACE ALL OCCURRENCES OF '{id}'
    IN lv_service_path
    WITH escape( val = gv_repository_name format = cl_abap_format=>e_url_full ).
  REPLACE ALL OCCURRENCES OF '{filename}'
    IN lv_service_path
    WITH escape( val = iv_filename format = cl_abap_format=>e_url_full ).
  REPLACE ALL OCCURRENCES OF '{branch}'
    IN lv_service_path
    WITH escape( val = gv_branch_name format = cl_abap_format=>e_url_full ).

  " create the http client
  cl_http_client=>create( EXPORTING host = gv_host_name scheme = cl_http_client=>schemetype_https
                          IMPORTING client = DATA(lo_http_client)
                         EXCEPTIONS OTHERS = 4 ).
  IF sy-subrc NE 0. RETURN. ENDIF.

  cl_http_utility=>set_request_uri( request = lo_http_client->request uri= lv_service_path ).

  IF gv_auth_token IS NOT INITIAL.
    lo_http_client->request->set_header_field( name  = 'PRIVATE-TOKEN' value = gv_auth_token ).
  ENDIF.

  lo_http_client->send( EXCEPTIONS  OTHERS = 4 ).
  IF sy-subrc NE 0.
    " get the error message
    lo_http_client->get_last_error( IMPORTING message = DATA(lv_send_message) ).

    RAISE EXCEPTION TYPE zcx_gitlab
      EXPORTING
        textid = VALUE #( msgid = '00' msgno = '001' attr1 = lv_send_message ).
  ENDIF.

  lo_http_client->receive( EXCEPTIONS OTHERS = 4 ).
  IF sy-subrc NE 0.
    " get the error message
    lo_http_client->get_last_error( IMPORTING message = DATA(lv_receive_message) ).

    RAISE EXCEPTION TYPE zcx_gitlab
      EXPORTING
        textid = VALUE #( msgid = '00' msgno = '001' attr1 = lv_receive_message ).
  ENDIF.

  DATA(lv_response) = lo_http_client->response->get_cdata( ).
  lo_http_client->close( ).

  " if the result contains "message": * there is a problem
  IF lv_response CS '"message"'.
    RAISE EXCEPTION TYPE zcx_gitlab
      EXPORTING
        textid = VALUE #( msgid = '00' msgno = '001' attr1 = lv_response ).
  ENDIF.

  " now unmarshall the JSON response
  /ui2/cl_json=>deserialize( EXPORTING json = lv_response
                              CHANGING data = ls_file ).

  " only accept base64 encoded files
  IF ls_file-encoding NE 'base64'.
    RAISE EXCEPTION TYPE zcx_gitlab
      EXPORTING
        textid = VALUE #( msgid = '00' msgno = '001' attr1 = 'no BASE64 encoded response' ).
  ENDIF.

  " decode the base64 encoded file
  rv_file_content = cl_http_utility=>decode_x_base64( ls_file-content ).
ENDMETHOD.    
```

## Send file

Sending files to the GitLab API is a bit more complicated than receiving. There are two different API endpoints: [Create new file in repository](https://docs.gitlab.com/ee/api/repository_files.html#create-new-file-in-repository) and [Update existing file in repository](https://docs.gitlab.com/ee/api/repository_files.html#update-existing-file-in-repository). Since there is no endpoint like "create or update" we need to check whether the file exists before we actually send it.

For this check we've implemented an own method:

```abap
METHOD prepare_send_files.

  DATA: lv_path TYPE filep.
  DATA: ls_file TYPE t_file_send_internal.
  DATA: lt_file_listing TYPE tt_file_listing,
        lt_paths        TYPE TABLE OF filep.

  " prepare the given files
  LOOP AT it_files ASSIGNING FIELD-SYMBOL(<import>) WHERE path_and_name IS NOT INITIAL
                                                      AND content IS NOT INITIAL.
    ls_file-filename = <import>-path_and_name.
    ls_file-content_base64 = cl_http_utility=>encode_x_base64( <import>-content ).
    APPEND ls_file
      TO rt_files.

    " extract the path portion for the existing check later
    CLEAR lv_path.
    CALL FUNCTION 'CV120_SPLIT_PATH'
      EXPORTING
        pf_path  = CONV char1024( <import>-path_and_name )
      IMPORTING
        pfx_path = lv_path.
    APPEND lv_path
      TO lt_paths.
  ENDLOOP.

  CHECK rt_files IS NOT INITIAL.
  SORT lt_paths.
  DELETE ADJACENT DUPLICATES FROM lt_paths.

  " check whether the file already exists or not
  LOOP AT lt_paths ASSIGNING FIELD-SYMBOL(<path>).
    APPEND LINES OF get_file_listing( iv_subfolder = CONV #( <path> ) )
      TO lt_file_listing.
  ENDLOOP.

  LOOP AT rt_files ASSIGNING FIELD-SYMBOL(<file>).
    READ TABLE lt_file_listing WITH KEY path_and_name = <file>-filename
                               TRANSPORTING NO FIELDS.
    IF sy-subrc EQ 0.
      <file>-action = 'update'.
    ELSE.
      <file>-action = 'create'.
    ENDIF.

  ENDLOOP.

ENDMETHOD.
```

Having this helper method allows us to use a single method to send files to the GitLab repository:

```abap
METHOD send_files.
  CONSTANTS: lc_service_path TYPE string VALUE 'projects/{id}/repository/commits'.

  TYPES: BEGIN OF t_action,
           action    TYPE string,
           file_path TYPE string,
           content   TYPE string,
           encoding  TYPE string,
         END OF t_action.

  TYPES: tt_action TYPE STANDARD TABLE OF t_action
                  WITH DEFAULT KEY.

  TYPES: BEGIN OF t_body,
           branch         TYPE string,
           commit_message TYPE string,
           actions        TYPE tt_action,
         END OF t_body.

  DATA: lv_body TYPE string.
  DATA: ls_body   TYPE t_body,
        ls_action TYPE t_action.

  " prepare the files to be send
  DATA(lt_files) = prepare_send_files( it_files ).

  " build the POST body
  ls_body-branch = gv_branch_name.
  ls_body-commit_message = iv_commit_message.

  LOOP AT lt_files ASSIGNING FIELD-SYMBOL(<file>).
    CLEAR: ls_action.

    ls_action-action = <file>-action.
    ls_action-file_path = <file>-filename.
    ls_action-content = <file>-content_base64.
    ls_action-encoding = 'base64'.
    APPEND ls_action
      TO ls_body-actions.

  ENDLOOP.

  lv_body = /ui2/cl_json=>serialize( data = ls_body pretty_name = /ui2/cl_json=>pretty_mode-low_case ).

  " prepare the service path for the given project
  DATA(lv_service_path) = |{ gc_api_prefix }{ lc_service_path }|.
  REPLACE ALL OCCURRENCES OF '{id}'
    IN lv_service_path
    WITH escape( val = gv_repository_name format = cl_abap_format=>e_url_full ).

  " create the http client
  cl_http_client=>create( EXPORTING host = gv_host_name scheme = cl_http_client=>schemetype_https
                          IMPORTING client = DATA(lo_http_client)
                         EXCEPTIONS OTHERS = 4 ).
  IF sy-subrc NE 0. RETURN. ENDIF.

  cl_http_utility=>set_request_uri( request = lo_http_client->request uri = lv_service_path ).

  IF gv_auth_token IS NOT INITIAL.
    lo_http_client->request->set_header_field( name  = 'PRIVATE-TOKEN' value = gv_auth_token ).
  ENDIF.

  lo_http_client->request->set_method( method = if_http_request=>co_request_method_post ).
  lo_http_client->request->set_header_field( name  = 'Content-Type' value = 'application/json' ).
  lo_http_client->request->set_cdata( data = lv_body ).

  lo_http_client->send( EXCEPTIONS  OTHERS = 4 ).
  IF sy-subrc NE 0.
    " get the error message
    lo_http_client->get_last_error( IMPORTING message = DATA(lv_send_message) ).

    RAISE EXCEPTION TYPE zcx_gitlab
      EXPORTING
        textid = VALUE #( msgid = '00' msgno = '001' attr1 = lv_send_message ).
  ENDIF.

  lo_http_client->receive( EXCEPTIONS OTHERS = 4 ).
  IF sy-subrc NE 0.
    " get the error message
    lo_http_client->get_last_error( IMPORTING message = DATA(lv_receive_message) ).

    RAISE EXCEPTION TYPE zcx_gitlab
      EXPORTING
        textid = VALUE #( msgid = '00' msgno = '001' attr1 = lv_receive_message ).
  ENDIF.

  DATA(lv_response) = lo_http_client->response->get_cdata( ).
  lo_http_client->close( ).

ENDMETHOD.
```

With this toolbox of methods you are able to send/receive any sort of files in ABAP.

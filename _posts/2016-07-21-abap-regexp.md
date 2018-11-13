---
title: "Regular expressions in ABAP"
categories:
  - programming
tags:
  - sap
  - abap
summary:
  - image: sap.gif
---
Regular expressions is a common technique to parse or validate strings in various programming languages.

Regular expressions in ABAP are available for some statements like `FIND` and `REPLACE`:

```abap
FIND ALL OCCURRENCES OF REGEX lv_regexp
  IN lv_field
  SUBMATCHES lv_sub1 lv_sub2.

REPLACE ALL OCCURRENCES OF REGEX lv_regexp
  IN lv_input
  WITH lv_replacement.
```  

Since the release of ABAP 6.40 the classes `CL_ABAP_REGEX` and `CL_ABAP_MATCHER` are available which provide a Java like experience with regular expressions:

```abap
DATA:  lv_value TYPE string VALUE '1234567890'.

DATA:  lo_regexp  TYPE REF TO cl_abap_regex,
       lo_matcher TYPE REF TO cl_abap_matcher.

CREATE OBJECT lo_regexp
  EXPORTING
    pattern     = '[A-Z]'
    ignore_case = abap_true.

lo_matcher = lo_regex->create_matcher( text =  lv_value ).

IF lo_matcher->match( ) EQ abap_true.
  WRITE:/ 'Its a match'.

ELSE.
  WRITE:/ 'Not a match'.

ENDIF.
```

Testing regular expressions for ABAP is also super easy with the demo program `DEMO_REGEX_TOY`:

![DEMO_REGEX_TOY](/images/2016/07/demo_regex_toy.png){: .align-center}

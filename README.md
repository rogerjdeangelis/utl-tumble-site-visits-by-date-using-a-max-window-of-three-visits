# utl-tumble-site-visits-by-date-using-a-max-window-of-three-visits
    %let pgm=utl-tumble-site-visits-by-date-using-a-max-window-of-three-visits;

    Tumble site visits by date using a max window of three visits

    'I want to add a column labeled n_visit. However I want to only label n_visit per
    site out to the 3rd visit. Once it has reached the 3rd visits I want the label
    to start again going from 1_visit, 2_visit, 3_visit per site based on date.
    Essentially I want to know the date window in which 1-3_vistis occurred per site.'


        1. SAS
        2. R
           https://stackoverflow.com/users/3358272/r2evans
           Input not easily converted to SAS dataset
           (sas datsset does not have row.names or factors)


    github
    https://tinyurl.com/umx7usme
    https://github.com/rogerjdeangelis/utl-tumble-site-visits-by-date-using-a-max-window-of-three-visits

    stackoverflow R
    https://stackoverflow.com/questions/77053562/label-visit-number-based-on-date

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    options validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.have;
    data sd1.have;
     informat site $7. date $10.;
     input site date ;
    cards4;
    tryon 2023-03-31
    phelps 2023-03-31
    phelps 2023-04-05
    admin 2023-04-20
    wood_lab 2023-05-03
    admin 2023-05-10
    phelps 2023-05-11
    wood_lab 2023-05-15
    wood_lab 2023-05-16
    rv 2023-05-22
    rv 2023-05-23
    rv 2023-05-24
    tuttle 2023-05-29
    tuttle 2023-05-30
    vorisek 2023-05-30
    tuttle 2023-05-31
    vorisek 2023-05-31
    vorisek 2023-06-01
    rv 2023-06-05
    tryon 2023-06-05
    tryon 2023-06-06
    rv 2023-06-06
    tryon 2023-06-07
    rv 2023-06-07
    ;;;;
    run;quit;

    proc sort data=sd1.have out=doc;
    by site date;
    run;quit;

    /**************************************************************************************************************************/
    /*                               |                                                                                        */
    /* INPUT                         |                                                                                        */
    /*                               |                                                                                        */
    /* Sorted by site date for       |   RULES                                                                                */
    /* documentation purposes only.  |   Add N_Visit Column                                                                   */
    /*                               |                                                                                        */
    /* SD1.HAVE total obs=24         |   Limit visit sequences to max ot three. Tumble visits by threes.                      */
    /*                               |                                                                                        */
    /* Obs    SITE          DATE     |     N_VISIT                                                                            */
    /*                               |                                                                                        */
    /*   1    admin      2023-04-20  |     visit_1                                                                            */
    /*   2    admin      2023-05-10  |     visit_2  second visit same site (no more visits)                                   */
    /*                               |                                                                                        */
    /*   3    phelps     2023-03-31  |     visit_1  New site start sequence of 3                                              */
    /*   4    phelps     2023-04-05  |     visit_2                                                                            */
    /*   5    phelps     2023-05-11  |     visit_3                                                                            */
    /*                               |                                                                                        */
    /*   6    rv         2023-05-22  |     visit_1  New site start sequence of 3                                              */
    /*   7    rv         2023-05-23  |     visit_2                                                                            */
    /*   8    rv         2023-05-24  |     visit_3                                                                            */
    /*                               |                                                                                        */
    /*   9    rv         2023-06-05  |     visit_1 Reachede Max of 3 at same site start over with new sequence                */
    /*  10    rv         2023-06-06  |     visit_2                                                                            */
    /*  11    rv         2023-06-07  |     visit_3                                                                            */
    /*                               |                                                                                        */
    /*  12    tryon      2023-03-31  |     visit_1                                                                            */
    /*  13    tryon      2023-06-05  |     visit_2                                                                            */
    /*  14    tryon      2023-06-06  |     visit_3                                                                            */
    /*                               |                                                                                        */
    /*  15    tryon      2023-06-07  |     visit_1  Max of 3 visits start over with new sequence. Only 1 additional           */
    /*                               |                                                                                        */
    /*  16    tuttle     2023-05-29  |     visit_1                                                                            */
    /*  17    tuttle     2023-05-30  |     visit_2                                                                            */
    /*  18    tuttle     2023-05-31  |     visit_3                                                                            */
    /*                               |                                                                                        */
    /*  19    vorisek    2023-05-30  |     visit_1                                                                            */
    /*  20    vorisek    2023-05-31  |     visit_2                                                                            */
    /*  21    vorisek    2023-06-01  |     visit_3                                                                            */
    /*                               |                                                                                        */
    /*  22    wood_la    2023-05-03  |     visit_1                                                                            */
    /*  23    wood_la    2023-05-15  |     visit_2                                                                            */
    /*  24    wood_la    2023-05-16  |     visit_3                                                                            */
    /*                               |                                                                                        */
    /**************************************************************************************************************************/

    /*
    / |  ___  __ _ ___
    | | / __|/ _` / __|
    | | \__ \ (_| \__ \
    |_| |___/\__,_|___/

    */

    proc datasets lib=sd1 nolist nodetails;delete want; run;quit;

    /*---- Note macro triggers are resolved in the parent scope (dosubl)     ----*/

    %utl_submit_wps64x("
    libname sd1 'd:/sd1';

    Data sd1.want;
      retain n_visit 0;
      set sd1.havSrt %dosubl('proc sort data=sd1.have out=sd1.havSrt; by site date;run;quit;');
      by site;
      if first.site then n_visit = 1;
      else do;
         n_visit = n_visit + 1;
         n_visit = mod(n_visit,4);
      end;
      /*---- take care of recuring sequence at the same site (0 site)        ----*/
      if n_visit = 0 then n_visit = 1;
    run;quit;

    proc print data=sd1.want;
    run;quit;
    ");

    /*___    ____
    |___ \  |  _ \
      __) | | |_) |
     / __/  |  _ <
    |_____| |_| \_\

    */

    proc datasets lib=sd1 nolist nodetails;delete want; run;quit;

    options validvarname=any;
    libname sd1 "d:/sd1";
    %utl_submit_wps64x('
    libname sd1 "d:/sd1";
    proc r;
    export data=sd1.have r=have;
    submit;
    library(dplyr);
    raw <- structure(list(SITE = c("tryon_weber", "phelps_pond", "phelps_pond",
    "admin_pond", "wood_lab_pond", "admin_pond", "phelps_pond", "wood_lab_pond",
    "wood_lab_pond", "rv_pond", "rv_pond", "rv_pond", "tuttle_pond",
    "tuttle_pond", "vorisek_pond", "tuttle_pond", "vorisek_pond",
    "vorisek_pond", "rv_pond", "tryon_weber", "tryon_weber", "rv_pond",
    "tryon_weber", "rv_pond"), DATE = structure(c(19447, 19447, 19452,
    19467, 19480, 19487, 19488, 19492, 19493, 19499, 19500, 19501,
    19506, 19507, 19507, 19508, 19508, 19509, 19513, 19513, 19514,
    19514, 19515, 19515), class = "Date")), class = "data.frame", row.names = c(NA,
    -24L));
    want<-have %>%
      group_by(SITE) %>%
      mutate(
        n_visit = match(DATE, unique(DATE)),
        n_visit = paste0("visit_", (n_visit - 1) %% 3 + 1, sep = "")
      ) %>%
      ungroup();
    want;
    endsubmit;
    import data=sd1.want r=want;
    run;quit;
    proc print data=sd1.want width=min;
    run;quit;
    ');


    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* R                                                                                                                      */
    /*                                                                                                                        */
    /* The WPS System                                                                                                         */
    /*                                                                                                                        */
    /* # A tibble: 24 x 3                                                                                                     */
    /*    SITE    DATE       n_visit                                                                                          */
    /*    <chr>   <chr>      <chr>                                                                                            */
    /*  1 tryon   2023-03-31 visit_1                                                                                          */
    /*  2 phelps  2023-03-31 visit_1                                                                                          */
    /*  3 phelps  2023-04-05 visit_2                                                                                          */
    /*  4 admin   2023-04-20 visit_1                                                                                          */
    /*  5 wood_la 2023-05-03 visit_1                                                                                          */
    /*  6 admin   2023-05-10 visit_2                                                                                          */
    /*  7 phelps  2023-05-11 visit_3                                                                                          */
    /*  8 wood_la 2023-05-15 visit_2                                                                                          */
    /*  9 wood_la 2023-05-16 visit_3                                                                                          */
    /* 10 rv      2023-05-22 visit_1                                                                                          */
    /* # i 14 more rows                                                                                                       */
    /*                                                                                                                        */
    /* WPS                                                                                                                    */
    /*                                                                                                                        */
    /*     SITE         DATE       N_VISIT                                                                                    */
    /*                                                                                                                        */
    /*    tryon      2023-03-31    visit_1                                                                                    */
    /*    phelps     2023-03-31    visit_1                                                                                    */
    /*    phelps     2023-04-05    visit_2                                                                                    */
    /*    admin      2023-04-20    visit_1                                                                                    */
    /*    wood_la    2023-05-03    visit_1                                                                                    */
    /*    admin      2023-05-10    visit_2                                                                                    */
    /*    phelps     2023-05-11    visit_3                                                                                    */
    /*    wood_la    2023-05-15    visit_2                                                                                    */
    /*    wood_la    2023-05-16    visit_3                                                                                    */
    /*    rv         2023-05-22    visit_1                                                                                    */
    /*    rv         2023-05-23    visit_2                                                                                    */
    /*    rv         2023-05-24    visit_3                                                                                    */
    /*    tuttle     2023-05-29    visit_1                                                                                    */
    /*    tuttle     2023-05-30    visit_2                                                                                    */
    /*    vorisek    2023-05-30    visit_1                                                                                    */
    /*    tuttle     2023-05-31    visit_3                                                                                    */
    /*    vorisek    2023-05-31    visit_2                                                                                    */
    /*    vorisek    2023-06-01    visit_3                                                                                    */
    /*    rv         2023-06-05    visit_1                                                                                    */
    /*    tryon      2023-06-05    visit_2                                                                                    */
    /*    tryon      2023-06-06    visit_3                                                                                    */
    /*    rv         2023-06-06    visit_2                                                                                    */
    /*    tryon      2023-06-07    visit_1                                                                                    */
    /*    rv         2023-06-07    visit_3                                                                                    */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */

Tumble site visits by date using a max window of three visits 

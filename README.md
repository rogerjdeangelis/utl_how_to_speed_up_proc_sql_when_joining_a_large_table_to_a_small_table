# utl_how_to_speed_up_proc_sql_when_joining_a_large_table_to_a_small_table
How to speed up proc sql when joining a large table to a small table. Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverfl SAS community.
    How to speed up proc sql when joining a large table to a small table

    Selecting 20,000 observations out of 60,000,000

    github
    https://goo.gl/gRiXVB
    https://github.com/rogerjdeangelis/utl_how_to_speed_up_proc_sql_when_joining_a_large_table_to_a_small_table


    https://goo.gl/i3UyYw
    https://communities.sas.com/t5/SAS-Procedures/How-to-speed-up-proc-sql-when-joining-a-large-datset-to-a-small/m-p/440399/highlight/true#M69210

    For tiny data like this I like to use SQL, but the hash is faster

    No where neer 10 minutes on my boat anchor (2008 desktop DDR2 ram)

      Benchmarks
            1. SAS HASH 35 seconds
            2. WPS HASH 37 seconds

            3. SAS SQL  49 seconds
            4. WPS SQL 115 seconds (not sure my wps config file is optimum - took a lot of CPU)

    Ops explanation

    Where the dataset eq_securityId consists of ~16000 observations (here denoted a small dataset)
    and the dataset FOUNDATION_SECURITY consist of ~60 000 000 observations (here denoted a large dataset).

    As of now the runtime to execute this command is about 10min, is there anything I can do to
    speed up the sql-step? e.g, is it in general faster to join a small dataset on to a large
    dataset instead of me joining the large dataset onto the small dataset? could some kind of
    indexation of the large dataset help me speeding up the computations?


    INPUTS   (60 million and 20,000)
    ======

     G.SMALL total obs=60,000,000

       Obs    NAME    COMPANYID     SECURITYID

         1    IBM     AB12345678         1
         2    IBM     AB12345678         2
         3    IBM     AB12345678         3
         4    IBM     AB12345678         4
         5    IBM     AB12345678         5
         6    IBM     AB12345678         6
         7    IBM     AB12345678         7
         8    IBM     AB12345678         8
         9    IBM     AB12345678         9

     D.TINY  Middle Observation(10000 ) of d.tiny - Total Obs 20,000

        --- KEY----
       SECURITYID      N8       29997001

        -- CHARACTER --
       A1              C8       12345678
       A2              C8       12345678
       A3              C8       12345678
       A18             C8       12345678
      ............
       A19             C8       12345678
       A20             C8       12345678


        -- NUMERIC --
       N1              N8       12345678
       N2              N8       12345678
       N3              N8       12345678
      ...........
       N18             N8       12345678
       N19             N8       12345678
       N20             N8       12345678


    PROCESS
    =======

      1. HASH 37 seconds

         data want;
           if _n_=1 then do;
             if 0 then set g.small ;
             declare hash bdata (dataset:'g.small');
               bdata.definekey('securityid');
               bdata.definedata(all:'Y');
               bdata.definedone();
           end;
           set d.tiny;
           rc=bdata.find();
           if rc=0;
           keep companyid name securityid;
         run;

         NOTE: There were 60000000 observations read from the data set G.SMALL.
         NOTE: There were 20000 observations read from the data set D.TINY.
         NOTE: The data set WORK.WANT has 20000 observations and 3 variables.
         NOTE: DATA statement used (Total process time):
               real time           35.24 seconds
               cpu time            35.25 seconds

      2. SQL  49 seconds

        proc sql;
           create table want as select
                   a.securityId
                   ,b.name
                   ,b.companyId
           from d.tiny as a
           left join g.small as b on a.securityId=b.securityId
        ;quit;


        NOTE: Table WORK.WANT created, with 20000 rows and 3 columns.

        68  !  quit;
        NOTE: PROCEDURE SQL used (Total process time):
              real time           48.53 seconds
              cpu time            1:45.51

    OUTPUT
    =====

      WORK.WANT Up to 40 obs from want total obs=20,000

         Obs    NAME    COMPANYID     SECURITYID

           1    IBM     AB12345678           1
           2    IBM     AB12345678        3001
           3    IBM     AB12345678        6001
           4    IBM     AB12345678        9001
           5    IBM     AB12345678       12001
           6    IBM     AB12345678       15001
           7    IBM     AB12345678       18001
           8    IBM     AB12345678       21001
           9    IBM     AB12345678       24001
          ....

    *                _              _       _
     _ __ ___   __ _| | _____    __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \  / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/ | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|  \__,_|\__,_|\__\__,_|

    ;


    libname g "c:/wrk";
    libname d "d:/wrk";

    * small dataset;
    data g.small;
       retain name "IBM"  companyid "AB12345678";
       do securityId=1 to 60000000 by 1;
             output;
       end;
    run;quit;

    * tiny dataset;
    libname d "d:/wrk";
    data d.tiny;
       array as[20] $8 a1-a20 (20*'12345678');
       array ns[20] n1-n20 (20*12345678);
       do securityId=1 to 60000000 by 3000;
             output;
       end;
    run;quit;

    *
     ___  __ _ ___
    / __|/ _` / __|
    \__ \ (_| \__ \
    |___/\__,_|___/

    ;

    data want;
      if _n_=1 then do;
        if 0 then set g.small ;
        declare hash bdata (dataset:'g.small');
          bdata.definekey('securityid');
          bdata.definedata(all:'Y');
          bdata.definedone();
      end;
      set d.tiny;
      rc=bdata.find();
      if rc=0;
      keep companyid name securityid;
    run;quit;

    proc sql;
       create table want as select
               a.securityId
               ,b.name
               ,b.companyId
       from d.tiny as a
       left join g.small as b on a.securityId=b.securityId
    ;quit;

    *
    __      ___ __  ___
    \ \ /\ / / '_ \/ __|
     \ V  V /| |_) \__ \
      \_/\_/ | .__/|___/
             |_|
    ;

    %utl_submit_wps64('
    libname g "c:/wrk";
    libname d "d:/wrk";
    libname wrk "%sysfunc(pathname(work))";
    data want;
      if _n_=1 then do;
        if 0 then set g.small ;
        declare hash bdata (dataset:"g.small");
          bdata.definekey("securityid");
          bdata.definedata(all:"Y");
          bdata.definedone();
      end;
      set d.tiny;
      rc=bdata.find();
      if rc=0;
      keep companyid name securityid;
    run;uit;
    ');

    The data step took :
    real time : 36.061
    cpu time  : 36.020


    %utl_submit_wps64('
    libname g "c:/wrk";
    libname d "d:/wrk";
    libname wrk "%sysfunc(pathname(work))";
    proc sql;
       create table want as select
           a.securityId
           ,b.name
           ,b.companyId
       from d.tiny as a
       left join g.small as b on a.securityId=b.securityId
    ;quit;
    ');

    NOTE: Procedure sql step took :
          real time : 1:55.135
          cpu time  : 7:00.329




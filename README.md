# utl-iterating-in-sql-assigning-a-unique-number-to-each-group-when-groups-repeat-and-are-not-sorted
Iterating in sql assigning a unique number to each group when groups repeat and are not sorted
    %let pgm=utl-iterating-in-sql-assigning-a-unique-number-to-each-group-when-groups-repeat-and-are-not-sorted;

    Iterating in sql assigning a unique number to each group when groups repeat and are not sorted

    This has other applications

    Simple in proceedural languge complex in SQL.
    Hope to bridge add to the SAS skillset by bridging to R and python.

    %stop_submission;
              Solutions
                 1 sas datastep
                 2 r sql long version and sort version
                 3 python sql
                 4 prolog
                 5 related repos

    The SQL technique showcases how to adapt procedural programming concepts to
    SQL, but it should be used judiciously, considering SQL's strengths in set-based
    operations.

    The solution describes a method for iterating through data in SQL, which is often not
    the most natural or efficient approach in SQL but can be useful in certain scenarios. Let's break
    down the key points:

    SAS SQL programmers who are familiar with SQL syntax may find it easier to transition to using
    R and Python by leveraging SQLite extensions.
    This approach allows them to utilize their existing SQL knowledge while working with SQLLITE.

    github
    https://tinyurl.com/y7f68ee8
    https://github.com/rogerjdeangelis/utl-iterating-in-sql-assigning-a-unique-number-to-each-group-when-groups-repeat-and-are-not-sorted

    /*               _     _
     _ __  _ __ ___ | |__ | | ___ _ __ ___
    | `_ \| `__/ _ \| `_ \| |/ _ \ `_ ` _ \
    | |_) | | | (_) | |_) | |  __/ | | | | |
    | .__/|_|  \___/|_.__/|_|\___|_| |_| |_|
    |_|
    */

    /**************************************************************************************************************************/
    /*                          |                                                                 |                           */
    /*       INPUT              |                       PROCESS                                   |        OUTPUT             */
    /*       =====              |                       =======                                   |        ======             */
    /*                          |                                                                 |                           */
    /*   SD1.HAVE total obs=12  | PROCESS SAS DATASTEP                                            | LTR GRP                   */
    /*                          | ====================                                            |                           */
    /*   Obs    LTR             |                                                                 |   S   1                   */
    /*                          | data want;                                                      |   S   1                   */
    /*     1     S              |   set sd1.have;                                                 |   S   1                   */
    /*     2     S              |   by ltr notsorted;                                             |   A   2                   */
    /*     3     S              |   if first.ltr then grp++1;                                     |   A   2                   */
    /*     4     A              | run;quit;                                                       |   S   3 Bump up 2nd S grp */
    /*     5     A              |                                                                 |   S   3                   */
    /*     6     S              |                                                                 |   Q   4                   */
    /*     7     S              | PROCESS R AND PYTHON SQL                                        |   Q   4                   */
    /*     8     Q              | ========================                                        |   R   5                   */
    /*     9     Q              |                                                                 |   Q   6                   */
    /*    10     R              |                                                                 |   Q   6                   */
    /*    11     Q              |  NUMBER THE ROWS (VIEW NUMBERED ROWS)                           |                           */
    /*    12     Q              |  ====================================                           |                           */
    /*                          |                                                                 |                           */
    /*                          |     LTR row_num                                                 |                           */
    /*                          |  1    S       1                                                 |                           */
    /*                          |  2    S       2                                                 |                           */
    /*                          |  3    S       3                                                 |                           */
    /*                          |  4    A       4                                                 |                           */
    /*                          |  5    A       5                                                 |                           */
    /*                          |  6    S       6                                                 |                           */
    /*                          |  7    S       7                                                 |                           */
    /*                          |  8    Q       8                                                 |                           */
    /*                          |  9    Q       9                                                 |                           */
    /*                          |  10   R      10                                                 |                           */
    /*                          |  11   Q      11                                                 |                           */
    /*                          |  12   Q      12                                                 |                           */
    /*                          |                                                                 |                           */
    /*                          |                                                                 |                           */
    /*                          |  LAG DIFF TO BUMP UP COUNTER (VIEW LAGGED_ROWS)                 |                           */
    /*                          |  ===============================================                |                           */
    /*                          |                                                                 |                           */
    /*                          |     row_num  ltr prev_ltr zero_one                              |                           */
    /*                          |      1        S     <NA>    1                                   |                           */
    /*                          |      2        S        S    0                                   |                           */
    /*                          |      3        S        S    0   when ltr ne prev_ltr            |                           */
    /*                          |                                 OR prev_ltr IS NULL then 1      |                           */
    /*                          |      4        A        S    1   else 0                          |                           */
    /*                          |                                                                 |                           */
    /*                          |      5        A        A    0                                   |                           */
    /*                          |      6        S        A    1                                   |                           */
    /*                          |      7        S        S    0                                   |                           */
    /*                          |      8        Q        S    1                                   |                           */
    /*                          |      9        Q        Q    0                                   |                           */
    /*                          |     10        R        Q    1                                   |                           */
    /*                          |     11        Q        R    1                                   |                           */
    /*                          |     12        Q        Q    0                                   |                           */
    /*                          |                                                                 |                           */
    /*                          |                                                                 |                           */
    /*                          |  ASSIGN THE CHANGE COUNTER (FINAL WANT DATASET)                 |                           */
    /*                          |  ===============================================                |                           */
    /*                          |                                                                 |                           */
    /*                          |                          zero    change                         |                           */
    /*                          |  ltr row_num prev_ltr    one     counter                        |                           */
    /*                          |                                                                 |                           */
    /*                          |    S       1     <NA>      1      1  sum the 1s and 0s          |                           */
    /*                          |    S       2        S      0      1  holding the sum            |                           */
    /*                          |    S       3        S      0      1  over each row              |                           */
    /*                          |                                      (uses sql over row_num)    |                           */
    /*                          |                                      no group by needed         |                           */
    /*                          |    A       4        S      1      2                             |                           */
    /*                          |    A       5        A      0      2                             |                           */
    /*                          |    S       6        A      1      3                             |                           */
    /*                          |    S       7        S      0      3                             |                           */
    /*                          |    Q       8        S      1      4                             |                           */
    /*                          |    Q       9        Q      0      4                             |                           */
    /*                          |    R      10        Q      1      5                             |                           */
    /*                          |    Q      11        R      1      6                             |                           */
    /*                          |    Q      12        Q      0      6                             |                           */
    /*                          |                                                                 |                           */
    /*                          |                                                                 |                           */
    /*                          |  %utl_rbeginx;                                                  |                           */
    /*                          |  parmcards4;                                                    |                           */
    /*                          |  library(sqldf)                                                 |                           */
    /*                          |  library(haven)                                                 |                           */
    /*                          |  source("c:/oto/fn_tosas9x.R")                                  |                           */
    /*                          |  have<-read_sas("d:/sd1/have.sas7bdat")                         |                           */
    /*                          |  have                                                           |                           */
    /*                          |  want <- sqldf('                                                |                           */
    /*                          |    WITH numbered_rows AS (                                      |                           */
    /*                          |      SELECT                                                     |                           */
    /*                          |         ltr                                                     |                           */
    /*                          |       ,ROW_NUMBER() OVER (ORDER BY rowid) AS row_num            |                           */
    /*                          |      FROM                                                       |                           */
    /*                          |         have                                                    |                           */
    /*                          |    ),                                                           |                           */
    /*                          |    lagged_rows AS (                                             |                           */
    /*                          |      SELECT                                                     |                           */
    /*                          |         ltr                                                     |                           */
    /*                          |        ,row_num                                                 |                           */
    /*                          |        ,LAG(ltr) OVER (ORDER BY row_num) AS prev_ltr            |                           */
    /*                          |      FROM                                                       |                           */
    /*                          |       numbered_rows                                             |                           */
    /*                          |    )                                                            |                           */
    /*                          |    SELECT                                                       |                           */
    /*                          |        ltr                                                      |                           */
    /*                          |       ,row_num                                                  |                           */
    /*                          |       ,prev_ltr                                                 |                           */
    /*                          |       ,CASE                                                     |                           */
    /*                          |          WHEN ltr != prev_ltr OR prev_ltr IS NULL THEN 1        |                           */
    /*                          |          ELSE 0                                                 |                           */
    /*                          |        END AS zero_one                                          |                           */
    /*                          |       ,sum(CASE                                                 |                           */
    /*                          |          WHEN ltr != prev_ltr OR prev_ltr IS NULL THEN 1        |                           */
    /*                          |          ELSE 0                                                 |                           */
    /*                          |        END) OVER (ORDER BY row_num) AS change_counter           |                           */
    /*                          |      FROM                                                       |                           */
    /*                          |       lagged_rows                                               |                           */
    /*                          |  ')                                                             |                           */
    /*                          |  want                                                           |                           */
    /*                          |  ;;;;                                                           |                           */
    /*                          |  %utl_rendx;                                                    |                           */
    /*                          |                                                                 |                           */
    /**************************************************************************************************************************/

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    options validvarname=upcase
    libname sd1 "d:/sd1";
    data sd1.have;
       input ltr$;
    cards4;
    S
    S
    S
    A
    A
    S
    S
    Q
    Q
    R
    Q
    Q
    ;;;;
    run;quit;

    /*                       _       _            _
    / |  ___  __ _ ___    __| | __ _| |_ __ _ ___| |_ ___ _ __
    | | / __|/ _` / __|  / _` |/ _` | __/ _` / __| __/ _ \ `_ \
    | | \__ \ (_| \__ \ | (_| | (_| | || (_| \__ \ ||  __/ |_) |
    |_| |___/\__,_|___/  \__,_|\__,_|\__\__,_|___/\__\___| .__/
                                                         |_|
    */

    data want;
      set sd1.have;
      by ltr notsorted;
      if first.ltr then grp++1;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* WANT total obs=12                                                                                                      */
    /*                                                                                                                        */
    /* Obs    LTR    GRP                                                                                                      */
    /*                                                                                                                        */
    /*   1     S      1                                                                                                       */
    /*   2     S      1                                                                                                       */
    /*   3     S      1                                                                                                       */
    /*   4     A      2                                                                                                       */
    /*   5     A      2                                                                                                       */
    /*   6     S      3                                                                                                       */
    /*   7     S      3                                                                                                       */
    /*   8     Q      4                                                                                                       */
    /*   9     Q      4                                                                                                       */
    /*  10     R      5                                                                                                       */
    /*  11     Q      6                                                                                                       */
    /*  12     Q      6                                                                                                       */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*___                     _       _           _      ___     _
    |___ \   _ __   ___  __ _| |  ___| |__   ___ | |_   ( _ )   | | ___  _ __   __ _
      __) | | `__| / __|/ _` | | / __| `_ \ / _ \| __|  / _ \/\ | |/ _ \| `_ \ / _` |
     / __/  | |    \__ \ (_| | | \__ \ | | | (_) | |_  | (_>  < | | (_) | | | | (_| |
    |_____| |_|    |___/\__, |_| |___/_| |_|\___/ \__|  \___/\/ |_|\___/|_| |_|\__, |
     _                     |_|                                                 |___/
    | | ___  _ __   __ _
    | |/ _ \| `_ \ / _` |
    | | (_) | | | | (_| |
    |_|\___/|_| |_|\__, |
                   |___/
    */

    %utl_rbeginx;
    parmcards4;
    library(sqldf)
    library(haven)
    source("c:/oto/fn_tosas9x.R")
    have<-read_sas("d:/sd1/have.sas7bdat")
    have
    want <- sqldf('
      WITH numbered_rows AS (
        SELECT
           ltr
         ,ROW_NUMBER() OVER (ORDER BY rowid) AS row_num
        FROM
           have
      ),
      lagged_rows AS (
        SELECT
           ltr
          ,row_num
          ,LAG(ltr) OVER (ORDER BY row_num) AS prev_ltr
        FROM
         numbered_rows
      )
      SELECT
          ltr
         ,row_num
         ,prev_ltr
         ,CASE
            WHEN ltr != prev_ltr OR prev_ltr IS NULL THEN 1
            ELSE 0
          END AS zero_one
         ,sum(CASE
            WHEN ltr != prev_ltr OR prev_ltr IS NULL THEN 1
            ELSE 0
          END) OVER (ORDER BY row_num) AS change_counter
        FROM
         lagged_rows
      ')
    want
      fn_tosas9x(
        inp    = want
       ,outlib ="d:/sd1/"
       ,outdsn ="rwant"
       )
    ;;;;
    %utl_rendx;

    proc print data=sd1.rwant;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* R                                                     SAS                                                              */
    /*                                                                                                              CHANGE_   */
    /*  > want                                                ROWNAMES    LTR    ROW_NUM    PREV_LTR    ZERO_ONE    COUNTER   */
    /*     ltr row_num prev_ltr zero_one change_counter                                                                       */
    /*  1    S       1     <NA>        1              1           1        S         1                      1          1      */
    /*  2    S       2        S        0              1           2        S         2         S            0          1      */
    /*  3    S       3        S        0              1           3        S         3         S            0          1      */
    /*  4    A       4        S        1              2           4        A         4         S            1          2      */
    /*  5    A       5        A        0              2           5        A         5         A            0          2      */
    /*  6    S       6        A        1              3           6        S         6         A            1          3      */
    /*  7    S       7        S        0              3           7        S         7         S            0          3      */
    /*  8    Q       8        S        1              4           8        Q         8         S            1          4      */
    /*  9    Q       9        Q        0              4           9        Q         9         Q            0          4      */
    /*  10   R      10        Q        1              5          10        R        10         Q            1          5      */
    /*  11   Q      11        R        1              6          11        Q        11         R            1          6      */
    /*  12   Q      12        Q        0              6          12        Q        12         Q            0          6      */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*   _                _
     ___| |__   ___  _ __| |_
    / __| `_ \ / _ \| `__| __|
    \__ \ | | | (_) | |  | |_
    |___/_| |_|\___/|_|   \__|

    */

    %utl_rbeginx;
    parmcards4;
    library(sqldf)
    library(haven)
    source("c:/oto/fn_tosas9x.R")
    have<-read_sas("d:/sd1/have.sas7bdat")
    have
    want <- sqldf('
      WITH numbered_rows AS (
        SELECT
           ltr
         ,ROW_NUMBER() OVER (ORDER BY rowid) AS row_num
        FROM
           have
      ),
      lagged_rows AS (
        SELECT
           ltr
          ,row_num
          ,LAG(ltr) OVER (ORDER BY row_num) AS prev_ltr
        FROM
         numbered_rows
      )
      SELECT
          ltr
         ,sum(CASE
            WHEN ltr != prev_ltr OR prev_ltr IS NULL THEN 1
            ELSE 0
          END) OVER (ORDER BY row_num) AS change_counter
        FROM
         lagged_rows
      ')
    want
      fn_tosas9x(
        inp    = want
       ,outlib ="d:/sd1/"
       ,outdsn ="rwant"
       )
    ;;;;
    %utl_rendx;

    proc print data=sd1.rwant;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /* R                         SAS                                                                                          */
    /*                                                                                                                        */
    /* > want                                                                                                                 */
    /*                                              CHANGE_                                                                   */
    /*    ltr change_counter     ROWNAMES    LTR    COUNTER                                                                   */
    /*                                                                                                                        */
    /* 1    S              1         1        S        1                                                                      */
    /* 2    S              1         2        S        1                                                                      */
    /* 3    S              1         3        S        1                                                                      */
    /* 4    A              2         4        A        2                                                                      */
    /* 5    A              2         5        A        2                                                                      */
    /* 6    S              3         6        S        3                                                                      */
    /* 7    S              3         7        S        3                                                                      */
    /* 8    Q              4         8        Q        4                                                                      */
    /* 9    Q              4         9        Q        4                                                                      */
    /* 10   R              5        10        R        5                                                                      */
    /* 11   Q              6        11        Q        6                                                                      */
    /* 12   Q              6        12        Q        6                                                                      */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*____               _   _                             _
    |___ /   _ __  _   _| |_| |__   ___  _ __    ___  __ _| |
      |_ \  | `_ \| | | | __| `_ \ / _ \| `_ \  / __|/ _` | |
     ___) | | |_) | |_| | |_| | | | (_) | | | | \__ \ (_| | |
    |____/  | .__/ \__, |\__|_| |_|\___/|_| |_| |___/\__, |_|
            |_|    |___/                                |_|
    */

    proc datasets lib=sd1 nolist nodetails;
       delete pywant;
    run;quit;

    %utl_pybeginx;
    parmcards4;
    exec(open('c:/oto/fn_python.py').read());
    have,meta = ps.read_sas7bdat('d:/sd1/have.sas7bdat');
    want=pdsql('''
      WITH numbered_rows AS (
        SELECT
           ltr
         ,ROW_NUMBER() OVER (ORDER BY rowid) AS row_num
        FROM
           have
      ),
      lagged_rows AS (
        SELECT
           ltr
          ,row_num
          ,LAG(ltr) OVER (ORDER BY row_num) AS prev_ltr
        FROM
         numbered_rows
      )
      SELECT
          ltr
         ,sum(CASE
            WHEN ltr != prev_ltr OR prev_ltr IS NULL THEN 1
            ELSE 0
          END) OVER (ORDER BY row_num) AS change_counter
        FROM
         lagged_rows
       ''');
    print(want);
    fn_tosas9x(want,outlib='d:/sd1/',outdsn='pywant',timeest=3);
    ;;;;
    %utl_pyendx;

    proc print data=sd1.pywant;
    run;quit;

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*  R                            SAS                                                                                      */
    /*                                       CHANGE_                                                                          */
    /*     ltr  change_counter        LTR    COUNTER                                                                          */
    /*                                                                                                                        */
    /*  0    S               1         S        1                                                                             */
    /*  1    S               1         S        1                                                                             */
    /*  2    S               1         S        1                                                                             */
    /*  3    A               2         A        2                                                                             */
    /*  4    A               2         A        2                                                                             */
    /*  5    S               3         S        3                                                                             */
    /*  6    S               3         S        3                                                                             */
    /*  7    Q               4         Q        4                                                                             */
    /*  8    Q               4         Q        4                                                                             */
    /*  9    R               5         R        5                                                                             */
    /*  10   Q               6         Q        6                                                                             */
    /*  11   Q               6         Q        6                                                                             */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*  _                     _
    | || |    _ __  _ __ ___ | | ___   __ _
    | || |_  | `_ \| `__/ _ \| |/ _ \ / _` |
    |__   _| | |_) | | | (_) | | (_) | (_| |
       |_|   | .__/|_|  \___/|_|\___/ \__, |
             |_|                      |___/
    */

     This technique showcases how to adapt procedural programming concepts to
     SQL, but it should be used judiciously, considering SQL's strengths in set-based
     operations.

     The solution describes a method for iterating through data in SQL, which is often not
     the most natural or efficient approach in SQL but can be useful in certain scenarios. Let's break
     down the key points:

     SQL ITERATION APPROACH

      The solution described mimics a
      loop-like behavior in SQL, similar to a SAS DO loop, which is more common in procedural programming
      languages. This approach is used when you need to process data row by row, which is not typically
      how SQL is designed to work.

     COUNTER MECHANISM

      The method involves using a counter
      that increments when certain conditions are met. Specifically:

       1. It uses the LAG function to
          compare the current row with the previous row.

       2. When the value changes (compared to the
      previous row), the counter is incremented.


     GROUPING BEHAVIOR

      This technique adds a
      unique counter to each group of data, even when group names repeat. The important characteristics
      are:

      1. Groups are not necessarily sorted, but they are grouped together.

      2. Each group
         gets a unique identifier, even if the group name appears multiple times in the dataset.


     USE CASE

     This approach can be useful in scenarios where you need to:

      1.  Assign unique
         identifiers to groups of data

      2. Process data sequentially in a way that's not typically handled
         by SQL's set-based operations

      3. Simulate loop-like behavior in SQL

     Considerations


     While this method can solve certain problems, it's important to note that:

     1. It's often
        less efficient than set-based operations in SQL.

     2. It may not be the best approach for large
        datasets.

     3. There might be more SQL-native ways to achieve the same result, depending on the
        specific problem.

      This technique showcases how to adapt procedural programming concepts to
      SQL, but it should be used judiciously, considering SQL's strengths in set-based
      operations

    /*___             _       _           _
    | ___|   _ __ ___| | __ _| |_ ___  __| |  _ __ ___ _ __   ___  ___
    |___ \  | `__/ _ \ |/ _` | __/ _ \/ _` | | `__/ _ \ `_ \ / _ \/ __|
     ___) | | | |  __/ | (_| | ||  __/ (_| | | | |  __/ |_) | (_) \__ \
    |____/  |_|  \___|_|\__,_|\__\___|\__,_| |_|  \___| .__/ \___/|___/
                                                      |_|
    */

    REPO
    -------------------------------------------------------------------------------------------------------------------------------------
    https://github.com/rogerjdeangelis/utl-adding-sequence-numbers-and-partitions-in-SAS-sql-without-using-monotonic
    https://github.com/rogerjdeangelis/utl-create-equally-spaced-values-using-partitioning-in-sql-wps-r-python
    https://github.com/rogerjdeangelis/utl-find-first-n-observations-per-category-using-proc-sql-partitioning
    https://github.com/rogerjdeangelis/utl-macro-to-enable-sql-partitioning-by-groups-montonic-first-and-last-dot
    https://github.com/rogerjdeangelis/utl-partitioning-your-table-for-a-big-parallel-systask-sort
    https://github.com/rogerjdeangelis/utl-pivot-long-pivot-wide-transpose-partitioning-sql-arrays-wps-r-python
    https://github.com/rogerjdeangelis/utl-pivot-transpose-by-id-using-wps-r-python-sql-using-partitioning
    https://github.com/rogerjdeangelis/utl-top-four-seasonal-precipitation-totals--european-cities-sql-partitions-in-wps-r-python
    https://github.com/rogerjdeangelis/utl-transpose-pivot-wide-using-sql-partitioning-in-wps-r-python
    https://github.com/rogerjdeangelis/utl-transposing-rows-to-columns-using-proc-sql-partitioning
    https://github.com/rogerjdeangelis/utl-transposing-words-into-sentences-using-sql-partitioning-in-r-and-python
    https://github.com/rogerjdeangelis/utl-using-sql-in-wps-r-python-select-the-four-youngest-male-and-female-students-partitioning

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */

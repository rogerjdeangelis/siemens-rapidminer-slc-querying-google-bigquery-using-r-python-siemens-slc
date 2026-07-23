# siemens-rapidminer-slc-querying-google-bigquery-using-r-python-siemens-slc
Siemens-rapidminer-slc-querying-google-bigquery-using-r-and-the-siemens-slc
    %let pgm=siemens-rapidminer-slc-querying-google-bigquery-using-r-python-siemens-slc;

    %stop_submission;

    Siemens-rapidminer-slc-querying-google-bigquery-using-r-and-the-siemens-slc;

    EXAMPLE; Select max birth weights by year and month

    Too long to post see
    https://github.com/rogerjdeangelis/siemens-rapidminer-slc-querying-google-bigquery-using-r-python-siemens-slc

    CONTENTS

    1 Preparation: Create Google API and install Google ODBC Driver
    2 R
    3 Python
    4 slc ODBC
    5 slc extensions (libname procs ...)
    6 related repos: google postgresql and sqlite

    MACROS
    https://github.com/rogerjdeangelis/utl-macros-used-in-many-of-rogerjdeangelis-repositories

    /*                                        _   _
    / |  _ __  _ __ ___ _ __   __ _ _ __ __ _| |_(_) ___  _ __
    | | | `_ \| `__/ _ \ `_ \ / _` | `__/ _` | __| |/ _ \| `_ \
    | | | |_) | | |  __/ |_) | (_| | | | (_| | |_| | (_) | | | |
    |_| | .__/|_|  \___| .__/ \__,_|_|  \__,_|\__|_|\___/|_| |_|
        |_|            |_|
    */

    1 You need a google API
        a https://docs.cloud.google.com/bigquery/docs/authentication
        b create project id
        c create json file with a authentification key
        d download JSO file and write down your project (you will need tis to configue the ODBC driver)
    2  Install google odbc driver
       Download
       https://storage.googleapis.com/bq-driver-releases/odbc/ODBCDriverforBigQuery_windows_x64_latest.msi
    3  Make sure after the install you configue the driver
        Open ODBC 64 on win 11
        a find the driver 'ODBC Driver for BigQuery'
          configure
             1 add your peoject I have shets-235114
             2 add the path to the JSON authentification file
               I saved it at d:/google/shets-235114-27fe0e08dcf7.json
        b Summary
          For ODBC Driver I Have
          dsn='BigQuery DSN',
          platform=64bit,
          driver='ODBC Driver for BigQuery'
          For Google Bigquery  API I have
          BigQuery Project = "shets-235114"
          path="d:/google/shets-235114-27fe0e08dcf7.json"

    /*___    _                   _                             _        _        _     _
    |___ \  (_)_ __  _ __  _   _| |_    __ _  ___   ___   __ _| | ___  | |_ __ _| |__ | | ___
      __) | | | `_ \| `_ \| | | | __|  / _` |/ _ \ / _ \ / _` | |/ _ \ | __/ _` | `_ \| |/ _ \
     / __/  | | | | | |_) | |_| | |_  | (_| | (_) | (_) | (_| | |  __/ | || (_| | |_) | |  __/
    |_____| |_|_| |_| .__/ \__,_|\__|  \__, |\___/ \___/ \__, |_|\___|  \__\__,_|_.__/|_|\___|
                    |_|                |___/             |___/
    Google provides a schema and sample builtin tables
    Schema=bigquery-public-data.samples
    Table= natality
    */

    Google table bigquery-public-data.samples.natality

                      weight
         year month   pounds

      1  2005    12    9.56
      2  2005     3    7.41
      3  2005    12    6.49
      4  2005     5    6.20
      5  2005    10    5.97
      6  2005    11    6.10
      7  2005    11    6.99
      8  2005     8    6.12
      9  2005     9    4.69
     10  2005    10    4.75
     ...

    /*___
    |___ \   _ __
      __) | | `__|
     / __/  | |
    |_____| |_|

    */

    libname workx sas7bdat "d:/wpswrkx"; /*--- put this in your autoexec ---*/

    proc datasets lib=workx kill;
    run;

    options validvarname=v7;
    options set=RHOME "C:\Progra~1\R\R-4.5.2\bin\r";
    proc r;
    submit;
    library(bigrquery)

    # 1. Authenticate
    bq_auth(path = "d:/google/shets-235114-27fe0e08dcf7.json")

    # 2. Set YOUR project for billing
    billing_project <- "shets-235114"

    # 3. SQL Query for a Public Dataset
    # This looks at the top 12 rows of birth data
    sql <- "SELECT year, month, max(weight_pounds) as maxwgt
            FROM `bigquery-public-data.samples.natality`
            GROUP BY year, month
            ORDER BY year, month
            LIMIT 12"

    # 4. Run the query
    tb <- bq_project_query(billing_project, sql)

    # 5. Download and view the results
    df <- bq_table_download(tb)
    print(df)
    endsubmit;
    import r=df data=workx.rnatality;
    run;

    proc print data=workx.rnatality;
    run;

    /**************************************************************************************************************************/
    /*              INPUT      |     OUTPUT MAX BIRTH WEIGHTS GROUPED NU YEAR MONTH                                           */
    /*                  weight |                                                                                              */
    /*      year month  pounds |   Obs    year    month    maxwgt                                                             */
    /*                         |                                                                                              */
    /*   1  2005    12   9.56  |     1    1969       1     17.4606                                                            */
    /*   2  2005     3   7.41  |     2    1969       2     17.8574                                                            */
    /*   3  2005    12   6.49  |     3    1969       3     17.6260                                                            */
    /*   4  2005     5   6.20  |     4    1969       4     16.1158                                                            */
    /*   5  2005    10   5.97  |     5    1969       5     16.0012                                                            */
    /*   6  2005    11   6.10  |     6    1969       6     17.8795                                                            */
    /*   7  2005    11   6.99  |     7    1969       7     16.3142                                                            */
    /*   8  2005     8   6.12  |     8    1969       8     15.0003                                                            */
    /*   9  2005     9   4.69  |     9    1969       9     13.6246                                                            */
    /*  10  2005    10   4.75  |    10    1969      10     15.7498                                                            */
    /*  ...                    |    11    1969      11     17.6260                                                            */
    /*                         |    12    1969      12     18.0007                                                            */
    /**************************************************************************************************************************/

    /*
    | | ___   __ _
    | |/ _ \ / _` |
    | | (_) | (_| |
    |_|\___/ \__, |
             |___/
    */


    1                                          Altair SLC         15:40 Thursday, July 23, 2026

    NOTE: Copyright 2002-2025 World Programming, an Altair Company
    NOTE: Altair SLC 2026 (05.26.01.00.000758)
          Licensed to Roger DeAngelis
    NOTE: This session is executing on the X64_WIN11PRO platform and is running in 64 bit mode


    NOTE: AUTOEXEC processing completed

    1         libname workx sas7bdat "d:/wpswrkx"; /*--- put this in your autoexec ---*/
    NOTE: Library workx assigned as follows:
          Engine:        SAS7BDAT
          Physical Name: d:\wpswrkx

    1    SASNATALITY10    DATA             66560      23JUL2026:15:17:40
    2
    3         proc datasets lib=workx kill;
    4         run;
    NOTE: Deleting WORKX.sasnatality10 (type=DATA)
    5
    6         options validvarname=v7;
    7         options set=RHOME "C:\Progra~1\R\R-4.5.2\bin\r";
    NOTE: Procedure datasets step took :
          real time : 0.027
          cpu time  : 0.000

    8         proc r;
    9         submit;
    10        library(bigrquery)
    11
    12        # 1. Authenticate
    13        bq_auth(path = "d:/google/shets-235114-27fe0e08dcf7.json")
    14
    15        # 2. Set YOUR project for billing
    16        billing_project <- "shets-235114"
    17
    18        # 3. SQL Query for a Public Dataset
    19        # This looks at the top 12 rows of birth data
    20        sql <- "SELECT year, month, max(weight_pounds) as maxwgt
    21                FROM `bigquery-public-data.samples.natality`
    22                GROUP BY year, month
    23                ORDER BY year, month
    24                LIMIT 12"
    25
    26        # 4. Run the query
    27        tb <- bq_project_query(billing_project, sql)
    28
    29        # 5. Download and view the results
    30        df <- bq_table_download(tb)
    31        print(df)
    32        endsubmit;
    NOTE: Using R version 4.5.2 (2025-10-31 ucrt) from C:\Program Files\R\R-4.5.2

    NOTE: Submitting statements to R:

    > library(bigrquery)
    Warning message:
    package 'bigrquery' was built under R version 4.5.3
    >
    > # 1. Authenticate
    > bq_auth(path = "d:/google/shets-235114-27fe0e08dcf7.json")
    >
    > # 2. Set YOUR project for billing
    > billing_project <- "shets-235114"
    >
    > # 3. SQL Query for a Public Dataset
    > # This looks at the top 12 rows of birth data
    > sql <- "SELECT year, month, max(weight_pounds) as maxwgt
    +         FROM `bigquery-public-data.samples.natality`
    +         GROUP BY year, month
    +         ORDER BY year, month
    +         LIMIT 12"
    >
    > # 4. Run the query
    > tb <- bq_project_query(billing_project, sql)
    >
    > # 5. Download and view the results
    > df <- bq_table_download(tb)
    > print(df)

    NOTE: Processing of R statements complete

    33        import r=df data=workx.rnatality;
    NOTE: Creating data set 'WORKX.rnatality' from R data frame 'df'
    NOTE: Data set "WORKX.rnatality" has 12 observation(s) and 3 variable(s)

    34        run;
    NOTE: Procedure r step took :
          real time : 4.005
          cpu time  : 0.015

    35
    36        proc print data=workx.rnatality;
    37        run;
    NOTE: 12 observations were read from "WORKX.rnatality"
    NOTE: Procedure print step took :
          real time : 0.005
          cpu time  : 0.000

    NOTE: Submitted statements took :
          real time : 4.171
          cpu time  : 0.109

    /*____               _   _
    |___ /   _ __  _   _| |_| |__   ___  _ __
      |_ \  | `_ \| | | | __| `_ \ / _ \| `_ \
     ___) | | |_) | |_| | |_| | | | (_) | | | |
    |____/  | .__/ \__, |\__|_| |_|\___/|_| |_|
            |_|    |___/
    */

    options validvarname=v7;
    options set=PYTHONHOME "D:\py314";
    proc python;
    submit;
    import os
    import requests
    import pandas as pd
    import json

    credential_path = "d:/google/shets-235114-27fe0e08dcf7.json"
    os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = credential_path

    # Load the service account credentials
    with open(credential_path, 'r') as f:
        creds = json.load(f)

    # Get an access token using the JSON key
    from google.oauth2 import service_account
    from google.auth.transport.requests import Request

    credentials = service_account.Credentials.from_service_account_file(
        credential_path,
        scopes=["https://www.googleapis.com/auth/bigquery"]
    )
    credentials.refresh(Request())
    token = credentials.token

    # Build the query request
    project = "shets-235114"
    sql = """
     SELECT year, month, max(weight_pounds) as maxwgt
     FROM `bigquery-public-data.samples.natality`
     GROUP BY year, month
     ORDER BY year, month
     LIMIT 12
    """

    url = f"https://bigquery.googleapis.com/bigquery/v2/projects/{project}/queries"
    headers = {
        "Authorization": f"Bearer {token}",
        "Content-Type": "application/json"
    }
    payload = {"query": sql, "useLegacySql": False}

    response = requests.post(url, headers=headers, json=payload)
    data = response.json()

    # Convert to DataFrame
    rows = data.get("rows", [])
    fields = [f["name"] for f in data["schema"]["fields"]]
    df = pd.DataFrame([[cell["v"] for cell in row["f"]] for row in rows], columns=fields)
    print(df)
    endsubmit;
    import python=df data=workx.pynatality;
    run;

    ods listing;
    proc print data=WORKX.pynatality;
    run;

    /**************************************************************************************************************************/
    /*  The PYTHON Procedure                  |    SLC WORKX.PYNATALITY total obs=12                                          */
    /*                                        |                                                                               */
    /*      year month              maxwgt    |    Obs    year    month    maxwgt                                             */
    /*                                        |                                                                               */
    /*  0   1969     1       17.4606111504    |      1    1969     1       17.4606111504                                      */
    /*  1   1969     2        17.857443222    |      2    1969     2       17.857443222                                       */
    /*  2   1969     3       17.6259578469    |      3    1969     3       17.6259578469                                      */
    /*  3   1969     4       16.1157913522    |      4    1969     4       16.1157913522                                      */
    /*  4   1969     5  16.001150975959998    |      5    1969     5       16.001150975959998                                 */
    /*  5   1969     6       17.8794894482    |      6    1969     6       17.8794894482                                      */
    /*  6   1969     7        16.314207388    |      7    1969     7       16.314207388                                       */
    /*  7   1969     8      15.00025230648    |      8    1969     8       15.00025230648                                     */
    /*  8   1969     9       13.6245677916    |      9    1969     9       13.6245677916                                      */
    /*  9   1969    10      15.74982399728    |     10    1969     10      15.74982399728                                     */
    /*  10  1969    11       17.6259578469    |     11    1969     11      17.6259578469                                      */
    /*  11  1969    12       18.0007436923    |     12    1969     12      18.0007436923                                      */
    /**************************************************************************************************************************/

    /*
    | | ___   __ _
    | |/ _ \ / _` |
    | | (_) | (_| |
    |_|\___/ \__, |
             |___/
    */

    1                                          Altair SLC         15:43 Thursday, July 23, 2026

    NOTE: Copyright 2002-2025 World Programming, an Altair Company
    NOTE: Altair SLC 2026 (05.26.01.00.000758)
          Licensed to Roger DeAngelis
    NOTE: This session is executing on the X64_WIN11PRO platform and is running in 64 bit mode

    NOTE: AUTOEXEC processing beginning; file is C:\wpsoto\autoexec.sas

    NOTE: AUTOEXEC processing completed

    1         options validvarname=v7;
    2         options set=PYTHONHOME "D:\py314";
    3         proc python;
    4         submit;
    5         import os
    6         import requests
    7         import pandas as pd
    8         import json
    9
    10        credential_path = "d:/google/shets-235114-27fe0e08dcf7.json"
    11        os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = credential_path
    12
    13        # Load the service account credentials
    14        with open(credential_path, 'r') as f:
    15            creds = json.load(f)
    16
    17        # Get an access token using the JSON key
    18        from google.oauth2 import service_account
    19        from google.auth.transport.requests import Request
    20
    21        credentials = service_account.Credentials.from_service_account_file(
    22            credential_path,
    23            scopes=["https://www.googleapis.com/auth/bigquery"]
    24        )
    25        credentials.refresh(Request())
    26        token = credentials.token
    27
    28        # Build the query request
    29        project = "shets-235114"
    30        sql = """
    31         SELECT year, month, max(weight_pounds) as maxwgt
    32         FROM `bigquery-public-data.samples.natality`
    33         GROUP BY year, month
    34         ORDER BY year, month
    35         LIMIT 12
    36        """
    37
    38        url = f"https://bigquery.googleapis.com/bigquery/v2/projects/{project}/queries"
    39        headers = {
    40            "Authorization": f"Bearer {token}",
    41            "Content-Type": "application/json"
    42        }
    43        payload = {"query": sql, "useLegacySql": False}
    44
    45        response = requests.post(url, headers=headers, json=payload)
    46        data = response.json()
    47
    48        # Convert to DataFrame
    49        rows = data.get("rows", [])
    50        fields = [f["name"] for f in data["schema"]["fields"]]
    51        df = pd.DataFrame([[cell["v"] for cell in row["f"]] for row in rows], columns=fields)
    52        print(df)
    53        endsubmit;

    NOTE: Submitting statements to Python:


    54        import python=df data=workx.pynatality;
    NOTE: Creating data set 'WORKX.pynatality' from Python data frame 'df'
    NOTE: Data set "WORKX.pynatality" has 12 observation(s) and 3 variable(s)

    55        run;
    NOTE: Procedure python step took :
          real time : 3.196
          cpu time  : 0.046

    NOTE: Submitted statements took :
          real time : 3.291
          cpu time  : 0.109

    /*  _         _
    | || |    ___| | ___
    | || |_  / __| |/ __|
    |__   _| \__ \ | (__
       |_|   |___/_|\___|

    */

    proc datasets lib=workx kill;
    run;quit;

    proc sql;
        /* Establish the ODBC connection using your DSN */
        connect to odbc as bq (dsn="BigQuery DSN");

        /* Create a SAS dataset from the query results */
        create table workx.sasnatality as
        select *
        from connection to bq (
          SELECT year, month, max(weight_pounds) as maxwgt
          FROM `bigquery-public-data.samples.natality`
          GROUP BY year, month
          ORDER BY year, month
          LIMIT 12
        );
    quit;

    proc print data=workx.sasnatality width=min;;
    run;

     /**************************************************************************************************************************/
     /*Altair SLC                                                                                                              */
     /*                                                                                                                        */
     /* Obs    YEAR    MONTH     MAXWGT                                                                                        */
     /*                                                                                                                        */
     /*   1    1969      1      17.4606                                                                                        */
     /*   2    1969      2      17.8574                                                                                        */
     /*   3    1969      3      17.6260                                                                                        */
     /*   4    1969      4      16.1158                                                                                        */
     /*   5    1969      5      16.0012                                                                                        */
     /*   6    1969      6      17.8795                                                                                        */
     /*   7    1969      7      16.3142                                                                                        */
     /*   8    1969      8      15.0003                                                                                        */
     /*   9    1969      9      13.6246                                                                                        */
     /*  10    1969     10      15.7498                                                                                        */
     /*  11    1969     11      17.6260                                                                                        */
     /*  12    1969     12      18.0007                                                                                        */
     /**************************************************************************************************************************/

    /*
    | | ___   __ _
    | |/ _ \ / _` |
    | | (_) | (_| |
    |_|\___/ \__, |
             |___/
    */

    1                                          Altair SLC         15:46 Thursday, July 23, 2026

    NOTE: Copyright 2002-2025 World Programming, an Altair Company
    NOTE: Altair SLC 2026 (05.26.01.00.000758)
          Licensed to Roger DeAngelis
    NOTE: This session is executing on the X64_WIN11PRO platform and is running in 64 bit mode

    NOTE: AUTOEXEC processing beginning; file is C:\wpsoto\autoexec.sas

    NOTE: AUTOEXEC processing completed


    Altair SLC

    The DATASETS Procedure

    1         proc datasets lib=workx kill;
    2         run;quit;
    NOTE: Deleting WORKX.pynatality (type=DATA)
    NOTE: Deleting WORKX.rnatality (type=DATA)
    NOTE: Procedure datasets step took :
          real time : 0.037
          cpu time  : 0.015

    3
    4         proc sql;
    5             /* Establish the ODBC connection using your DSN */
    6             connect to odbc as bq (dsn="BigQuery DSN");
    NOTE: Connected to DB: BigQuery DSN (BigQuery version 2)
    NOTE: Connected to DB: BigQuery DSN (BigQuery version 2)
    NOTE: Successfully connected to database ODBC as alias BQ.
    7
    8             /* Create a SAS dataset from the query results */
    9             create table workx.sasnatality as
    10            select *
    11            from connection to bq (
    12              SELECT year, month, max(weight_pounds) as maxwgt
    13              FROM `bigquery-public-data.samples.natality`
    14              GROUP BY year, month
    15              ORDER BY year, month
    16              LIMIT 12
    17            );
    NOTE: Data set "WORKX.sasnatality" has 12 observation(s) and 3 variable(s)
    18        quit;
    NOTE: Procedure sql step took :
          real time : 2.254
          cpu time  : 0.515

    19
    20        proc print data=workx.sasnatality width=min;;
    21        run;
    NOTE: 12 observations were read from "WORKX.sasnatality"
    NOTE: Procedure print step took :
          real time : 0.015
          cpu time  : 0.000

    NOTE: Submitted statements took :
          real time : 2.408
          cpu time  : 0.640

    /*___        _                  _                 _
    | ___|   ___| | ___    _____  _| |_ ___ _ __  ___(_) ___  _ __  ___
    |___ \  / __| |/ __|  / _ \ \/ / __/ _ \ `_ \/ __| |/ _ \| `_ \/ __|
     ___) | \__ \ | (__  |  __/>  <| ||  __/ | | \__ \ | (_) | | | \__ \
    |____/  |___/_|\___|  \___/_/\_\\__\___|_| |_|___/_|\___/|_| |_|___/
                 _   _           _           _                  _   _
      ___  _ __ | |_(_)_ __ ___ (_)_______  | | ___ _ __   __ _| |_| |__  ___
     / _ \| `_ \| __| | `_ ` _ \| |_  / _ \ | |/ _ \ `_ \ / _` | __| `_ \/ __|
    | (_) | |_) | |_| | | | | | | |/ /  __/ | |  __/ | | | (_| | |_| | | \__ \
     \___/| .__/ \__|_|_| |_| |_|_/___\___| |_|\___|_| |_|\__, |\__|_| |_|___/
          |_|                                             |___/
    */

    /*--- optimize lengths   ---*/

    proc datasets lib=workx kill;
    run;quit;

    /*--- PASSTHRU           ---*/
    proc sql;
        /* Establish the ODBC connection using your DSN */
        connect to odbc as bq (dsn="BigQuery DSN");

        /* Create a SAS dataset from the query results */
        create table workx.sasnatality10 as
        select *
        from connection to bq (
          SELECT  *
          FROM `bigquery-public-data.samples.natality`
          WHERE state IS NOT NULL
          limit 10
        );
    quit;

    ?*-- OPTIMIZE            ---*/
    %slc_optlenpos(inp=workx.sasnatality10,out=workx.sasnatality10 );


    /*--- MIDDLE OBSERVATION ---*/
    /*--- BEFORE             ---*/

     -- CHARACTER --
    Variable                 Typ      Value

    STATE                     C1024   AK
    MOTHER_RESIDENCE_STATE    C1024   AK
    LMP                       C1024   01191999
    MOTHER_BIRTH_STATE        C1024

     -- NUMERIC --
    SOURCE_YEAR               N8      1969
    YEAR                      N8      1969
    MONTH                     N8      11
    DAY                       N8      17
    WDAY                      N8      .
    IS_MALE                   N8      0
    CHILD_RACE                N8      1
    WEIGHT_POUNDS             N8      8.9992695348
    PLURALITY                 N8      .
    APGAR_1MIN                N8      .
    APGAR_5MIN                N8      .
    MOTHER_RACE               N8      1
    MOTHER_AGE                N8      21
    GESTATION_WEEKS           N8      43
    MOTHER_MARRIED            N8      1
    CIGARETTE_USE             N8      .
    CIGARETTES_PER_DAY        N8      .
    ALCOHOL_USE               N8      .
    DRINKS_PER_WEEK           N8      .
    WEIGHT_GAIN_POUNDS        N8      .
    BORN_ALIVE_ALIVE          N8      0
    BORN_ALIVE_DEAD           N8      0
    BORN_DEAD                 N8      0
    EVER_BORN                 N8      1
    FATHER_RACE               N8      1
    FATHER_AGE                N8      32
    RECORD_WEIGHT             N8      2


    /*--- AFTER              ---*/

     -- CHARACTER --
    Variable                 Typ    Value

    STATE                     C2    AK         Length 2 was 1024
    MOTHER_RESIDENCE_STATE    C2    AK         Length 2 was 1024
    LMP                       C8    01191999   Length 8 was 1024
    MOTHER_BIRTH_STATE        C1
    totobs                    C16   10


     -- NUMERIC --
    SOURCE_YEAR               N3    1969        All these were length 8 now 3
    YEAR                      N3    1969
    MONTH                     N3    11
    DAY                       N3    17
    WDAY                      N3    .
    IS_MALE                   N3    0
    CHILD_RACE                N3    1
    WEIGHT_POUNDS             N8    8.9992695348
    PLURALITY                 N3    .
    APGAR_1MIN                N3    .
    APGAR_5MIN                N3    .
    MOTHER_RACE               N3    1
    MOTHER_AGE                N3    21
    GESTATION_WEEKS           N3    43
    MOTHER_MARRIED            N3    1
    CIGARETTE_USE             N3    .
    CIGARETTES_PER_DAY        N3    .
    ALCOHOL_USE               N3    .
    DRINKS_PER_WEEK           N3    .
    WEIGHT_GAIN_POUNDS        N3    .
    BORN_ALIVE_ALIVE          N3    0
    BORN_ALIVE_DEAD           N3    0
    BORN_DEAD                 N3    0
    EVER_BORN                 N3    1
    FATHER_RACE               N3    1
    FATHER_AGE                N3    32
    RECORD_WEIGHT             N3    2

    /*
     _ __  _ __ ___   ___   _ __ ___   ___  __ _ _ __  ___
    | `_ \| `__/ _ \ / __| | `_ ` _ \ / _ \/ _` | `_ \/ __|
    | |_) | | | (_) | (__  | | | | | |  __/ (_| | | | \__ \
    | .__/|_|  \___/ \___| |_| |_| |_|\___|\__,_|_| |_|___/
    |_|
    */


    libname bq odbc dsn="BigQuery DSN" schema="bigquery-public-data.samples";

    proc means data= bq.natality(obs=100);
    class month;
    var weight_pounds;
    run;

    libname bq close;

    /**************************************************************************************************************************/
    /*  Altair SLC                                                                                                            */
    /*  he MEANS Procedure                                                                                                    */
    /*                                                                                                                        */
    /*  LOWEST NUMBER OF MONTHS HAS AT LEAST 100 VALUES                                                                       */
    /*                                            Summary statistics                                                          */
    /*            N                                                                                                           */
    /*  ONTH    Obs    Variable             N             Mean             Std Dev           Minimum         Maximum          */
    /*                                                                                                                        */
    /*     6    100    WEIGHT_POUNDS       99     6.9773633653        1.6492277937      2.5132697868    11.876302054          */
    /**************************************************************************************************************************/

    /*
    | | ___   __ _
    | |/ _ \ / _` |
    | | (_) | (_| |
    |_|\___/ \__, |
             |___/
    */

    1                                          Altair SLC         15:48 Thursday, July 23, 2026

    NOTE: Copyright 2002-2025 World Programming, an Altair Company
    NOTE: Altair SLC 2026 (05.26.01.00.000758)
          Licensed to Roger DeAngelis
    NOTE: This session is executing on the X64_WIN11PRO platform and is running in 64 bit mode

    NOTE: AUTOEXEC processing beginning; file is C:\wpsoto\autoexec.sas

    NOTE: AUTOEXEC processing completed

    1         libname bq odbc dsn="BigQuery DSN" schema="bigquery-public-data.samples";
    NOTE: Library bq assigned as follows:
          Engine:        ODBC
          Physical Name: BigQuery DSN (BigQuery version 2)

    WARNING: truncating character column state to 1024 characters long, based on dbmax_text/max_char_len setting.
    WARNING: truncating character column mother_residence_state to 1024 characters long, based on dbmax_text/max_char_len setting.
    WARNING: truncating character column lmp to 1024 characters long, based on dbmax_text/max_char_len setting.
    WARNING: truncating character column mother_birth_state to 1024 characters long, based on dbmax_text/max_char_len setting.
    2
    3         proc means data= bq.natality(obs=100);
    4         class month;
    5         var weight_pounds;
    6         run;
    NOTE: 100 observations were read from "BQ.natality"
    NOTE: Procedure means step took :
          real time : 17.960
          cpu time  : 4.765


    7
    8         libname bq close;
                         ^
    ERROR: The library engine CLOSE is not known
    ERROR: Error printed on page 1

    NOTE: Submitted statements took :
          real time : 19.136
          cpu time  : 5.390

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */

# utl-creating-a-sas7bdat-directly-from-r-using-dullesresearch-free-s-jdbc-driver
Creating a sas7bdat directly from r using dullesresearch free s-jdbc driver

    Creating a sas7bdat directly from r using dullesresearch free s-jdbc driver

    GitHub
    https://tinyurl.com/3xmzwabc
    https://github.com/rogerjdeangelis/utl-creating-a-sas7bdat-directly-from-r-using-dullesresearch-free-s-jdbc-driver

      This is a big step forward in integrating SAS with other languages.
      Much better then V5 xport files? (which are not lossless for unix and windows)

         1. Supports long variable names
         2. Appears so far to be lossless in windows (sas floats > r floast > sas floats)
         3. The dbWriteTable function has the ODBC resriction and limits the lenght of
            character variables to 255 bytes.
         4. Have not tested date, time or other formats. (dont consider this a issue. I like to work without formats)


    See
    https://dullesresearch.com/

    Download the free driver to create SAS7BDATS.

    https://dullesresearch.com/products/
    Scroll down to S-JDBC

    Documentation
    https://www.dullesresearch.com/wp-content/uploads/2017/11/Carolina-S-JDBC-2.1-User-Guide.pdf
    https://dullesresearch.com/wp-content/uploads/2020/04/Carolina-S-JDBC-2.4-User-Guide-v20.pdf
    https://dullesresearch.com/wp-content/uploads/2017/10/Carolina-S-JDBC-Product-Sheet.pdf

    You will get an email with with instructions to download the driver.

    I downloaded the driver and license to d:/carolina

    I also added d:/carolina to environment variable CLASSPATH

    COMPUTER/PROPERTIES/ADVANCED SYSTEM SETTINGS/ENVIONMENT SETTINGS

     Variable CLASSPATH
     Value    d:/carolina

    *_                   _
    (_)_ __  _ __  _   _| |_
    | | '_ \| '_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    ;

    * just in case you rerun;
    %utlfkil(d:/sd1/want.sas7bdat);

    libname sd1 "d:/sd1";
    data sd1.have;
      length long $10000;
      set sashelp.class;
      long=repeat('1234567890',1000);
    run;quit;

    /*
    Up to 40 obs SD1.HAVE total obs=19

      LONG(length=10,000)      NAME       SEX    AGE    HEIGHT    WEIGHT

     12345678901234567890...   Joyce       F      11     51.3       50.5
     12345678901234567890      Louise      F      12     56.3       77.0
     12345678901234567890      Alice       F      13     56.5       84.0
     12345678901234567890      James       M      12     57.3       83.0
     12345678901234567890      Thomas      M      11     57.5       85.0
     ....


      Variables in Creation Order

         Variable    Type      Len

         LONG        Char    10000
         NAME        Char        8
         SEX         Char        1
         AGE         Num         8
         HEIGHT      Num         8
         WEIGHT      Num         8
    */

    *
     _ __  _ __ ___   ___ ___  ___ ___
    | '_ \| '__/ _ \ / __/ _ \/ __/ __|
    | |_) | | | (_) | (_|  __/\__ \__ \
    | .__/|_|  \___/ \___\___||___/___/
    |_|
    ;

    %utl_submit_r64(%tslit(
    library(RJDBC);
    library(haven);
    want<-read_sas("d:/sd1/have.sas7bdat");
    str(want);
    drv<- JDBC("com.dullesopen.jdbc.Driver","d:/carolina/carolina-jdbc-2.4.3.jar");
    conn <- dbConnect(drv, "jdbc:carolina:bulk:libnames=(dir='d:/sd1')", "", "");
    colnames(iris)=c("SepalLength", "SepalWidth", "PetalLength", "PetalWidth", "Species");
    head(want);
    rc<- dbWriteTable(conn,"want",want);
    ));

    proc print data=sd1.want ;
    run;quit;

    *            _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| '_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    ;
    Up to 40 obs SD1.HAVE total obs=19

      LONG(length=255)          NAME       SEX    AGE    HEIGHT    WEIGHT

     12345678901234567890...   Joyce       F      11     51.3       50.5
     12345678901234567890      Louise      F      12     56.3       77.0
     12345678901234567890      Alice       F      13     56.5       84.0
     12345678901234567890      James       M      12     57.3       83.0
     12345678901234567890      Thomas      M      11     57.5       85.0
     ....


     Variables in Creation Order

    #    Variable    Type    Len

    1    LONG        Char    255   * was 10,0000;
    2    NAME        Char    255
    3    SEX         Char    255
    4    AGE         Num       8
    5    HEIGHT      Num       8
    6    WEIGHT      Num       8

    * reduce to lossless minimum lengths;

    %utl_optlen(inp=sd1.want,out=sd1.want);

     Variables in Creation Order

    #    Variable    Type    Len

    1    LONG        Char    255
    2    NAME        Char      7
    3    SEX         Char      1
    4    AGE         Num       3
    5    HEIGHT      Num       8
    6    WEIGHT      Num       3


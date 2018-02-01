# utl_using_integer_arithmetic_in_sas_to_get_exact_results_pennies_instead_of_dollars
Using Integer arithmetic in SAS to get exact results (pennies instead of dollars). Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.
    
    **** OPEN QUESTION USE AT YOUR OWN RISK ****
    see assumptions and analysis on the end
    
    SAS Forum: Using Integer arithmetic in SAS to get exact results (pennies instead of dollars)

    Great question

    When dealing with dollars and cents mutiply by 100 and
    deal with pennies.

    If we rephase the ops proble in pennies we get the exact
    result the op wants.

    original topic:  Mysterious In Function

    https://communities.sas.com/t5/Base-SAS-Programming/Mysterious-In-Function/m-p/431578

    *                _     _
     _ __  _ __ ___ | |__ | | ___ _ __ ___
    | '_ \| '__/ _ \| '_ \| |/ _ \ '_ ` _ \
    | |_) | | | (_) | |_) | |  __/ | | | | |
    | .__/|_|  \___/|_.__/|_|\___|_| |_| |_|
    |_|
    ;

    %let nodes=-0.1 0 0.1 0.2;

    %let x_min=-0.2;
    %let x_max=0.2;
    %let x_step=0.01;

    data xxx(keep=x nodes_flag);
        nstep=round((&x_max-&x_min)/&x_step);
        do iter=0 to nstep;
          nodes_flag=0;
          x=&x_min+&x_step*iter;
          if x in (&nodes) then nodes_flag=1;
          output;
        end;
    run;

    data nodes;
      set xxx;
      if nodes_flag=1;
    run;

    proc print data=nodes;
    run;

    YEILDS

           NODES_
    Obs     FLAG        X

     1        1      -0.1
     2        1       0.0
     3        1       0.2

    BUT THE CORRECT ANSWER IS

           NODES_
    Obs     FLAG       X

     1        1      -10
     2        1        0
     3        1       10
     4        1       20

    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __
    / __|/ _ \| | | | | __| |/ _ \| '_ \
    \__ \ (_) | | |_| | |_| | (_) | | | |
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|
    ;

    %let nodes=-10 0 10 20;  * mutiply by 100;
    %let x_min=-20;
    %let x_max=20;
    %let x_step=1;

    data xxx(keep=x nodes_flag);
          nstep=(&x_max-&x_min)/&x_step;   ** remove the round;
          put nstep;
          do iter=0 to nstep;
          nodes_flag=0;
          x=&x_min+&x_step*iter;
          if x in (&nodes) then nodes_flag=1;

          output;
          end;
    run;

    data nodes;
      set xxx;
      if nodes_flag=1;
       x=x/100;          ** divide by 100  this is the only non exact operation;
      run;
      proc print data=nodes;
      run;

           NODES_
    Obs     FLAG        X

     1        1      -0.1
     2        1       0.0
     3        1       0.1
     4        1       0.2


     Example of the issue

     It depends on the numbers and the floating point representation;

    data many;
     do i=1 to 20000;
       x=1.00000001;
       output;
     end;
    run;quit;

    proc sql;
      select sum(x) as sumx fromat=18.9 from many;
    ;quit;

                  sumx
    ------------------
       20000.000200009    *** wrong sum;



    data many;
     do i=1 to 20000;
       x=100000001;
       output;
     end;
    run;quit;

    proc sql;
      select sum(x)/10**8 as sumx fromat=18.10 from many;
    ;quit;

                  sumx
    ------------------
      20000.0002000000    ** correct sum;

    Thanks for the details on floating point.

    This analysis only applies to pennies

    My major concern is  can I rely on either ceil(100*100.01) or
    input(cats(scan(x,1,'.'),cats(scan(x,2,'.'),12.)

    I added the context and a warning to my github site when using
    ceil along or the input method with additions and subtracts in the original post.

      **** OPEN QUESTION USE AT YOUR OWN RISK  ***

    The original post used purely integer additions and
    subtractions, except the last x/100


    You have to be carefull about overflow, but the numbers are huge.

    ASSUMPTIONS

     1. Pennies have to be between +/-90 quadrillion pennies
        Thus dollars are limited to plus or minus 900 trillion dollars
        Approx Largest and smallest pennies
          +/-90,007,199,254,740,992
        In dollars
         +/-900,719,925,474
        basicall plus or minus 90 quadrillion pennies
        which is roughly equivalent to
           900,719,925,474
        900 triliion dollars
     2, I eliminated then SINGLE final division by 100 with a character operation

        chr=put(x,z13.);
        chrnum=cats(substr(chr,1,11),'.',substr(chr,12,2));


    %let nodes=-10 0 10 20;  * mutiply by 100;
    %let x_min=-20;
    %let x_max=20;
    %let x_step=1;

    data xxx(keep=x nodes_flag);
          nstep=(&x_max-&x_min);  /* no need to divide by one */
          put nstep;
          do iter=0 to nstep;
          nodes_flag=0;
          x=&x_min+&x_step*iter;  /* these are all integers */
          if x in (&nodes) then nodes_flag=1;

          output;
          end;
    run;

    data nodes;
      set xxx;
      if nodes_flag=1;
      /*
       x=x/100;          ** divide by 100;
      run;               ** this is not exact but unlikley to be a problem;
      */

      chr=put(x,z13.);
      chrnum=cats(substr(chr,1,11),'.',substr(chr,12,2));
      drop chr;
    run;quit;


    proc print data=nodes;
    run;quit;


           NODES_
    Obs     FLAG       X        CHRNUM

     1        1      -10    -0000000000.10
     2        1        0    00000000000.00
     3        1       10    00000000000.10
     4        1       20    00000000000.20


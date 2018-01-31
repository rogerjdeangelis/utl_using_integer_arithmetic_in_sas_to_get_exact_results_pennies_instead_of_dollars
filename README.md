# utl_using_integer_arithmetic_in_sas_to_get_exact_results_pennies_instead_of_dollars
Using Integer arithmetic in SAS to get exact results (pennies instead of dollars). Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.
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
       x=x/100;          ** divide by 100;
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


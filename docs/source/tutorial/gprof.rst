GProf
==========================

.. admonition:: Overview
   :class: Overview

    * **Tutorial:** 30 min

        **Objectives:**
            #. Understand the basics of GProf and how to use it for profiling applications.


GProf (GNU Profiler) is a performance analysis tool for Unix-like operating systems. 
It helps developers identify which parts of their code are consuming the most execution time, 
allowing them to optimize performance-critical sections. GProf works by collecting and analyzing 
data about function calls and execution times during the program's runtime.


The cmake command used is

..  code-block:: bash
    :linenos:

    cmake -G Ninja \
        -DCMAKE_BUILD_TYPE=Debug \
        -DMPI_CXX_COMPILER="$(which mpicxx)" \
        -DWITH_MPI=On \
        -DWITH_OPENMP=On \
        -DCMAKE_INSTALL_PREFIX="$INSTALLDIR" \
        -DCMAKE_CXX_FLAGS="-pg" \
        "$SRCDIR"

where ``-DCMAKE_CXX_FLAGS="-pg"`` is used to enable profiling with gprof. 

To run the application with gprof enabled, use the following command:   

..  code-block:: bash
    :linenos:

    ./lulesh2.0 -s 20

This will generate a file named ``gmon.out`` in the current directory after the program completes.
To analyze the profiling data, use the following command:   

..  code-block:: bash
    :linenos:

    gprof ./lulesh2.0 gmon.out > profile.txt

This will create a file named ``profile.txt`` containing the profiling results, which can be
viewed using any text editor or command-line tools like ``less`` or ``cat``.


GProf columns in the output file:

1. % time

    * The percentage of the total program runtime that was spent inside this function (not including functions it calls).

    * Example: If your program ran for 10 seconds total and foo() spent 4 seconds in its own code, then % time = 40%.

2. cumulative seconds

    * The total time spent in this function and all functions it calls, up to and including this function.

    * Example: If foo() called bar() which took 3 seconds, and foo() itself took 4 seconds, then cumulative seconds = 7 seconds.

3. self seconds

    * The total time spent in this function alone, excluding time spent in functions it calls.

    * Example: If foo() took 4 seconds and called bar() which took 3 seconds, then self seconds = 4 seconds.

4. calls

    * The number of times this function was called during the program's execution.

    * Example: If foo() was called 5 times, then calls = 5.

5. self s/call

    * The average time spent in this function per call, calculated as self seconds divided by calls.

    * Example: If foo() took 4 seconds and was called 5 times, then self ms/call = 800 ms/call.

6. total s/call

    * The average time spent in this function and all functions it calls per call, calculated as cumulative seconds divided by calls.

    * Example: If foo() took 7 seconds (including calls to bar()) and was called 5 times, then total s/call = 1400 ms/call.

7. name

    * The name of the function being profiled.

    * Example: If the function is named foo(), then name = foo().





.. admonition:: Key Points
   :class: hint
   
    #. GProf is a performance analysis tool for Unix-like operating systems.
    #. It helps identify performance bottlenecks in code by analyzing function calls and execution times
    #. To use GProf with a C++ application, compile the code with the `-pg` flag.
    #. Run the application to generate a `gmon.out` file.
    #. Analyze the data using the `gprof` command.
    




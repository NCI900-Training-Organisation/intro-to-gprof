Valgrind
==========================

.. admonition:: Overview
   :class: Overview

    * **Tutorial:** 45 min

        **Objectives:**
            #. Understand the basics of Valgrind and how to use it for profiling applications.



Valgrind is a programming tool for memory debugging, memory leak detection, and profiling. It helps developers 
identify memory-related issues in their applications, such as memory leaks, uninitialized memory reads, and 
invalid memory accesses. 

First, start an interactive job on the cluster 



..  code-block:: bash
    :linenos:

    qsub -I -q normal -P vp91 -l walltime=02:00:00 -l ncpus=48 -l mem=192GB -l wd


and load the necessary modules.

..  code-block:: bash
    :linenos:

    module load gcc/15.1.0


Run Valgrind to detect memory leaks
---------------------------------


Compile the code with debugging information

..  code-block:: bash
    :linenos:

    cd /scratch/vp91/$USER/intro-to-gprof/code/valgrind
    gcc -g -O0 1_leak.c -o 1_leak

* `-g` enables debugging information.
* `-O0` disables optimizations to make debugging easier.

Now run the program with Valgrind to check for memory leaks

..  code-block:: bash
    :linenos:

    valgrind ./1_leak

This will produce output indicating any memory leaks detected in the program. Look for lines that mention 
"definitely lost" to identify memory that was allocated but not freed.

Sample output will be similar to:

..  code-block:: bash
    :linenos:

    ==384063== HEAP SUMMARY:
    ==384063==     in use at exit: 40 bytes in 1 blocks
    ==384063==   total heap usage: 1 allocs, 0 frees, 40 bytes allocated
    ==384063==
    ==384063== LEAK SUMMARY:
    ==384063==    definitely lost: 40 bytes in 1 blocks
    ==384063==    indirectly lost: 0 bytes in 0 blocks
    ==384063==      possibly lost: 0 bytes in 0 blocks
    ==384063==    still reachable: 0 bytes in 0 blocks
    ==384063==         suppressed: 0 bytes in 0 blocks
    ==384063== Rerun with --leak-check=full to see details of leaked memory
    ==384063==
    ==384063== For lists of detected and suppressed errors, rerun with: -s
    ==384063== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)


* **Definitely lost**: No pointer to the allocated memory exists at program exit. This a real memory leak that must be fixed.

* **Indirectly lost**: Memory is only reachable through another leaked block. Fixing the *definitely lost* allocation will fix this too.

* **Possibly lost**: A pointer exists but does not point to the start of the allocation → **ambiguous**, often caused by pointer arithmetic and usually a bug.

* **Still reachable**: Memory is still referenced at exit and could be freed → **not a true leak**, typically safe to ignore.

Indirect leak example
----------------------

Compile the code with debugging information

..  code-block:: bash
    :linenos:

    cd /scratch/vp91/$USER/intro-to-gprof/code/valgrind

    gcc -g -O0 2_indirect.c -o 2_indirect

and run Valgrind

..  code-block:: bash
    :linenos:

    valgrind ./2_indirect

Sample output will be similar to:

..  code-block:: bash
    :linenos:

    ==384410== HEAP SUMMARY:
    ==384410==     in use at exit: 48 bytes in 2 blocks
    ==384410==   total heap usage: 2 allocs, 0 frees, 48 bytes allocated
    ==384410==
    ==384410== LEAK SUMMARY:
    ==384410==    definitely lost: 8 bytes in 1 blocks
    ==384410==    indirectly lost: 40 bytes in 1 blocks
    ==384410==      possibly lost: 0 bytes in 0 blocks
    ==384410==    still reachable: 0 bytes in 0 blocks
    ==384410==         suppressed: 0 bytes in 0 blocks


Possibly lost example
----------------------

Compile the code with debugging information

..  code-block:: bash
    :linenos:

    cd /scratch/vp91/$USER/intro-to-gprof/code/valgrind

    gcc -g -O0 3_possible.c -o 3_possible


run Valgrind

..  code-block:: bash
    :linenos:

    valgrind ./3_possible


Sample output will be similar to:

..  code-block:: bash
    :linenos:
    
    ==384772== HEAP SUMMARY:
    ==384772==     in use at exit: 100 bytes in 1 blocks
    ==384772==   total heap usage: 1 allocs, 0 frees, 100 bytes allocated
    ==384772==
    ==384772== LEAK SUMMARY:
    ==384772==    definitely lost: 0 bytes in 0 blocks
    ==384772==    indirectly lost: 0 bytes in 0 blocks
    ==384772==      possibly lost: 100 bytes in 1 blocks
    ==384772==    still reachable: 0 bytes in 0 blocks
    ==384772==         suppressed: 0 bytes in 0 blocks


Still reachable example
----------------------

Comnpile the code with debugging information

..  code-block:: bash
    :linenos:

    cd /scratch/vp91/$USER/intro-to-gprof/code/valgrind

    gcc -g -O0 4_still_reachable.c -o 4_still_reachable

Run Valgrind

..  code-block:: bash
    :linenos:

    valgrind ./4_still_reachable

Sample output will be similar to:

..  code-block:: bash
    :linenos:


    ==384651== HEAP SUMMARY:
    ==384651==     in use at exit: 100 bytes in 1 blocks
    ==384651==   total heap usage: 1 allocs, 0 frees, 100 bytes allocated
    ==384651==
    ==384651== LEAK SUMMARY:
    ==384651==    definitely lost: 0 bytes in 0 blocks
    ==384651==    indirectly lost: 0 bytes in 0 blocks
    ==384651==      possibly lost: 0 bytes in 0 blocks
    ==384651==    still reachable: 100 bytes in 1 blocks
    ==384651==         suppressed: 0 bytes in 0 blocks


Detailed leak report
----------------------

To see more details about the leaks, rerun Valgrind with the `--leak-check=full` option:

..  code-block:: bash
    :linenos:

    cd /scratch/vp91/$USER/intro-to-gprof/code/valgrind

    gcc -g -O0 5_leaks_all.c -o 5_leaks_all


Run Valgrind

..  code-block:: bash
    :linenos:

    valgrind --leak-check=full --show-leak-kinds=all ./5_leaks_all


* **--leak-check=full**: provides detailed information about each memory leak.

* **--show-leak-kinds=all**: shows all types of leaks: definitely lost, indirectly lost, possibly lost, and still reachable.

Sample output will be similar to:

..  code-block:: bash
    :linenos:

    ==384913== HEAP SUMMARY:
    ==384913==     in use at exit: 428 bytes in 5 blocks
    ==384913==   total heap usage: 5 allocs, 0 frees, 428 bytes allocated
    ==384913==
    ==384913== 40 bytes in 1 blocks are definitely lost in loss record 1 of 5
    ==384913==    at 0x4C39185: malloc (vg_replace_malloc.c:442)
    ==384913==    by 0x400617: main (5_leak_all.c:16)
    ==384913==
    ==384913== 80 bytes in 1 blocks are indirectly lost in loss record 2 of 5
    ==384913==    at 0x4C39185: malloc (vg_replace_malloc.c:442)
    ==384913==    by 0x400633: main (5_leak_all.c:21)
    ==384913==
    ==384913== 88 (8 direct, 80 indirect) bytes in 1 blocks are definitely lost in loss record 3 of 5
    ==384913==    at 0x4C39185: malloc (vg_replace_malloc.c:442)
    ==384913==    by 0x400625: main (5_leak_all.c:20)
    ==384913==
    ==384913== 100 bytes in 1 blocks are possibly lost in loss record 4 of 5
    ==384913==    at 0x4C39185: malloc (vg_replace_malloc.c:442)
    ==384913==    by 0x400647: main (5_leak_all.c:25)
    ==384913==
    ==384913== 200 bytes in 1 blocks are still reachable in loss record 5 of 5
    ==384913==    at 0x4C39185: malloc (vg_replace_malloc.c:442)
    ==384913==    by 0x400664: main (5_leak_all.c:30)
    ==384913==
    ==384913== LEAK SUMMARY:
    ==384913==    definitely lost: 48 bytes in 2 blocks
    ==384913==    indirectly lost: 80 bytes in 1 blocks
    ==384913==      possibly lost: 100 bytes in 1 blocks
    ==384913==    still reachable: 200 bytes in 1 blocks
    ==384913==         suppressed: 0 bytes in 0 blocks

.. admonition:: Key Points
   :class: hint

    #. Valgrind is a programming tool for memory debugging, memory leak detection, and profiling.
    #. It helps identify memory-related issues in applications, such as memory leaks and invalid memory accesses.
    #. Compile code with debugging information using the `-g` flag to get detailed reports from Valgrind.
    #. Run Valgrind with the `--leak-check=full` option to see detailed information about memory leaks.
    #. Understand the different types of memory leaks reported by Valgrind: definitely lost, indirectly lost, possibly lost, and still reachable.
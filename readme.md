
# Compiling *minisat-2.2.0* using Visual Stidio

**Abstract**: This repository provides a Visual Studio project, which can 
successfully compile minisat-2.2.0 on Windows. You can directly generate a 
binary program using this project, or develop your own SAT applications based on
minisat API.


-----------
I attempted to find a recent minisat version for windows and 
didn't found it. Then, I googled an approach to compile minisat-2.2.0 using
Visual Stidio from the link <https://groups.google.com/forum/#!topic/minisat/61QrnOg07d4>.
All steps are arranged as below:
1. Download minisat from the website <https://github.com/niklasso/minisat> 
 or <http://minisat.se> and extract extract the files.
2. Open visual studio (I'm working with 2015, butI did it for 2012, 2010 and 2008 as 
well). Create an empty Win32 Console Application:
     + File -> New -> Project Win32 Console Application 
     + Location -> the location of the project I'll use "Location" (we will 
need later) 
     + Name -> I will use "Minisat2" 
     + Ok -> Next 
     + Choose "Empty Project", Finish 
3. Copy mtl, core, simp, utils directories to Location/Minisat2/minisat/. 
 From now we will work with visual studio: 
    + Add all header and cc files to the project
    + Pay attention to **add only one main.cc** file (usually the one under simp 
directory in order to use preprocessing) 

4. Create another directory "win7" under Location/Minisat2/minisat (under 
project directory). Copy the following files to win7/
    + [zconf.h, zdll.exp, zdll.lib, zlib.def, zlib.h]
    + Add .h and .lib to the project 
    + Put *zlib1.dll* under Location/Minisat2/minisat (This step seems unuseful)
5. Go to visual studio, and open project properties -> C/C++ -> General,  
Set Additional Include Directories as $(SolutionDir)\$(ProjectName)\; 
6. Replace staff in files: 
    + In file **IntTypes.h** replace
~~~~~cpp
#else 
#include <stdint.h> 
#include <inttypes.h> 
#endif 
~~~~~
with: 
~~~~~cpp
#elif _MSC_VER 
#include <stdint.h> 
#else 
#include <stdint.h> 
#include <inttypes.h> 

#endif 
~~~~~

   + In file **Main.cc** replace: 
~~~~~cpp
#include <zlib.h> 
#include <sys/resource.h> 
~~~~~
with: 
~~~~~cpp
#ifdef _MSC_VER 
#include <win7/zlib.h> 
#else 
#   include <zlib.h> 
#   include <sys/resource.h> 
#endif 
~~~~~cpp

and replace: 
~~~~~cpp
void printStats(Solver& solver) 
{ 
    double cpu_time = cpuTime(); 
    double mem_used = memUsedPeak(); 
    printf("restarts              : %"PRIu64"\n", solver.starts); 
    printf("conflicts             : %-12"PRIu64"   (%.0f /sec)\n", 
solver.conflicts   , solver.conflicts   /cpu_time); 
    printf("decisions             : %-12"PRIu64"   (%4.2f %% random) 
(%.0f /sec)\n", solver.decisions, (float)solver.rnd_decisions*100 / 
(float)solver.decisions, solver.decisions   /cpu_time); 
    printf("propagations          : %-12"PRIu64"   (%.0f /sec)\n", 
solver.propagations, solver.propagations/cpu_time); 
    printf("conflict literals     : %-12"PRIu64"   (%4.2f %% deleted) 
\n", solver.tot_literals, (solver.max_literals - 
solver.tot_literals)*100 / (double)solver.max_literals); 
    if (mem_used != 0) printf("Memory used           : %.2f MB\n", 
mem_used); 
    printf("CPU time              : %g s\n", cpu_time); 
} 
~~~~~
with:
~~~~~cpp
void printStats(Solver& solver) 
{ 
    double cpu_time = cpuTime(); 
    double mem_used = memUsedPeak(); 
    printf("restarts              : %-12lld\n", solver.starts); 
    printf("conflicts             : %-12lld   (%.0f /sec)\n", 
solver.conflicts   , solver.conflicts   /cpu_time); 
    printf("decisions             : %-12lld   (%4.2f %% random) (%.0f / 
sec)\n", solver.decisions, (float)solver.rnd_decisions*100 / 
(float)solver.decisions, solver.decisions   /cpu_time); 
    printf("propagations          : %-12lld   (%.0f /sec)\n", 
solver.propagations, solver.propagations/cpu_time); 
    printf("conflict literals     : %-12lld   (%4.2f %% deleted)\n", 
solver.tot_literals, (solver.max_literals - solver.tot_literals)*100 / 
(double)solver.max_literals); 
    if (mem_used != 0) printf("Memory used           : %.2f MB\n", 
mem_used); 
    printf("CPU time              : %g s\n", cpu_time); 
} 
~~~~~
After: 
~~~~~cpp
signal(SIGINT, SIGINT_exit); 
~~~~~cpp
add: 
~~~~~cpp
#ifndef _MSC_VER 
~~~~~

Before: 
~~~~~cpp
if (argc == 1) 
    printf("Reading from standard input... Use '--help' for 
help.\n"); 
~~~~~
add: 
~~~~~cpp
#endif 
~~~~~
replace: 
~~~~~cpp
 signal(SIGXCPU,SIGINT_interrupt); 
~~~~~
with: 
~~~~~cpp
#ifndef _MSC_VER 
        signal(SIGXCPU,SIGINT_interrupt); 
#endif 
~~~~~



   +  In file **ParseUtils.h** replace: 
~~~~~cpp
#   include <zlib.h> 
~~~~~
with:
~~~~~cpp 
#ifdef _MSC_VER 
#include <win7/zlib.h> 
#else 
#include <zlib.h> 
#endif 
~~~~~

PS 1: Although with these changes, the minisat can also be compiled successfully on Linux.

PS 2: Another repository <https://github.com/jasax/Minisat-2.2.0-for-windows> do similar work, i.e., compiling minisat-2.2.0 on windows, but it uses mingw instead of visual studio.

Thanks for your reading.
Best regards.
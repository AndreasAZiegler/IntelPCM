# IntelPCM

This program is based on the IntelPCM tool (http://software.intel.com/en-us/articles/intel-performance-counter-monitor-a-better-way-to-measure-cpu-utilization). And similar modified as in https://github.com/GeorgOfenbeck/perfplot

I actually tried to get the funtions measurement_start(), measurement_stop(unsigned long runs) and measurement_end() running with the current IntelPCM tool.

Install: (Mostly as in https://github.com/GeorgOfenbeck/perfplot)

-----------------------------------------------------------------------
  Install PCM
-----------------------------------------------------------------------

First make sure that PCM works on your system.
To do so go into the folder perfplot/pcm and follow the instructions for your OS.

Short version for linux:
  cd perfplot/pcm
  make
  ./pcm.x 1
 
  This will build and run the original pcm and run it outputting the timing at fixed intervals.
  If you see the output the pcm is ready to use. Otherwise resolve the issues as output by the program.
  
  If it fails to execute even after you fixed the permissons, try if it will work using root.

  I had to execute the following commands as root to get it running:

  sudo modprobe msr
  sudo echo -1 > /proc/sys/kernel/perf_event_paranoid
  sudo echo 0 > /proc/sys/kernel/nmi_watchdog

  If it still doesn't work, you may have to recompile the linux kernel with deactivated CONFIG_STRICT_DEVMEM
  On a Ubuntu Live CD, it worked without recompiling the kernel, whereas on my Arch Linux, I had to recompile the kernel, as it seems, that Arch Linux allows a smaller part of the memory to be addressed.

-----------------------------------------------------------------------
  Using the modified PCM
-----------------------------------------------------------------------  
 
The make file will also generate a .lib and a .a library
This is the modified version of PCM that is enriched with the following functions:

int measurement_init(long * custom_counters , unsigned long offcore_response0 , unsigned long offcore_response1 );
void measurement_start();
void measurement_stop(unsigned long runs);
void measurement_end();  


To use the library instrument your code such that at some point it calls init,
then as many start/stop pairs as required and finally calls end.

Init takes an array of 8 elements of long that describe the counters you want to use.
E.g. for Flops for Sandybridge from the Intel manual:
    "10H","01H","FP_COMP_OPS_EXE.X87","Counts number of X87 uops executed."
    "10H","80H","FP_COMP_OPS_EXE.SSE_SCALAR_DOUBLE","Counts number of SSE* double precision FP scalar uops executed."
    "10H","10H","FP_COMP_OPS_EXE.SSE_FP_PACKED_DOUBLE","Counts number of SSE* double precision FP packed uops executed."
    "11H","02H","SIMD_FP_256.PACKED_DOUBLE","Counts 256-bit packed double-precision floating- point instructions."

    should be passed as:
     10H,01H,10H,80H etc. etc.
If you don't want to use custom counters, just use: measurement_init(NULL, 0, 0); 
     
Offcore parameters are used if offcore response should be queried (see intel manual for codes) - but can be ignored for most users. (passing 0 ignores them)

Every cycle of start/stop will query the counters and put them internally into a list (divided by #runs passed in stop)
This list will get dumped into many files once end is called.

There will be a txt file per Core and Counter containing all numbers separated.

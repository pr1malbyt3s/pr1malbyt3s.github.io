
## Memory
In the context of computers, memory contains the current running state of a system. 
A memory's current state is only retained given a power supply and is constantly changing. 
We care about memory content because it provides the closest possible snapshot of a systems state at a past moment in time. 
Should an incident occur, the memory contents at that instantaneous point of occurence serve as the most reputable source of evidence.
Unfortunately, there is no longer a clean, safe way to capture a memory dump on Linux without the use of external utilities (prior to kernel version 2.6 series, all ranges of memory could be read directly from the /dev/mem device file). 
Given this restriction, we concern ourselves with memory usage statistics. 
Although we can't access the memory contents, the memory usage statistics provide a quantitative measurable of what may be happening on a system.  
  
__Overall Memory Usage:__
```bash
free -h
# Free displays the total amount of free and used system memory. 
# The -h option displays usage in gigabytes.
```
```bash
cat /proc/meminfo
# The cat utility is used to concatenante files and print on the standard output.
# The /proc/ directory is a pseudo filesystem, non-file object representation, for system information.
# The meminfo file reports information about the system's current memory usage.
```
__Memory Usage Timeline:__
```bash
sar -r 1 10
# The sar utility is used to report system activity information.
# The -r option is used to report memory utilizations statistics.
# The 1 and 10 are used to print information every second, ten times.
# kbmemfree = amount of free memory available in KB.
# kbmemused = amount ot used memory in KB.
# %memused = percentage of used memory.
# kbbuffers = amount of memory used as buffers by the kernel in KB.
# kbcached = amount of memory used to cache data by the kernel in KB.
# kbcommit = amount of memory in KB needed for current workload.
# %commit = percentage of memory needed for current workload in relation to total amount of memory.
# kbactive = amount of active memory in KB.
# kbinact = amount of inactive memory in KB.
# kbdirty = amount of memory in KB waiting to get written back to disk.
```
__Memory Usage Per Process:__
```bash
top
# Top allows real time process monitoring for system resource usage. Most of the fields are self explanatory, but some aren't:
# M or Shift+m sorts by memory usage
# PR = scheduling priority 
# NI = "nice" value
# VIRT = total memory consumed by a process (including data)
# RES = memory consumed by the process in RAM
# %MEM = RES value expressed as percent of total RAM
# SHR = amount of shared memory with other processes
# S = process state
# Reference: https://www.booleanworld.com/guide-linux-top-command/
```
#### Demonstration:
We'll use a small C program to take a look at what happens in memory using the above methods. The program:
```C
/* Program that allocates 100MB of memory every five seconds. */
#include <stdlib.h>
#include <memory.h>
/* Make variable to represent a MB size of data */
#define MB 1024 * 1024

int main() {
        while(1) {
                /* Use malloc to allocate 100MB memory space to pointer p */
                void *p = malloc(100*MB);
                /* Use memset to fill allocated memory space with 0s */
                memset(p, 0, 100*MB);
                /* Sleep for 5 seconds */
                sleep(5);
        }
}
```
You can save the program as memtest.c and compile it using the command:
```bash
gcc memdemo.c -o memdemo
```
Then run it using the command:
```bash
./memdemo
```
You can bounce back and forth between the different programs to get a feel for observing noticable changes in memory usage. 
 Be sure to kill the process before it eats up all your memory.

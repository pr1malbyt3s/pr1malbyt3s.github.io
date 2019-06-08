## Processes
A process is commonly defined as a running instance of a computer program.
Each process has a set instructions, which are executed in memory, that tell the system what to execute.
Processes also have multiple attributes, such as a process ID (PID), user, etc.
Analyzing processes gives information about what actions and/or programs are currently running on a system.
Knowing this information is useful in identifying evidence of compromise because if we know what processes are normally ran on a system, we can identify unusual processes when they exist.
Additionally, being familiar with process attributes will help us confirm unusual attributes for both normal and unusual proccesses.  
  
__Running Process List:__
```bash
ps -ef
# PS gives a current process snapshot. Most of the fields are self-explanatory, but some aren't:
# C = processor utiliztion expressed as integer value percent usage over process lifetime
# TTY = controlling terminal
# Note: TIME is the cumulative CPU time, not total run time 
```
```bash
ps auxf
# %CPU = CPU time used divided by the process run time
# %MEM = percentage expression of processes resident set size to machine physical memory
# VSZ = virtual memory size of process (KiB)
# RSS = resident set size, the non-swapped physical memory used (KB)
# STAT = muti-character process state
```
__Process Attributes:__
```bash
ls -alh /proc/<PID>/cwd
# The cwd directory underneath an individaul process directory in /proc/ gives the current working directory for the process when it was started.
# The ls utility is used to list directory contents.
# The -a option is used to include ALL files, not ignoring entires starting with .
# The -h option is used to display contents with sizes in human readable format.
# The -l option is used to display contents in a long listing format.
```
```bash
ls -alh /proc/<PID>/fd
# The fd directory underneath an individual process directory in /proc/ gives the file descriptors for the process.
# A file descriptor is essentially a number that represents an open file.
```
```bash
cat /proc/<PID>/cmd
# The cmd file underneat an individual process directory in /proc/ gives the command used to start the process, including parameters.
```
__Open Files for Process:__
```bash
lsof -p <PID>
# The -p option is used to select listings of files for a specific process ID.
```
```bash
lsof -c <PROCESS_NAME>
# The -c option is used to select listings of files for a specfic process name.
```
__Process Statistics:__
```bash
top
# Back to our good old friend, top. This time we'll use shortcuts to do deeper analysis.
# P or Shift+p sorts by CPU usage
# N or Shift+n sorts by PID
# T or Shift+t sorts by running time
# Can use m and t to change top display
# Reference: https://www.booleanworld.com/guide-linux-top-command/
```
__CPU Utilization Timeline:__
```bash
sar -u 1 10
# The -u option is used to report CPU utilization.
# %user = percentage of CPU utilization that occurred while executing at the user level.
# %nice = percentage of CPU utilization that occurred while executing at the user level with nice priority.
# %system = percentage of CPU utilization that occurred while executing at the system level.
# %iowait = percentage of time that the CPU or CPUs were idle during which the system had an outstanding disk I/O request.
# %steal = percentage of time spent in involuntary wait while another process was being serviced.
# %idle = percentage of time that the CPU or CPUs were idle and the system did not have an outstanding disk I/O request.
```
  
#### Demonstration:
To practice identifying unusual processes, we will make our own "malicious" program and run it so that it becomes a process. 
Using the text editor of your choice, create a new script titled 'firefox'  with the following contents:
```bash
#!/bin/bash

while true; do
        echo "This is not malicious, I promise."
        nc -nvlp 2222
done
```
If you're not using Firefox, go ahead and open a new browser window. 
Take a look at the command titled 'firefox' in top and get a quick familiarity with its attributes.
Using the PID for the process, use some of the other methods mentioned to explore the 'firefox' process.
Leaving the firefox browser open, we're going to run our "malicious" program and identify differences between the two that may tip us off.
Run your "malicious" 'firefox' program using the following command in the directory it is saved to:
```bash
./firefox
```
Now open up top again and filter for processes titled 'firefox' by pressing the 'o' key and then entering the filter 'COMMAND=firefox'.
This will bring up both 'firefox' processes. 
You can probably already tell which process is which, but utilize some of the other utilities to identify some other noticable differences.

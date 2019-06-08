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

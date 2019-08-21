---
layout: default
title: Temporary Files
parent: Linux Analysis
nav_order: 2
---

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

---
layout: default
title: Logs
parent: Linux Analysis
nav_order: 2
---

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

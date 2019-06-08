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

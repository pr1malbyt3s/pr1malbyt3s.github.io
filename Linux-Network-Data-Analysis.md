---
layout: default
title: Network Data
parent: Linux Analysis
nav_order: 2
---

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

---
layout: default
title: Host Firewall
parent: Linux Analysis
nav_order: 2
---

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
Try pinging the Google DNS server again. This time you won't receive an error message, but you won't see any responses either because by default, inbound packets are still dropped. Make sure to hit 'Ctrl+c' to stop the process and you'll get your ping statistics. 
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

---
layout: default
title: Linux Analysis
nav_order: 2
has_children: true
permalink: /Linux-Analysis
---

# Linux Analysis Reference

*The purpose of this guide is to serve as a reference for baseline analysis tasks when investigating a Linux system.*  

A couple of notes worth mentioning:
1) Currently, this reference is designed for Red Hat Enterprise Linux (RHEL) and CentOS, but does contain crossover to other Linux versions.
2) Although the listed tasks are useful in forensic investigations, the end goal is initial forensic inference based on an identified collection need, not a final determination.
3) The sections are taught in order of predicted volatility, or the likelihood of data disappearance should a system's state change. Although the method used in each section may not directly correlate to the order, the decision was made to keep section integrity.
4) Each method given is designed to use native utilities, or those included by default. This guide purposefully avoids non-native utilities, as they are not always available.

---
layout: post
title:  "CPU C P T states"
date:   2017-04-09 11:28:30 +0800
categories: blog
---
All the states presented in the title are about power management.

According to [intel-t-state][intel-t-state], it is now `not relevant` to newer cpus..

According to [intel-p-c-state][intel-p-c-state], p-state is not harmful to HPC(high performance computing)..
Also according to that article, comparing to p-state's `execution power saving`, c-states are `idle power saving`,
means it is idle.

When running in C0 state, processors are actually not idle and executing.

We have c-state, also there's a pc-state, means package idle state. All cpus in a package should be in the same idle
state.

According to [intel-p-c-state][intel-p-c-state], we have 2 c-states, c1 and c6 (of course there's a c0). But according
to ![cpu-c-states-img][cpu-c-states-img], there are more states..

[intel-t-state]:https://software.intel.com/en-us/blogs/2013/10/15/c-states-p-states-where-the-heck-are-those-t-states
[intel-p-c-state]:https://software.intel.com/en-us/articles/power-management-states-p-states-c-states-and-package-c-states
[cpu-c-states-img]:https://raw.githubusercontent.com/sansna/sansna.github.io/master/_pic/cpu-c-states.png
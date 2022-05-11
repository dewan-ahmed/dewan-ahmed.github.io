---
title: "Principles and patterns - A tale of infrastructure as code"
date: 2022-05-10T06:50:00-04:00
author: Dewan Ahmed
header:
  teaser: "/assets/images/iac-part2.jpg"
tags:
- infrastructure-as-code
- devops
---

A few years ago, I wrote about common objections to infrastructure-as-code (IaC) and how [stability comes from making changes](https://www.dewanahmed.com/stability-from-iac/). If your leadership is now convinced to adopt IaC, how do you implement this in your organization? In the second part of the IaC blog series, I will discuss the principles and patterns of IaC.
For reference, the following table lists things found on a server:
 
| Type  | Description  | Keep in mind  |
|---|---|---|
| Software  | Applications, libraries, and other code  | Makes sure it's the same on every relevant server  |
| Configuration  | Files used to control how the system and/or applications work.  | Make sure it's consistent and correct  |
| Data  | Files generated and updated by the system, applications, and so on. It may change frequently  | Naturally occuring and changing; may need to preserve it, but won't try to manage what's inside  |
 
 
## Infrastructure as Code Principles

The ability to effortlessly create, destroy, and rebuild servers and other related resources depends on sound principles.

The first principle is that **systems can be easily reproduced**. Your IaC tooling/scripts should handle the build/rebuild of systems, and your engineers should not waste time arguing about how to choose a hostname.

The next principle states **systems are disposable** which is related to the common expression *cattles, not pets*. In large-scale cloud infrastructure, the failure of underlying hardware is not a matter of *if* but *when*. Software should continue running even if some of the underlying hardware is modified. Service continuity in the cloud era depends on embracing disposable infrastructure.

In order to trust your automation, the third principle is **systems are consistent**. If your test environment and the production environment differ (in terms of vCPU, memory, or network performance), you cannot predict the reliability of your applications. Inconsistent infrastructure leads to configuration drift and vice versa. 

The fourth principle is **processes are repeatable**. Effective infrastructure teams have a strong automation culture. It's easier to click-ops* a one-off request rather than writing (and testing) a script. But what if you have 20 future requests to do the same task? Manual processes are also enemies of consistent systems, as you might not recall the exact configuration.

The final IaC principle is **design is always changing**. One way to ensure that a system can be changed safely and quickly is to make frequent changes. The benefit of a dynamic infrastructure is that change is not dreaded; it's expected.

## Infrastructure as Code patterns

While it's important to know the principles of infrastructure-as-code, you also need to know the patterns of provisioning, updating, and decommissioning servers. The patterns mentioned for servers can be applied to other infrastructure resources as well.
 
The first pattern is to **create a server template image** when provisioning servers that can be used to create multiple servers. Templates can be as simple as specifying an operating system image or customized with all the software and packages that will run on that server. As much as your ops team loves, do not hot clone servers (that's an anti-pattern). They are one of the main reasons for server sprawl and configuration drift. Cloned servers also suffer from all the runtime data from the original servers.
 
The second pattern is around updating servers, and that says **server updates should be done through automation**. A team member should commit a change, and that change should update the servers if the commit passes all tests. There can be a manual check in place for production servers, but the engineering team should have a high degree of confidence in the tests that are put in place. This pattern also applies when decommissioning servers.
 
The process of updating servers should be the same **regardless of how many servers are affected by this change**. This is the third IaC pattern that relies on the design of your system and the scalability of the IaC tool you're using.
 
To avoid configuration drift, the fourth IaC pattern is based on immutability. The configuration files, system packages, and software are not modified on the running server instance once a server is created from a template. In case of a required change, **a new server instance is created from the modified template rather than modifying the existing server instance**. Of course, immutable servers aren't truly immutable. The transitory runtime state, application and system data constantly change among running servers.
 
Continuous synchronization helps to maintain the discipline for a well-automated
infrastructure. Continuous synchronization model means that **a tool runs on an unattended schedule, usually at least once an hour, and it applies the current set of definitions**. This prevents configuration drift or at least catches the drift fast. This is the fifth IaC pattern. Writing and maintaining configuration definitions, however, is time-consuming. There is a limit to how much of a server's surface area can be reasonably managed by definitions. Any areas not explicitly managed by configuration definitions might be vulnerable to configuration drift. When using continuous synchronization, it's important to be aware that the majority of a server will not be managed by the configuration definitions. If you find an urge to modify parts of an unmanaged server, remember the second pattern around updating servers.
 
## Wrap up
 
In this blog, I covered some of the main principles around Infrastructure-as-Code (IaC) and a tiny subset of the patterns to implement IaC in your organization. The above is not an exhaustive list, but rather the most important ones to help you get started. In my next blog of the IaC blog series, I'll talk about the best practices on this topic.
 
I really hope to not make you wait two years this time ;)
 
 
 
---
 
 
 
*click-ops: An ops task that is accomplished by clicking on a GUI rather than automation.

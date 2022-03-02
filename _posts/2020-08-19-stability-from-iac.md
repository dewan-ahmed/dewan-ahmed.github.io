---
title: "Stability comes from making changes - A tale of infrastructure as code"
last_modified_at: 2020-08-19T0:00:00-05:00
author: Dewan Ahmed
tags:
  - infrastructure-as-code
  - devops
---

When mentioning **code**, it’s commonly understood as application code. What about the system your application runs on? Infrastructure as Code(IaC) applies the same software engineering principles used for application code to the infrastructure. Simply put, it means to codify your IT infrastructure to better manage large-scale, distributed systems and cloud-native platforms. In part one of Infrastructure as Code blog series, I talk about the need for dynamic infrastructure to be able to handle change and point out some of the common objections to Infrastructure as Code.

> The only constant in life is change - Heraclitus

In a production system, changes possess the biggest risk. However, changes are also inevitable due to the pace of agile engineering projects. Therefore, making changes is the ONLY way to improve the resiliency of your cloud infrastructure.

In the traditional approach, an application developer “asks for” an infrastructure for their code to run by opening a ticket. A team of operation folks provision that infrastructure either manually or running some scripts (which are customized to each ticket request). Now imagine a large team of developers working on a product that requires them to tear down the infrastructure frequently and provision new ones. The manual ticketing approach would soon become a bottleneck as the project would scale. With wide adoption of cloud, speed cannot be compromised when allocating required resources to the development team. This calls for a dynamic infrastructure where:

- System configuration is not only written as code but also version controlled
- Systems are provisioned and torn down frequently
- Change is expected rather than controlled

If you’re already planning to pitch about Infrastructure as Code to your customers, here are three common objections you might face:

### “We don’t make changes often enough to justify automating them”

Your production code likely has many dependencies. Even if the code for one particular component does not change, one cannot ignore the possibility of changes in other parts of the application, or the platform, or the infrastracture. Some common examples of changes:

- Performance profiling shows that the current application deployment architecture is limiting performance. You need to redeploy the applications across different application servers. Doing this requires changes to the clustering and network architecture
- Your web servers experience intermittent failures. You need to make a series of configuration changes to diagnose the problem. Then you need to update a module to resolve the issue
- An essential new application feature requires adding a new database

### “We should build first and automate later”

Automation should enable faster delivery, even for new systems. Adjusting to Infrastructure as Code model for an existing system offers a much steeper curve than a greenfield automation. Automation is a part of a system’s design and implementation. To add automation to a system built without it, you need to change the design and implementation of that system significantly.

### We don’t have a < insert your favorite IaC tool here > expert and our system is running JUST FINE

This is similar to the above point where your design vision should encompass future scaleabity. Just because your team can work through manual provisioning does not guarantee that there won’t be future bottlenecks. If your team is in the business of building software products on cloud; investing time/$$$ on peoplewho can support the IaC journey will be one of the best investment you can make.

To recap,

- Define everything as code; even your system that your applications run on.
- Changes are inevitable. Embrace changes by embracing a dynamic infrastructure.
- Understand the objections towards IaC and prepare to reason with those.

#### Reference
Infrastructure as Code, 2nd Edition by Kief Morris

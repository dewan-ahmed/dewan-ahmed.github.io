---
title: "When to go K8s-native - a tale of CI/CD servers"
last_modified_at: 2020-09-21T0:00:00-05:00
author: Dewan Ahmed
tags:
  - kubernetes
  - devops
  - cicd
---

## Why CI/CD?

The modern software delivery is **FAST**. Most of the software companies will claim to work in an agile way while having good DevOps practices. However, the “fastness” of software delivery for the industry is a wide spectrum. It can range from hundreds of deployments every day via complex CI/CD pipeline to having a manual upload of .war files to some servers.

Even without considering the speed of delivery, as the codebase and number of engineers pushing code grow, your team will require some CI/CD process to avoid bottlenecks (i.e. pinging the “DevOps Guy” to upload the binary to the prod server). The main benefit of continuous integration and delivery is neither the speed of delivery nor the ease of collaboration. It is the fast feedback loop. The faster you can ship your product (even with bugs), the faster you can identify the bugs and fix ‘em (or get feedback on improving the features). In this blog, I’ll write about the dilemma clients face when choosing CI/CD tools and make a few comparisons between traditional CI/CD tools and K8s-native CI/CD tools.

## What are your options?

> Would you like some orange juice?   


–> Sure!  


> Organic or regular?  


–> Regular.  


> With or without added calcium?  


–> With calcium pls.  


> No pulp, some pulp or lots of pulp? Brand-name or no-name? Would you like it cold or normal temperature?  

Whether it’s a consumer product or a software product, MORE is not always helpful. Having choices is good but too many choices can lead to confusion. If you’re trying to find the right open-source CI/CD tool for your team, [these](https://landscape.cncf.io/category=continuous-integration-delivery&format=card-mode&grouping=category) are your options (and more projects are added every month).

You might be listening to a DevOps talk from Netflix where they mention about their complex build pipelines with hundreds of deployments every day. You might be reading an AWS whitepaper on latest container security and how to integrate that into the CI/CD pipeline. But then you feel lost when you come back to your product - a Java monolith with two developers pushing code every week.

So the answer to the choice of the right CI/CD tool is:

> It depends.

## Kubernetes or not, THAT is the question.

There are many avenues to tackle the question of the right CI/CD tool. I’ll answer based on the software architecture and deployment platform. So whether you need a K8s-native CI/CD tool is not the first question to ask. The first two questions I ask the clients I talk to:

Are your apps microservices and runnings as containers?
Are you using Kubernetes?
Whether to choose Kubernetes or not is not in the scope of this blog but [this medium article](https://medium.com/better-programming/why-not-use-kubernetes-52a89ada5e22) can be a good start. 

Once you decide that containers and orchestration tools like Kubernetes can be a good fit for your software product, you can compare K8s-native CI/CD tools with traditional CI/CD tools.

## Comparing K8s-native CI/CD with traditional CI/CD

The following table summarizes the challenges around a traditional CI/CD server and makes comparison to a K8s-native CI/CD server:

| Traditional CI/CD Server  | Kubernetes-native CI/CD  |
|---|---|
| Logging and monitoring is on an external agent  | Centralized logging and monitoring  |
| High-availability is not possible or complex to achieve  | The platform guarantees high-availability  |
| Self-healing is not possible (retry logic for some tools)  | Self-healing comes by default –resources are pods  |
| Most common CI/CD servers are from a pre-container era  | Developed for containers and Kubernetes; runs as containers on Kubernetes  |


## Challenges around K8s-native CI/CD adoption

There is no free lunch. The benefits of a K8s-native CI/CD server come at a cost. It is paramount that you know of these challenges before going all in.

- Steep learning curve: Kubernetes is hard. Since the pre-requisite is Kubernetes adoption, transitioning from traditional tools might take some time.
- Fairly new technology: Most K8s-native CI/CD tools have been out only for a few years and have not GA-ed yet. So these tools are not “battle-tested” yet and very few companies are running them in production. This also means that there will be limited resources available online for these tools.
- Resistance: If your team is already using an established CI/CD tool, there might be resistance towards a new tool.

## Next steps?

Now that you know the benefits as well as the challenges around a K8s-native CI/CD tool, here are the three steps you can follow for a smooth transition:

1. Research: As a team, do your own research if the K8s-native tool is a good fit for your product.
2. PoC: Play around with the tool with non-production codebase. This is the time to explore it’s capabilities and weaknesses. This is also a good time to learn about the community and the level of support available.
3. Go live (with caution): If you’re satisfied, start using the tool with production codebase BUT with a backup plan, in case you need to rollback.
If you’ve enjoyed reading this blog and would like to get started with a popular K8s-native CI/CD tool, you can give [Tekton](https://tekton.dev) a try. [This repo](https://github.com/dewan-ahmed/java-tekton-demo) contains a hands-on tutorial.


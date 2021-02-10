---
title: "7 OpenShift Questions and 8 Answers"
date: 2021-01-29T20:14:00-04:00
author: Dewan Ahmed
categories:
  - Blog
tags:
  - OpenShift
  - Kubernetes
---

Few weeks ago, I gave an IBM-internal talk on [OpenShift](https://www.openshift.com/). During my favorite part of the talk (the Q&A), a number of good questions came up. I took away some of the questions I didn't have answers for, as a to-do and gathered answers from my Red Hat colleagues. This blog is a collection of those questions and their answers as the information would be useful to many others. If you'd like to improve any of the answer(s), please [reach out to me](https://linkedin.com/in/diahmed) and I'll update this post. 

![Photo by Edwin Andrade on Unsplash](/assets/images/openshift-questions-image.jpg)

> Is OpenShift the same as Kubernetes?

If Kubernetes is the engine, OpenShift is the car. Just like you cannot drive using the engine itself, you'll need a bunch of other services (for monitoring, storage etc.) alongside container orchestration (a.k.a. Kubernetes). A number of these essential services come out of the box with OpenShift and you have the option to [install more](https://docs.openshift.com/container-platform/4.1/applications/operators/olm-understanding-operatorhub.html) based on your project needs. Some specific mentions: *RBAC* and *Networking*; i.e. try to implement using vanilla Kubernetes versus how OpenShift provides the out-of-the-box capabilities. 

At the heart of OpenShift IS Kubernetes, and that it is a [100% certified Kubernetes](https://www.cncf.io/certification/software-conformance/), fully open source and non-proprietary, which means:
- The API to the OpenShift  cluster is 100% Kubernetes.
- Nothing changes between a container running on any other Kubernetes and running on OpenShift. No changes to the application.
For a more detailed answer, you can read [this excellent blog](https://www.redhat.com/en/blog/openshift-and-kubernetes-whats-difference).

> Any good docs on how to design/deploy apps to OpenShift clusters?

Estimating the cluster in terms of HA and designing your projects/apps in terms of performance/scalability is a massive topic. This is also a direct factor of the type of apps you deploy and what kind of resiliency you expect. To get started, you can refer to [this doc](https://docs.openshift.com/container-platform/4.6/scalability_and_performance/planning-your-environment-according-to-object-maximums.html).  

> Are cluster services part of the platform? Or workload built on the platform?

If we're talking about [Kubernetes Services](https://kubernetes.io/docs/concepts/services-networking/service/), that is considered a workload which you can find under _Networking --> Service_ under Administrator perspective on your OpenShift cluster. If the question is more along the way how OpenShift manages the core services for its operators; the answer starts with Operator Lifecycle Manager (OLM). Beginning OpenShift 4.X, the OLM helps users install, update, and manage the lifecycle of all Operators and their associated services running across their clusters. It is part of the [Operator Framework](https://github.com/operator-framework), an open source toolkit designed to manage Kubernetes native applications (Operators) in an effective, automated, and scalable way. For more details on OLM, please [read this OpenShift doc](https://docs.openshift.com/container-platform/4.1/applications/operators/olm-understanding-olm.html).

> How are SSL/TLS handled on OpenShift? If I deploy an app, do I have to configure these manually or done by default?

It depends on how you configure the OpenShift route you create for that app (i.e. the service).  [This blog](https://www.openshift.com/blog/self-serviced-end-to-end-encryption-approaches-for-applications-deployed-in-openshift) is an excellent source to learn various OpenShift route configuration (i.e. whether the platform or the developer handles SSL/TLS).

> What firewall rules should be open to the internet if the OCP cluster is on-site?

Usually 443 or 80 unless you are doing [NodePort](https://docs.openshift.com/container-platform/4.6/networking/configuring_ingress_cluster_traffic/configuring-ingress-cluster-traffic-nodeport.html).

> Does OpenShift have a way to encrypt kube secrets at rest (in etcd) and as they are being made available to the pods (in transit)?

[ETCD](https://docs.openshift.com/container-platform/4.6/security/encrypting-etcd.html) can be encrypted for securing the data at rest.  There is something called "sealed secrets" that might be what you'd need for pods. For more information on using "sealed secrets" on OpenShift, please read [this blog](https://www.openshift.com/blog/gitops-secret-management). 

> How many apps/pods should I run per OpenShift project?

This is another _it depends_ answer.  [OpenShift Docs](https://docs.openshift.com/container-platform/3.9/scaling_performance/cluster_limits.html) indicates cluster limits which are the maximum numbers but whether anything below that is going to be "usable" is going to depend a lot on the app(s) in question.

These were the 7 answers of the 7 questions I took away from my talk. The 8th answer is my own learning over the recent years: whether we're maintaining a legacy system or choosing a shiny new tool, we should always keep the end-user in mind when making technical decisions. At the end, it's their experiences with the product that matters.

---
title: "StackConf 2022 Conference - Reflection"
date: 2022-08-12T00:50:00-04:00
author: Dewan Ahmed
header:
  teaser: "/assets/images/stackconf-dewan-talk.jpg"
tags:
- devrel
---

<!-- Google tag (gtag.js) -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-6Y8EX6JQCW"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'G-6Y8EX6JQCW');
</script>

Last month I travelled across the pond to attend and give a talk at [StackConf EU 2022](https://www.youtube.com/watch?v=I_xBBGeFULU) - an open-source infrastructure-focused conference. After two virtual events in 2020 and 2021, the conference took place live this year in the capital city of Berlin. The two-day conference was packed with exciting talks from industry leaders in the areas of cloud, DevOps, databases, machine learning, and other areas. You can check out the [conference archive](https://stackconf.eu/archives/2022-2/) for recordings of all the talks. In this blog, I write about my favorite talks and moments from StackConf. 

![Speakers' dinner](/assets/images/stackconf-speakers-dinner.jpg)

<p align = "center">
Speakers' dinner on the evening before the conference
</p>

## July 19 - Talks to talk about

The first talk I'm excited to mention is "Scaling the Grail – Cloud-Native Computing on Encrypted Data using Carbyne Stack". I met [Sven Trieflinger](https://twitter.com/SvenTrieflinger) on the evening before the conference during the speakers' dinner. Sven mentioned that Carbyne Stack relies on Secure Multiparty Computation (MPC) to keep data encrypted end-to-end: in transit, at rest, and in use. Although started at Bosch, the [Carbyne Stack](https://github.com/carbynestack/carbynestack) project is now open-source and is gaining traction to build scalable infrastructure for sensitive data workloads.


Whether it's a one repo for all or having no test for infrastructure, [Kris Buytaert](https://twitter.com/KrisBuytaert) talked about the different patterns and anti-patterns
we've seen over the past decade of automation. It was reassuring to see that some of the points I was going to mention in my talk are repeated in other speakers' talks as well - for example, the challenges around configuration drift for infrastructure as code.

![Kris Buytaert talk](/assets/images/stackconf-kris-talk.jpg)

<p align = "center">
Kris Buytaert talk
</p>

[Peter Zaitsev](https://twitter.com/PeterZaitsev), the founder and CEO of Percona, gave a high-level talk on databases and storage in the cloud. Here are the types of storage to consider, according to Peter:

- Node local storage
- Network attached block storage
- Network file system
- HTTP(S) accessible object store
- Queues/Streams/Pipelines
- Databases

![Peter Zaitsev talk](/assets/images/stackconf-peter-talk.jpg)

<p align = "center">
Peter Zaitsev talk
</p>

Peter mentioned "shapechangers", i.e., some databases that can speak multiple protocols. For example, ClickHouse can speak PostgreSQL and MySQL protocols, while FerretDB allows you to use PostgreSQL as if it were MongoDB, etc. It was nice to see a mention of Aiven in Peter's slides.

[Dr. Dawn Foster](https://twitter.com/geekygirldawn) from VMware talked about the dynamics of collaboration among individuals, companies, and communities in her talk "How to Be a Good Corporate Citizen in Open Source". She emphasised that community comes **before** company or individual needs. It's better for the company to align their goals with the community/project goals rather than try to push something that the community/project does not benefit from. Most folks in the room were quick to snap a photo of the reference slide that Dawn shared, which included the following useful links:

- [The Linux Foundation TODO Group](https://todogroup.org/)
- [TAG Docs & Templates for CNCF Contributor Strategy](https://github.com/cncf/project-template)
- [The Open Source Way Guidebook](https://github.com/theopensourceway/guidebook)

## July 20 - Talks to talk about

The next morning, I kicked-off StackConf with my own talk, "Do NOT click-ops your data infrastructure". I hope it's okay for me to shamelessly plug my talk. In my talk, I made a point about using automation tools like Terraform to build and manage data infrastructure rather than manually creating the resources. During the demonstration, I used [Aiven for Apache Kafka®](https://developer.aiven.io/docs/products/kafka.html) and [Aiven for Apache Kafka® MirrorMaker 2](https://developer.aiven.io/docs/products/kafka/kafka-mirrormaker.html) to demonstrate cross-cluster replication and how [Aiven Terraform Provider](https://developer.aiven.io/docs/tools/terraform.html) can be used to deploy all resources in your preferred cloud. You can check out the recording of my talk [on YouTube](https://www.youtube.com/watch?v=YBxt5uLz00I).

![Dewan Ahmed talk](/assets/images/stackconf-dewan-talk.jpg)

<p align = "center">
Dewan Ahmed talk
</p>

I got into a real dilemma when two of the talks I wanted to attend were happening at the same time. [Dotan Horovits](https://twitter.com/horovits) from Logz.io, a seasoned developer advocate, had a great talk "Open Source for Better Observability" and [Sayantika Banik](https://twitter.com/sayabanik) from Quansight was covering data pipelines using open-source.

![Dotan Horovits talk](/assets/images/stackconf-dotan-talk.jpg)

<p align = "center">
Dotan Horovits talk
</p>

I really liked how Dotan broke down the open-source observability into "what", "why", and "where":

- Metrics (the "what"): Prometheus, OpenMetrics, Grafana
- Logs (the "why"): OpenSearch
- Traces (the "where"): Jaeger, Zipkin, Apache SkyWalking®

Sayantika had "fun" in her talk title, so that drew a lot of folks to her talk. She used [Dagster](https://github.com/dagster-io/dagster) - an open-source orchestration tool for the development, production, and observation of data assets. If you'd like to learn about the basics of data pipeline by doing so, check out [her GitHub repo](https://github.com/sayantikabanik/DataJourney).

![Sayantika Banik talk](/assets/images/stackconf-sayantika-talk.jpg)

<p align = "center">
Sayantika Banik talk
</p>

My friend and ex-colleague [JJ Asghar](https://twitter.com/jjasghar) from IBM gave a super interesting talk titled "We accidentally created a Serverless Application". I took a ride from JJ on my way from the airport, so I was indebted enough to attend his talk ;). While waiting for the talk to start, JJ amused the audience with an unlimited supply of dad-jokes as GIFs. JJ mentioned the challenges where he and the team supported Kubernetes cluster creation requests and the entire process was manual. JJ's talk was a step-by-step recollection of how he made the process automated, and that was the serverless application he developed "accidentally". 

![JJ Asghar talk](/assets/images/stackconf-jj-talk.jpg)

<p align = "center">
JJ Asghar talk
</p>

[Laura Ham](https://twitter.com/laura_hamham) from SeMI Technologies asked a question that if you do a text search on a database for the word "Python" and there are no computer-related books with that title, the search will show books related to Python snakes. But with a vector database like Weaviate, the search can be a semantic search instead of a traditional text-based search. Besides showing how to perform your first semantic search with the vector database Weaviate, Laura covered other functionalities of Weaviate, like multi-modal search, data classification, connecting custom ML models, etc. 

![Laura Ham talk](/assets/images/stackconf-laura-talk.jpg)

<p align = "center">
Laura Ham talk
</p>

## Reflecting back

![Dewan with Markus, the host and conference organizer](/assets/images/stackconf-dewan-markus.jpg)

<p align = "center">
Dewan with Markus, the host and conference organizer
</p>

StackConf and a few other in-person conferences have been a great change after more than two years of virtual conferences. The confused looks or nodding of heads from the audience, the hallway chats, and great food make in-person conferences like StackConf a great success. I really hope that the flight delays and cancellations don't become killjoys to the excitements of in-person conferences.
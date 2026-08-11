---
title: "The Third Reset: Becoming a Developer Advocate in the AI Era"
date: 2026-08-11T00:00:00-04:00
author: Dewan Ahmed
header:
  teaser: "/assets/images/2026/nguyen-dang-hoang-nhu-rmCrHocHG88-unsplash.jpg"
tags:
  - devrel
  - career
---

# How to Become a Developer Advocate in the AI Era

Someone asked me recently what advice I'd give to a person trying to break into DevRel in the AI era. I gave an answer on the spot, and it bothered me for a week afterward, because it was the wrong shape of answer.

Here's the right one. The AI era is not a new job. It's the third reset of the same job, and it only looks unprecedented if you weren't paying attention during the first two.

Ask five experienced developer advocates what to focus on and you'll get seven different answers, all correct. Learn AI. Learn AI. Learn AI. Learn video. Learn marketing and events, because that's what the postings actually ask for. Learn agentic development. Learn how to size up an organization so you don't become the first DevRel hire at a ten person company that has no idea what it just bought. When five correct answers point in five directions, you don't have advice. You have noise. What's missing is the model that explains why they're all correct at once.

I started my big tech career at IBM in 2014, spent 2020 to 2022 at Red Hat, and have been at startup unicorns since. I've also been organizing conferences and meetups for the last decade or so, which means twice a year I read a stack of CFP submissions and watch the industry's obsessions turn over in real time, usually about nine months before they show up in job descriptions. From that seat, the pattern is hard to miss.

I want to explore five things in this blog:

1. The three eras of developer advocacy, what changed at each reset, and what has never changed at all.
2. How the demos we build changed, including the sharp break of the last three years.
3. How DevRel measurement shifted across the eras, and why the AI era quietly broke the metrics most teams still report.
4. The uncomfortable truth: you are not hired for what you value. You are hired to remove a cost.
5. How to tell whether the job you're being offered is one where the work is even possible.

## Three eras, three complete resets

### Era 1: Before the cluster

Roughly 2008 to 2014. The title was usually "developer evangelist," and the product surface you advocated for was small enough to hold in your head: a REST API, an SDK in four languages, a quickstart, and a set of docs.

The distribution channel was **people in rooms**. Conference stages, user groups, hackathons, a book, a blog on your own domain, a mailing list. Companies like Twilio, SendGrid, EngineYard, and New Relic took the model mainstream by putting engineers, not marketers, on stage in front of other engineers.

When I joined IBM in 2014, the hardest part of the job was explaining that the job existed. "What do you do at IBM?" never ended in one sentence. I eventually got so tired of the explanation that I [wrote a blog post](https://medium.com/@dewanahmed/what-i-do-as-a-developer-advocate-at-ibm-da2252179f6) I could just send people. Some of my colleagues then were engineers who had spent twenty years shipping software and chose advocacy over management or a principal engineer track. A few of them retired out of the role. That was the profile: [not an entry level job](https://www.dewanahmed.com/why-paying-devrel/#developer-advocacy---not-an-entry-level-role), and nobody pretended otherwise.

Credibility was earned in production and spent on stage. The unit of work was **the talk**.

### Era 2: The microservices, Kubernetes, and API era

Docker landed in 2013. Kubernetes in 2014. The CNCF in 2015. Within about three years, the surface area a developer advocate had to cover went from an SDK and an API to YAML, Helm charts, operators, CRDs, service meshes, observability stacks, CI pipelines, and Terraform.

Two things changed at once, and they're worth separating.

The technical bar moved sideways, not up. Writing good application code stopped being sufficient. You had to run a distributed system, explain why it fell over, and reason about failure across a network boundary. Some developer advocates looked at Kubernetes in 2016, decided it was accidental complexity dressed up as progress, and sat that cycle out. That was a defensible engineering opinion. It was a bad advocacy decision, and the two are not the same thing. An engineer can stay valuable maintaining an unfashionable system. An advocate for an unfashionable system has no audience.

Sit with that, because it's the pattern that governs everything else in this post. **The market's demand did not need to be architecturally justified in order to be real.**

My own version of that lesson came at the transition. I came up as a backend Java developer. Around 2016 I had to decide whether the container thing was a serious shift or a fashion, and I chose to treat it as serious mostly because the CFP submissions I was reading as an organizer had gone from ten percent container talks to sixty percent in about two years. That was not a technical judgment. It was a market reading, and it was the more useful of the two.

By the time I got to Red Hat in 2020, the reset was complete and the demo had become an environment rather than a snippet. It was also the pandemic, so every event I organized or spoke at moved to a webcam, which stripped out the hallway track and left only the artifact. That accelerated something already underway.

**The distribution channel had moved to search and GitHub.** The talk stopped being the unit of work. The unit became the artifact that outlived you: the tutorial that ranked on page one, the reference architecture repo people forked, the docs page that surfaced when someone pasted an error message into Google. A conference talk was a spike. A tutorial was an annuity.

The unit of work was **the artifact**.

### Era 3: The AI era

Now look at what's actually happening, and look past the word "AI," which is so overloaded it carries almost no information.

**The product surface changed again.** It's a model, a set of tools, an MCP server, a retrieval layer, an eval suite, a context budget, and a bill that scales with tokens instead of nodes. If you advocate for a developer product today, a growing fraction of your product's usage is driven by an agent that a human is loosely supervising.

**The distribution channel changed again, and this is the part most people are underweighting.** In Era 1 you reached a human in a room. In Era 2 you reached a human through a search engine. In Era 3, a large share of your content is read by a model before a human sees it, and frequently instead of. RTFM is being replaced by the AI reading the manual on the developer's behalf. Your developer portal's job is no longer only to be readable. It has to be **retrievable and unambiguous** to something that will never scroll.

I wrote about [llms.txt](https://www.dewanahmed.com/llms-txt/) a while back and the reaction was mostly polite curiosity. It isn't curious anymore. If an agent cannot get from zero to a working integration with your product without a human intervening, you have a developer experience problem that no amount of conference presence will fix.

There's a second order effect that I think is the real story of this era. **The cost of producing a competent-looking artifact has fallen to nearly zero.** A tutorial, a demo app, a blog post, a sample repo, all of it can be generated in an afternoon by someone who understands none of it.

That breaks the Era 2 economy completely. For fifteen years the artifact was evidence. It was expensive to produce, so producing one proved something about you. It no longer does. Volume stopped signalling effort, and effort stopped signalling competence.

When artifacts stop working as evidence, the market re-prices two things: **judgment** and **verifiable outcomes**. Knowing what to build, knowing when the agent is confidently wrong, knowing the exact moment to take manual control back. That's the scarce input now. Not the typing.

The unit of work is **the judgment call**.

## The demo tells the whole story

If you want a single artifact that tracks all three eras, watch what happened to the demo.

**Era 1: the demo was proof the API worked.** A laptop, a terminal, a browser, five minutes of live coding that ended with a text message arriving on a phone in front of 300 people. Small, deterministic, genuinely persuasive because the audience saw the whole thing. Shelf life of two or three years. You could give it at eight conferences and only bump the SDK version.

**Era 2: the demo became an environment.** You weren't showing an API call, you were showing a system. A polyglot microservices app, a GitOps pipeline, a cluster that had to exist before the talk started. A week to build, half a day to reset. Half the room had seen a variation already, because the whole ecosystem was demoing against the same reference apps. The demo stopped proving your product worked and started proving your product fit into a stack. Shelf life of about a year, because a minor version bump could break it on stage. I have watched a demo die at a meetup because of a Helm chart change that shipped that morning, and the recovery mattered more to that room than the demo would have.

**Era 3, specifically the last three years, is where it gets interesting.** The trajectory ran chatbot, then RAG pipeline, then agent, then multi-agent, roughly one per year. But the version number isn't the point. Three deeper things changed:

**The demo stopped being scarce, so it stopped being persuasive.** When the audience knows anyone can generate a polished demo app in an hour, watching one generated live impresses nobody. The demo lost its function as proof of competence. That's a genuine loss and our field has not adjusted to it. Plenty of teams are still shipping Era 2 style demos and wondering why engagement is flat.

**Demos became non-deterministic.** You can no longer guarantee the same output twice. Anyone who has run a live agent demo knows the specific dread of watching the model take a path you never saw in forty rehearsals. The honest responses are to pin versions and seeds, to pre-record and narrate over it, or to lean into the non-determinism deliberately and make the recovery the point.

**Shelf life collapsed to weeks.** A demo built against a model release from six months ago is often just wrong, not merely dated. The cost structure inverted: cheap to produce, expensive to maintain. Most teams have not updated their budgets for that.

So what actually persuades a technical audience now? Three things, in my experience:

- **The demo that fails on purpose.** Show the agent getting it wrong, show how you caught it, show the guardrail. Everyone in the room has been burned by something that didn't survive contact with real data. Showing the failure mode buys more credibility than a clean run ever will.
- **The eval, not the output.** A passing eval suite is the new working demo. It survives a version bump, and it's the artifact a skeptical staff engineer actually respects.
- **The demo an agent can run without you.** If someone points their own agent at your repo and reaches the same result unattended, you've proven something about your product's agent experience that no stage performance can.

## What we measured, and what quietly broke

Metrics moved in lockstep with distribution, and the AI era broke a set of proxies most teams are still reporting.

**Era 1 measured presence.** Attendance, booth conversations, t-shirts, tweets, a warm feeling in the exec team. Nobody had solved attribution because nobody was seriously asking.

**Era 2 measured traffic and community.** Docs pageviews, search rankings, GitHub stars, forks, contributor counts, chat community size, attributed signups, time to first hello world. Imperfect proxies, but with one enormous advantage: a click was a reasonable stand-in for a human. Someone read the thing, so someone existed.

**Era 3 broke that assumption, and almost nobody has repriced accordingly.** Consider what's happening to your numbers right now:

- **Docs pageviews are falling while docs usage is rising.** The agent read your page, synthesized the answer, and the developer never clicked. Flat traffic on your best page may mean it's being consumed more than ever. Report that chart to leadership without the caveat and you are arguing against your own budget.
- **Stars and content volume are trivially manufactured.** Anything that was cheap to game in 2022 is free to game in 2026.
- **Search ranking is a shrinking proxy for reach.** The question is no longer only where you rank, but whether you are represented, and represented accurately, in the answer a model gives when your category comes up.

What replaces them, in my view, are proxies built around machine consumption and verified outcomes:

- **Agent completion rate.** Can an agent go from zero to a working integration, unattended, without hallucinating an endpoint that doesn't exist?
- **Time to first successful agentic call**, which is the Era 3 version of time to first hello world.
- **Answer share and answer accuracy.** How often does your product surface when a developer asks a model a question in your category, and how wrong is the answer when it does?
- **Docs ambiguity rate.** How many pages produce inconsistent agent behaviour across runs? This is measurable and almost nobody measures it.
- **Retrieval coverage.** What fraction of your documented surface is reachable and correct through a machine-readable path.
- Unchanged from before: **feedback that reached the roadmap**, and **technical blockers removed from real deals**.

I'll be honest about the limits. None of this is standardized, none of it has agreed benchmarks, and you will be inventing the framework at whatever company you join. That has been true in every era, and I've written at length about [why DevRel measurement is broken in a way other roles' measurement is not](https://www.dewanahmed.com/devrel-measurement-paradox/). The difference now is that the old proxies aren't merely imperfect. They point the wrong direction.

## What changed, what didn't

The compressed version:

|                 | Pre-cluster (2008 to 2014) | Microservices and K8s (2014 to 2022) | AI (2023 to now)                     |
| --------------- | -------------------------- | ------------------------------------ | ------------------------------------ |
| Product surface | SDK, REST API              | Cluster, YAML, pipelines             | Model, tools, MCP, evals             |
| Distribution    | People in rooms            | Search and GitHub                    | Retrieval layer and agents           |
| Unit of work    | The talk                   | The artifact                         | The judgment call                    |
| The demo        | Five minute live API call  | A whole environment                  | Non-deterministic, cheap, disposable |
| Measured by     | Presence                   | Traffic and community                | Agent success and verified outcomes  |
| Scarce input    | Stage presence             | Depth across a wide stack            | Taste and verification               |

Here's what has not moved a millimetre in twelve years:

**Technical credibility is still earned by building, not by talking about building.** Every era produces advocates who let the code atrophy, and every era punishes them at the next reset. This one is more dangerous, not less, because the tooling lets you appear technical far longer before anyone notices.

**Empathy still can't be faked or shortcut.** You develop it by having been the person on the other side of a broken quickstart at 11 PM.

**A great DevRel team still cannot save a bad product.** I wrote that in 2023, it's still true, and it will be true in 2030.

**Trust is still the entire asset.** As more generated content reaches the world, human connection and narrative appreciate rather than depreciate. I'd sharpen that further: the reason live work and video are gaining value is precisely the reason some people are skeptical of them, which is that models don't ingest them well. That isn't a flaw in the strategy. **That's the moat.**

Which gives you a practical framing for content. You're running two funnels now, and they optimize for opposite things. The machine funnel is docs, llms.txt, structured content, MCP servers, SDKs an agent calls correctly on the first attempt, error messages that are parseable. Optimize that for retrievability and correctness. The human funnel is video, live workshops, hallway conversations, opinions, narrative, the trip report nobody asked for. Optimize that for trust. Stop trying to make one piece of content serve both. That's why so much DevRel content right now reads like it was written for nobody in particular. It was.

## You are not hired for what you value

Now the uncomfortable part, and the actual reason I wrote this.

**You are hired to remove a cost or a risk.**

A job description is not a description of the work. It's a compression artifact of a hiring manager's anxiety, run through a recruiter who has to filter 400 applicants. When it says "experience building agentic workflows," the sentence underneath is closer to: our CEO asked what our AI story is, our docs are being consumed by models and misrepresented, activation is flat, and I need someone who won't take six months to ramp.

You are allowed to think a requirement is silly. In 2016, "must have production Kubernetes experience" was a questionable ask for a role that mostly involved writing tutorials, and smart people said so out loud. The ask was still real. The people who satisfied it got the roles. The people who wrote thoughtful posts about why the industry was overreacting got to keep writing those posts, from home, for longer than they intended.

I've watched this from both sides. Since 2022 I've been at startup unicorns, where the pressure to name a number is constant and the tolerance for "this will pay off in eighteen months" is close to zero. And from the organizer chair, I've seen the CFP pool turn over completely three times. Both views teach the same lesson: the market's vocabulary changes faster than the work does, and being fluent in the current vocabulary is not the same as being a sellout.

So hold two things at once. Know the work, which means understanding the systems for real and never becoming the person who demos something they cannot debug. And signal what is being searched for right now, not what should be searched for.

Most people in DevRel are excellent at the first and treat the second as beneath them. That's a moral position, not a strategy, and the market doesn't grade on moral positions. The people who struggle during a reset are almost never the ones who lacked skill. They're the ones whose skill wasn't legible in the vocabulary the market was currently using.

## Then check whether the job is even doable

Getting a job is not the win condition. Getting a job where the work is possible is.

Before you accept anything, find out who DevRel reports to and whether that person wanted a DevRel team or inherited one. Find out what stage the business is in and whether the proposed metrics match that stage. If you'd be the first DevRel hire, find out who defined the mandate before the requisition opened, because "we'll figure it out together" means you're being hired to invent your own job and then take the blame when it fails to produce a number nobody defined. Find out whether anyone senior will protect you when the CEO asks for ROI the week after a conference. And find out whether the product is any good, because you cannot advocate your way out of a bad one, and attempting it costs you community trust you spent a decade building.

"First DevRel hire, ten person company, no prior DevRel function, open to someone junior" is not an opportunity. It's a story you'll be telling at a meetup in eighteen months.

## Time to ship

Three eras. Three product surfaces, three distribution channels, three complete resets of what the market asked for. Underneath all of it, the same job: understand the technology well enough to be trusted, and be useful to developers in whatever medium they actually use.

What changes every eight years is the medium and the vocabulary. What never changes is that you have to build in order to stay credible, and you have to be legible in order to get hired.

So if you're trying to break in this year, pick one thing in the agentic stack and go deep enough to have opinions about its failure modes. Publish the failures, not the happy path. Write down how you'd measure your own work in the first ninety days, because almost no candidate does. And make all of it verifiable by a stranger in ninety seconds.

**Do the work that matters, and ship the proof in the format the market is currently searching for. Skipping either one is how good people end up unemployed for reasons they refuse to name.**

---

_A note on how this was made: I developed this blog with an AI assistant. The structure, the arguments, the three era framing, and the opinions are mine, pressure-tested and organized through that conversation. It is not an AI-generated post, and I'd rather tell you that directly than have you wonder._

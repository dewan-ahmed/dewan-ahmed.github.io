---
title: "From Cloud-First to AI-First: Productivity-Maxxing or Token-Maxxing?"
date: 2026-10-07T00:00:00-04:00
author: Dewan Ahmed
header:
  teaser: "/assets/images/2026/cloud-first-ai-first-cover.png"
tags:
  - ai
  - cloud
  - software-engineering
---

Imagine a company in 2016. An engineer is standing in a meeting room, trying to convince the CTO and CIO to move an application to the cloud. The engineer has prepared a demo, a cost estimate, and answers to the questions that came up in the previous meeting. There have been several previous meetings.

The CIO asks where the data will live. The CTO asks how the team will recover from an outage. Finance points out that the servers downstairs are already paid for. The engineer explains that waiting weeks for capacity has a cost too, especially when the team wants to try something small and has to ask for something large.

Everyone wants the company to do better. They disagree about which risk is worth taking. I want to follow that engineer for a bit, because the next transformation puts them in an uncomfortable position. Ten years later, they will be the person asking the questions.

## Ten years ago, getting permission was the work

Stay in that room for a moment. The engineer believes cloud could help the team ship more easily. They can picture a development environment appearing when someone needs it, an experiment that doesn't require a hardware purchase, and less time spent coordinating infrastructure before the useful work can begin.

The executives can picture a sensitive application running somewhere they don't control. Their questions force the engineer to think through access, recovery, cost, and dependence on a provider. The proposal improves because the engineer has to explain how the team would actually live with the decision.

Eventually, in our imaginary company, they agree to start with one application. It gives the team enough room to learn without making the entire business part of the experiment.

The migration still has awkward moments. A deployment assumption breaks. Somebody discovers an unexpected charge. A process that made sense in the server room needs to change. The engineer who wanted the move gets to help fix all of it.

That seems like a useful bargain to me. Enthusiasm earns you the opportunity to try something. Taking responsibility for what follows earns you the next opportunity.

Gregor Hohpe [writes about accepting some lock-in when the payoff is worthwhile](https://martinfowler.com/articles/oss-lockin.html). I like the decision that invites: work out what you're gaining, understand what it will cost to change your mind, and choose with both in view. Our engineer's cloud proposal needs that kind of judgment.

As the team learns, the new way of working becomes familiar. People stop discussing the cloud as an abstract position and start discussing the application they need to operate. The meeting room fills with other arguments.

## Back in the room, in 2026

Now bring the same company forward ten years. The engineer has more experience, a larger codebase to look after, and a healthy collection of incidents they would prefer never to repeat. The CEO has called a meeting about AI.

This time, the executives are already convinced. The CEO wants an AI strategy for the board. The CTO wants the engineers using coding agents. The CIO has started reviewing vendors, although several departments have helpfully skipped ahead to purchasing them.

Our engineer arrives with questions. Which work are we trying to improve? What data can these tools see? Who will review the changes? What happens if the team produces code faster than it can understand it?

The questions sound familiar. The skepticism has moved across the table.

I think that shift explains some of the tension around AI adoption. An executive hears an engineer raising concerns and wonders why the team is resisting a chance to move faster. The engineer hears an executive announce a productivity target and wonders how much cleanup has just been added to next quarter.

There are reasonable motives on both sides. A CEO watching competitors move has to decide under uncertainty. An engineer responsible for an application has to think about the consequences after the announcement. Each can see a cost the other might underestimate.

And the willingness to fund this work is genuinely welcome. Our engineer spent months trying to get permission for the cloud experiment. Now the budget is available before the experiment has been designed. I'd want to make good use of that change.

First, though, someone needs to explain what the company intends to do with all these tools. Procurement has had an excellent quarter. Engineering would like to have one too.

## Then someone says "software factory"

The CTO describes a more ambitious possibility. An agent picks up a task, explores the repository, prepares a change, runs tests, and brings it back for review. Over time, more of that path becomes automated. The team could take on maintenance work that keeps losing out to the next feature request.

Eno Reyes [did an X post](https://x.com/EnoReyes/status/2107858448914518174?s=20) about software improving through user feedback and making an organization's unwritten knowledge part of how its systems work. That gives our engineer something more interesting to consider: what would the team learn from each piece of work, and how would that learning help with the next one?

Our engineer is interested. There is a migration in the backlog nobody wants to do by hand. There are tests the team has been meaning to add. There are old modules that take a new hire far too long to understand. A factory that helps with those things would earn its place quickly.

Then the CTO suggests trying it on a small ticket: add retries to the payment flow. The engineer asks what happens when a payment succeeds but the response gets lost. Would the retry charge the customer again?

Now the room has a concrete problem to discuss. Someone needs to establish how the system recognizes a transaction it has already processed. Someone needs to decide what the customer sees while the result is uncertain. The agent can help find the relevant code and investigate the behavior. The team still needs to agree on the promise it is making.

Our engineer knows why this matters because they've been through a similar incident before. The important detail lives in their memory and a comment buried in an old ticket. Getting it into a regression test would help the next engineer and the next agent. Asking the experienced person to remember to mention it every time is a fairly fragile operating model.

This is a developer objection I take seriously. "It passes the tests" leaves me wanting to know which behavior those tests cover, especially when money is involved. I'd also want to understand how the change fits the application we'll be extending next month.

In [his talk about software factories](https://www.youtube.com/watch?v=Ib5GBkD555M), Dex Horthy proposes working through product review, architecture, and program design before implementation, then building smaller vertical slices while people continue reading the code. I can see our team using that approach to get the payment question settled before the agent starts editing.

There is another worry underneath the engineer's questions. Simon Willison [describes losing his mental model of some of his own projects after generating features without reviewing their implementation](https://simonwillison.net/2026/Feb/15/cognitive-debt/), making subsequent decisions harder. I would want our engineer to keep that possibility in mind as the factory grows.

The team can have source code, passing checks, and gradually less confidence about how the whole thing works. Add a busy review queue and it becomes tempting to skim the next change. A few weeks of impressive output can leave a surprisingly long reading assignment.

So the engineer's concern about review capacity belongs in the strategy. So does their question about permissions. An agent experimenting on a branch and an agent holding production credentials deserve different levels of trust. The person carrying the pager should help decide where the boundary goes.

Those concerns are useful inputs to a pilot. I would expect the engineer to help design it, just as they expected the executives to consider the cloud proposal ten years earlier.

## A proposal they can both support

The engineer suggests starting with that neglected migration, leaving the payment change for later. The team already understands the intended behavior, can compare the old and new paths, and has someone willing to own the result. It is a manageable place to find out how much help the agent can provide.

Before implementation, they agree on the shape of the change. The agent gets a bounded workspace and the context it needs. The work comes back in pieces small enough to review. The team can ask it to explain an assumption, investigate a failure, or revise an approach before the next piece begins.

They also decide what to keep from the review. A recurring design decision goes into the team's documented conventions. A mistake the checks can catch gets a test or a lint rule. Context that helped the agent belongs somewhere the next run can find it. Each improvement still needs review; nobody is giving the system permission to quietly rewrite its own standards.

I like the possibilities here. A reviewer could open a pull request with the relevant context already assembled. A newer engineer could use the agent to walk through the unfamiliar parts. An experienced engineer could explore an alternative without spending an afternoon preparing scaffolding.

They also agree to follow the work all the way through. Time saved during implementation matters. So do review, rework, release, and the effort of maintaining the result. The token bill belongs in the conversation alongside the human effort.

After release, they follow the migration into actual use. A customer asking for another button may be struggling with a step the team assumed was obvious. The team investigates that problem before accepting the suggested solution, then carries the finding into the next change. I'd want that feedback to travel from observation to a tested release while someone still remembers why it mattered.

If the experiment leaves the team with a quicker release and a change everyone can explain, they have a reason to expand it. If review takes longer or the design becomes harder to follow, they have something specific to improve. An unsuccessful tool can be retired without anyone being accused of failing to believe in AI.

The executives get movement. The engineers get a say in how that movement happens. Both get evidence that is more useful than a count of activated seats.

Our engineer also has a responsibility here. Their experience gives them good reasons to ask hard questions. It should help them identify worthwhile experiments too. I'd want the person who once argued for a cloud pilot to leave room for the next engineer's enthusiasm.

## The next meeting

A few weeks later, imagine the team returning to the same room. The slide still has an AI heading. Underneath it is the migration they attempted, where the agent helped, where a person intervened, and what the team would change before trying again.

Our engineer now has something to show. The executives can ask whether the migration was worth the effort and whether another team should try the same approach. The discussion can get specific: a review that took too long, context the agent was missing, a useful chunk of work the team would happily delegate again.

There is something else on the slide: what the team kept. A test for a missed failure case. A clearer convention. A customer observation that changed the next task. Those are things the team can use again, even if the engineer who ran the pilot is on holiday. I'd count that growing capacity among the reasons to fund the next experiment.

In 2016, that engineer had to answer difficult questions before getting permission to use the cloud. In 2026, permission arrived first. The questions about cost, ownership, and what happens when something breaks still need answers. Buying the tools doesn't make that work disappear.

If you're leading this effort, put budget behind a task the team actually needs done. Give the engineers time to learn the tool and a say in how it gets used. Expect them to try it seriously, and expect to hear when it creates more work than it saves.

Then follow the result past the demo. Can the next engineer explain why the change works? Can the person on call recover when it fails? What did the team learn that will make the next change easier? If the token dashboard disappeared tomorrow, could you still show that the customer got a better product or the team got a better working day?

I'd fund a second experiment when the first one gives us useful answers, and expand the workflows the team wants to keep using. If the main thing growing is the invoice, check whether you've budgeted for more review and repair work too. That would be a fairly expensive way to discover you've been token-maxxing.

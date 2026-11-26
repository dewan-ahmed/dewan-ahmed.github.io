---
title: "A2A Is for Crossing Boundaries, Not for Every Subagent"
date: 2026-12-15T00:00:00-04:00
author: Dewan Ahmed
excerpt: "A2A 1.0 is a good protocol for agents that cross organizational, vendor, trust, or time boundaries. Inside one codebase, your subagents are functions."
series: agent-builder
series_part: 3
header:
  teaser: "/assets/images/2026/agent-builder-part-3.png"
tags:
  - ai
  - agents
  - a2a
---

{% include agent-builder-series.html %}

[Part 2]({% post_url 2026-12-08-mcp-tool-design %}) ended with a question: what do you do when the thing on the other end of a call isn't a tool but another agent?

The usual answer now is the Agent2Agent protocol, A2A. Google announced it in April 2025, it moved to the Linux Foundation, it shipped [a 1.0 specification](https://a2a-protocol.org/latest/whats-new-v1/), and in August 2026 it [joined the Agentic AI Foundation](https://a2a-protocol.org/latest/blog/) alongside MCP. Both protocols in this series now sit in the same foundation, which makes "which vendor owns this?" a much shorter conversation.

I like A2A. I also think a lot of teams reach for it before they need it. The question I'd ask of any multi-agent design is simple: where's the boundary? If nobody can point to one, I don't think A2A belongs in the diagram.

<!-- TODO(Dewan): if you've seen (or been tempted by) a design that put A2A between agents that didn't need it, a two-line version of that story would make a stronger opening than the history paragraph. -->

## Tools versus peers

This is how I explain the split to myself:

- MCP is for tools. You call a function, it does one thing, and it returns. You can see its schema, and you decide when to call it. The tool has no goals of its own.
- A2A is for peers. You hand a task to another agent with its own model, tools, memory, and policies. You can't see inside it, and you shouldn't need to. It might finish in two seconds or two days. It might come back with a question.

The second description is where A2A gets interesting, and it's also where it gets expensive. If the other side is opaque, slow, and allowed to ask you things, you need discovery, authentication, a task lifecycle, and a way to get results back later. A2A gives you all of that. You pay for it in moving parts.

## The protocol in one walkthrough

I'll use an example close to my own tools. Parts 1 and 2 leaned on [flowfacts](https://github.com/dewan-ahmed/flowfacts), the CLI that pulls facts out of a Jenkinsfile or Harness pipeline YAML. flowfacts is deliberately not an agent. It extracts facts and stops, and anything it can't resolve deterministically comes back marked as `unresolved`.

Now imagine the layer above it. A platform team runs an agent that assesses CI/CD pipelines for migration, and my agent, in a different team, wants to use it. That's a real boundary: a different owner, a different release schedule, and probably different credentials.

**1. Discovery.** The platform agent publishes an Agent Card at `/.well-known/agent-card.json`. It says what the agent does, where to reach it, and how to authenticate:

```json
{
  "name": "Pipeline Migration Assessor",
  "description": "Assesses a CI/CD pipeline and returns a migration plan, with the risks that need human review called out.",
  "version": "0.4.0",
  "provider": {
    "organization": "Platform Engineering",
    "url": "https://platform.example.com"
  },
  "supportedInterfaces": [
    {
      "url": "https://agents.example.com/a2a",
      "protocolBinding": "JSONRPC",
      "protocolVersion": "1.0"
    }
  ],
  "capabilities": {
    "streaming": true,
    "pushNotifications": true
  },
  "securitySchemes": {
    "platformOAuth": {
      "oauth2SecurityScheme": {
        "flows": {
          "clientCredentials": {
            "tokenUrl": "https://auth.example.com/oauth/token",
            "scopes": { "assess": "Run pipeline assessments" }
          }
        }
      }
    }
  },
  "securityRequirements": [
    { "schemes": { "platformOAuth": { "list": ["assess"] } } }
  ],
  "defaultInputModes": ["text/plain", "application/json"],
  "defaultOutputModes": ["application/json", "text/markdown"],
  "skills": [
    {
      "id": "assess-pipeline",
      "name": "Assess a pipeline for migration",
      "description": "Reads a Jenkinsfile or pipeline YAML and returns stages, risks, and a proposed target layout.",
      "tags": ["ci", "migration"],
      "examples": ["Assess this Jenkinsfile for a move to GitHub Actions."]
    }
  ]
}
```

Three things stood out to me in this card. **Skills** describe tasks a caller can ask for, which is the same advice I gave for tool design in part 2. **`supportedInterfaces`** can list several bindings (JSON-RPC, gRPC, HTTP+JSON), each with its own protocol version. And security is declared up front, so my agent knows it needs a client-credentials token with the `assess` scope before it sends anything.

Because the card lives at a well-known URL, it's also a cheap thing to check before a factory starts a long, expensive job. That's what I built [factory-doctor](https://github.com/dewan-ahmed/factory-doctor) for, and one `http` check covers it:

```json
{
  "version": 1,
  "checks": [
    {
      "name": "Pipeline assessor Agent Card",
      "kind": "http",
      "target": "https://agents.example.com/.well-known/agent-card.json",
      "expect_status": 200,
      "timeout_ms": 3000
    }
  ]
}
```

If the card isn't there, `factory-doctor check` exits with `1` and the job never starts. Keep in mind what that check proves: something answered with a 200. It says nothing about whether the card is honest. I'll come back to that.

**2. Send a message.** My agent calls `SendMessage` with a message made of parts. In 1.0, a part is just text, a URL, raw bytes, or structured data:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "SendMessage",
  "params": {
    "message": {
      "messageId": "9b1f5c2e-6d0a-4a7e-9a51-0c2f3d1e8b77",
      "role": "ROLE_USER",
      "parts": [
        {
          "text": "Assess this pipeline for a move to GitHub Actions."
        },
        {
          "url": "https://git.example.com/org/billing/raw/main/Jenkinsfile",
          "filename": "Jenkinsfile",
          "mediaType": "text/plain"
        }
      ]
    }
  }
}
```

**3. Work with the task.** The remote agent returns a **Task** with an ID, a `contextId` that groups related work, and a status. Tasks move through states: submitted, working, completed, failed, canceled, rejected, and the two I care about most, **input required** and **auth required**. My agent can poll with `GetTask`, stream updates with `SendStreamingMessage` or `SubscribeToTask`, or register a webhook for push notifications if the work will take a while.

This is the state that sold me on A2A:

```json
{
  "id": "task-4821",
  "contextId": "ctx-billing-migration",
  "status": {
    "state": "TASK_STATE_INPUT_REQUIRED",
    "message": {
      "messageId": "c0a8e7d2-1f4b-4e0e-8f3a-2b6d9e5c7a10",
      "role": "ROLE_AGENT",
      "parts": [
        { "text": "Stage 'deploy-prod' loads credentials from a Groovy helper I can't resolve. Which secret store should the new pipeline use?" }
      ]
    }
  }
}
```

The remote agent hit a credential lookup it couldn't resolve deterministically, the same kind of thing flowfacts reports as `unresolved`, and it asked instead of guessing. I'd much rather get that question than a confident migration plan with an invented secret store in it. My agent can answer, escalate to a person, or cancel. Results come back as **artifacts**, which are made of the same parts as messages.

If you built against the 0.3 spec, 1.0 is a real migration: methods were renamed (`message/send` became `SendMessage`), the `kind` discriminators on parts are gone, enum values are now `SCREAMING_SNAKE_CASE`, and the card's `url` and `protocolVersion` moved into `supportedInterfaces`. The [what's new page](https://a2a-protocol.org/latest/whats-new-v1/) has the full list.

I checked the JSON above against the 1.0 Python SDK's types. Please do the same with anything you copy from a blog, including this one.

<!-- TODO(Dewan): if validating against the SDK caught a 0.3-era habit in your first draft of these examples, name it here. One real mistake is a better warning than the list above. -->

## When A2A earns its keep

This is the table I go through before adding an A2A hop:

| Situation | Use | Why |
| --- | --- | --- |
| Your agent needs a capability that's a function | An MCP tool | You want a schema, not a conversation |
| You want to keep a big exploration out of the main context | An in-process subagent | Context isolation without network, auth, or a task store |
| Another team owns the agent and ships it on their own schedule | A2A | Independent ownership and versioning |
| The other agent runs on a different framework or vendor | A2A | A shared protocol instead of a shared codebase |
| The work takes hours and may need a human answer midway | A2A | Task lifecycle, input-required, push notifications |
| The other side holds data or credentials you shouldn't | A2A | The boundary is the point |

The row I left out on purpose is "we have a planner, a coder, and a reviewer." If all three run in your process, on your framework, under your credentials, and ship together, they're functions. Putting each one behind an HTTP server with an Agent Card adds latency, failure modes, and an auth story, and you get nothing back that a function call didn't already give you.

So my rule of thumb is that A2A is for crossing a boundary between organizations, vendors, trust levels, or long stretches of time. If I can't name the boundary, I leave the protocol out.

## Treat a remote agent like a contractor

When I hand work to an agent I don't control, I think about it the way I'd think about hiring a contractor. A contractor gets a badge for one floor, and their work gets reviewed before it ships. Calling a library doesn't need any of that. A remote agent does:

- **Its output is untrusted input.** An artifact from a remote agent can carry prompt injection just like a web page can. Validate structured data against a schema, and don't let returned text steer your agent's tool use without a check. [artifact-bridge](https://github.com/dewan-ahmed/artifact-bridge) is my small version of this habit for files: it rejects non-2xx responses and HTML login pages, enforces a size limit, and can verify a SHA-256 digest before the file lands.
- **Don't forward your credentials.** Get a token scoped to this agent and this task. If the remote agent needs your data, give it a URL it can fetch with its own identity, or a short-lived grant. That's why the Jenkinsfile in the example travels as a URL part. It's also why artifact-bridge reads auth headers from a named environment variable instead of taking them as command arguments.
- **Verify who you're talking to.** A2A 1.0 Agent Cards can carry signatures (JWS over a canonicalized card). An unsigned card fetched over the internet only tells you what someone claims. The factory-doctor check above proves the card is reachable, and nothing more.
- **Bound the task.** Set your own timeouts and budgets. The remote agent's idea of "still working" may not match yours. The same instinct is behind [runway](https://github.com/dewan-ahmed/runway), which checks agent usage against a budget and returns an exit code a job can act on. The limit should live on your side of the boundary.

## Where I think A2A is heading

Here's where I'd put my money:

- It wins at company and vendor boundaries. I expect the most valuable A2A deployments to be the boring ones: a vendor's support agent, a partner's fulfillment agent, a platform team's internal service. Anywhere a shared protocol can replace a one-off integration.
- Inside a single codebase, it mostly loses. Most "multi-agent" systems will stay in-process, and I think they should.
- MCP and A2A keep borrowing from each other. MCP now has an official tasks extension and an `input_required` result pattern. A2A has structured data parts and skills that look a lot like tools. I don't see that as a problem. I expect them to coexist the way RPC and messaging do, with different defaults for different problems.
- Agent identity is the hard part. Publishing a card is easy. Knowing which agent you're talking to, what it's allowed to do, and who's accountable when it's wrong is where I expect the next few years of work to go.

## Talks worth your time

- [Introduction to Agent2Agent (A2A) Protocol](https://www.youtube.com/watch?v=Fbr_Solax1w), Google Cloud Tech.
- [A2A vs MCP: AI Agent Communication Explained](https://www.youtube.com/watch?v=BMDFPOyezH4), IBM Technology.
- [Don't Build Agents, Build Skills Instead](https://www.youtube.com/watch?v=CEvIs9y1uog), Barry Zhang and Mahesh Murag (Anthropic), AI Engineer. Not about A2A, but a useful counterweight to reaching for more agents.

## Next up

Part 4 is about the agents most of us use every day: coding agents. I'll argue that their quality depends less on the model than on the factory around them, and I'll show the small tools I've built to make that factory more predictable.

---

**A note on how this was made:** I developed this series with an AI assistant. The structure, the examples from my own projects, and the opinions are mine, pressure-tested and organized through that conversation.
{: .disclaimer}

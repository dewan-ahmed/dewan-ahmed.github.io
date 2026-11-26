---
title: "A2A Is for Crossing Boundaries, Not for Every Subagent"
date: 2026-12-15T00:00:00-04:00
author: Dewan Ahmed
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

The Agent2Agent protocol, A2A, is the answer the industry has converged on. Google announced it in April 2025, it moved to the Linux Foundation, it shipped [a 1.0 specification](https://a2a-protocol.org/latest/whats-new-v1/), and in August 2026 it [joined the Agentic AI Foundation](https://a2a-protocol.org/latest/blog/) alongside MCP. So both of the protocols this series covers now live under the same neutral roof.

I like A2A. I also think it's the most over-applied idea in agent architecture right now. This post is about telling the difference.

## Tools versus peers

The cleanest way I've found to explain the split:

- **MCP is for tools.** You call a function, it does one thing, and it returns. You see its schema. You decide when to call it. The tool has no goals of its own.
- **A2A is for peers.** You hand a task to another agent that has its own model, its own tools, its own memory, and its own policies. You can't see inside it, and you shouldn't need to. It might finish in two seconds or two days. It might come back and ask you a question.

That second description is what makes A2A interesting. It's also what makes it expensive. Opacity, long-running work, and back-and-forth negotiation need discovery, authentication, a task lifecycle, and a way to deliver results later. A2A gives you all of that. You pay for it in moving parts.

## The protocol in one walkthrough

Say the platform team runs an agent that assesses CI/CD pipelines for migration. My agent, in a different team, wants to use it.

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

A few things worth noticing. **Skills** describe tasks, not functions, which is the same lesson as tool design in part 2. **`supportedInterfaces`** can list several bindings (JSON-RPC, gRPC, HTTP+JSON), each with its own protocol version. And security is declared up front, so my agent knows it needs a client-credentials token with the `assess` scope before it sends anything.

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

Here's the state that makes A2A worth it:

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

The remote agent got stuck on something it couldn't resolve deterministically (the same kind of item flowfacts reports as `unresolved`) and asked instead of guessing. My agent can answer, escalate to a person, or cancel. Results come back as **artifacts**, which are made of the same parts as messages.

If you built against the 0.3 spec, 1.0 is a real migration: methods were renamed (`message/send` became `SendMessage`), the `kind` discriminators on parts are gone, enum values are now `SCREAMING_SNAKE_CASE`, and the card's `url` and `protocolVersion` moved into `supportedInterfaces`. The [what's new page](https://a2a-protocol.org/latest/whats-new-v1/) has the full list. I checked the JSON above against the 1.0 Python SDK's types, which I'd recommend doing with anything you copy from a blog, this one included.

## When A2A earns its keep

Here's the decision table I use:

| Situation | Use | Why |
| --- | --- | --- |
| Your agent needs a capability that's a function | An MCP tool | You want a schema, not a conversation |
| You want to keep a big exploration out of the main context | An in-process subagent | Context isolation without network, auth, or a task store |
| Another team owns the agent and ships it on their own schedule | A2A | Independent ownership and versioning |
| The other agent runs on a different framework or vendor | A2A | A shared protocol instead of a shared codebase |
| The work takes hours and may need a human answer midway | A2A | Task lifecycle, input-required, push notifications |
| The other side holds data or credentials you shouldn't | A2A | The boundary is the point |

Notice what's missing: "we have a planner, a coder, and a reviewer." If all three run in your process, on your framework, under your credentials, and ship together, they're functions. Wrapping each in an HTTP server with an Agent Card adds latency, failure modes, and an auth story, and gives you nothing a function call didn't.

**A2A is for crossing a boundary: organizational, vendor, trust, or time.** If there's no boundary, you probably don't need it.

## Treat a remote agent like a contractor

When you hand work to an agent you don't control, the security posture should look like hiring a contractor, not calling a library:

- **Its output is untrusted input.** An artifact from a remote agent can contain prompt injection just like a web page. Validate structured data against a schema. Don't let returned text steer your agent's tool use without a check.
- **Don't forward your credentials.** Get a token scoped to this agent and this task. If the remote agent needs access to your data, give it a URL it can fetch with its own identity, or a short-lived grant.
- **Verify who you're talking to.** A2A 1.0 Agent Cards can carry signatures (JWS over a canonicalized card). An unsigned card fetched over the internet is a claim, not an identity.
- **Bound the task.** Set your own timeouts and budgets. The remote agent's idea of "still working" and yours may differ.

## Where I think A2A is heading

- **It wins at company and vendor boundaries.** The most valuable A2A deployments will be the boring ones: a vendor's support agent, a partner's fulfillment agent, a platform team's internal service. Places where a shared protocol replaces a bespoke integration.
- **It loses inside a single codebase.** Most "multi-agent" systems will stay in-process, and that's correct.
- **The line between MCP and A2A gets blurrier, and that's fine.** MCP now has an official tasks extension and an `input_required` result pattern. A2A has structured data parts and skills that look a lot like tools. They'll coexist the way RPC and messaging coexist: different defaults for different problems.
- **Agent identity becomes the real work.** Discovery is easy. Knowing which agent you're talking to, what it's allowed to do, and who's accountable when it's wrong is where the next few years of effort will go.

## Talks worth your time

- [Introduction to Agent2Agent (A2A) Protocol](https://www.youtube.com/watch?v=Fbr_Solax1w), Google Cloud Tech.
- [A2A vs MCP: AI Agent Communication Explained](https://www.youtube.com/watch?v=BMDFPOyezH4), IBM Technology.
- [Don't Build Agents, Build Skills Instead](https://www.youtube.com/watch?v=CEvIs9y1uog), Barry Zhang and Mahesh Murag (Anthropic), AI Engineer. Not about A2A, but a useful counterweight to reaching for more agents.

## Next up

Part 4 is about the agents most of us actually use every day: coding agents. My argument is that their quality depends less on the model than on the factory around them, and I'll show the small tools I've built to make that factory more predictable.

---

**A note on how this was made:** I developed this series with an AI assistant. The structure, the examples from my own projects, and the opinions are mine, pressure-tested and organized through that conversation.
{: .disclaimer}

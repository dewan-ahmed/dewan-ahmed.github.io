---
title: "Agents Are Loops: What I Look for in an Agent Framework"
date: 2026-12-01T00:00:00-04:00
author: Dewan Ahmed
series: agent-builder
series_part: 1
header:
  teaser: "/assets/images/2026/agent-builder-part-1.png"
tags:
  - ai
  - agents
  - software-engineering
---

{% include agent-builder-series.html %}

Every few weeks someone asks me which agent framework they should use. I usually answer with a question: what happens when your agent crashes halfway through a task?

If the answer is "it starts over and we pay for the tokens again", the framework decision is still open. If the answer is "I don't know", we should probably talk about that before we talk about frameworks.

This is the first post in a five-part series about building with agents. It's written for people who have already wired a model to a tool, watched it do something impressive, then watched it do something baffling five minutes later. Over the series I'll cover the agent loop, MCP, A2A, coding agents, and sandboxes, and I'll say where I think each one is heading.

Most of what follows comes from building small tools that agents call: [factory-doctor](https://github.com/dewan-ahmed/factory-doctor), [artifact-bridge](https://github.com/dewan-ahmed/artifact-bridge), [flowfacts](https://github.com/dewan-ahmed/flowfacts), and [runway](https://github.com/dewan-ahmed/runway), plus a lot of time watching agents use them. They're open source, so you can check my work.

## Strip the branding and every agent is the same loop

An agent is a model in a loop with tools. The model reads the conversation so far, either answers or asks for a tool call, your code runs the tool, appends the result, and the loop goes around again. That's it. Everything else is a feature built on top of that loop.

Here's a complete agent in about 50 lines of Python. It has one tool that runs factory-doctor's preflight checks and reports back.

```python
import os
import subprocess

import anthropic

client = anthropic.Anthropic()        # reads ANTHROPIC_API_KEY
MODEL = os.environ["AGENT_MODEL"]     # any tool-capable model

TOOLS = [{
    "name": "run_preflight",
    "description": (
        "Run factory-doctor preflight checks and return its JSON report. "
        "Use this before starting work to confirm tools, services and repo access."
    ),
    "input_schema": {
        "type": "object",
        "properties": {
            "config": {"type": "string", "description": "Path to a factory-doctor config file."}
        },
        "required": ["config"],
    },
}]


def run_preflight(config: str) -> str:
    result = subprocess.run(
        ["factory-doctor", "check", "--config", config, "--json"],
        capture_output=True, text=True, timeout=60,
    )
    # Exit code 1 means a critical check failed; the JSON report is still on stdout.
    return result.stdout or result.stderr


HANDLERS = {"run_preflight": run_preflight}


def run_agent(task: str, max_turns: int = 10) -> str:
    messages = [{"role": "user", "content": task}]
    for _ in range(max_turns):
        response = client.messages.create(
            model=MODEL, max_tokens=2048, tools=TOOLS, messages=messages,
        )
        messages.append({"role": "assistant", "content": response.content})

        if response.stop_reason != "tool_use":
            return "".join(b.text for b in response.content if b.type == "text")

        results = []
        for block in response.content:
            if block.type != "tool_use":
                continue
            try:
                output = HANDLERS[block.name](**block.input)
                results.append({"type": "tool_result", "tool_use_id": block.id, "content": output})
            except Exception as exc:  # errors go back to the model, not up the stack
                results.append({"type": "tool_result", "tool_use_id": block.id,
                                "content": f"{type(exc).__name__}: {exc}", "is_error": True})
        messages.append({"role": "user", "content": results})

    raise RuntimeError(f"agent did not finish within {max_turns} turns")
```

I used the Anthropic SDK because it's what I had open. The OpenAI, Gemini, and open-weight equivalents look almost identical: a list of tool schemas, a response that says "call this tool with these arguments", and a message type for sending results back.

Three details in that loop matter more than they look:

- **`max_turns`.** An agent without a turn limit is a credit card without a limit. You will eventually find out what it costs.
- **Errors become tool results.** When a tool fails, the model gets the error as content and can try something else. If you raise instead, a typo in a file path ends the whole run.
- **The tool returns facts, not prose.** factory-doctor emits stable JSON and stable exit codes. The model doesn't have to guess whether "mostly fine" means pass or fail.

If you have never written this loop by hand, do it once. Everything a framework sells you is easier to evaluate once you know which parts of it you could write in an afternoon.

## What the loop doesn't give you

The 50-line version works on your laptop. It stops working the moment a task takes longer than one sitting, involves a person, or costs real money. This is where frameworks earn their keep, and it's where I'd focus your evaluation:

| Problem | What you need | Where you'll find it |
| --- | --- | --- |
| The process dies on turn 14 | Durable state you can resume from | Checkpointers (LangGraph), durable execution engines (Temporal and friends) |
| A person must approve a step | Pause, persist, resume hours later | Interrupts and approval hooks |
| The conversation outgrows the context window | Compaction, summarization, trimming tool output | Agent harnesses like the Claude Agent SDK; your own code otherwise |
| You can't tell why it did that | Traces of every model call, tool call, and token count | OpenTelemetry instrumentation, vendor tracing |
| One agent's context is full of the wrong things | Subagents with isolated context | Handoffs (OpenAI Agents SDK), subagents, graph nodes |
| The bill surprised you | Turn, token, and spend budgets | Usually your own code |

Notice that none of those rows are "a nicer way to define a tool". Every framework already has that. The differences that matter show up in the runtime: state, interruption, context, observability, and cost.

## My framework checklist

When I evaluate a framework now, I ask five questions before I look at a single tool decorator.

1. **Can I resume after a crash without repeating side effects?** If a tool already opened a pull request, the resumed run shouldn't open a second one.
2. **Can I pause for a human and resume tomorrow?** Real workflows wait on people. "Keep the Python process alive" is not a strategy.
3. **Can I see every model call and tool call, with token counts?** If I can't trace it, I can't debug it, and I definitely can't explain the invoice.
4. **Can I swap the model provider without rewriting the agent?** Models leapfrog each other every few months. Your tool layer shouldn't care.
5. **Can I unit-test the loop with a fake model?** If the only way to test the agent is to call a real model, your test suite becomes an eval suite with a monthly bill.

That last one is cheap to do yourself. Here's the loop above, tested with a scripted fake client:

```python
import os
from types import SimpleNamespace as NS

os.environ.setdefault("ANTHROPIC_API_KEY", "test")
os.environ.setdefault("AGENT_MODEL", "fake-model")
import agent  # the module above

script = [
    NS(stop_reason="tool_use", content=[
        NS(type="tool_use", id="t1", name="run_preflight", input={"config": "factory-doctor.json"}),
    ]),
    NS(stop_reason="end_turn", content=[NS(type="text", text="All critical checks passed.")]),
]
agent.client = NS(messages=NS(create=lambda **_: script.pop(0)))
agent.HANDLERS["run_preflight"] = lambda config: '{"passed": 4, "failed": 0}'

assert agent.run_agent("Is this machine ready?") == "All critical checks passed."
```

Test the loop and the tools deterministically. Save model calls for evals that measure behaviour, and run those on purpose, not on every commit.

## Four opinions I hold more strongly every month

### 1. Put deterministic work in tools, not prompts

The most reliable way I know to improve an agent isn't a better model or a longer prompt. It's moving work out of the model.

flowfacts is the clearest example. Ask a migration agent to read a long Jenkinsfile and reason about it, and you're spending context on boilerplate and inviting the model to confidently "remember" stages that aren't there. So the agent calls flowfacts instead. It parses the pipeline and returns compact JSON: stages, dependencies, credential IDs (never values), artifacts, and an explicit list of `unresolved` items like dynamic Groovy.

That `unresolved` list is my favourite part. It tells the agent exactly where deterministic extraction stops and judgment has to start. The model spends its attention on the hard ten percent instead of re-deriving the easy ninety.

If a task has a correct answer a program can compute, a program should compute it.

### 2. Exit codes and JSON are agent UX

We spent decades making CLIs pleasant for humans: colours, spinners, friendly messages. Agents want the opposite. They want stable JSON on stdout, diagnostics on stderr, and exit codes that mean one thing.

All four of my tools follow the same contract: `0` means proceed, a small set of non-zero codes mean specific failures, and `2` or `3` means you called me wrong. That's boring on purpose. The same binary works from a terminal, a CI step, an agent hook, or an MCP server, which brings us to the next post.

### 3. Start with one agent and good tools

Multi-agent architectures look great on a whiteboard. In practice, most of the ones I've seen would have worked better as one agent with better tools.

There's one excellent reason to split: **context isolation**. A subagent that reads 200 files to answer "where is retry logic configured?" and returns one paragraph keeps those 200 files out of the main agent's context. That's a real win. "The planner agent talks to the coder agent who talks to the reviewer agent" is usually not. It's three prompts sharing one problem and tripling the ways to lose information between them.

Split when the context or the trust boundary demands it. Part 3 is about the second case.

### 4. Frameworks are thinning, and that's good

Looking across the major options today (LangGraph, the OpenAI Agents SDK, the Claude Agent SDK, Google's ADK, Microsoft Agent Framework, Pydantic AI, Mastra), they're converging on the same core: a loop, typed tools, state, tracing, and some form of handoff. The clever abstractions from 2023 have mostly been sanded away.

I think that trend continues. The loop becomes a commodity, and the differentiation moves to two places:

- **The tool layer**, which is increasingly MCP.
- **The environment** the agent runs in, which is increasingly a sandbox.

That's the shape of this series. If you get the tools and the environment right, the framework choice becomes much less dramatic. If you get them wrong, no framework will save you.

## Talks worth your time

If you want to go deeper on the ideas in this post, these are the talks I point people to:

- [How We Build Effective Agents](https://www.youtube.com/watch?v=D7_ipDqhtwk), Barry Zhang (Anthropic), AI Engineer.
- [12-Factor Agents: Patterns of reliable LLM applications](https://www.youtube.com/watch?v=8kMaTybvDUw), Dex Horthy (HumanLayer), AI Engineer.
- [The Great Loops Debate](https://www.youtube.com/watch?v=c35YoMdnI78), Dex Horthy, Geoff Huntley, Ian Livingstone, and Greg Pstrucha, AI Engineer.
- [Software Is Changing (Again)](https://www.youtube.com/watch?v=LCEmiRjPEtQ), Andrej Karpathy, Y Combinator.

## Next up

In part 2, I'll take the tool from this post and expose it over MCP, then talk about what I've learned designing tools for models: fewer of them, shaped like tasks, with errors the model can act on. MCP also changed a lot in 2026, and some of those changes are worth understanding even if you never write a server.

---

**A note on how this was made:** I developed this series with an AI assistant. The structure, the examples from my own projects, and the opinions are mine, pressure-tested and organized through that conversation.
{: .disclaimer}

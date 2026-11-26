---
title: "Coding Agents Are a Software Factory Problem"
date: 2026-12-22T00:00:00-04:00
author: Dewan Ahmed
series: agent-builder
series_part: 4
header:
  teaser: "/assets/images/2026/agent-builder-part-4.png"
tags:
  - ai
  - agents
  - software-engineering
---

{% include agent-builder-series.html %}

Most of my agent time this year went into coding agents. Not building them, using them: handing off tasks, reviewing pull requests, and figuring out why a session that should have taken twenty minutes took two hours.

Here's what I keep relearning. When a coding agent fails, the model is rarely the first thing I'd fix. The session started on a machine without the test runner. It couldn't reach the internal package registry. It spent forty turns exploring a repo with no instructions. It fetched an artifact that turned out to be a login page. Nobody told it when to stop spending.

That's not a model problem. It's a factory problem. In [part 1]({% post_url 2026-12-01-agents-are-loops %}) I said deterministic work belongs in tools. This post is what that looks like when the agent writes code for a living.

## The factory around the agent

When I say "software factory," I mean everything around a coding agent session:

| Stage | What can go wrong | What I want there |
| --- | --- | --- |
| Before the session | Missing tools, unreachable services, wrong repo access | A preflight check that fails in seconds |
| Repo context | The agent guesses at build commands and conventions | An `AGENTS.md` and a fast, honest test command |
| During the session | Big files and artifacts flood the context | Small tools that return facts, not files |
| Credentials | Tokens end up in prompts, logs, or command arguments | Scoped credentials the model never sees |
| Cost | Sessions run long and nobody notices until the invoice | A budget forecast that can stop work |
| Review | Plausible-looking PRs that nobody can verify | CI, tests, and a reviewer who reads the diff |

None of this is new. It's the same discipline we've always applied to CI runners and build farms. The difference is that a coding agent will happily try to work around a broken environment for an hour instead of failing fast like a build would. So the factory has to do the failing for it.

I built a few small, vendor-neutral CLIs while learning this. They're early, but each one came from a specific failure I kept seeing.

## Fail before the expensive part

The cheapest agent session is the one you don't start on a broken machine. That's why I wrote [factory-doctor](https://github.com/dewan-ahmed/factory-doctor): a config of checks that runs in milliseconds and returns stable JSON and exit codes.

```json
{
  "version": 1,
  "checks": [
    {"name": "Source repository", "kind": "git_remote", "target": "https://github.com/dewan-ahmed/flowfacts", "args": ["HEAD"], "timeout_ms": 5000},
    {"name": "Git executable", "kind": "command", "target": "git", "args": ["--version"], "timeout_ms": 1000},
    {"name": "Test runner", "kind": "command", "target": "pytest", "args": ["--version"], "timeout_ms": 2000},
    {"name": "Optional local cache", "kind": "file", "target": "./cache", "critical": false, "timeout_ms": 100}
  ]
}
```

I ran that on a fresh VM while writing this post, and it caught exactly the kind of thing I'd otherwise find out about forty minutes into a session:

```json
{"version":1,"status":"fail","checks":[{"name":"Source repository","kind":"git_remote","target":"https://github.com/dewan-ahmed/flowfacts","status":"pass","code":"ok","message":"exit code 0","duration_ms":537},{"name":"Git executable","kind":"command","target":"git","status":"pass","code":"ok","message":"exit code 0","duration_ms":10},{"name":"Test runner","kind":"command","target":"pytest","status":"fail","code":"executable_not_found","message":"executable \"pytest\" was not found","duration_ms":0},{"name":"Optional local cache","kind":"file","target":"./cache","status":"warn","code":"file_not_found","message":"path does not exist","duration_ms":0}],"summary":{"total":4,"passed":2,"warned":1,"failed":1}}
```

Exit code `1`. The test runner is missing, so the session shouldn't start. The cache is a warning because I marked it non-critical. Under a second of wall time, and no tokens spent.

The design choices matter more than the code. Exit codes are stable (`0` pass, `1` critical failure, `2` bad config), so a CI step, a session hook, or an agent can all branch on them. Targets are scrubbed of credentials and query strings before they're printed. And the `code` field is machine-readable, so an agent that does see the report can tell `executable_not_found` from `unexpected_http_status` without parsing prose.

## Give the agent the repo's rules

The single cheapest improvement I've made to coding agent sessions is a good [`AGENTS.md`](https://agents.md/) at the repo root. It's just Markdown: how to set up the environment, the exact commands for build, lint, and test, the conventions that aren't obvious from the code, and the things you never want the agent to do.

What I've learned writing them:

- **Commands beat descriptions.** "Run `v -silent test .`" is better than "we use V's test framework."
- **The test command has to be fast and honest.** If the suite takes 20 minutes or fails on main, the agent will either skip it or learn to ignore failures. Both are bad.
- **Write down the traps.** Every repo has one: the generated file you shouldn't edit, the integration test that needs a VPN. The agent can't know.

Eno Reyes from Factory gave a good talk on this, [Making Codebases Agent Ready](https://www.youtube.com/watch?v=ShuJ_CN6zr4). His framing stuck with me: the things that make a codebase easy for an agent are mostly the things that make it easy for a new hire.

## Return facts, not files

Coding agents burn context fast. A 900-line Jenkinsfile, a full build log, a 50 MB artifact: every one of these competes with the code the agent is actually supposed to change.

[flowfacts](https://github.com/dewan-ahmed/flowfacts) is my answer for CI/CD pipelines. Instead of reading the file, the agent asks for facts: stages, dependencies, runners, credential IDs (never values), artifacts, and a list of **unresolved** items where the pipeline does something dynamic that a deterministic parser can't follow. That unresolved list is the part I'm proudest of. It tells the agent exactly where its judgment is needed and where it isn't.

For artifacts, [artifact-bridge](https://github.com/dewan-ahmed/artifact-bridge) does the boring parts nobody wants the model improvising with `curl`:

```sh
export ARTIFACT_TOKEN="Bearer ..."

artifact-bridge fetch "https://factory.example.com/build.zip" \
  --out build.zip \
  --header-env "Authorization=ARTIFACT_TOKEN"
```

The token comes from the environment and is never printed. It rejects non-2xx responses and HTML login pages, enforces a size limit, can verify a SHA-256, and writes atomically. The login-page check is there because an expired session often returns a 200 with an HTML sign-in page, and an agent will happily save that as `build.zip` and then spend turns trying to unzip it.

## Put a budget on the loop

Coding agents are the first software I've used where a bug can cost money by the minute. I don't want to find out at the end of the month, so I wrote [runway](https://github.com/dewan-ahmed/runway). It reads the usage data a platform officially exposes and forecasts the rest of the billing period:

```sh
runway check \
  --source devin \
  --file consumption.json \
  --budget 1500 \
  --period-start 2026-11-01 \
  --period-end 2026-11-30 \
  --warn-at 80 \
  --stop-at 100
```

On a sample export with usage ramping through the month, it returned:

```json
{"version":1,"source":"devin","unit":"ACU","period_start":"2026-11-01","period_end":"2026-11-30","as_of":"2026-11-20","elapsed_days":20,"period_days":30,"used":925,"budget":1500,"used_percent":61.67,"daily_rate":46.25,"forecast":1387.5,"forecast_percent":92.5,"remaining":575,"runway_days":12.43,"exhaustion_date":"","status":"warning"}
```

Only 62% used, but the forecast is 92.5%, so the status is `warning` and the exit code is `1`. At `stop` it exits `2`. Thresholds apply to the forecast, not the balance, which is the whole point: I want to slow down while there's still budget left. It also reads Factory analytics and OpenTelemetry trace exports, because the budget problem isn't specific to one vendor.

## Review is still the bottleneck

All of this gets the agent to a pull request faster and more reliably. It doesn't make the pull request correct. The constraint in my workflow has moved from writing code to reviewing it, and I don't think a better model fixes that by itself.

What helps:

- **Smaller tasks.** A PR I can review in ten minutes beats a PR that "does everything" and sits for a week.
- **Plans before code.** For anything non-trivial, I ask for a short plan first. It's much cheaper to correct a plan than a diff. Dex Horthy's [No Vibes Allowed](https://www.youtube.com/watch?v=rmvDxxNubIg) talk makes this case with real numbers from a large codebase.
- **CI that the agent has to pass.** Formatting, lint, tests, and the build. If the agent can run the same checks locally, it fixes most failures before I see them.
- **A second reviewer, human or agent.** I've been using a separate agent to review my own drafts, including this series. It catches different things than I do.

## Where I think coding agents are heading

- **Background agents become the default.** The IDE copilot isn't going away, but more work moves to agents that run in a sandbox, open a PR, and wait for review. That makes the factory, not the editor, the product surface.
- **Repos get optimized for agents.** `AGENTS.md`, fast tests, deterministic tooling, and clean CI become table stakes, the same way READMEs and CI did. Teams that do this will get much more out of the same models.
- **Small deterministic tools win.** A 400-line CLI with stable JSON and exit codes is more useful to a coding agent than another feature in the agent itself. I expect a lot more of these, and most of them should be vendor-neutral.
- **Cost controls move into the loop.** Today budgets are a dashboard. They'll become a check the agent runs before it starts and between steps, like a test.

## Talks worth your time

- [Making Codebases Agent Ready](https://www.youtube.com/watch?v=ShuJ_CN6zr4), Eno Reyes (Factory), AI Engineer.
- [No Vibes Allowed: Solving Hard Problems in Complex Codebases](https://www.youtube.com/watch?v=rmvDxxNubIg), Dex Horthy (HumanLayer), AI Engineer.
- [12 Rules for a Software Factory That Doesn't Ship Slop](https://www.youtube.com/watch?v=-NFMsM2y0Vk), Dex Horthy, AI Engineer.
- [Claude Code & the evolution of agentic coding](https://www.youtube.com/watch?v=Lue8K2jqfKk), Boris Cherny (Anthropic), AI Engineer.

## Next up

Every coding agent in this post runs code on a machine. In the last part of the series, I'll argue that the sandbox around that machine is now the real permission model, and show how I test that a sandbox actually blocks what it claims to.

---

**A note on how this was made:** I developed this series with an AI assistant. The structure, the examples from my own projects, and the opinions are mine, pressure-tested and organized through that conversation.
{: .disclaimer}

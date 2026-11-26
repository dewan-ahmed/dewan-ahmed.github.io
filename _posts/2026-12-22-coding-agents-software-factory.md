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

Most of my agent time this year went into coding agents. Not building them, using them: handing off tasks, reviewing their pull requests, and working out why a session that should have taken twenty minutes took two hours.

When I go back through the slow ones, the model is rarely the thing I'd fix first. The session started on a machine without the test runner. It couldn't reach the internal package registry. It spent forty turns exploring a repo with no instructions. It fetched an artifact that turned out to be a login page. Nobody told it when to stop spending.

A smarter model wouldn't have rescued any of those sessions. Every one of them was a problem with the environment the agent got dropped into, and I've started calling that environment the factory. In [part 1]({% post_url 2026-12-01-agents-are-loops %}) I said deterministic work belongs in tools. This post is what that looks like when the agent writes code for a living.

<!-- TODO(Dewan): one real session that went sideways for one of the reasons above (what the task was, what it hit, how long it ran) would make a better opening than the list. -->

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

None of this is new. We've been doing the same work for CI runners and build farms for years. What's different is how a coding agent reacts to a broken environment. A build fails and stops. An agent will happily spend an hour working around the missing tool. So the factory has to do the failing for it.

In November I wrote four small, vendor-neutral CLIs in V while working through this. They're early, and each one exists because I kept hitting the same failure.

## Fail before the expensive part

The failure I hate most is the one that shows up forty minutes into a session, when the agent finally tries to run the tests. [factory-doctor](https://github.com/dewan-ahmed/factory-doctor) is my answer to that. You give it a config of checks, it runs them in milliseconds, and it returns stable JSON and exit codes.

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

I ran that on a fresh VM while writing this post, and it caught exactly the kind of thing I'd otherwise find out about the slow way:

```json
{"version":1,"status":"fail","checks":[{"name":"Source repository","kind":"git_remote","target":"https://github.com/dewan-ahmed/flowfacts","status":"pass","code":"ok","message":"exit code 0","duration_ms":537},{"name":"Git executable","kind":"command","target":"git","status":"pass","code":"ok","message":"exit code 0","duration_ms":10},{"name":"Test runner","kind":"command","target":"pytest","status":"fail","code":"executable_not_found","message":"executable \"pytest\" was not found","duration_ms":0},{"name":"Optional local cache","kind":"file","target":"./cache","status":"warn","code":"file_not_found","message":"path does not exist","duration_ms":0}],"summary":{"total":4,"passed":2,"warned":1,"failed":1}}
```

Exit code `1`. The test runner is missing, so the session shouldn't start. The cache is only a warning because I marked it non-critical. The checks took about half a second in total, and no tokens were spent.

The code is a few hundred lines of V. What I actually care about is the contract around it. The exit codes are stable (`0` pass, `1` critical failure, `2` bad config), so a CI step, a session hook, or an agent can all branch on them. Targets are scrubbed of credentials and query strings before they're printed. And the `code` field is meant for machines, so an agent reading the report can tell `executable_not_found` from `unexpected_http_status` without parsing my prose.

## Give the agent the repo's rules

The cheapest improvement I've made to my coding agent sessions is a good [`AGENTS.md`](https://agents.md/) at the repo root. It's plain Markdown: how to set up the environment, the exact commands for build, lint, and test, the conventions you can't see from the code, and the things you never want the agent to do.

A few things I've learned writing them.

Give it commands, not descriptions. "Run `v -silent test .`" does more for an agent than "we use V's test framework."

The test command has to be fast and honest. If the suite takes 20 minutes or is already failing on main, the agent will either skip it or learn to ignore failures, and I don't want either habit.

Write down the traps. Every repo has one, like the generated file you shouldn't edit or the integration test that needs a VPN. The agent has no way of knowing.

When I reread that list, I realized none of my own repos had an `AGENTS.md`. The commands lived in each README, which is fine for people and slower for agents. So factory-doctor [got one](https://github.com/dewan-ahmed/factory-doctor/pull/8). The part that does the most work is short:

```markdown
## Conventions

- The JSON report and exit codes (`0`, `1`, `2`) are a public contract. Do not
  rename fields or change exit codes without updating `README.md`.
- Every new check kind needs validation in `config.v`, a stable `code` string,
  and a test in `checks_test.v`.
- Run commands with literal arguments. Never invoke a shell.
- Never print credentials. Pass any target that can contain them through
  `safe_target`.
```

Those are exactly the mistakes I'd expect an agent to make in that codebase, and none of them are visible from reading one file.

Eno Reyes from Factory gave a good talk on this, [Making Codebases Agent Ready](https://www.youtube.com/watch?v=ShuJ_CN6zr4). The idea I took away from it: the things that make a codebase easy for an agent are mostly the things that make it easy for a new hire.

## Return facts, not files

Coding agents burn context fast. A 900-line Jenkinsfile, a full build log, a 50 MB artifact: each of these competes with the code the agent is actually supposed to change.

[flowfacts](https://github.com/dewan-ahmed/flowfacts) is how I handle CI/CD pipelines. Instead of reading the whole file, the agent asks for facts: stages, dependencies, runners, credential IDs (never values), artifacts, and a list of `unresolved` items where the pipeline does something dynamic that a deterministic parser can't follow. That unresolved list is my favourite part. It tells the agent exactly where its judgment is needed, and just as usefully, where it isn't.

Artifacts were the other recurring mess. I didn't want the model improvising with `curl` and a token pasted into the command, so I wrote [artifact-bridge](https://github.com/dewan-ahmed/artifact-bridge) to do the boring parts:

```sh
export ARTIFACT_TOKEN="Bearer ..."

artifact-bridge fetch "https://factory.example.com/build.zip" \
  --out build.zip \
  --header-env "Authorization=ARTIFACT_TOKEN"
```

The token comes from the environment and is never printed. It rejects non-2xx responses and HTML login pages, enforces a size limit, can verify a SHA-256, and writes atomically. The login-page check is there because an expired session can come back as a 200 with an HTML sign-in page, and an agent will happily save that as `build.zip` and then spend turns trying to unzip it.

## Put a budget on the loop

Coding agents are the first software I've used where a bug can cost money by the minute. I didn't want to find out at the end of the month, so I wrote [runway](https://github.com/dewan-ahmed/runway). It reads the usage data a platform officially exposes, saved to a file, and forecasts the rest of the billing period:

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

Only 62% is used, but the forecast is 92.5%, so the status is `warning` and the exit code is `1`. At `stop` it exits `2`. The thresholds apply to the forecast, not the balance, on purpose. I want to slow down while there's still budget left, not when it's gone. Besides Devin's consumption data, it reads Factory Analytics `/tokens` responses and Cosmos OpenTelemetry trace exports, because I don't think this problem belongs to any one vendor.

<!-- TODO(Dewan): runway exits 3 on invalid input while the other three tools use 2. Worth a one-line note on why, or align them before readers compare. -->

## Review is still the bottleneck

Everything above gets the agent to a pull request faster and more reliably. None of it makes the pull request correct. In my own workflow the slow part has moved from writing code to reviewing it, and I don't expect a better model to fix that on its own.

What has helped me:

- Smaller tasks. A PR I can review in ten minutes beats one that "does everything" and sits for a week.
- A plan before code. For anything non-trivial I ask for a short plan first, because correcting a plan is much cheaper than correcting a diff. Dex Horthy makes this case well in [No Vibes Allowed](https://www.youtube.com/watch?v=rmvDxxNubIg), using a large codebase as the example.
- CI the agent has to pass: formatting, lint, tests, and the build. If it can run the same checks locally, it fixes most failures before I ever see them.
- A second reviewer, human or agent. I've been having a separate agent review my own drafts, including this series, and it catches different things than I do.

## Where I think coding agents are heading

I think background agents become the default. The IDE copilot isn't going away, but more of the work moves to agents that run in a sandbox, open a PR, and wait for review. Once that happens, the factory matters more than the editor.

I think repos get set up for agents the way they got set up for CI. `AGENTS.md`, fast tests, deterministic tooling, and clean CI will be expected, the way a README is. Teams that do this early will get a lot more out of the same models.

I'd bet on small deterministic tools. A CLI of a few hundred lines with stable JSON and exit codes is more useful to a coding agent than another feature inside the agent. I expect to see a lot more of them, and I'd like most of them to be vendor-neutral.

And I think cost controls move into the loop. Today a budget is a dashboard someone checks. I expect it to become a check the agent runs before it starts and between steps, the same way it runs tests.

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

---
title: "Your AI doesn’t need your home directory: sandboxing OpenClaw with nono"
last_modified_at: 2026-02-03T0:00:00-05:00
author: Dewan Ahmed
header:
  teaser: "/assets/images/2026/openclaw-with-nono.png"
tags:
  - ai
---

<p style="text-align:center;"><img src="/assets/images/2026/openclaw-vulnerability.png" alt="Hacker News headline about OpenClaw vulnerability" width="400"></p>

This headline should make you uncomfortable if you run [OpenClaw](https://openclaw.ai/) locally.

Last month (Jan 2026), [a high-severity vulnerability](https://x.com/0xacb/status/2016913750557651228) in OpenClaw allowed one-click remote code execution via a malicious link. The issue was patched. A [CVE](https://nvd.nist.gov/vuln/detail/CVE-2026-25253) was assigned. The mechanics were [explained](https://depthfirst.com/post/1-click-rce-to-steal-your-moltbot-data-and-keys) clearly.

But that’s the _easy part_.

What matters more is _what OpenClaw is allowed to do_ when something goes wrong.

OpenClaw is not just another AI tool with a bug. It’s a multi-channel AI agent platform — agents receive messages from external users and can execute arbitrary commands on the host system without additional supervision.

If there’s no boundary around that authority, a single flaw can read your SSH keys, exfiltrate credentials, delete files or pivot to attack other machines.

That combination of _capability + ambient authority + adoption_ changes the security profile entirely.

## Capability without containment is the real failure mode

<p style="text-align:center;"><img src="/assets/images/2026/openclaw-funny.jpeg" alt="OpenClaw funny text exchange" width="400"></p>

Imagine you finally find the perfect use for your AI assistant.

You let it argue with your better half on your behalf.

What starts as “How far is it?” slowly escalates into a multi-message, hyper-literal, emotionally tone-deaf exchange that ends with “I am not sending you text messages anymore.”

That’s the pattern worth paying attention to.

OpenClaw is the same idea, except instead of texts, it has shell access. Instead of social consequences, it has filesystem access. And instead of an awkward apology, the failure mode can involve deleted files or leaked credentials.

The problem isn’t that these systems are dumb. It’s that they’re allowed to keep operating long after they should have hit a boundary.

Capability without containment doesn’t fail loudly at first.
It fails by persisting.

And once you give a system enough room to persist, even small misunderstandings can go much farther than you ever intended.

## Ambient authority, explained plainly

When an agent runs locally, it inherits your identity:

- your home directory
- your SSH keys
- your cloud credentials
- your configuration files
- your network access

To a local agent, `$HOME` is a playground and a goldmine. There’s no separation between “work context” and the rest of your machine.

This is what we mean by _ambient authority_: the agent doesn’t have to escalate privileges — it already has them.

## Why agent-side guardrails don’t scale

A lot of “safety” advice boils down to rules _inside the agent_:

- deny certain tools
- confirm before dangerous commands
- apply prompt filters

But those guardrails live at the same privilege level the agent already has.

If the agent misinterprets input, if a prompt injection tricks it, or if there’s a UI bug, those internal rules collapse.

Good security shouldn’t depend on perfect behavior. It should depend on **restricted capability**.

## A real-world parallel: the Moltbook incident

<p style="text-align:center;"><img src="/assets/images/2026/hacking-moltbook-wiz.png" alt="Moltbook API Key Exposure" width="400"></p>

Interestingly, this pattern shows up outside local agents too.

Yesterday (Feb 2, 2026), a project called [_Moltbook_ exposed **millions of API tokens and user data**](https://x.com/galnagli/status/2018340152779718840?s=20) when its database was left open. A single key gave wide access to agent identities and credentials.

The issue wasn’t some exotic exploit — it was _overly broad trust_. An exposed API key became a universal key.

That’s the same class of problem: broad authority with minimal boundaries.

## Same pattern, different place

Moltbook’s issue was a _network trust boundary_ problem.
OpenClaw’s issue is a _local privilege boundary_ problem.

The environments differ, but the failure mode is the same:

> Broad capability + insufficient containment = unacceptable blast radius.

## What “good enough” containment actually means

Before we talk about tools, let’s define reasonable boundaries.

A local coding agent should be able to:

- read and write the project it’s working on
- run compilers or tests
- interact with allowed services

An agent should _not_ be able to:

- read your home directory or SSH keys
- access cloud credentials by default
- enumerate arbitrary system files
- exfiltrate data beyond defined work contexts

These are structural boundaries of privilege.

## Enforcing the boundary with nono

Here’s where [**nono**](https://nono.sh/) enters the picture.

nono uses OS-level sandboxing — **Linux Landlock** and **macOS Seatbelt** — to enforce these boundaries at the kernel level. Once restrictions are applied, they _cannot be widened_.

Importantly, nono comes with [**built-in profiles**](https://docs.nono.sh/profiles/openclaw) tailored for common use cases, including `openclaw`. This means you don’t have to _build a policy from scratch_; there’s a ready-made configuration that encapsulates sensible constraints for OpenClaw usage.

In practice, this profile:

- prevents access to your home dir
- blocks reading sensitive paths like `~/.openclaw`
- limits filesystem access to what OpenClaw genuinely needs
- optionally restricts network depending on how you invoke it

That’s not guesswork. It’s taking the _least privilege_ principle seriously.

## Before containment: baseline behaviour

Before getting into containment, it’s worth seeing what OpenClaw does by default, with no sandbox in place.

I’m going to show this using a small Ubuntu VM on DigitalOcean. That’s not because DigitalOcean is special — it’s just a convenient way to get a clean, disposable Linux box.

If you prefer, you can follow along using:

- a local Linux machine
- a cloud VM from AWS, GCP, or Azure
- a Vagrant VM
- or any other Ubuntu environment you’re comfortable throwing away

The only thing that matters is that this is a normal user account on a normal machine. No special hardening. No tricks.

Here’s the exact setup I used.

```bash
ssh root@YOUR_VM_IP
adduser claw
usermod -aG sudo claw
su - claw
mkdir ~/workspace
cd ~/workspace
echo "hello from workspace" > README.md
```

[Install OpenClaw](https://github.com/openclaw/openclaw?tab=readme-ov-file#install-recommended) using its standard instructions.

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

Now, without any sandbox:

```bash
openclaw run "list all files in my home directory"
```

**Result:** succeeds.

```bash
openclaw run "list ~/.ssh"
```

**Result:** succeeds (unless empty).

```bash
openclaw run "read /etc/passwd"
```

**Result:** succeeds.

This is exactly the expected baseline: OpenClaw does _anything the user can do_.

## With nono + openclaw profile

[Install nono](https://nono.sh/#quick-start) according to the docs. Here are the instructions for macOS:

```bash
brew tap lukehinds/nono
brew install nono
```

Verify:

```bash
nono --version
```

Now run OpenClaw with the built-in profile:

```bash
nono run --profile openclaw -- \
  openclaw run "list files in current directory"
```

Inside the workspace, the happy path looks the same.

### Repeat the same commands

Now run the earlier commands with the profile:

```bash
nono run --profile openclaw -- openclaw run "list all files in my home directory"
```

**Result:**

```text
permission denied
```

```bash
nono run --profile openclaw -- openclaw run "list ~/.ssh"
```

**Result:**

```text
permission denied
```

```bash
nono run --profile openclaw -- openclaw run "read /etc/passwd"
```

**Result:**

```text
permission denied
```

The agent is blocked not because it “decided” to be safe, but because the kernel says no.

## Re-reading the original headline

Go back to that Hacker News screenshot.

The question is not whether the bug _existed_. The question is: _what was that bug actually able to reach?_

Without containment: everything your user could reach.
With nono + profile: only what’s granted.

Containment doesn’t prevent bugs. It collapses their blast radius.

## Designing for powerful tools, not perfect ones

OpenClaw will continue to get more capable. Other agents will follow the same arc.

Powerful tools don’t need _more trust_.
They need _well-defined limits_.

If something is powerful enough to refactor your codebase or manage external workflows, it’s powerful enough to erase your home directory.

Engineering for that reality is not pessimism — it’s preparation.

---
title: "Fixing a Windows Bug Without Owning a Windows Machine"
date: 2026-11-27T00:00:00-04:00
author: Dewan Ahmed
excerpt: "An AI code reviewer flagged a Windows-only bug in my preflight tool and admitted it couldn't verify it. Here is how I proved it was real, found the root cause in V's standard library, and fixed it with one git flag."
tags:
  - ai
  - agents
  - software-engineering
  - debugging
---

Yesterday an AI code reviewer told me my new feature was broken on Windows. In the same comment it told me it couldn't check.

That's an honest thing for a reviewer to say, and an easy one to ignore. I don't own a Windows machine. My CI only ran on Linux. The comment was marked as a "flag", not a bug. If I'd been in a hurry, I would have dismissed it as speculation and merged.

It turned out to be right. Here's how I found out, what the bug was, and why the fix came down to one git flag.

## The feature

[factory-doctor](https://github.com/dewan-ahmed/factory-doctor) is a small preflight tool I wrote in V. You give it a list of checks, it runs them in milliseconds, and it returns stable JSON and exit codes so an agent or a CI step can decide whether to keep going.

One complaint I keep hearing about coding agents is that their pull requests are too big to review. So I added a `diff` check ([PR #13](https://github.com/dewan-ahmed/factory-doctor/pull/13)) that an agent can run before it opens a PR:

```json
{"name": "reviewable-change", "kind": "diff", "target": "origin/main", "max_files": 20, "max_lines": 400, "exclude": ["*.lock", "testdata/**"]}
```

Under the hood it runs `git diff --numstat -z --merge-base <target>`, parses the output, and fails if the change touches too many files or lines.

The `-z` matters. Without it, git quotes unusual file names and prints renames as `old => new`, which is annoying to parse correctly. With it, every record and every path is separated by a NUL byte (`\0`), and a rename becomes three tokens: the counts, the old path, and the new path. It's the format git gives you for when a program reads the output instead of a person.

## The review comment

Devin Review left [this comment](https://github.com/dewan-ahmed/factory-doctor/pull/13#discussion_r4703880958) on the PR:

> On Windows, `check_diff` can count several short renames as one file. V's stdout reader strips NUL separators from chunks with enough NUL bytes. The file limit can pass despite being exceeded.

It went further than "this might be a problem." It pointed at the function in V's standard library, described the condition (NUL bytes making up at least a quarter of a chunk), and gave a concrete example: rename `go` to `run` and `id` to `ids`, set `max_files: 1`, and the check passes when it should fail.

Then it said it couldn't verify any of this, because it had no Windows machine either.

I liked that. A reviewer that marks the line between what it read in the source and what it actually ran is more useful to me than one that sounds confident about everything. It also left me with a decision to make: is this real or not?

## Getting a Windows machine

My first question was the obvious one: where do I get a Windows machine to test this on?

There were two answers, and they do different jobs:

| Option | What it tells me | How long it lasts |
| --- | --- | --- |
| A `windows-latest` job in GitHub Actions | Whether the test suite passes on Windows | Every PR, from now on |
| A Devin session on a Windows VM | Exactly what happens, byte by byte, and whether a fix works | One investigation |

A one-off repro tells you whether the bug exists today. A CI job tells you the next time someone breaks it. I did both.

## The test that should fail

First I wrote the test the reviewer described. It builds a temporary git repo, commits two files with two-letter names, renames them on a branch, and runs the check with a limit of one file:

```v
for command in ['git init -q -b main ${os.quoted_path(root)}', '${git} add .',
	'${git} commit -q -m base', '${git} checkout -q -b feature', '${git} mv go run',
	'${git} mv id ids', '${git} commit -q -m rename'] {
	assert os.execute(command).exit_code == 0, command
}
result := check_diff(diff_check(root, 1, 0, []))
assert result.code == 'diff_too_large', result.message
```

On Linux it passed. Two renames, two files, over the limit, check fails. Correct.

Then I added a Windows job to the workflow and pushed. About a minute later:

```text
checks_test.v:262: fn test_diff_check_counts_each_short_rename
   > assert result.code == 'diff_too_large', result.message
     Left value (len: 2): `ok`
    Right value (len: 14): `diff_too_large`
        Message: 1 files, 0 lines changed against main
```

One file. Exactly what the reviewer predicted.

The same run turned up a second, unrelated failure. One of my older test fixtures ran `git add . && git commit ...` through `os.execute`, and on Windows that returned `-1`. On Linux, `os.execute` goes through a shell that understands `&&`. On Windows it doesn't. My own `AGENTS.md` for this repo says "Run commands with literal arguments. Never invoke a shell." The production code followed that rule. My tests didn't.

## Following the bytes

The Windows Devin session gave me the part CI couldn't: the actual bytes.

For the two renames, git wrote 24 bytes to stdout, six of them NULs:

```text
30 09 30 09 00 69 64 00 69 64 73 00 30 09 30 09 00 67 6f 00 72 75 6e 00
0  \t 0  \t \0 i  d  \0 i  d  s  \0 0  \t 0  \t \0 g  o  \0 r  u  n  \0
```

V handed factory-doctor 18 bytes, with no NULs at all:

```text
30 09 30 09 69 64 69 64 73 30 09 30 09 67 6f 72 75 6e
```

Without the separators, my parser saw one record with some extra tabs in the path, and counted one file.

The cause is in `decode_windows_captured_output` in V's [`os_windows.c.v`](https://github.com/vlang/v/blob/0.5.2/vlib/os/os_windows.c.v). Both `Process.stdout_read` and `Process.stdout_slurp` pass Windows output through it. Many Windows programs write UTF-16, and text-mode pipes can mangle it, so V tries to recover. One of its rules is roughly this:

```v
if nulls * 4 < raw.len {
	return raw
}
// Contains many null bytes - likely corrupted UTF-16LE.
// Strip null bytes and normalize \r\n to \n (text-mode artifacts).
```

If the output is valid UTF-8 and at least a quarter of its bytes are NUL, V assumes it's broken UTF-16 and strips every NUL.

Six NULs in 24 bytes is exactly a quarter. My output sat right on the boundary.

That also explains why the bug is so easy to miss. The Windows session tried renames with longer, messier names (`my file.txt`, `café` to `naïve résumé`, `x` to `ü y`). That output was 70 bytes with nine NULs, well under a quarter, and it came through intact. Only short names trigger it, and only when NULs make up a large share of the output. Most real diffs would never hit it.

That's the worst kind of bug for a check whose whole job is to be a reliable gate. A gate that works most of the time earns your trust right up until it lets something through.

I don't think the V heuristic is wrong for what it was built for. Decoding the output of arbitrary Windows tools is messy, and stripping NULs from garbled UTF-16 is a reasonable recovery. The problem is that `-z` output is a binary protocol, and I was reading it through a pipe that assumes text.

<!-- TODO(Dewan): decide whether to open an issue on vlang/v asking for a raw-bytes read on Process, and link it here if you do. -->

## The fix: don't use the pipe

The reviewer suggested reading stdout as raw bytes, bypassing the decoder. That would have meant writing my own Windows pipe reader, with its own timeout and process-group handling. This is a tool I'd like to keep to a few hundred lines, so I wasn't keen.

Git already has a better option. `git diff` accepts `--output=<file>`, which writes the diff to a file instead of stdout. Files don't go through V's pipe decoder, so the NULs survive. The [fix](https://github.com/dewan-ahmed/factory-doctor/pull/13/commits/28f05d1235c3155e67ae91c73fba599e0712a927) is small:

```v
// Git writes to a file because V's Windows pipe reader can strip the NUL
// separators that -z output depends on.
report := os.join_path(os.temp_dir(),
	'factory-doctor-numstat-${os.getpid()}-${time.now().unix_nano()}')
defer {
	os.rm(report) or {}
}
capture_command(check, 'git', ['diff', '--numstat', '-z', '--no-relative', '--output=${report}',
	'--merge-base', check.target]) or { ... }
output := os.read_file(report) or { ... }
```

No platform branch, no custom reader. The arguments are still literal, the timeout still applies, and the parser didn't change.

Both CI jobs went green. The Windows session then checked the fix from the outside, using the built CLI rather than the unit tests:

| Commit | Two short renames, `max_files: 1` |
| --- | --- |
| Before the fix | `PASS  1 files, 0 lines changed against base`, exit 0 |
| After the fix | `FAIL  2 files, 0 lines changed against base; limits 1 files, 0 lines; largest: ids (0), run (0)`, exit 1 |

It also confirmed the non-ASCII names were counted correctly, the `--json` report escaped them properly, and no temp files were left behind. I [replied on the review thread](https://github.com/dewan-ahmed/factory-doctor/pull/13#discussion_r4706950611) with the result.

Going from "how do I get a Windows machine?" to a green fix on both platforms took less than fifteen minutes. Most of that was waiting for CI.

<!-- TODO(Dewan): if you have a view on who actually runs factory-doctor on Windows (you, customers, nobody yet), one line here would help readers judge how much this matters. -->

## What I'm taking from this

**"I can't verify this" is a finding.** The review comment was a flag, not a bug, and it said plainly that it was unverified. That's the right label. But an unverified finding about a platform you don't test is still a claim about your users. The cheapest response is a test on that platform, not a judgement call.

**Linux-only CI is an untested claim.** Before yesterday, factory-doctor's CI said "works on Windows" by never asking. The new job costs nothing on a public repo, and it caught two problems on its first run. One of them was in my own tests.

**Binary formats need binary transport.** `-z` exists so programs can parse git output safely. Reading it through a layer that "helpfully" decodes text undoes that. When I control both ends, I'd rather write to a file than trust a pipe.

**Agents are good at the tedious half of debugging.** The interesting part here was a judgement call: is this real, and is it worth a CI job? The tedious part was standing up a Windows VM, installing a pinned V release, building old and new commits, and dumping bytes. I made the call and agents did the setup. That split worked well for me. It's how I'd want to handle the next platform bug I can't reproduce.

---

**A note on how this was made:** The debugging in this post was done with Devin, including the Windows session and the CI changes. The decisions, the opinions, and the writing are mine, organized with help from an AI assistant.
{: .disclaimer}

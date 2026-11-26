---
permalink: /projects/
title: "Projects"
---

Things I build and maintain in the open.

## Agent and software factory tools

Small, vendor-neutral command-line tools for AI agents and software factories. Each one does a single deterministic job, emits stable JSON, and uses exit codes an agent or CI job can act on. They're all early development, so expect the schemas to change.

### factory-doctor

Fast, deterministic preflight checks. It catches missing tools, unreachable services, broken repository access, and unavailable files before a factory starts an expensive agent session. Checks can also be marked `"expect": "fail"` to prove a sandbox blocks access it shouldn't have.

[GitHub](https://github.com/dewan-ahmed/factory-doctor){% include project-posts.html slugs="agents-are-loops,coding-agents-software-factory,sandboxes-permission-model" %}

### flowfacts

Turns a CI/CD pipeline file into compact JSON facts (stages, dependencies, runners, credentials, artifacts, unresolved dynamic code) so a migration agent can reason over a pipeline without reading the whole file into context. It extracts facts only.

[GitHub](https://github.com/dewan-ahmed/flowfacts){% include project-posts.html slugs="mcp-tool-design" %}

### artifact-bridge

Downloads an artifact without putting credentials into the model prompt or command arguments. It validates HTTP status, response type, size, and an optional SHA-256 digest before atomically moving the file into place.

[GitHub](https://github.com/dewan-ahmed/artifact-bridge){% include project-posts.html slugs="coding-agents-software-factory" %}

### runway

Reads the usage data a software factory officially exposes, forecasts consumption through the end of the budget period, and returns an exit code an agent or CI job can act on. No dashboard scraping, no private APIs.

[GitHub](https://github.com/dewan-ahmed/runway){% include project-posts.html slugs="coding-agents-software-factory" %}

## Kubernetes

### k8s-workers

Kubernetes tools on Cloudflare Workers. The [deprecations checker](https://deprecations.k8s.workers.dev) flags Kubernetes API deprecations and removals for a target release (1.16–1.37) in pasted YAML, rendered Helm output, or a packaged Helm chart, with replacement and migration guidance for each finding. It also has a JSON API you can call from CI to fail a build on removed APIs.

[GitHub](https://github.com/dewan-ahmed/k8s-workers) · [Try it](https://deprecations.k8s.workers.dev)

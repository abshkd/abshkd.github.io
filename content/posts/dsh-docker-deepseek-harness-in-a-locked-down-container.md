---
title: "dsh-docker: DeepSeek Harness in a Locked-Down Container"
slug: dsh-docker-deepseek-harness-in-a-locked-down-container
date: 2026-09-12T16:50:24+08:00
draft: false
description: "A Docker Compose setup for running DeepSeek Harness with persistent sessions, an isolated workspace, a read-only filesystem, and least-privilege defaults."
tags:
  - deepseek
  - docker
  - ai-agents
  - self-hosted
  - security
---

I built [dsh-docker](https://github.com/abshkd/dsh-docker) to run the DeepSeek Harness Web UI in a small, locked-down Docker container. The Harness is useful, but it can also read and edit files, run commands, install plugins, and store model credentials. I wanted a repeatable setup where the agent starts with a much smaller view of the host.

This builds on my earlier note about [why I want to try DeepSeek Harness](/2026/08/deepseek-released-the-agent-harness-too/). The difference here is packaging and containment rather than another agent plugin.

<!--more-->

## What the container changes

The image pins `@deepseek-ai/dsh@0.1.5-rc.1` and runs it as an unprivileged user. The default Compose configuration:

- makes the image filesystem read-only;
- drops every Linux capability and disables privilege escalation;
- exposes the Web UI only on `127.0.0.1`;
- keeps Harness state and the agent workspace in separate persistent volumes;
- retains the upstream `workspace-write` sandbox and approval prompts;
- disables optional telemetry; and
- applies CPU, memory, process, and temporary-storage limits.

The agent cannot see host files unless I explicitly bind-mount them. There is a separate Compose override for mounting only the local `workspace/` directory when I need to work on real files. I deliberately do not mount the Docker socket, home directory, SSH keys, or cloud credentials.

## Running it

The basic workflow is short:

```bash
docker compose build
docker compose up -d
docker compose logs -f dsh
```

The logs print an authenticated local URL beginning with `http://127.0.0.1:3080/`. Sessions, settings, credentials, attachments, installed profiles, and workspace files survive container recreation through the two named volumes.

The same image can also run a one-off headless task:

```bash
docker compose run --rm --entrypoint dsh dsh \
  --profile headless "inspect this workspace"
```

I use [Hugging Face Pro](https://huggingface.co/pro), which includes $2 in monthly compute credits. I use that credit through [Hugging Face Inference Providers](https://huggingface.co/docs/inference-providers/index), and it has been enough to build and test a DSH plugin while trying different models.

## Security notes

This reduces accidental host exposure, but Docker is not a separate virtual machine. The agent still has outbound network access for model providers, and anything placed in its persistent volumes is inside its trust boundary. DeepSeek Harness is also developer-preview software and has not had a security audit.

I documented the reviewed upstream commit, pinned npm artifact, package-manager advisories, mitigations, and remaining risks in the repository's [security notes](https://github.com/abshkd/dsh-docker/blob/main/SECURITY.md). For genuinely hostile repositories, prompts, plugins, or MCP servers, I would still put Docker inside a disposable VM and use an API key that is easy to revoke.

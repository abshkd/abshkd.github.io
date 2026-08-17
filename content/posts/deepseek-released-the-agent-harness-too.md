---
title: "DeepSeek Released the Agent Harness Too"
slug: deepseek-released-the-agent-harness-too
date: 2026-08-17T09:33:52+08:00
draft: false
description: "DeepSeek has released an open-source agent harness with a plugin-based runtime, Web UI, headless mode, tools, sandboxing, approvals, and persistent sessions."
tags:
  - deepseek
  - ai-agents
  - coding-agents
  - agent-harness
  - open-source
---

DeepSeek has released [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness), or `dsh`, as an open-source agent harness under the MIT license. Model releases usually give us weights or an API and leave the actual agent runtime to tools such as OpenCode. This release includes that missing layer.

## What is different

The main idea is that everything is a plugin. The model adapter, tool registry, session log, agent loop, filesystem, sandbox, approval policy, persistence, and telemetry can all be replaced or patched through configuration. Profiles and bundles compose those plugins into a working runtime, with both a Web UI and a headless runner.

This is more than a prompt and a shell tool. The default agent can read and edit files, run commands, delegate work, maintain a plan, and request approval for restricted operations. Its append-only session log is also used for replay, resume, forks, transcripts, and persistence. The [architecture documentation](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/architecture.md) is unusually detailed about how all of this fits together.

## Why I want to try it

I want to compare DeepSeek inside its own harness with the same model running through a general-purpose coding agent. The plugin boundaries also make it interesting for experimenting with a different model endpoint, sandbox, or tool set without rewriting the agent loop.

The closest project I have built to this is [Airith](https://airith.com), an enterprise financial analysis platform that used LLMs and AI/ML for deep analysis, charting, and market microstructure work. Airith had what I would now call a complete harness: models, financial data, tools, execution, and presentation working as one system. We stopped development before MCP and "agent harness" became common vocabulary, but we were already solving many of the same orchestration problems.

I want to see whether DeepSeek Harness can provide a modern foundation for revisiting that work. Instead of rebuilding the orchestration layer, I could map the financial data sources, analysis tools, and chart generation into plugins and see how far DeepSeek's latest models can take the workflow.

The quickest way to start the Web UI is:

```bash
npx @deepseek-ai/dsh web
```

DeepSeek calls this a developer preview and explicitly warns that compatibility-breaking changes are coming. It looks complete in scope rather than finished in maturity, which is exactly why I want to test it now.

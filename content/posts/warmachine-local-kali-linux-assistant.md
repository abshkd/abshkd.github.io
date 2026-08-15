---
title: "WarMachine: A Local Kali Linux Assistant for HTB and Pentesting"
slug: warmachine-local-kali-linux-assistant
date: 2026-08-15T12:20:00+08:00
draft: false
tags:
  - machine-learning
  - cybersecurity
  - kali-linux
  - local-llm
  - tool-calling
---

[WarMachine](https://huggingface.co/ovedrive/WarMachine) is my work-in-progress model for running a Kali Linux assistant locally. The current 4B version operates more like an OpenCode-compatible security assistant than a fully autonomous pentesting agent.

I run it on a local GPU inside a sandbox for Hack The Box and other authorized pentesting work. It helps construct commands, runs them through structured tool calls, and uses the output to suggest what to try next. I find it most useful for code and command completion, especially when I know the objective but do not want to stop and reconstruct the exact syntax for each tool.

## What is in the 4B version

The current model is based on Qwen3.5-4B and fine-tuned with LoRA through Axolotl. The training data combines synthetic and curated pentesting tool-calling conversations focused on CEH, PNPT, and HTB-style engagements. I retained the Qwen3.5 multimodal wrapper, so the base vision stack is still available for experiments with screenshots and other visual inputs.

The practical workflow is straightforward:

1. Give it a target and an authorized objective.
2. Let it construct the relevant Kali Linux command.
3. Run the command inside the sandbox.
4. Return the output so it can help interpret the result and construct the next command.

## Current limitation

The 4B model cannot reliably complete a full attack chain yet. I do not know whether the main constraint is model size, the amount and quality of training data, or a combination of both. It is still useful as an assistant, but I keep a human in the loop instead of expecting it to plan and execute an entire engagement.

## What comes next

The next version will use a larger base model and a larger dataset, probably in the 9B or 27B range. The main question is whether the additional capacity and more complete multi-step training examples improve planning across a longer sequence of tool calls without losing the practical command-completion behavior that already works well in the 4B model.

The current model and 4B files are available on [Hugging Face](https://huggingface.co/ovedrive/WarMachine/tree/main/4B). It is intended only for authorized security testing and education, and I run it in an isolated environment.

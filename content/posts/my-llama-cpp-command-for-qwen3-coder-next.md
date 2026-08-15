---
title: "My llama.cpp Command for Qwen3-Coder-Next"
slug: my-llama-cpp-command-for-qwen3-coder-next
date: 2026-08-15T12:10:00+08:00
draft: false
tags:
  - llama.cpp
  - qwen
  - local-llm
  - inference
  - coding
---

I use this command to run Qwen3-Coder-Next locally with `llama-server`. I am keeping the complete command and an explanation of each option here so I do not have to reconstruct it later.

This is the best configuration I have found so far for an RTX 4090 with 24 GB of VRAM and 64 GB of system RAM. My priority is a longer context window rather than maximum inference speed. There is still room to optimize it further, but this setup works well enough for regular use.

```bash
./llama.cpp/llama-server \
  --model unsloth/Qwen3-Coder-Next-GGUF/Qwen3-Coder-Next-UD-Q4_K_XL.gguf \
  --alias "unsloth/Qwen3-Coder-Next" \
  --seed 3407 \
  --temp 1.0 \
  --top-p 0.95 \
  --min-p 0.01 \
  --top-k 40 \
  --port 8008 \
  --jinja \
  --cache-type-k q4_1 \
  --ctx-size 256000 \
  --fit on
```

## What each option does

| Option | Meaning and reason for using it |
| --- | --- |
| `./llama.cpp/llama-server` | Starts llama.cpp's HTTP server, web interface, and OpenAI-compatible API. This path assumes the executable is inside my local `llama.cpp` directory. |
| `--model ...gguf` | Loads the local UD-Q4_K_XL GGUF file. The path is relative to the directory where I launch the command; `--model` does not download it from Hugging Face. The model file is approximately 49.6 GB. |
| `--alias "unsloth/Qwen3-Coder-Next"` | Gives the server a stable model ID for `/v1/models` and API requests. Clients can use this name instead of the full filesystem path. |
| `--seed 3407` | Fixes the random-number seed. This makes comparisons easier, although identical output is not guaranteed across different llama.cpp builds, hardware, or settings. |
| `--temp 1.0` | Sets sampling temperature. A value of `1.0` preserves the model's normal probability distribution instead of making it more conservative. This is Qwen's recommended value for this model. |
| `--top-p 0.95` | Keeps the smallest group of likely tokens whose combined probability reaches 95%. This removes the unlikely tail while leaving multiple plausible choices. |
| `--min-p 0.01` | Removes tokens whose probability is below 1% of the most likely token's probability. This is less restrictive than llama.cpp's current `0.05` default and retains more alternatives. |
| `--top-k 40` | Limits sampling to the 40 most probable tokens before the other probability filters run. This prevents very unlikely tokens from entering the candidate set. |
| `--port 8008` | Serves the UI and API on port `8008` instead of the default `8080`. Without a separate `--host`, llama-server listens on `127.0.0.1`, so it remains local to the machine. |
| `--jinja` | Uses the model's Jinja chat template to turn roles, messages, and tools into the prompt format expected by Qwen. It is particularly important for OpenAI-style tool calling. |
| `--cache-type-k q4_1` | Stores the key side of the KV cache in Q4_1 instead of the default F16. This reduces the memory cost of the long context. Only K is changed here; the value cache remains F16 because `--cache-type-v` is not set. |
| `--ctx-size 256000` | Creates a context window of 256,000 tokens, slightly below the model's native 262,144-token limit. This is the combined working space for the prompt, conversation history, and generated tokens—not a guarantee that every response can generate 256,000 new tokens. |
| `--fit on` | Lets llama.cpp adjust arguments that I did not explicitly set so the model fits available device memory, leaving a default safety margin. Since I explicitly set `--ctx-size 256000`, fit will not reduce that context; it can still adjust automatic device and layer placement. |

## How the settings work together

Qwen recommends `temperature=1.0`, `top_p=0.95`, and `top_k=40` for Qwen3-Coder-Next. llama.cpp applies these sampling filters as a chain rather than as independent choices. My additional `min-p=0.01` removes only tokens that are extremely unlikely relative to the current best token.

The 256,000-token context is the expensive part of this setup. Quantizing the K cache helps control its memory usage, while `--fit on` tries to place as much of the model as practical on available devices. On my RTX 4090 and 64 GB RAM system, this is a practical compromise that favors context length over raw speed. If the server still runs out of memory, the first setting to reduce is `--ctx-size`; the model card suggests `32768` as a fallback.

These sampling arguments establish server defaults. An API client can send its own `temperature`, `top_p`, `top_k`, `min_p`, or `seed` for an individual request and override them.

## References

- [Qwen3-Coder-Next GGUF model and recommended settings](https://huggingface.co/unsloth/Qwen3-Coder-Next-GGUF)
- [llama-server options and API documentation](https://github.com/ggml-org/llama.cpp/blob/master/tools/server/README.md)
- [llama.cpp function-calling documentation](https://github.com/ggml-org/llama.cpp/blob/master/docs/function-calling.md)

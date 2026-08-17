---
title: "Compiling PyTorch CUDA Extensions in Docker Without a GPU"
slug: compiling-pytorch-cuda-extensions-in-docker-without-a-gpu
date: 2026-08-17T14:25:36+08:00
draft: false
description: "Build PyTorch CUDA extensions in a Docker image without GPU access by setting the CUDA paths and an explicit TORCH_CUDA_ARCH_LIST target."
tags:
  - cuda
  - docker
  - nvidia
  - pytorch
  - gpu-containers
  - rootless-docker
---

One detail that is easy to forget when building CUDA containers is that the GPU itself is normally not available during `docker build`. The compiler and CUDA toolkit can be present, but PyTorch cannot inspect the target card and choose an architecture automatically.

For an RTX 3090 build, the important setting is:

```dockerfile
ENV TORCH_CUDA_ARCH_LIST="8.6"
```

Compute capability `8.6` is the correct target for the RTX 3090. This variable is specifically used by PyTorch CUDA extension builds. A plain `nvcc` or CMake CUDA project needs its equivalent compiler or `CMAKE_CUDA_ARCHITECTURES` setting instead.

<!--more-->

## Base image and environment

I use the CUDA development image because it includes the compiler, headers, and development libraries needed during the build:

```dockerfile
FROM nvidia/cuda:13.1.2-cudnn-devel-ubuntu24.04 AS runtime

# Set shell and noninteractive environment variables
SHELL ["/bin/bash", "-c"]
ENV DEBIAN_FRONTEND=noninteractive
ENV PYTHONUNBUFFERED=1
ENV SHELL=/bin/bash
ENV CUDA_HOME=/usr/local/cuda
ENV PATH="/usr/local/cuda/bin:${PATH}"
ENV LD_LIBRARY_PATH="/usr/local/cuda/lib64:${LD_LIBRARY_PATH}"
ENV PIP_BREAK_SYSTEM_PACKAGES=1
ENV PIP_ROOT_USER_ACTION=ignore
ENV TORCH_CUDA_ARCH_LIST="8.6"
```

The variables each have a small, practical purpose:

| Setting | Why it is there |
| --- | --- |
| `SHELL ["/bin/bash", "-c"]` | Makes shell-form `RUN` instructions use Bash. |
| `DEBIAN_FRONTEND=noninteractive` | Prevents APT from waiting for interactive input. |
| `PYTHONUNBUFFERED=1` | Sends Python output directly to container logs. |
| `SHELL=/bin/bash` | Tells programs inside the image which shell to expect. |
| `CUDA_HOME=/usr/local/cuda` | Gives PyTorch and build tools an explicit CUDA toolkit location. |
| CUDA in `PATH` | Makes tools such as `nvcc` directly executable. |
| CUDA in `LD_LIBRARY_PATH` | Makes CUDA shared libraries discoverable. |
| `PIP_BREAK_SYSTEM_PACKAGES=1` | Allows pip to install into an externally managed system Python environment. |
| `PIP_ROOT_USER_ACTION=ignore` | Suppresses pip's root-user warning during the image build. |
| `TORCH_CUDA_ARCH_LIST="8.6"` | Builds PyTorch CUDA extensions for the RTX 3090 architecture without GPU detection. |

The NVIDIA base image already sets the standard CUDA `PATH` and `LD_LIBRARY_PATH`; keeping them explicit in a downstream Dockerfile makes the build assumptions obvious. Docker also recommends scoping `DEBIAN_FRONTEND=noninteractive` to an `ARG` or individual `RUN` command when the final image may be used interactively.

## Rootless Docker

This works with the [rootless NVIDIA Docker setup from my earlier Ubuntu guide](/2026/08/ubuntu-26.04-nvidia-cuda-13.3-and-rootless-docker/). Rootless mode changes how the daemon and runtime expose the GPU, but it does not change what the compiler needs during the image build.

With the `devel` base image, the required Python and build dependencies, explicit CUDA paths, and `TORCH_CUDA_ARCH_LIST`, a typical PyTorch CUDA extension can compile without a GPU being present. The GPU is needed later when the compiled code is run and tested.

## References

- [PyTorch CUDA extension documentation](https://docs.pytorch.org/docs/main/cpp_extension.html)
- [NVIDIA CUDA 13.1.2 cuDNN development image](https://hub.docker.com/layers/nvidia/cuda/13.1.2-cudnn-devel-ubuntu24.04/images/sha256-c8365d2cde6d16aad4e9981ebfaff1b8556152b85f51082316a9667f87a49604)
- [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)

---
title: "Running Qwen Image Edit 2511 4-bit on Amazon EKS"
date: 2026-08-15T12:00:00+08:00
draft: false
tags:
  - machine-learning
  - quantization
  - qwen
  - hugging-face
  - aws-eks
---

I published [Qwen Image Edit 2511 4-bit](https://huggingface.co/ovedrive/Qwen-Image-Edit-2511-4bit) as a selective NF4 quantization rather than quantizing every transformer layer indiscriminately. Some layers remain at higher precision to preserve output quality. The resulting Diffusers-compatible model can operate below 20 GB of VRAM, including on 16 GB GPUs with CPU offload.

Gary Stafford used the model in his [Amazon EKS deployment](https://garystafford.medium.com/deploying-qwen-image-edit-model-to-amazon-eks-with-gpu-acceleration-d71c7f4fca61). His implementation packages the 17 GB model for an NVIDIA L40S, distributes it from S3 to node-local EBS, loads it into a FastAPI inference service, and reports model and GPU readiness through health checks. The accompanying [deployment repository](https://github.com/garystafford/qwen-image-edit-2511-eks-react) includes the containers, Kubernetes manifests, caching DaemonSet, API, and end-to-end inference tests.

## Other quantized models

- [Qwen Image Edit 2509 4-bit](https://huggingface.co/ovedrive/Qwen-Image-Edit-2509-4bit)
- [Qwen Image 2512 4-bit](https://huggingface.co/ovedrive/Qwen-Image-2512-4bit)
- [Qwen Image 2512 8-bit](https://huggingface.co/ovedrive/Qwen-Image-2512-8bit)
- [Wan 2.2 I2V A14B 4-bit](https://huggingface.co/ovedrive/Wan2.2-I2V-A14B-4bit)
- [ERNIE Image NF4](https://huggingface.co/ovedrive/ERNIE-Image-nf4)
- [Krea 2 Raw 4-bit](https://huggingface.co/ovedrive/Krea-2-Raw-4bit)
- [Krea 2 Turbo 4-bit](https://huggingface.co/ovedrive/Krea-2-Turbo-4bit)

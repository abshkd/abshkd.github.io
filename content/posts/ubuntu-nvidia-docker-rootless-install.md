---
title: "Ubuntu 26.04: NVIDIA CUDA 13.3 and Rootless Docker"
date: 2026-08-16T15:17:17+08:00
draft: false
description: "Install CUDA 13.3 and rootless Docker with NVIDIA GPU support on Ubuntu 26.04, including Secure Boot and NVIDIA Container Toolkit."
tags:
  - ubuntu
  - nvidia
  - cuda
  - docker
  - rootless-docker
  - gpu-containers
  - secure-boot
---

**GPU support:** Tested on an RTX 3090. Theoretically compatible cards are listed at the end of this guide.

Validated on 16 August 2026 with:

- Ubuntu 26.04 LTS (`resolute`), x86-64
- Linux kernel `7.0.0-29-generic`
- NVIDIA GeForce RTX 3090
- UEFI Secure Boot enabled
- Canonical-signed NVIDIA 595 server-open kernel driver
- CUDA Toolkit 13.3
- Rootless Docker Engine
- NVIDIA Container Toolkit 1.20.0

<!--more-->

Run all commands as a normal user with `sudo` access. Do not run rootless Docker setup commands from a root login.

## 1. Inspect the existing NVIDIA driver

```bash
cat /etc/os-release
uname -m
uname -r
sudo mokutil --sb-state
lspci -nnk -d 10de:
lsmod | grep -E '^(nvidia|nouveau)' || true
modinfo nvidia 2>/dev/null |
  grep -E '^(filename|version|license|signer):' || true
```

For this installation, the expected kernel module is the Canonical-signed 595 server-open module:

```text
version: 595.71.05
license: Dual MIT/GPL
signer: Canonical Ltd. Kernel Module Signing
```

Retain this signed module when Secure Boot is enabled. Do not replace it with NVIDIA DKMS packages.

## 2. Install the matching headless NVIDIA utilities

Confirm that the utility and compute-library versions match:

```bash
apt-cache policy \
  nvidia-utils-595-server \
  libnvidia-compute-595-server
```

Install the headless utilities:

```bash
sudo apt install --no-install-recommends nvidia-utils-595-server
```

Confirm that the driver can access the GPU:

```bash
nvidia-smi
```

Expected driver version: `595.71.05`.

## 3. Add NVIDIA's CUDA repository

Install prerequisites:

```bash
sudo apt update
sudo apt install -y ca-certificates wget
```

Install NVIDIA's CUDA repository keyring for Ubuntu 26.04:

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2604/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt update
```

Confirm that CUDA Toolkit 13.3 is available:

```bash
apt-cache policy cuda-toolkit-13-3
```

Ensure the toolkit will not replace the signed driver:

```bash
apt-get --simulate install cuda-toolkit-13-3 |
  awk '/^(Inst|Remv) / && $2 ~ /^(nvidia|libnvidia|cuda-drivers)/'
```

The simulation should print nothing.

## 4. Install CUDA Toolkit 13.3

Install the driver-independent toolkit package:

```bash
sudo apt install -y cuda-toolkit-13-3
```

Do not install the broader `cuda`, `cuda-13-3`, `cuda-runtime-13-3`, `cuda-drivers`, or `nvidia-open` packages because they can change the driver stack.

Add CUDA to the login PATH:

```bash
grep -qxF 'export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}' ~/.profile ||
  printf '\n# NVIDIA CUDA Toolkit\nexport PATH=/usr/local/cuda/bin${PATH:+:${PATH}}\n' >> ~/.profile

. ~/.profile
```

Confirm the toolkit:

```bash
nvcc --version
readlink -f /usr/local/cuda
nvidia-smi
```

Expected results:

- `nvcc` reports CUDA 13.3.
- `/usr/local/cuda` resolves to `/usr/local/cuda-13.3`.
- `nvidia-smi` continues to report driver 595.71.05.

`nvidia-smi` may display `CUDA Version: 13.2`. This is the driver's native CUDA capability, not the installed toolkit version. Driver 595 supports CUDA 13.x minor-version compatibility.

## 5. Add Docker's official Ubuntu repository

Remove conflicting distribution packages:

```bash
sudo apt remove -y \
  docker.io docker-compose docker-compose-v2 docker-doc docker-buildx \
  podman-docker containerd runc
```

If APT proposes removing LXD or another required package, cancel the operation and resolve the conflict before continuing.

Install repository and rootless-mode prerequisites:

```bash
sudo apt update
sudo apt install -y ca-certificates curl uidmap dbus-user-session
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Add Docker's repository:

```bash
sudo tee /etc/apt/sources.list.d/docker.sources >/dev/null <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

## 6. Install Docker Engine with rootless support

```bash
sudo apt install -y \
  docker-ce \
  docker-ce-cli \
  containerd.io \
  docker-buildx-plugin \
  docker-compose-plugin \
  docker-ce-rootless-extras
```

Docker's package starts a system-wide rootful daemon. Disable it:

```bash
sudo systemctl disable --now docker.service docker.socket
sudo rm -f /var/run/docker.sock
```

Confirm subordinate UID and GID mappings exist for the current user:

```bash
grep "^$(whoami):" /etc/subuid /etc/subgid
```

Each file should assign at least 65,536 subordinate IDs.

Create the rootless daemon as the normal user, without `sudo`:

```bash
dockerd-rootless-setuptool.sh install
```

Keep it running after logout and start it at boot:

```bash
sudo loginctl enable-linger "$(whoami)"
systemctl --user enable --now docker
docker context use rootless
```

Do not add the user to the `docker` group. That group is for the rootful daemon and effectively grants root-level access.

Confirm rootless Docker:

```bash
docker info | grep -A5 "Security Options"
docker run --rm hello-world
```

The Docker security options must include `rootless`.

## 7. Install NVIDIA Container Toolkit

Install repository prerequisites:

```bash
sudo apt update
sudo apt install -y --no-install-recommends ca-certificates curl gnupg2
```

Add NVIDIA's production repository:

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey |
  sudo gpg --dearmor \
    -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
```

```bash
curl -sL https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list |
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' |
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt update
```

Install the documented production version:

```bash
NVIDIA_CONTAINER_TOOLKIT_VERSION=1.20.0-1

sudo apt install -y \
  nvidia-container-toolkit=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
  nvidia-container-toolkit-base=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
  libnvidia-container-tools=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
  libnvidia-container1=${NVIDIA_CONTAINER_TOOLKIT_VERSION}
```

## 8. Configure GPU access for rootless Docker

Configure the current user's Docker daemon. Do not use `sudo` for this command:

```bash
nvidia-ctk runtime configure \
  --runtime=docker \
  --config="$HOME/.config/docker/daemon.json"
```

Enable NVIDIA's rootless cgroup configuration:

```bash
sudo nvidia-ctk config \
  --set nvidia-container-cli.no-cgroups \
  --in-place
```

Restart the rootless daemon:

```bash
systemctl --user restart docker
```

Confirm GPU access from a rootless container:

```bash
docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi
```

The output should identify the NVIDIA GeForce RTX 3090 and driver 595.71.05.

## Official references

- [Ubuntu Server: NVIDIA drivers installation](https://ubuntu.com/server/docs/how-to/graphics/install-nvidia-drivers/)
- [Ubuntu: UEFI Secure Boot](https://documentation.ubuntu.com/security/security-features/platform-protections/secure-boot/)
- [NVIDIA Driver Installation Guide for Ubuntu](https://docs.nvidia.com/datacenter/tesla/driver-installation-guide/ubuntu.html)
- [NVIDIA CUDA Installation Guide for Linux](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/)
- [NVIDIA CUDA compatibility](https://docs.nvidia.com/deploy/cuda-compatibility/latest/)
- [Docker Engine installation on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
- [Docker rootless mode](https://docs.docker.com/engine/security/rootless/)
- [NVIDIA Container Toolkit installation](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)

## Theoretical GPU compatibility

This exact installation was validated only on the RTX 3090. Based on the [NVIDIA 595.71.05 supported-products list](https://download.nvidia.com/XFree86/Linux-x86_64/595.71.05/README/supportedchips.html), NVIDIA's open kernel module support for Turing and newer GPUs, and the [compute targets supported by CUDA 13.3](https://docs.nvidia.com/cuda/cuda-compiler-driver-nvcc/#list-of-supported-gpu-codes), the same stack should theoretically support these GPUs:

- **RTX 30 series:** RTX 3090 Ti, RTX 3090, RTX 3080 Ti, RTX 3080, RTX 3070 Ti, RTX 3070, RTX 3060 Ti, RTX 3060, RTX 3050
- **RTX 40 series:** RTX 4090, RTX 4080 SUPER, RTX 4080, RTX 4070 Ti SUPER, RTX 4070 Ti, RTX 4070 SUPER, RTX 4070, RTX 4060 Ti, RTX 4060
- **RTX 50 series:** RTX 5090, RTX 5080, RTX 5070 Ti, RTX 5070, RTX 5060 Ti, RTX 5060, RTX 5050
- **Ada workstation:** RTX 6000 Ada Generation, RTX 5880 Ada Generation, RTX 5000 Ada Generation, RTX 4500 Ada Generation, RTX 4000 Ada Generation, RTX 4000 SFF Ada Generation, RTX 2000 Ada Generation
- **Ada data center:** L40S, L40, L20, L4, L2
- **Blackwell workstation and server:** RTX PRO 6000 Blackwell Workstation Edition, RTX PRO 6000 Blackwell Max-Q Workstation Edition, RTX PRO 6000 Blackwell Server Edition, RTX PRO 5000 Blackwell, RTX PRO 5000 72GB Blackwell, RTX PRO 4500 Blackwell, RTX PRO 4000 Blackwell, RTX PRO 4000 Blackwell SFF Edition, RTX PRO 2000 Blackwell
- **Blackwell data center:** B200, GB200, B300, GB300

Board-vendor firmware, hybrid graphics, and device-specific packaging can still affect compatibility. Data-center GPUs may also require platform-specific firmware, fabric, and provisioning steps outside this guide. Treat GPUs other than the RTX 3090 as theoretically compatible rather than validated by this guide.

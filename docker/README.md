# NCCL Tests - Docker

## Requirements

- Multiple machines on the **same VPN** with **SSH access enabled**
- NVIDIA GPUs with compatible CUDA drivers installed
- Docker with NVIDIA Container Toolkit
- A valid `hostfile` listing all participating machines
- NCCL test image: `ghcr.io/voltagepark/nccl-tests:cuda12.4.1-ubuntu22.04`

## Overview

### 🔗 How Do Docker Hosts Communicate?

Unlike Kubernetes, Docker does not handle pod networking or orchestrate SSH. You must ensure:

1. **Passwordless SSH Between Hosts**  
   All machines must have SSH access to each other via shared public keys.

2. **Hostfile Definition**  
   A `hostfile` must list the internal IPs or hostnames of each participating machine:

3. **Launcher Node**  
   Choose one machine as the **launcher**. SSH into it to start the job.

## Example Usage

1. SSH into the Launcher Node

```bash
ssh user@192.168.1.101
```

2. Create the Hostfile

```bash
echo -e "192.168.1.101\n192.168.1.102\n192.168.1.103" > hostfile.txt
```
3. Pull the dockerfile (use the docker file without --ssh server tags)

```bash
docker pull ghcr.io/voltagepark/nccl-tests:cuda12.4.1-ubuntu22.04
```


## Development

### 🧪 Local Docker Build & Test

To build and test the Docker image locally before pushing:

```bash
CUDA_VERSION=12.6.3
UBUNTU_VERSION=22.04

docker build \
  --build-arg CUDA_VERSION=${CUDA_VERSION} \
  --build-arg UBUNTU_VERSION=${UBUNTU_VERSION} \
  -f docker/Dockerfile \
  -t nccl-tests:cuda${CUDA_VERSION}-ubuntu${UBUNTU_VERSION} \
  .

docker run --rm -it --gpus all \
  --network=host \
  --privileged \
  nccl-tests:cuda12.6.3-ubuntu22.04 \
  mpirun -np 2 ./build/all_reduce_perf -b 8 -e 512M -f 2 -g 1
```

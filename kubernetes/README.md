# NCCL Tests - AllReduce Benchmark

## Overview

This repository provides performance and correctness tests for **NCCL (NVIDIA Collective Communications Library)**, focusing on the `all_reduce` collective operation.

NCCL is a high-performance library developed by NVIDIA for accelerating collective communication primitives (such as all-reduce, all-gather, broadcast, reduce, and reduce-scatter) on multi-GPU systems. It is optimized for NVIDIA hardware and widely used in deep learning frameworks like PyTorch and TensorFlow to scale training across multiple GPUs and nodes.

## About the AllReduce Test

The `all_reduce` operation sums arrays of data across all GPUs and distributes the result back to each GPU. It is fundamental in distributed deep learning for synchronizing gradients across devices.

The NCCL `all_reduce` test measures:

- **Bandwidth** (GB/s): How much data can be reduced per second.
- **Latency** (μs): Time taken for the operation to complete.
- **Scalability**: How performance changes with more GPUs or nodes.

### 🚀 Example Usage

#### Requirements

- Kubernetes cluster with GPU nodes
- [Kubeflow MPI Operator](https://github.com/kubeflow/mpi-operator)
- A valid `nccl_tests.yaml` job spec (included in this repo or customized to your environment)

---

#### 1. Deploy the MPI Operator

The MPI Operator is required to manage distributed MPI workloads across your Kubernetes cluster:
```bash
kubectl apply --server-side -f https://raw.githubusercontent.com/kubeflow/mpi-operator/v0.6.0/deploy/v2beta1/mpi-operator.yaml
```

#### 2. Configure MPI Command

In the `nccl_tests.yaml` file you will find a command like:
```bash
mpirun -np 64 ./build/all_reduce_perf -b 8M -e 8M -f 2 -g 1
```
**Explanation of key flags:**
- `-np 64`: Total number of MPI processes = total GPUs across all nodes (e.g., 8 nodes × 8 GPUs each).
- `-g 1`: One GPU per MPI process (recommended for multi-process).
- `-b`, `-e`: Start and end message size.
- `-f 2`: Number of iterations per message size.

⚠️ Ensure the number of Worker replicas in your YAML matches the number of nodes. For example, for 8 nodes:
```yaml
spec:
  mpiReplicaSpecs:
    Worker:
      replicas: 8
```

⚠️ Ensure the image matches the CUDA version supported on the nodes and has SSH support:
```yaml
spec:
  containers:
    - name: mpi-launcher
      image: ghcr.io/voltagepark/nccl-tests:cuda12.4.1-ubuntu22.04-ssh

spec:
  containers:
    - name: mpi-worker
      image: ghcr.io/voltagepark/nccl-tests:cuda12.4.1-ubuntu22.04-ssh
```

#### 3. Launch the NCCL Test Job

Apply your job definition (nccl_tests.yaml):
```bash
kubectl apply -f nccl_tests.yaml

kubectl get pods --watch
```
This will start the NCCL all_reduce_perf benchmark across the configured GPU workers.

Note: The launcher pod may enter a CrashLoopBackOff state until all worker pods are up and running. 

For debugging add the flag `-x NCCL_DEBUG=INFO` to the MPI command.

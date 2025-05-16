# NCCL Tests
This project incorporates [NVIDIA NCCL Tests](https://github.com/NVIDIA/nccl-tests) as part of our benchmarking and validation framework. 

NCCL is a high-performance library developed by NVIDIA for accelerating collective communication primitives (such as all-reduce, all-gather, broadcast, reduce, and reduce-scatter) on multi-GPU systems. It is optimized for NVIDIA hardware and widely used in deep learning frameworks like PyTorch and TensorFlow to scale training across multiple GPUs and nodes.

### Kubernetes
For details on how we deploy and manage these tests in Kubernetes, see our [Kubernetes README](kubernetes/README.md).

### Docker
For details on how we deploy and manage these tests in Docker, see our [Docker README](docker/README.md).

## NCCL Tests - AllReduce Benchmark

### About the AllReduce Test

The `all_reduce` operation sums arrays of data across all GPUs and distributes the result back to each GPU. It is fundamental in distributed deep learning for synchronizing gradients across devices.

The NCCL `all_reduce` test measures:

- **Bandwidth** (GB/s): How much data can be reduced per second.
- **Latency** (μs): Time taken for the operation to complete.
- **Scalability**: How performance changes with more GPUs or nodes.

## Copyright
NCCL tests are provided under the BSD license. All source code and accompanying documentation is copyright (c) 2016-2025, NVIDIA CORPORATION. All rights reserved.

# NCCL Tests - Kubernetes

## Requirements

- Kubernetes cluster with GPU nodes
- [Kubeflow MPI Operator](https://github.com/kubeflow/mpi-operator)
- A valid `nccl_tests.yaml` job spec (included in this repo or customized to your environment)
- NCCL test image w/ SSH server: `ghcr.io/voltagepark/nccl-tests:cuda12.4.1-ubuntu22.04-ssh-server` (matched to your local cuda version)

## Overview

#### 📦 What is the MPI Operator?

The MPI operator automates the deployment and management of **MPI-based distributed jobs** in Kubernetes. It:

- Manages **MPI launcher and worker pods**.
- Sets up **hostfile configuration and SSH** for MPI to work across pods.
- Integrates with Kubernetes to monitor pod lifecycle and job completion.
- Enables **scaling and reproducibility** for GPU-intensive distributed training or benchmarking.

When you create an `MPIJob` custom resource, the MPI Operator spawns:
- One **launcher pod** that executes `mpirun`.
- Multiple **worker pods**, one per replica, typically tied to a physical GPU.

#### 🔗 How Do MPI Pods Find Each Other?

MPI requires all participating processes to know how to **reach each other over the network**. Here's how the MPI Operator facilitates this:

1. **DNS-based Pod Discovery**  
   Worker pods are created with predictable names (e.g., `nccl-tests-worker-0`, `nccl-tests-worker-1`, etc.). These names resolve to pod IPs via Kubernetes DNS within the job's namespace.

2. **Generated Hostfile**  
   The MPI Operator automatically generates a **hostfile** containing the FQDNs/IPs of all worker pods. This file is passed to `mpirun` by the launcher pod.

3. **SSH Communication**  
   All pods use an image with an **SSH server** and shared key-based authentication, allowing passwordless SSH between launcher and workers. This is critical for `mpirun` to launch and monitor processes across nodes.

4. **MPIJob Spec**  
   The YAML defines how many replicas (`Worker.replicas`) should be created. The operator ensures these pods are running and ready before allowing the launcher pod to initiate MPI.


## Example Usage

#### 1. Deploy the MPI Operator

The MPI Operator is required to manage distributed MPI workloads across your Kubernetes cluster:
```bash
kubectl apply --server-side -f https://raw.githubusercontent.com/kubeflow/mpi-operator/v0.6.0/deploy/v2beta1/mpi-operator.yaml
```

#### 2. Configure MPI Command

In the `nccl_tests.yaml` file you will find a command like:
```bash
mpirun -np 64 ./build/all_reduce_perf -b 512M -e 8G -f 2 -g 1
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

⚠️ Ensure the image matches the CUDA version supported on the nodes and has SSH Server:
```yaml
spec:
  containers:
    - name: mpi-launcher
      image: ghcr.io/voltagepark/nccl-tests:cuda12.4.1-ubuntu22.04-ssh-server

spec:
  containers:
    - name: mpi-worker
      image: ghcr.io/voltagepark/nccl-tests:cuda12.4.1-ubuntu22.04-ssh-server
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

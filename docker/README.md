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

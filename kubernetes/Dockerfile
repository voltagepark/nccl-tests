ARG CUDA_VERSION=12.6.3
ARG UBUNTU_VERSION=22.04
FROM nvidia/cuda:${CUDA_VERSION}-devel-ubuntu${UBUNTU_VERSION}

RUN apt-get update && apt-get install -y \
    openssh-server \
    libibverbs-dev \
    ibverbs-utils \
    libnuma1 libnuma-dev \
    rdma-core \
    openmpi-bin libopenmpi-dev \
    git \
    build-essential && \
    rm -rf /var/lib/apt/lists/*

# Install latest NCCL
ARG NCCL_VERSION=2.27.7-1+cuda12.4
RUN apt-mark unhold libnccl2 libnccl-dev || true && \
    apt-get update && apt-get install -y libnccl2=${NCCL_VERSION} libnccl-dev=${NCCL_VERSION} && \
    apt-mark hold libnccl2 libnccl-dev && \
    rm -rf /var/lib/apt/lists/*

# Install HPC-X. Upgrade to DOCA_OFED once VP moves away from MLNX_OFED
RUN mkdir -p /opt/hpcx && \
    wget -q http://content.mellanox.com/hpc/hpc-x/v2.18.1/hpcx-v2.18.1-gcc-mlnx_ofed-ubuntu22.04-cuda12-x86_64.tbz && \
    tar -xjf hpcx-v2.18.1-gcc-mlnx_ofed-ubuntu22.04-cuda12-x86_64.tbz -C /opt/hpcx --strip-components=1 && \
    rm hpcx-v2.18.1-gcc-mlnx_ofed-ubuntu22.04-cuda12-x86_64.tbz
RUN echo "source /opt/hpcx/hpcx-init.sh && hpcx_load" >> /root/.bashrc
      
# Prepares SSH for mpi-operator
RUN mkdir -p /var/run/sshd && \
    echo "UserKnownHostsFile /dev/null" >> /etc/ssh/ssh_config && \
    echo "    StrictHostKeyChecking no" >> /etc/ssh/ssh_config && \
    sed -i 's/#\(StrictModes \).*/\1no/g' /etc/ssh/sshd_config

ENV MPI_HOME=/usr/lib/x86_64-linux-gnu/openmpi

RUN git clone https://github.com/NVIDIA/nccl-tests.git /opt/nccl-tests && \
    cd /opt/nccl-tests && \
    make MPI=1 MPI_HOME=$MPI_HOME

WORKDIR /opt/nccl-tests

CMD ["/usr/sbin/sshd", "-D"]

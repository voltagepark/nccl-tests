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
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Prepares SSH for mpi-operator
RUN mkdir -p /var/run/sshd && \
    echo "UserKnownHostsFile /dev/null" >> /etc/ssh/ssh_config && \
    echo "StrictHostKeyChecking no" >> /etc/ssh/ssh_config && \
    sed -i 's/#\(StrictModes \).*/\1no/g' /etc/ssh/sshd_config

ENV MPI_HOME=/usr/lib/x86_64-linux-gnu/openmpi

RUN git clone https://github.com/NVIDIA/nccl-tests.git /opt/nccl-tests && \
    cd /opt/nccl-tests && \
    make MPI=1 MPI_HOME=$MPI_HOME

WORKDIR /opt/nccl-tests

CMD ["/usr/sbin/sshd", "-D"]

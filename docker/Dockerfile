FROM hpcaitech/colossalai:0.1.10-torch1.11-cu11.3

##############################################################################
# Temporary Installation Directory
##############################################################################
ENV STAGE_DIR=/tmp/stage_dir
RUN mkdir -p ${STAGE_DIR}

##############################################################################
# Installation/Basic Utilities
##############################################################################
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && \
        apt-get install -y --no-install-recommends \
        software-properties-common build-essential autotools-dev \
        pdsh g++ gcc \
        curl wget vim tmux emacs less unzip \
        htop iftop iotop ca-certificates \
        rsync iputils-ping net-tools sudo \
        libfuse-dev fuse \
        git git-lfs \
        # libnuma-dev is required by MLNX
        libnuma-dev \
        dos2unix psmisc graphviz llvm-10-dev ninja-build npm \
        libaio-dev \
        jq \
        lshw \
        dmidecode \
        util-linux \
        automake \
        autoconf \
        libtool \
        perftest \
        net-tools \
        openssh-client \
        openssh-server \
        pciutils \
        libaio-dev \
        libcap2 \
        lsb-release

RUN apt-get clean -y all

# Remove apt intermmediate files
RUN rm -rf /var/lib/apt/lists/*

##############################################################################
# Mellanox OFED
##############################################################################
ENV MLNX_OFED_VERSION=5.1-2.5.8.0
RUN cd ${STAGE_DIR} && \
        wget -q -O - http://content.mellanox.com/ofed/MLNX_OFED-${MLNX_OFED_VERSION}/MLNX_OFED_LINUX-${MLNX_OFED_VERSION}-ubuntu20.04-x86_64.tgz | tar xzf - && \
        cd MLNX_OFED_LINUX-${MLNX_OFED_VERSION}-ubuntu20.04-x86_64 && \
        ./mlnxofedinstall --user-space-only --without-fw-update --force --all -q --skip-unsupported-devices-check && \
        rm -rf ${STAGE_DIR}/MLNX_OFED_LINUX-${MLNX_OFED_VERSION}-ubuntu20.04-x86_64*

##############################################################################
# Python (MLNX 5.1 requires python2 .......)
##############################################################################
ENV DEBIAN_FRONTEND=noninteractive
ENV PYTHON_VERSION=3
RUN python -V && pip -V
        
##############################################################################
# nv_peer_mem
##############################################################################
ENV NV_PEER_MEM_VERSION=1.1
ENV NV_PEER_MEM_TAG=1.1-0
RUN mkdir -p ${STAGE_DIR} && \
        git clone https://github.com/Mellanox/nv_peer_memory.git --branch ${NV_PEER_MEM_TAG} ${STAGE_DIR}/nv_peer_memory && \
        cd ${STAGE_DIR}/nv_peer_memory && \
        ./build_module.sh && \
        cd /tmp && \
        tar xzf /tmp/nvidia-peer-memory_${NV_PEER_MEM_VERSION}.orig.tar.gz && \
        cd nvidia-peer-memory-${NV_PEER_MEM_VERSION} && \
        apt-get update && \
        apt-get install -y dkms && \
        dpkg-buildpackage -us -uc && \
        dpkg -i /tmp/nvidia-peer-memory_${NV_PEER_MEM_TAG}_all.deb && \
        rm -rf /var/lib/apt/lists/* ${STAGE_DIR}/nv_peer_memory /tmp/nvidia-peer-memory_${NV_PEER_MEM_VERSION}.orig.tar.gz /tmp/nvidia-peer-memory-${NV_PEER_MEM_VERSION}

##############################################################################
# NCCL RDMA Sharp plugin
##############################################################################
RUN cd ${STAGE_DIR} && \
    mkdir -p /usr/local/nccl-rdma-sharp-plugins && \
    apt-get update && \
    apt-get install -y zlib1g-dev && \
    git clone https://github.com/Mellanox/nccl-rdma-sharp-plugins.git && \
    cd nccl-rdma-sharp-plugins && \
    git checkout v2.0.x-ar && \
    ./autogen.sh && \
    ./configure --prefix=/usr/local/nccl-rdma-sharp-plugins --with-cuda=/usr/local/cuda && \
    make && \
    make install && \
    LD_LIBRARY_PATH=/usr/local/nccl-rdma-sharp-plugins/lib:${LD_LIBRARY_PATH} && \
    LD_PRELOAD=/usr/local/nccl-rdma-sharp-plugins/lib/libnccl-net.so:${LD_PRELOAD}

ENV LD_LIBRARY_PATH=/usr/local/nccl-rdma-sharp-plugins/lib:${LD_LIBRARY_PATH}

##############################################################################
# User Packages
##############################################################################
# install apex (91fcaaf718306e3441e9a8369aa4553eea2e326b)
RUN cd ${STAGE_DIR} && \
    git clone https://github.com/NVIDIA/apex && \
    cd apex && \
    pip install -v --disable-pip-version-check --no-cache-dir --global-option="--cpp_ext" --global-option="--cuda_ext" --global-option="--fast_layer_norm" ./

# install titans
RUN pip install --no-cache-dir titans==0.0.7

# install tensornvme (0316931784c552d3981a84a9db88da6f09d4a9df)
RUN cd ${STAGE_DIR} && \
    conda install cmake && \
    git clone https://github.com/hpcaitech/TensorNVMe.git && \
    cd TensorNVMe && \
    pip install -r requirements.txt && \
    pip install -v --no-cache-dir .

# install colossalai
RUN pip install colossalai==0.1.12+torch1.11cu11.3 -f https://release.colossalai.org
RUN pip install datasets==2.8.0 accelerate==0.15.0 transformers==4.25.1
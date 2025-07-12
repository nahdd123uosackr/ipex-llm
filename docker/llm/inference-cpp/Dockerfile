# Copyright (C) 2025 Intel Corporation
# SPDX-License-Identifier: Apache-2.0

# Stage 1: Build stage to handle file preparation
FROM ubuntu:24.04 as build
ENV DEVICE=iGPU
ENV no_proxy=localhost,127.0.0.1
ENV OLLAMA_NUM_GPU=999
ENV OLLAMA_KEEP_ALIVE=10m
ENV OLLAMA_HOST=0.0.0.0:11434
ENV ONEAPI_DEVICE_SELECTOR="level_zero:0"
ENV SYCL_CACHE_PERSISTENT=1
ENV ZES_ENABLE_SYSMAN=1
ENV SYCL_PI_LEVEL_ZERO_USE_IMMEDIATE_COMMANDLISTS=1
ENV IPEX_LLM_FORCE_BATCH_FORWARD=1
ENV SYCL_DEVICE_FILTER=level_zero:gpu
ENV SYCL_DEVICE_TYPE=GPU
ENV LEVEL_ZERO_CACHE_PERSISTENT=1
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Asia/Seoul
ENV IPEX_LLM_NUM_CTX=16384

# Copy the files to the build image
COPY docker/llm/inference-cpp/start-llama-cpp.sh docker/llm/inference-cpp/start-ollama.sh docker/llm/inference-cpp/benchmark_llama-cpp.sh /llm/scripts/


# Set build arguments for proxy
ARG http_proxy
ARG https_proxy
# Disable pip cache
ARG PIP_NO_CACHE_DIR=false

# Set environment variables
ENV TZ=Asia/Seoul \
    PYTHONUNBUFFERED=1 \
    SYCL_CACHE_PERSISTENT=1

# Install dependencies and configure the environment
RUN set -eux && \
    #
    # Ensure scripts are executable
    chmod +x /llm/scripts/*.sh && \
    apt-get update && \
    apt-get install -y --no-install-recommends curl wget git sudo libunwind8-dev vim less gnupg gpg-agent software-properties-common && \
    # Intel OneAPI 및 GPU 저장소 추가
    wget -O- https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB | gpg --dearmor | tee /usr/share/keyrings/intel-oneapi-archive-keyring.gpg > /dev/null && \
    echo "deb [signed-by=/usr/share/keyrings/intel-oneapi-archive-keyring.gpg] https://apt.repos.intel.com/oneapi all main" | tee /etc/apt/sources.list.d/oneAPI.list && \
    chmod 644 /usr/share/keyrings/intel-oneapi-archive-keyring.gpg && \
    rm -f /etc/apt/sources.list.d/intel-graphics.list || true && \
    wget -O- https://repositories.intel.com/graphics/intel-graphics.key | gpg --dearmor | tee /usr/share/keyrings/intel-graphics.gpg > /dev/null && \
    echo "deb [arch=amd64,i386 signed-by=/usr/share/keyrings/intel-graphics.gpg] https://repositories.intel.com/graphics/ubuntu jammy arc" | tee /etc/apt/sources.list.d/intel.gpu.jammy.list && \
    chmod 644 /usr/share/keyrings/intel-graphics.gpg && \
    apt-get update && \
    # Intel GPU 관련 패키지 설치
    apt-get install -y --no-install-recommends libze-dev libze-intel-gpu1 libze1 intel-metrics-discovery intel-opencl-icd clinfo intel-gsc intel-ocloc && \
    # Python 3.11 설치
    add-apt-repository ppa:deadsnakes/ppa -y && \
    apt-get install -y --no-install-recommends python3.11 python3-pip python3.11-dev python3.11-distutils python3-wheel && \
    rm /usr/bin/python3 && ln -s /usr/bin/python3.11 /usr/bin/python3 && ln -s /usr/bin/python3 /usr/bin/python && \
    # IPEX-LLM 재설치
    pip3 install --break-system-packages --pre --upgrade ipex-llm[xpu]==2.3.0b20250630 --extra-index-url https://pytorch-extension.intel.com/release-whl/stable/xpu/us/ && \
    # Intel GPU 런타임 및 드라이버 패키지 레포지토리에서 설치
    apt-get install -y --no-install-recommends intel-opencl-icd libigdgmm12 intel-metrics-discovery clinfo intel-gsc intel-ocloc intel-basekit && \
    # Clean up unnecessary dependencies to reduce image size
    find /usr/lib/python3/dist-packages/ -name 'blinker*' -exec rm -rf {} + && \
    rm -rf /root/.cache/Cypress
RUN cd / && mkdir /dev/dri && wget https://github.com/ipex-llm/ipex-llm/releases/download/v2.3.0-nightly/ollama-ipex-llm-2.3.0b20250630-ubuntu.tgz && \
    tar -xvf ./ollama-ipex-llm-2.3.0b20250630-ubuntu.tgz --strip-components=1 -C / && \
    rm ./ollama-ipex-llm-2.3.0b20250630-ubuntu.tgz

ENTRYPOINT ["/bin/bash", "/start-ollama.sh"]

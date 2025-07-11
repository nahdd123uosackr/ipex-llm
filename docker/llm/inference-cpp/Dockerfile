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
    apt-get install -y --no-install-recommends libze-dev libze-intel-gpu1 libze1 intel-metrics-discovery intel-opencl-icd clinfo intel-gsc intel-ocloc libze-intel-gpu-raytracing && \
    # Python 3.11 설치
    add-apt-repository ppa:deadsnakes/ppa -y && \
    apt-get install -y --no-install-recommends python3.11 python3-pip python3.11-dev python3.11-distutils python3-wheel && \
    rm /usr/bin/python3 && ln -s /usr/bin/python3.11 /usr/bin/python3 && ln -s /usr/bin/python3 /usr/bin/python && \
    # IPEX-LLM 재설치
    pip3 install --break-system-packages --pre --upgrade ipex-llm[xpu]==2.3.0b20250630 --extra-index-url https://pytorch-extension.intel.com/release-whl/stable/xpu/us/ && \
    # Intel GPU 드라이버 및 런타임(Compute Runtime, Level Zero 등) 수동 설치
    mkdir -p /tmp/gpu && cd /tmp/gpu && \
    echo "Downloading Intel Compute Runtime (24.52) for Gen12+..." && \
    wget https://github.com/intel/intel-graphics-compiler/releases/download/v2.5.6/intel-igc-core-2_2.5.6+18417_amd64.deb && \
    wget https://github.com/intel/intel-graphics-compiler/releases/download/v2.5.6/intel-igc-opencl-2_2.5.6+18417_amd64.deb && \
    wget https://github.com/intel/compute-runtime/releases/download/24.52.32224.5/intel-level-zero-gpu_1.6.32224.5_amd64.deb && \
    wget https://github.com/intel/compute-runtime/releases/download/24.52.32224.5/intel-opencl-icd_24.52.32224.5_amd64.deb && \
    wget https://github.com/intel/compute-runtime/releases/download/24.52.32224.5/libigdgmm12_22.5.5_amd64.deb && \
    echo "Downloading Legacy Compute Runtime (24.35) for pre-Gen12 support..." && \
    wget https://github.com/intel/compute-runtime/releases/download/24.35.30872.22/intel-level-zero-gpu-legacy1_1.3.30872.22_amd64.deb && \
    wget https://github.com/intel/compute-runtime/releases/download/24.35.30872.22/intel-opencl-icd-legacy1_24.35.30872.22_amd64.deb && \
    wget https://github.com/intel/intel-graphics-compiler/releases/download/igc-1.0.17537.20/intel-igc-core_1.0.17537.20_amd64.deb && \
    wget https://github.com/intel/intel-graphics-compiler/releases/download/igc-1.0.17537.20/intel-igc-opencl_1.0.17537.20_amd64.deb && \
    dpkg -i *.deb && rm -rf /tmp/gpu && \
    # Install oneAPI Level Zero Loader
    mkdir /tmp/level-zero && cd /tmp/level-zero && \
    wget https://github.com/oneapi-src/level-zero/releases/download/v1.20.2/level-zero_1.20.2+u22.04_amd64.deb && \
    wget https://github.com/oneapi-src/level-zero/releases/download/v1.20.2/level-zero-devel_1.20.2+u22.04_amd64.deb && \
    dpkg -i *.deb && rm -rf /tmp/level-zero && \
    # Clean up unnecessary dependencies to reduce image size
    find /usr/lib/python3/dist-packages/ -name 'blinker*' -exec rm -rf {} + && \
    rm -rf /root/.cache/Cypress && \
    # Ollama 바이너리 설치
    cd / && \
    wget -q https://github.com/ipex-llm/ipex-llm/releases/download/v2.3.0-nightly/ollama-ipex-llm-2.3.0b20250630-ubuntu.tgz && \
    mkdir -p /lib/ollama && \
    tar -xf ./ollama-ipex-llm-2.3.0b20250630-ubuntu.tgz --strip-components=1 -C /lib/ollama && \
    rm -f ./ollama-ipex-llm-2.3.0b20250630-ubuntu.tgz && \
    apt install intel-basekit -y && \
    # 라이브러리 링크 수정
    if [ -f "/opt/intel/oneapi/compiler/2024.2/lib/libur_loader.so.0" ]; then ln -sf /opt/intel/oneapi/compiler/2024.2/lib/libur_loader.so.0 /usr/lib/libur_loader.so.0; fi && \
    # GPU 디바이스 확인
    ls -la /dev/dri/ || echo "No DRI devices found" && \
    # 라이브러리 호환성 테스트
    python3 -c "try: import torch; print('PyTorch loaded successfully'); import intel_extension_for_pytorch as ipex; print('IPEX loaded successfully, version:', ipex.__version__); print('XPU device count:', ipex.xpu.device_count() if hasattr(ipex, 'xpu') else 'XPU not available') except Exception as e: print('Library test failed:', str(e))" || true
ENV LD_LIBRARY_PATH=/lib/ollama:/usr/lib/ollama:/opt/intel/oneapi/compiler/2024.2/lib:$LD_LIBRARY_PATH
ENTRYPOINT ["/lib/ollama/ollama", "serve"]

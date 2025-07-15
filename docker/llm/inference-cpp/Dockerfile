FROM ubuntu:24.04
ENV DEBIAN_FRONTEND=noninteractive
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

# Base packages
RUN apt update && \
    apt install --no-install-recommends -q -y \
    software-properties-common \
    ca-certificates \
    wget \
    ocl-icd-libopencl1

# Intel GPU compute user-space drivers
RUN mkdir -p /tmp/gpu && \
 cd /tmp/gpu && \
 wget https://github.com/oneapi-src/level-zero/releases/download/v1.21.9/level-zero_1.21.9+u24.04_amd64.deb && \
 wget https://github.com/intel/intel-graphics-compiler/releases/download/v2.8.3/intel-igc-core-2_2.8.3+18762_amd64.deb && \
 wget https://github.com/intel/intel-graphics-compiler/releases/download/v2.8.3/intel-igc-opencl-2_2.8.3+18762_amd64.deb && \
 wget https://github.com/intel/compute-runtime/releases/download/25.09.32961.7/intel-level-zero-gpu_1.6.32961.7_amd64.deb && \
 wget https://github.com/intel/compute-runtime/releases/download/25.09.32961.7/intel-opencl-icd_25.09.32961.7_amd64.deb && \
 wget https://github.com/intel/compute-runtime/releases/download/25.09.32961.7/libigdgmm12_22.6.0_amd64.deb && \
 dpkg -i *.deb && \
 rm *.deb

RUN mkdir -p /dev/dri 
RUN cd / && wget https://github.com/ipex-llm/ipex-llm/releases/download/v2.3.0-nightly/ollama-ipex-llm-2.3.0b20250630-ubuntu.tgz && \
    tar -xvf ./ollama-ipex-llm-2.3.0b20250630-ubuntu.tgz --strip-components=1 -C /lib && tar -xvf ./ollama-ipex-llm-2.3.0b20250630-ubuntu.tgz --strip-components=1 -C / &&\
    rm ./ollama-ipex-llm-2.3.0b20250630-ubuntu.tgz

EXPOSE 11434

ENTRYPOINT ["/bin/bash", "/lib/start-ollama.sh"]
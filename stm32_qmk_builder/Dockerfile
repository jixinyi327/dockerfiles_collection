# 基于 Debian 12 (ARM兼容版本)
FROM --platform=$TARGETPLATFORM debian:bookworm-slim

ARG TARGETPLATFORM
ARG BUILDPLATFORM
ARG TOOLS_PATH=/opt/gcc-arm-none-eabi
ARG ARM_VERSION=14.2.rel1
ENV QMK_HOME=/home/qmk_firmware
ENV STM32_HOME=/home/stm32_projects
# 安装基础工具和依赖
RUN apt-get update && apt-get install -y \
    build-essential \
    git gnupg2\
    wget \
    make \
    python3 \
    python3-pip \
    python3-venv \
    cmake \
    ninja-build \
    libusb-1.0-0-dev \
    pkg-config \
    software-properties-common \
    openocd \
    dfu-util \
    stlink-tools \
    xz-utils curl \
    clang-format clang-tidy \
    && rm -rf /var/lib/apt/lists/*

# Get ARM Toolchain
RUN if [ "$TARGETPLATFORM" = "linux/amd64" ]; then export ARM_ARCH=x86_64; \
	else export ARM_ARCH=aarch64; \
	fi \
	&& echo "Downloading ARM GNU GCC for Platform: $ARM_ARCH" \
	&& mkdir ${TOOLS_PATH} \
	&& curl -Lo gcc-arm-none-eabi.tar.xz "https://developer.arm.com/-/media/Files/downloads/gnu/${ARM_VERSION}/binrel/arm-gnu-toolchain-${ARM_VERSION}-${ARM_ARCH}-arm-none-eabi.tar.xz" \
	&& tar xf gcc-arm-none-eabi.tar.xz --strip-components=1 -C ${TOOLS_PATH} \
	&& rm gcc-arm-none-eabi.tar.xz \
	&& rm ${TOOLS_PATH}/*.txt \
	&& rm -rf ${TOOLS_PATH}/share/doc \
	&& echo "https://developer.arm.com/-/media/Files/downloads/gnu/${ARM_VERSION}/binrel/arm-gnu-toolchain-${ARM_VERSION}-${ARM_ARCH}-arm-none-eabi.tar.xz"


# Add Toolchain to PATH
ENV PATH="$PATH:${TOOLS_PATH}/bin"

# 安装 QMK 依赖
RUN python3 -m pip install --upgrade pip --break-system-packages && \
    python3 -m pip install qmk --break-system-packages

# 创建工作目录
RUN mkdir -p $QMK_HOME
RUN mkdir -p $STM32_HOME
WORKDIR /home

RUN qmk setup -y -H $QMK_HOME

# Get static analysis tools: clang-format clang-tidy, Lizard, cpplint
# 设置容器默认命令
CMD ["/bin/bash"]

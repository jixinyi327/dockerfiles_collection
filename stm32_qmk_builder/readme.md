# STM32 QMK Builder Docker

适用于 STM32 QMK 固件开发的 Docker 镜像，集成 ARM 工具链、QMK CLI 及常用调试工具，支持 x86_64 和 aarch64 平台。

## 目录
- [构建镜像](#构建镜像)
- [运行容器](#运行容器)
- [QMK 工程编译](#qmk-工程编译)
- [STM32 工程编译](#stm32-工程编译)
- [CMakeLists.txt 自动生成 bin/hex](#cmakelists.txt-自动生成-binhex)
- [容器内开发流程](#容器内开发流程)
- [其他说明](#其他说明)
- [English Version](#english-version)

## 构建镜像

在 Dockerfile 所在目录执行：

```sh
docker build -t stm32-qmk-builder .
```

## 运行容器

假设你的 QMK 项目和 STM32 项目分别在本地如下目录：

- QMK 固件目录：`/path/to/qmk_firmware`
- STM32 项目目录：`/path/to/stm32_projects`

启动容器并挂载本地目录：

```sh
docker run -it --rm \
  -v /path/to/keyboards:/home/qmk_firmware/keyboards \
  -v /path/to/build:/home/qmk_firmware/.build \
  -v /path/to/stm32_projects:/home/stm32_projects \
  stm32-qmk-builder
```

## QMK 工程编译

在容器中执行 QMK 编译：

```sh
qmk compile -kb <keyboard> -km <keymap>
```

## STM32 工程编译

通过 STM32CubeMX 创建 CMake 工程后，在容器中执行：

```sh
cd /home/stm32_projects/your_project/
mkdir build
cmake -B build -G "Ninja" -DCMAKE_BUILD_TYPE=Release
cd build
ninja
```

### CMakeLists.txt 自动生成 bin/hex

每次创建新的 STM32 工程后，可在对应工程目录下的 `CMakeLists.txt` 添加以下内容：

```cmake
# 自动生成 .bin 和 .hex 文件
find_program(OBJCOPY arm-none-eabi-objcopy HINTS ${TOOLCHAIN_BIN_PATH})
set(BIN_FILE ${CMAKE_PROJECT_NAME}.bin)
set(HEX_FILE ${CMAKE_PROJECT_NAME}.hex)

add_custom_command(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD
    COMMAND ${OBJCOPY} -O binary $<TARGET_FILE:${CMAKE_PROJECT_NAME}> ${BIN_FILE}
    COMMENT "Generating ${BIN_FILE}"
    VERBATIM
)
add_custom_command(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD
    COMMAND ${OBJCOPY} -O ihex $<TARGET_FILE:${CMAKE_PROJECT_NAME}> ${HEX_FILE}
    COMMENT "Generating ${HEX_FILE}"
    VERBATIM
)
```

## 容器内开发流程

1. QMK CLI 已安装并初始化，工作目录为 `/home/qmk_firmware`
2. ARM GCC 工具链已加入 PATH，可直接编译 STM32 固件
3. 支持 openocd、dfu-util、stlink-tools 烧录调试
4. 可使用 clang-format、clang-tidy 进行代码静态分析

## 其他说明

- 支持 x86_64 和 aarch64 架构自动下载对应 ARM 工具链
- 如需自定义 QMK 或 STM32 目录，修改挂载参数即可
- 默认进入 `/home` 目录，可根据需要切换到具体项目目录

---

如有问题请参考 Dockerfile 注释或联系维护者。

---

## English Version

# STM32 QMK Builder Docker

A Docker image for STM32 QMK firmware development, integrating ARM toolchain, QMK CLI, and common debugging tools. Supports x86_64 and aarch64 platforms.

## Table of Contents
- [STM32 QMK Builder Docker](#stm32-qmk-builder-docker)
  - [目录](#目录)
  - [构建镜像](#构建镜像)
  - [运行容器](#运行容器)
  - [QMK 工程编译](#qmk-工程编译)
  - [STM32 工程编译](#stm32-工程编译)
    - [CMakeLists.txt 自动生成 bin/hex](#cmakeliststxt-自动生成-binhex)
  - [容器内开发流程](#容器内开发流程)
  - [其他说明](#其他说明)
  - [English Version](#english-version)
- [STM32 QMK Builder Docker](#stm32-qmk-builder-docker-1)
  - [Table of Contents](#table-of-contents)
  - [Build Image](#build-image)
  - [Run Container](#run-container)
  - [QMK Firmware Build](#qmk-firmware-build)
  - [STM32 Project Build](#stm32-project-build)
    - [Auto bin/hex Generation in CMakeLists.txt](#auto-binhex-generation-in-cmakeliststxt)
  - [Development Workflow](#development-workflow)
  - [Notes](#notes)

## Build Image

Run in the directory containing the Dockerfile:

```sh
docker build -t stm32-qmk-builder .
```

## Run Container

Assume your QMK and STM32 projects are in:

- QMK firmware: `/path/to/qmk_firmware`
- STM32 projects: `/path/to/stm32_projects`

Start the container and mount local directories:

```sh
docker run -it --rm \
  -v /path/to/keyboards:/home/qmk_firmware/keyboards \
  -v /path/to/build:/home/qmk_firmware/.build \
  -v /path/to/stm32_projects:/home/stm32_projects \
  stm32-qmk-builder
```

## QMK Firmware Build

Compile QMK firmware in the container:

```sh
qmk compile -kb <keyboard> -km <keymap>
```

## STM32 Project Build

After creating a CMake project with STM32CubeMX, run:

```sh
cd /home/stm32_projects/your_project/
mkdir build
cmake -B build -G "Ninja" -DCMAKE_BUILD_TYPE=Release
cd build
ninja
```

### Auto bin/hex Generation in CMakeLists.txt

Add the following to your project's `CMakeLists.txt` to auto-generate `.bin` and `.hex` files:

```cmake
# Auto-generate .bin and .hex files
find_program(OBJCOPY arm-none-eabi-objcopy HINTS ${TOOLCHAIN_BIN_PATH})
set(BIN_FILE ${CMAKE_PROJECT_NAME}.bin)
set(HEX_FILE ${CMAKE_PROJECT_NAME}.hex)

add_custom_command(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD
    COMMAND ${OBJCOPY} -O binary $<TARGET_FILE:${CMAKE_PROJECT_NAME}> ${BIN_FILE}
    COMMENT "Generating ${BIN_FILE}"
    VERBATIM
)
add_custom_command(TARGET ${CMAKE_PROJECT_NAME} POST_BUILD
    COMMAND ${OBJCOPY} -O ihex $<TARGET_FILE:${CMAKE_PROJECT_NAME}> ${HEX_FILE}
    COMMENT "Generating ${HEX_FILE}"
    VERBATIM
)
```

## Development Workflow

1. QMK CLI is installed and initialized at `/home/qmk_firmware`
2. ARM GCC toolchain is in PATH, ready for STM32 firmware build
3. openocd, dfu-util, stlink-tools available for flashing/debugging
4. clang-format, clang-tidy available for static code analysis

## Notes

- Supports x86_64 and aarch64 platforms, auto-downloads correct ARM toolchain
- Customize QMK/STM32 directories by changing mount parameters
- Default working directory is `/home`; switch as needed

---

For questions, refer to Dockerfile comments or contact the maintainer.
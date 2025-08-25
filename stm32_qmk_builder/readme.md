## English Version

# STM32 QMK Builder Docker

A Docker image for STM32 QMK firmware development, integrating ARM toolchain, QMK CLI, and common debugging tools. Supports x86_64 and aarch64 platforms.

## Table of Contents
- [STM32 QMK Builder Docker](#stm32-qmk-builder-docker)
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
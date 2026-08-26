cmake_minimum_required(VERSION 3.13)

# Copy pico_sdk_import.cmake from $PICO_SDK_PATH/external/ into this folder
include(pico_sdk_import.cmake)

project(bughunt3 C CXX ASM)

pico_sdk_init()

add_executable(bughunt3
    bughunt3.c
)

target_link_libraries(bughunt3 pico_stdlib)

pico_enable_stdio_usb(bughunt3 1)
pico_enable_stdio_uart(bughunt3 0)

pico_add_extra_outputs(bughunt3)

# --------------------------------------------------------------------
# Bug Hunt #3 requires you to compare optimisation levels.
# Configure the build twice:
#
#   cmake -DPICO_BOARD=pico_w -DCMAKE_BUILD_TYPE=Debug   ..   # -O0
#   cmake -DPICO_BOARD=pico_w -DCMAKE_BUILD_TYPE=Release ..   # -O3
#
# ...into two SEPARATE build directories, and keep both .uf2 files.
# --------------------------------------------------------------------

cmake_minimum_required(VERSION 3.13)

# Copy pico_sdk_import.cmake from $PICO_SDK_PATH/external/ into this folder
include(pico_sdk_import.cmake)

project(bughunt6 C CXX ASM)

pico_sdk_init()

add_executable(bughunt6
    bughunt6_pico.c
    bughunt6.c
)

target_link_libraries(bughunt6 pico_stdlib)

pico_enable_stdio_usb(bughunt6 1)
pico_enable_stdio_uart(bughunt6 0)

pico_add_extra_outputs(bughunt6)

# --------------------------------------------------------------------
# Configure this project TWICE, into two separate build directories:
#
#   cmake -DPICO_BOARD=pico_w -DCMAKE_BUILD_TYPE=Debug   ..   # -O0
#   cmake -DPICO_BOARD=pico_w -DCMAKE_BUILD_TYPE=Release ..   # -O3
#
# Several defects in this hunt exist in only one of the two.
# --------------------------------------------------------------------

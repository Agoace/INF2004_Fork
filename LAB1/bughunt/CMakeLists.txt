cmake_minimum_required(VERSION 3.13)

# Copy pico_sdk_import.cmake from $PICO_SDK_PATH/external/ into this folder
include(pico_sdk_import.cmake)

project(bughunt1 C CXX ASM)

pico_sdk_init()

add_executable(bughunt1
    bughunt1.c
)

target_link_libraries(bughunt1 pico_stdlib)

# Serial output over USB so you can read the test results
pico_enable_stdio_usb(bughunt1 1)
pico_enable_stdio_uart(bughunt1 0)

pico_add_extra_outputs(bughunt1)

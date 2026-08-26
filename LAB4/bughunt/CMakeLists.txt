cmake_minimum_required(VERSION 3.13)

# Copy pico_sdk_import.cmake from $PICO_SDK_PATH/external/ into this folder
include(pico_sdk_import.cmake)

project(bughunt4 C CXX ASM)

pico_sdk_init()

add_executable(bughunt4
    bughunt4.c
    algo.c
)

target_link_libraries(bughunt4
    pico_stdlib
    hardware_pwm
    hardware_adc
    hardware_clocks
)

pico_enable_stdio_usb(bughunt4 1)
pico_enable_stdio_uart(bughunt4 0)

pico_add_extra_outputs(bughunt4)

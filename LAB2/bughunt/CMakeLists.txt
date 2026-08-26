cmake_minimum_required(VERSION 3.13)

# Copy pico_sdk_import.cmake from $PICO_SDK_PATH/external/ into this folder
include(pico_sdk_import.cmake)

project(bughunt2 C CXX ASM)

pico_sdk_init()

add_executable(bughunt2
    bughunt2_pico.c
    frame.c
)

target_link_libraries(bughunt2
    pico_stdlib
    hardware_uart
)

# printf goes over USB; the link under test is UART1 on GP8/GP9
pico_enable_stdio_usb(bughunt2 1)
pico_enable_stdio_uart(bughunt2 0)

pico_add_extra_outputs(bughunt2)

cmake_minimum_required(VERSION 3.13)

include(pico_sdk_import.cmake)

project(lab1 C CXX ASM)

pico_sdk_init()

add_executable(basic basic.c)
target_link_libraries(basic pico_stdlib)
pico_enable_stdio_usb(basic 1)
pico_enable_stdio_uart(basic 0)
pico_add_extra_outputs(basic)
target_compile_options(basic PRIVATE -Wall -Wextra)

add_executable(blinky blinky.c)
target_link_libraries(blinky pico_stdlib)
pico_enable_stdio_usb(blinky 1)
pico_enable_stdio_uart(blinky 0)
pico_add_extra_outputs(blinky)
target_compile_options(blinky PRIVATE -Wall -Wextra)

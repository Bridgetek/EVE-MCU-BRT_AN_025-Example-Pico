# Set minimum required version of CMake
cmake_minimum_required(VERSION 3.12)
#include build functions from Pico SDK
include($ENV{PICO_SDK_PATH}/external/pico_sdk_import.cmake)
# Set name of project (as current working directory)
get_filename_component(ProjectId ${CMAKE_CURRENT_LIST_DIR} NAME)
string(REPLACE " " "_" ProjectId ${ProjectId})
project(${ProjectId} C CXX)
# Set C/C++ Standards
set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)
# Creates a pico-sdk subdirectory in our project for the libraries
pico_sdk_init()
# point out the CMake, where to find the executable source file
add_executable(${PROJECT_NAME}
        main/main.c
)
include_directories(
        lib/eve/include
        example/
)
add_compile_definitions(PLATFORM_RP2040)
add_library(eve_library
        lib/eve/source/EVE_API.c
        lib/eve/source/EVE_HAL.c
        lib/eve/ports/eve_arch_rpi/EVE_MCU_RP2040.c
)
target_include_directories(eve_library PUBLIC
        lib/eve/include
)
add_library(eve_example
        example/eve_calibrate.c
        example/eve_example.c
        example/eve_fonts.c
        example/eve_helper.c
        example/eve_images.c
)
target_include_directories(eve_example PUBLIC
        example/
        lib/eve/include
)
target_link_libraries(eve_example
        eve_library
)
target_link_libraries(${PROJECT_NAME} 
        eve_library
        eve_example
)
# create map/bin/hex/uf2 files.
pico_add_extra_outputs(${PROJECT_NAME})
# Pull in our pico_stdlib which pulls in commonly used features (gpio, timer-delay etc)
target_link_libraries(${PROJECT_NAME}
            pico_stdlib
            hardware_spi
)
target_link_libraries(eve_library
            pico_stdlib
            hardware_spi
)
# Enable usb output, disable uart output
pico_enable_stdio_usb(${PROJECT_NAME} 1)
pico_enable_stdio_uart(${PROJECT_NAME} 0)

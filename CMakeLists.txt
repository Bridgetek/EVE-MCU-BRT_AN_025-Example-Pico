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
# Set the target board to be a standard pico, you may change this for different targets
set(PICO_BOARD pico)

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
# Tell EVE library to use RP2040 as base class
add_compile_definitions(PLATFORM_RP2040)

# Source code for EVE library (targetted at RP2040)
add_library(eve_library # EVE library name
        # Source code for EVE library
        lib/eve/source/EVE_API.c
        lib/eve/source/EVE_HAL.c
        lib/eve/ports/eve_arch_rpi/EVE_MCU_RP2040.c
)
# Header files used for EVE library
target_include_directories(eve_library PUBLIC
        lib/eve/include
)
# EVE library dependencies
target_link_libraries(eve_library
            pico_stdlib
            hardware_spi
)

# Source for example code
add_library(eve_example # Example code library name
        # Source files for example code library
        example/eve_calibrate.c
        example/eve_example.c
        example/eve_fonts.c
        example/eve_helper.c
        example/eve_images.c
)
# Header files used for example code library
target_include_directories(eve_example PUBLIC
        example/ # Headers for example code
        lib/eve/include # Headers for EVE library
)
# Example code dependencies
target_link_libraries(eve_example
        eve_library # Example code depends on EVE library
)

# Project dependencies
target_link_libraries(${PROJECT_NAME} 
        eve_example # Project depends on example code library
        hardware_flash # For non-volatile storage
        hardware_sync # For non-volatile storage
        pico_stdlib # Commonly used features (gpio, timer-delay etc)
)
# Create map/bin/hex/uf2 files.
pico_add_extra_outputs(${PROJECT_NAME})

# Enable stdio on UART
pico_enable_stdio_uart(${PROJECT_NAME} 1)
# Enable stdio on USB
pico_enable_stdio_usb(${PROJECT_NAME} 1)

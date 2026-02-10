cmake_minimum_required(VERSION 3.5...3.28)

# Download SOF & PFG into build dir
# ===============================

# URLs (replace with real ones)
set(SOF_URL "https://releases.rocketboards.org/2025.10/gsrd/agilex5_dk_a5e065bb32aes1_gsrd/ghrd_a5ed065bb32ae6sr0.sof")
set(PFG_URL "https://releases.rocketboards.org/2024.11/zephyr/agilex5/hps_zephyr/hello_world/qspi_boot/qspi_flash_image_agilex5_boot.pfg")

# Destination inside build directory
set(BITSTREAM_DIR ${CMAKE_BINARY_DIR})
file(MAKE_DIRECTORY ${BITSTREAM_DIR})

# Final paths used by build
set(SOF_PATH ${BITSTREAM_DIR}/ghrd_a5ed065bb32ae6sr0.sof CACHE STRING "SOF File path" FORCE)
set(PFG_PATH ${BITSTREAM_DIR}/qspi_flash_image_agilex5_boot.pfg CACHE STRING "PFG File path" FORCE)

# Download SOF
if(NOT EXISTS ${SOF_PATH})
    message(STATUS "Downloading SOF from ${SOF_URL} using wget")
    execute_process(
        COMMAND wget -O ${SOF_PATH} ${SOF_URL}
        RESULT_VARIABLE res
    )
    if(NOT res EQUAL 0)
        message(FATAL_ERROR "Failed to download SOF")
    endif()
endif()

# Download PFG
if(NOT EXISTS ${PFG_PATH})
    message(STATUS "Downloading PFG from ${PFG_URL} using wget")
    execute_process(
        COMMAND wget -O ${PFG_PATH} ${PFG_URL}
        RESULT_VARIABLE res
    )
    if(NOT res EQUAL 0)
        message(FATAL_ERROR "Failed to download PFG")
    endif()
endif()



include(FetchContent)

set(FREERTOS_FATFS n)
set(FREERTOS_USB n)
set(FREERTOS_TCPIP n)
set(FREERTOS_LIBRSU n)
set(FREERTOS_LIBFCS n)

FetchContent_Declare(
    freertos
    GIT_REPOSITORY https://github.com/Ignitarium-Technology/freertos-socfpga.git
    GIT_TAG main
)

# Get the freertos repo
FetchContent_MakeAvailable(freertos)

# project
project(freertos_hello_world C CXX ASM)

set(LINKER_SCRIPT "${freertos_SOURCE_DIR}/lscript.ld")
include(${freertos_SOURCE_DIR}/tools/target_socfpga.cmake)

# target
add_executable(${PROJECT_NAME}.elf)

#Add image generation
generate_bin_file(${PROJECT_NAME}.elf)

# link to the baremetal library
target_link_libraries(${PROJECT_NAME}.elf
    PRIVATE freertos_socfpga
)

# sources
target_sources(${PROJECT_NAME}.elf
    PRIVATE hello_world.c
)

# specify linker script
target_link_options(${PROJECT_NAME}.elf PRIVATE
   -T ${LINKER_SCRIPT}
)


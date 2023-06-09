# Copyright (c) 2020 ARM Limited. All rights reserved.
# SPDX-License-Identifier: Apache-2.0

cmake_minimum_required(VERSION 3.19.0 FATAL_ERROR)

set(MBED_PATH ${CMAKE_CURRENT_SOURCE_DIR}/mbed-os CACHE INTERNAL "")
set(MBED_CONFIG_PATH ${CMAKE_CURRENT_BINARY_DIR} CACHE INTERNAL "")
set(APP_TARGET NuMaker-mbed-Azure-IoT-CSDK-OTA-example)

include(${MBED_PATH}/tools/cmake/app.cmake)

project(${APP_TARGET})

# Mbed CLI 2 fails to pass MBED_APP_START and friends to build system. See:
# https://github.com/ARMmbed/mbed-tools/issues/156
message(FATAL_ERROR
        "Mbed CLI 2 doesn't support locating application to another address. "
        "Use Mbed CLI 1 instead. "
        "See: https://github.com/ARMmbed/mbed-tools/issues/156"
)

add_subdirectory(${MBED_PATH})
add_subdirectory(mbed-client-for-azure)
add_subdirectory(ntp-client)

add_executable(${APP_TARGET})

mbed_configure_app_target(${APP_TARGET})

target_include_directories(${APP_TARGET}
    PRIVATE
        .
)

target_sources(${APP_TARGET}
    PRIVATE
        hsm_custom/custom_hsm_example.c
        src/main.cpp
        src/mem_stats.cpp
)

if("AZIOT_OTA" IN_LIST MBED_TARGET_LABELS)
    target_include_directories(${APP_TARGET}
        PRIVATE
            configs/COMPONENT_AZIOT_OTA
    )
endif()

target_link_libraries(${APP_TARGET}
    PRIVATE
        mbed-client-for-azure
        mbed-os
        mbed-storage-kvstore
        ntp-client
        mbed-mbedtls
        mbed-netsocket
        mbed-wifi
)

mbed_set_post_build(${APP_TARGET})

option(VERBOSE_BUILD "Have a verbose build process")
if(VERBOSE_BUILD)
    set(CMAKE_VERBOSE_MAKEFILE ON)
endif()

# Copyright 2019 Joe Drago. All rights reserved.
# SPDX-License-Identifier: BSD-2-Clause

cmake_minimum_required(VERSION 3.5)
project(libavif)

if(CMAKE_CXX_COMPILER_ID MATCHES "GNU")
  add_definitions(-std=c99) # Enforce C99 for gcc
endif()

option(AVIF_CODEC_AOM "Use the AOM codec for encoding (and decoding if no other decoder is present)" OFF)
option(AVIF_CODEC_DAV1D "Use the dav1d codec for decoding (overrides AOM decoding if also enabled)" OFF)
option(AVIF_CODEC_RAV1E "Use the rav1e codec for encoding (overrides AOM encoding if also enabled)" OFF)

option(AVIF_LOCAL_AOM "Build the AOM codec by providing your own copy of the repo in ext/aom" OFF)
option(AVIF_LOCAL_DAV1D "Build the dav1d codec by providing your own copy of the repo in ext/dav1d" OFF)
option(AVIF_LOCAL_RAV1E "Build the rav1e codec by providing your own copy of the repo in ext/rav1e" OFF)

add_subdirectory(ext)

include_directories(include)

# Enable all warnings
if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  MESSAGE(STATUS "libavif: Enabling warnings for Clang")
  add_definitions(
    -Weverything
    -Werror
    -Wno-bad-function-cast
    -Wno-c11-extensions
    -Wno-cast-align
    -Wno-cast-qual
    -Wno-covered-switch-default
    -Wno-deprecated-declarations
    -Wno-disabled-macro-expansion
    -Wno-documentation
    -Wno-documentation-unknown-command
    -Wno-double-promotion
    -Wno-format-nonliteral
    -Wno-language-extension-token
    -Wno-missing-noreturn
    -Wno-padded
    -Wno-reserved-id-macro
    -Wno-sign-conversion
    -Wno-switch-enum
    -Wno-undef
  )
elseif(CMAKE_CXX_COMPILER_ID MATCHES "GNU")
  add_definitions(-std=c99) # Enforce C99 for gcc
  MESSAGE(STATUS "libavif: Enabling warnings for GCC")
  add_definitions(-Werror -Wall -Wextra)
elseif(CMAKE_CXX_COMPILER_ID MATCHES "MSVC")
  MESSAGE(STATUS "libavif: Enabling warnings for MS CL")
  add_definitions(
    /Wall   # All warnings
    /WX     # Warnings as errors
    /wd4255 # Disable: no function prototype given
    /wd4324 # Disable: structure was padded due to alignment specifier
    /wd4668 # Disable: is not defined as a preprocessor macro, replacing with '0'
    /wd4710 # Disable: function not inlined
    /wd4711 # Disable: function selected for inline expansion
    /wd4738 # Disable: storing 32-bit float result in memory, possible loss of performance
    /wd4820 # Disable: bytes padding added after data member
    /wd4996 # Disable: potentially unsafe stdlib methods
    /wd5045 # Disable: Compiler will insert Spectre mitigation for memory load if /Qspectre switch specified
  )
else()
  MESSAGE(FATAL_ERROR "libavif: Unknown compiler, bailing out")
endif()

set(AVIF_SRCS
    include/avif/avif.h
    include/avif/internal.h
    src/avif.c
    src/colr.c
    src/mem.c
    src/rawdata.c
    src/read.c
    src/reformat.c
    src/stream.c
    src/utils.c
    src/write.c
)

set(AVIF_PLATFORM_LIBRARIES)
if(NOT WIN32)
    set(AVIF_PLATFORM_LIBRARIES m pthread)
endif()

set(AVIF_CODEC_LIBRARIES)

if(AVIF_CODEC_DAV1D)
    message(STATUS "libavif: Codec enabled: dav1d (decode)")
    add_definitions(-DAVIF_CODEC_DAV1D=1)
    set(AVIF_SRCS ${AVIF_SRCS}
        src/codec_dav1d.c
    )

    if(AVIF_LOCAL_DAV1D)
        set(LIB_FILENAME "${CMAKE_CURRENT_SOURCE_DIR}/ext/dav1d/build/src/libdav1d.a")
        if(NOT EXISTS "${LIB_FILENAME}")
            message(FATAL_ERROR "libavif: ${LIB_FILENAME} is missing, bailing out")
        endif()

        include_directories(
            "${CMAKE_CURRENT_SOURCE_DIR}/ext/dav1d/build"
            "${CMAKE_CURRENT_SOURCE_DIR}/ext/dav1d/build/include"
            "${CMAKE_CURRENT_SOURCE_DIR}/ext/dav1d/build/include/dav1d"
            "${CMAKE_CURRENT_SOURCE_DIR}/ext/dav1d/include"
        )
        set(AVIF_CODEC_LIBRARIES ${AVIF_CODEC_LIBRARIES} ${LIB_FILENAME})
    else()
        # Check to see if dav1d is independently being built by the outer CMake project
	if(NOT TARGET dav1d)
            find_library(LIBDAV1D_LIBRARY_PATH NAMES dav1d)
            if(NOT LIBDAV1D_LIBRARY_PATH)
              message(FATAL_ERROR "libavif: dav1d library not found")
            endif()
            find_path (LIBDAV1D_INCLUDE_PATH dav1d/dav1d.h)
            if(NOT LIBDAV1D_INCLUDE_PATH)
              message(FATAL_ERROR "libavif: dav1d library includes not found")
            endif()
            message(STATUS "LIBDAV1D_INCLUDE_PATH: ${LIBDAV1D_INCLUDE_PATH}")
            include_directories(${LIBDAV1D_INCLUDE_PATH})
        endif()
        set(AVIF_CODEC_LIBRARIES ${AVIF_CODEC_LIBRARIES} dav1d)
    endif()
endif()

if(AVIF_CODEC_RAV1E)
    message(STATUS "libavif: Codec enabled: rav1e (encode)")
    add_definitions(-DAVIF_CODEC_RAV1E=1)
    set(AVIF_SRCS ${AVIF_SRCS}
        src/codec_rav1e.c
    )

    if(AVIF_LOCAL_RAV1E)
        set(LIB_FILENAME "${CMAKE_CURRENT_SOURCE_DIR}/ext/rav1e/target/release/rav1e.lib")
        if(NOT EXISTS "${LIB_FILENAME}")
            set(LIB_FILENAME "${CMAKE_CURRENT_SOURCE_DIR}/ext/rav1e/target/release/librav1e.a")
            if(NOT EXISTS "${LIB_FILENAME}")
                message(FATAL_ERROR "libavif: compiled rav1e library is missing (in ext/rav1e/target/release), bailing out")
            endif()
        endif()

        include_directories(
            "${CMAKE_CURRENT_SOURCE_DIR}/ext/rav1e/target/release/include"
        )
        set(AVIF_CODEC_LIBRARIES ${AVIF_CODEC_LIBRARIES} ${LIB_FILENAME})
    else()
        # Check to see if rav1e is independently being built by the outer CMake project
	if(NOT TARGET rav1e)
            find_library(LIBRAV1E_LIBRARY_PATH NAMES rav1e)
            if(NOT LIBRAV1E_LIBRARY_PATH)
                message(FATAL_ERROR "libavif: rav1e library not found")
            endif()
            find_path (LIBRAV1E_INCLUDE_PATH rav1e/rav1e.h)
            if(NOT LIBRAV1E_INCLUDE_PATH)
                message(FATAL_ERROR "libavif: rav1e library includes not found")
            endif()
            message(STATUS "LIBRAV1E_INCLUDE_PATH: ${LIBRAV1E_INCLUDE_PATH}")
            include_directories(${LIBRAV1E_INCLUDE_PATH})
        endif()
        set(AVIF_CODEC_LIBRARIES ${AVIF_CODEC_LIBRARIES} rav1e)
    endif()

    # Unfortunately, rav1e requires a few more libraries
    if(WIN32)
        set(AVIF_PLATFORM_LIBRARIES ${AVIF_PLATFORM_LIBRARIES} ws2_32.lib userenv.lib)
    elseif(UNIX AND NOT APPLE)
        set(AVIF_PLATFORM_LIBRARIES ${AVIF_PLATFORM_LIBRARIES} dl) # for backtrace
    endif()
endif()

if(AVIF_CODEC_AOM)
    message(STATUS "libavif: Codec enabled: aom (encode/decode)")
    add_definitions(-DAVIF_CODEC_AOM=1)
    set(AVIF_SRCS ${AVIF_SRCS}
        src/codec_aom.c
    )
    if(AVIF_LOCAL_AOM)
        include_directories(
            "${CMAKE_CURRENT_SOURCE_DIR}/ext/aom"
            "${AOM_BINARY_DIR}"
        )
        message(STATUS "LIBAOM_INCLUDE_PATH: ${CMAKE_CURRENT_SOURCE_DIR}/ext/aom")
    else()
        # Check to see if aom is independently being built by the outer CMake project
        if(NOT TARGET aom)
            find_library(LIBAOM_LIBRARY_PATH NAMES aom)
            if(NOT LIBAOM_LIBRARY_PATH)
              message(FATAL_ERROR "libavif: aom library not found")
            endif()
            find_path (LIBAOM_INCLUDE_PATH aom/aom.h)
            if(NOT LIBAOM_INCLUDE_PATH)
              message(FATAL_ERROR "libavif: aom library includes not found")
            endif()
            message(STATUS "LIBAOM_INCLUDE_PATH: ${LIBAOM_INCLUDE_PATH}")
            include_directories(${LIBAOM_INCLUDE_PATH})
        endif()
    endif()
    set(AVIF_CODEC_LIBRARIES ${AVIF_CODEC_LIBRARIES} aom)
endif()

if(NOT AVIF_CODEC_AOM AND NOT AVIF_CODEC_DAV1D)
    message(FATAL_ERROR "libavif: No decoding library is enabled, bailing out.")
endif()

add_library(avif ${AVIF_SRCS})
target_link_libraries(avif ${AVIF_CODEC_LIBRARIES} ${AVIF_PLATFORM_LIBRARIES})
target_include_directories(avif SYSTEM INTERFACE include)

option(AVIF_BUILD_EXAMPLES "Build avif Examples." OFF)
if(AVIF_BUILD_EXAMPLES)
    add_executable(avif_example1 examples/avif_example1.c)
    target_link_libraries(avif_example1 avif ${AVIF_PLATFORM_LIBRARIES})
endif()

if(NOT SKIP_INSTALL_ALL)
    include(GNUInstallDirs)
endif()

option(AVIF_BUILD_APPS "Build avif apps." OFF)
if(AVIF_BUILD_APPS)
    include_directories(apps/shared)
    add_executable(avifenc
        apps/avifenc.c
        apps/shared/y4m.c
        apps/shared/avifutil.c
    )
    target_link_libraries(avifenc avif ${AVIF_PLATFORM_LIBRARIES})
    add_executable(avifdec
        apps/avifdec.c
        apps/shared/y4m.c
        apps/shared/avifutil.c
    )
    target_link_libraries(avifdec avif ${AVIF_PLATFORM_LIBRARIES})

    if(NOT SKIP_INSTALL_APPS AND NOT SKIP_INSTALL_ALL)
        install(TARGETS avifenc avifdec
            RUNTIME DESTINATION "${CMAKE_INSTALL_FULL_BINDIR}"
            ARCHIVE DESTINATION "${CMAKE_INSTALL_FULL_LIBDIR}"
            LIBRARY DESTINATION "${CMAKE_INSTALL_FULL_LIBDIR}"
        )
    endif()
endif()

option(AVIF_BUILD_TESTS "Build avif tests (fuzzing)." OFF)
if(AVIF_BUILD_TESTS)
    add_executable(aviffuzz
        tests/aviffuzz.c
    )
    target_link_libraries(aviffuzz avif ${AVIF_PLATFORM_LIBRARIES})
endif()

if(NOT SKIP_INSTALL_LIBRARIES AND NOT SKIP_INSTALL_ALL)
    install(TARGETS avif
        ARCHIVE DESTINATION "${CMAKE_INSTALL_FULL_LIBDIR}"
        LIBRARY DESTINATION "${CMAKE_INSTALL_FULL_LIBDIR}"
    )
endif()
if(NOT SKIP_INSTALL_HEADERS AND NOT SKIP_INSTALL_ALL)
    install(FILES include/avif/avif.h include/avif/internal.h
        DESTINATION "${CMAKE_INSTALL_FULL_INCLUDEDIR}/avif"
    )
endif()

macro(avif_set_folder_safe target folder)
    if(TARGET ${target})
        set_target_properties(${target} PROPERTIES FOLDER ${folder})
    endif()
endmacro()

if(WIN32)
    option(AVIF_BUILD_STATIC "Build static avif library" OFF)
    if(AVIF_BUILD_STATIC)
        add_custom_command(
            OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_CFG_INTDIR}/avif_static_x64.lib
            COMMAND link.exe /lib /nologo /MACHINE:x64 /OUT:${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_CFG_INTDIR}/avif_static_x64.lib $<TARGET_FILE:avif> $<TARGET_FILE:aom>
            DEPENDS avif
            COMMENT "Creating static avif library..."
        )

        add_custom_target(avif_static ALL
            DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_CFG_INTDIR}/avif_static_x64.lib
        )
    endif()

    set_property(GLOBAL PROPERTY USE_FOLDERS ON)

    avif_set_folder_safe(aom "ext/avif")
    avif_set_folder_safe(aom_av1 "ext/avif")
    avif_set_folder_safe(aom_av1_common "ext/avif")
    avif_set_folder_safe(aom_av1_common_avx2_intrinsics "ext/avif")
    avif_set_folder_safe(aom_av1_common_sse2_intrinsics "ext/avif")
    avif_set_folder_safe(aom_av1_common_sse4_intrinsics "ext/avif")
    avif_set_folder_safe(aom_av1_common_ssse3_intrinsics "ext/avif")
    avif_set_folder_safe(aom_av1_decoder "ext/avif")
    avif_set_folder_safe(aom_av1_encoder "ext/avif")
    avif_set_folder_safe(aom_av1_encoder_avx2_intrinsics "ext/avif")
    avif_set_folder_safe(aom_av1_encoder_sse2 "ext/avif")
    avif_set_folder_safe(aom_av1_encoder_sse2_intrinsics "ext/avif")
    avif_set_folder_safe(aom_av1_encoder_sse3_intrinsics "ext/avif")
    avif_set_folder_safe(aom_av1_encoder_sse42_intrinsics "ext/avif")
    avif_set_folder_safe(aom_av1_encoder_sse4_intrinsics "ext/avif")
    avif_set_folder_safe(aom_av1_encoder_ssse3 "ext/avif")
    avif_set_folder_safe(aom_av1_encoder_ssse3_intrinsics "ext/avif")
    avif_set_folder_safe(aom_dsp "ext/avif")
    avif_set_folder_safe(aom_dsp_common "ext/avif")
    avif_set_folder_safe(aom_dsp_common_avx2_intrinsics "ext/avif")
    avif_set_folder_safe(aom_dsp_common_sse2 "ext/avif")
    avif_set_folder_safe(aom_dsp_common_sse2_intrinsics "ext/avif")
    avif_set_folder_safe(aom_dsp_common_sse4_1_intrinsics "ext/avif")
    avif_set_folder_safe(aom_dsp_common_ssse3 "ext/avif")
    avif_set_folder_safe(aom_dsp_common_ssse3_intrinsics "ext/avif")
    avif_set_folder_safe(aom_dsp_decoder "ext/avif")
    avif_set_folder_safe(aom_dsp_encoder "ext/avif")
    avif_set_folder_safe(aom_dsp_encoder_avx "ext/avif")
    avif_set_folder_safe(aom_dsp_encoder_avx2_intrinsics "ext/avif")
    avif_set_folder_safe(aom_dsp_encoder_sse2 "ext/avif")
    avif_set_folder_safe(aom_dsp_encoder_sse2_intrinsics "ext/avif")
    avif_set_folder_safe(aom_dsp_encoder_sse4_1_intrinsics "ext/avif")
    avif_set_folder_safe(aom_dsp_encoder_ssse3 "ext/avif")
    avif_set_folder_safe(aom_dsp_encoder_ssse3_intrinsics "ext/avif")
    avif_set_folder_safe(aom_mem "ext/avif")
    avif_set_folder_safe(aom_ports "ext/avif")
    avif_set_folder_safe(aom_rtcd "ext/avif")
    avif_set_folder_safe(aom_scale "ext/avif")
    avif_set_folder_safe(aom_util "ext/avif")
    avif_set_folder_safe(aom_version "ext/avif")
    avif_set_folder_safe(aom_version_check "ext/avif")
    avif_set_folder_safe(avif "ext/avif")
    avif_set_folder_safe(dist "ext/avif")
endif()

# Copyright 2019 Joe Drago. All rights reserved.
# SPDX-License-Identifier: BSD-2-Clause

cmake_minimum_required(VERSION 3.5)
project(avif)

if(CMAKE_CXX_COMPILER_ID MATCHES "GNU")
  add_definitions(-std=c99) # Enforce C99 for gcc
endif()

add_subdirectory(ext)

include_directories(
    include
    "${CMAKE_CURRENT_SOURCE_DIR}/ext/aom"
    "${AOM_BINARY_DIR}"
)
add_library(avif STATIC
    include/avif/avif.h
    include/avif/internal.h
    src/avif.c
    src/mem.c
    src/rawdata.c
    src/read.c
    src/reformat.c
    src/stream.c
    src/utils.c
    src/write.c
)
target_link_libraries(avif aom)

option(AVIF_BUILD_EXAMPLES "Build avif Examples." OFF)

if(AVIF_BUILD_EXAMPLES)
    set_property(GLOBAL PROPERTY USE_FOLDERS ON)

    add_executable(avif_example1 examples/avif_example1.c)
    target_link_libraries(avif_example1 avif)
endif()

macro(avif_set_folder_safe target folder)
    if(TARGET ${target})
        set_target_properties(${target} PROPERTIES FOLDER ${folder})
    endif()
endmacro()

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
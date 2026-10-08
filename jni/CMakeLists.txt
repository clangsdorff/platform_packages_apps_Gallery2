cmake_minimum_required(VERSION 3.22.1)
project(gallery2_jni C CXX)

set(JPEG_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../third_party/libjpeg-turbo)
set(JPEGSTREAM_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../jni_jpegstream/src)
set(JNI_WARNINGS -Wall -Wextra -Werror -Wno-error=constant-conversion -Wno-unused-parameter -Wno-vla-cxx-extension)

add_library(libjni_eglfence SHARED jni_egl_fence.cpp)
set_target_properties(libjni_eglfence PROPERTIES OUTPUT_NAME jni_eglfence)
target_compile_options(libjni_eglfence PRIVATE ${JNI_WARNINGS} -DEGL_EGLEXT_PROTOTYPES)
target_link_libraries(libjni_eglfence log EGL)

file(GLOB FILTER_SRCS filters/*.c filters/*.cc)
add_library(libjni_filtershow_filters SHARED ${FILTER_SRCS})
set_target_properties(libjni_filtershow_filters PROPERTIES OUTPUT_NAME jni_filtershow_filters)
target_compile_options(libjni_filtershow_filters PRIVATE ${JNI_WARNINGS} -ffast-math -O3 -funroll-loops)
target_link_libraries(libjni_filtershow_filters log jnigraphics)

file(GLOB JPEG_SRCS ${JPEG_DIR}/*.c)
list(REMOVE_ITEM JPEG_SRCS
    ${JPEG_DIR}/jccolext.c ${JPEG_DIR}/jdcol565.c ${JPEG_DIR}/jdcolext.c
    ${JPEG_DIR}/jdmrg565.c ${JPEG_DIR}/jdmrgext.c ${JPEG_DIR}/jstdhuff.c)
file(GLOB JPEG_SIMD_SRCS ${JPEG_DIR}/simd/arm/*.c ${JPEG_DIR}/simd/arm/aarch64/*.c)
list(REMOVE_ITEM JPEG_SIMD_SRCS
    ${JPEG_DIR}/simd/arm/jcgryext-neon.c ${JPEG_DIR}/simd/arm/jdcolext-neon.c
    ${JPEG_DIR}/simd/arm/jdmrgext-neon.c ${JPEG_DIR}/simd/arm/aarch64/jccolext-neon.c)
add_library(jpeg_static STATIC ${JPEG_SRCS} ${JPEG_SIMD_SRCS})
target_include_directories(jpeg_static PUBLIC ${JPEG_DIR} PRIVATE ${JPEG_DIR}/simd/arm)
target_compile_options(jpeg_static PRIVATE -DWITH_SIMD -DNEON_INTRINSICS -DNO_GETENV -O3
    -fstrict-aliasing -Wno-sign-compare -Wno-unused-parameter -Wno-vla-cxx-extension)

file(GLOB JPEGSTREAM_SRCS ${JPEGSTREAM_DIR}/*.cpp)
add_library(libjni_jpegstream SHARED ${JPEGSTREAM_SRCS})
set_target_properties(libjni_jpegstream PROPERTIES OUTPUT_NAME jni_jpegstream)
target_include_directories(libjni_jpegstream PRIVATE ${JPEGSTREAM_DIR})
target_compile_options(libjni_jpegstream PRIVATE -ffast-math -O3 -funroll-loops -Wall -Wextra -Werror -Wno-vla-cxx-extension)
target_link_libraries(libjni_jpegstream jpeg_static log)

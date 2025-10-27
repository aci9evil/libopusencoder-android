# For more information about using CMake with Android Studio, read the
# documentation: https://d.android.com/studio/projects/add-native-code.html

# Sets the minimum version of CMake required to build the native library.
cmake_minimum_required(VERSION 3.4.1)
set(CMAKE_CXX_STANDARD 14)
project(OpusEncoder CXX)

# === 16 KB PAGE ALIGNMENT FOR ANDROID 15+ ===
# Required for compatibility with 16 KB page size devices (Android 15+)
if (CMAKE_SYSTEM_NAME STREQUAL "Android")
    if (CMAKE_ANDROID_ARCH_ABI STREQUAL "arm64-v8a" OR CMAKE_ANDROID_ARCH_ABI STREQUAL "x86_64")
        # NDK r26 and lower require both flags
        set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,-z,max-page-size=16384")
        set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,-z,common-page-size=16384")
        message(STATUS "Enabled 16 KB page alignment for ${CMAKE_ANDROID_ARCH_ABI}")
    endif()
endif()

set(distribution_OPUS_DIR ${CMAKE_SOURCE_DIR}/opus)

set(LIBOPUS_DIR ${distribution_OPUS_DIR}/libopus)
set(LIBOPUSENC_DIR ${distribution_OPUS_DIR}/libopusenc)

if (USE_PREBUILTS)
    message("Using prebuilt libraries instead of building from sources.")
    add_library(opus SHARED IMPORTED)
    set_target_properties(opus PROPERTIES IMPORTED_LOCATION
            ${distribution_OPUS_DIR}/prebuilt_libs/${ANDROID_ABI}/libopus.so)

    add_library(opusenc SHARED IMPORTED)
    set_target_properties(opusenc PROPERTIES IMPORTED_LOCATION
        ${distribution_OPUS_DIR}/prebuilt_libs/${ANDROID_ABI}/libopusenc.so)
else()
    # Builds libopus and libopusenc libraries as dependencies.
    add_subdirectory(${LIBOPUS_DIR})
    add_subdirectory(${LIBOPUSENC_DIR})
endif()

# Creates the library as a shared library, adds the needed sources.
add_library(opuscodec
        SHARED
        codec/CodecOggOpus.cpp
        opuscodec.cpp)

target_include_directories(opuscodec PRIVATE
        ${LIBOPUS_DIR}/include
        ${LIBOPUSENC_DIR}/include)

find_library(log-lib log)

target_link_libraries(opuscodec
        android
        opusenc
        opus
        ${log-lib})

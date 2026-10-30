#---------------------------------------------------------------------------------------------
#  iModel Native CMake Build System (HarmonyOS Port)
#  Copyright (c) Bentley Systems, Incorporated. All rights reserved.
#  See LICENSE.md in the repository root for full copyright notice.
#
#  这是为鸿蒙移植重写的 CMake 构建系统，替代原有的 BentleyBuild (bb) 构建系统。
#  构建策略：自底向上，逐步添加模块，先 Windows 验证，再鸿蒙移植。
#---------------------------------------------------------------------------------------------

cmake_minimum_required(VERSION 3.20)

# 项目名称和版本
project(imodel_native
    VERSION 1.0.0
    DESCRIPTION "iModel Native C++ Engine - HarmonyOS Port"
    LANGUAGES CXX C
)

# C++ 标准
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS ON)

# 构建类型默认 Release
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
endif()

message(STATUS "========================================================")
message(STATUS "  iModel Native - HarmonyOS Port Build")
message(STATUS "  Build type: ${CMAKE_BUILD_TYPE}")
message(STATUS "  System: ${CMAKE_SYSTEM_NAME}")
message(STATUS "  Processor: ${CMAKE_SYSTEM_PROCESSOR}")
message(STATUS "  C++ Compiler: ${CMAKE_CXX_COMPILER_ID} ${CMAKE_CXX_COMPILER_VERSION}")
message(STATUS "========================================================")

# ============================================================
#  平台检测
# ============================================================
if(CMAKE_SYSTEM_NAME STREQUAL "HarmonyOS")
    set(BENTLEY_PLATFORM_HARMONYOS TRUE)
    message(STATUS "Platform: HarmonyOS")
elseif(WIN32)
    set(BENTLEY_PLATFORM_WINDOWS TRUE)
    message(STATUS "Platform: Windows")
elseif(ANDROID)
    set(BENTLEY_PLATFORM_ANDROID TRUE)
    message(STATUS "Platform: Android")
elseif(APPLE)
    if(IOS)
        set(BENTLEY_PLATFORM_IOS TRUE)
        message(STATUS "Platform: iOS")
    else()
        set(BENTLEY_PLATFORM_MACOS TRUE)
        message(STATUS "Platform: macOS")
    endif()
elseif(LINUX)
    set(BENTLEY_PLATFORM_LINUX TRUE)
    message(STATUS "Platform: Linux")
endif()

# ============================================================
#  编译选项
# ============================================================

# 公共编译选项
add_compile_options(
    $<$<CXX_COMPILER_ID:MSVC>:/W3 /MP>
    $<$<CXX_COMPILER_ID:GNU,Clang,AppleClang>:-Wall -Wno-unknown-pragmas>
)

# Debug 定义
add_compile_definitions(
    $<$<CONFIG:Debug>:DEBUG>
    __BENTLEYDLL_BUILD__
    BC__RWSTD_LIB_SRC
)

# Windows 特定
if(BENTLEY_PLATFORM_WINDOWS)
    add_compile_definitions(
        _CRT_SECURE_NO_WARNINGS
        WIN32_LEAN_AND_MEAN
        NOMINMAX
    )
endif()

# ============================================================
#  输出目录
# ============================================================
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# ============================================================
#  第三方库根目录
# ============================================================
set(THIRD_PARTY_DIR ${CMAKE_CURRENT_SOURCE_DIR}/iModelCore/libsrc)

# ============================================================
#  子模块（按依赖顺序添加）
# ============================================================

# --- 第 1 层：基础工具库（无内部依赖） ---
add_subdirectory(iModelCore/Bentley)

# --- 第 2 层：依赖 Bentley 的模块 ---
# TODO: add_subdirectory(iModelCore/BeSQLite)
# TODO: add_subdirectory(iModelCore/ecobjects)

# --- 第 3 层：依赖 ECDb / ECObjects 的模块 ---
# TODO: add_subdirectory(iModelCore/ECDb)
# TODO: add_subdirectory(iModelCore/GeomLibs)
# TODO: add_subdirectory(iModelCore/Units)
# TODO: add_subdirectory(iModelCore/iModelPlatform)
# TODO: add_subdirectory(iModelCore/ECPresentation)
# TODO: add_subdirectory(iModelCore/GeoCoord)

# ============================================================
#  状态总结
# ============================================================
message(STATUS "")
message(STATUS "========================================================")
message(STATUS "  Modules included in this build:")
message(STATUS "    [x] Bentley      - 基础工具库")
message(STATUS "    [ ] BeSQLite     - (待添加)")
message(STATUS "    [ ] ECObjects    - (待添加)")
message(STATUS "    [ ] ECDb         - (待添加)")
message(STATUS "    [ ] ...          - more to come")
message(STATUS "========================================================")

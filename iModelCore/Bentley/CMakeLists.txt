#---------------------------------------------------------------------------------------------
#  Bentley 基础工具库 - CMake 构建文件
#  对应原 BentleyBuild 的 Bentley.mke / Bentley.PartFile.xml
#---------------------------------------------------------------------------------------------

# 平台无关的源文件（Bentley/ 目录）
set(BENTLEY_COMMON_SOURCES
    Bentley/WString.cpp
    Bentley/DateTime.cpp
    Bentley/DateTimeConverter.cpp
    Bentley/GlobalHandleContainer.cpp
    Bentley/RefCounted.cpp
    Bentley/Logging.cpp
    Bentley/md5.cpp
    Bentley/sha1.cpp
)

# 平台相关的源文件（nonport/ 目录）- 所有平台通用
set(BENTLEY_NONPORT_COMMON_SOURCES
    Bentley/nonport/Base64Utilities.cpp
    Bentley/nonport/BeDebugLog.cpp
    Bentley/nonport/BeDebugUtilities.cpp
    Bentley/nonport/BeDirectoryIterator.cpp
    Bentley/nonport/BeFile.cpp
    Bentley/nonport/BeFileListIterator.cpp
    Bentley/nonport/BeFileName.cpp
    Bentley/nonport/BeGetProcAddress.cpp
    Bentley/nonport/BeStringUtilities.cpp
    Bentley/nonport/BeSystemInfo.cpp
    Bentley/nonport/BeTest.cpp
    Bentley/nonport/BeTextFile.cpp
    Bentley/nonport/BeThread.cpp
    Bentley/nonport/BeTimeUtilities.cpp
    Bentley/nonport/ValueFormat.cpp
    Bentley/nonport/strfunc.cpp
)

# Android 特定源文件
if(BENTLEY_PLATFORM_ANDROID)
    list(APPEND BENTLEY_PLATFORM_SOURCES
        Bentley/nonport/BeJStringUtilities.cpp
    )
endif()

# iOS 特定源文件 (Objective-C++)
if(BENTLEY_PLATFORM_IOS)
    list(APPEND BENTLEY_PLATFORM_SOURCES
        Bentley/nonport/BeSystemInfoIos.mm
    )
endif()

# macOS 特定源文件 (Objective-C++)
if(BENTLEY_PLATFORM_MACOS)
    list(APPEND BENTLEY_PLATFORM_SOURCES
        Bentley/nonport/BeSystemInfoMacOS.mm
    )
endif()

# 鸿蒙特定源文件（待添加，先用 Linux 风格 stub）
if(BENTLEY_PLATFORM_HARMONYOS)
    # TODO: 添加鸿蒙平台特定实现
    # 目前先用 POSIX 风格的实现（从 Android/Linux 版本适配）
    message(STATUS "[Bentley] HarmonyOS platform - using POSIX-compatible implementations")
endif()

# 汇总所有源文件
set(BENTLEY_ALL_SOURCES
    ${BENTLEY_COMMON_SOURCES}
    ${BENTLEY_NONPORT_COMMON_SOURCES}
    ${BENTLEY_PLATFORM_SOURCES}
)

# 创建静态库
add_library(Bentley STATIC ${BENTLEY_ALL_SOURCES})

# 头文件搜索路径
target_include_directories(Bentley
    PUBLIC
        # 公开头文件 - 其他模块通过 #include <Bentley/WString.h> 引用
        ${CMAKE_CURRENT_SOURCE_DIR}/PublicAPI
    PRIVATE
        # 内部头文件
        ${CMAKE_CURRENT_SOURCE_DIR}/Bentley
)

# 编译定义
target_compile_definitions(Bentley
    PRIVATE
        __BENTLEYDLL_BUILD__
        BC__RWSTD_LIB_SRC
        CREATE_STATIC_LIBRARIES
    PUBLIC
        # Windows 平台必须定义 BENTLEY_WIN32（Bentley.h 中有检查）
        $<$<BOOL:${BENTLEY_PLATFORM_WINDOWS}>:BENTLEY_WIN32>
        # 鸿蒙平台宏（用于 BentleyConfig.h 中的条件编译）
        $<$<BOOL:${BENTLEY_PLATFORM_HARMONYOS}>:BENTLEY_HARMONYOS>
        $<$<BOOL:${BENTLEY_PLATFORM_HARMONYOS}>:BENTLEYCONFIG_OS_HARMONYOS>
)

# ============================================================
#  第三方依赖
# ============================================================

# TODO: ICU4C - 暂时注释掉，等后面需要时再加
# （初期可以先不加 ICU，看看编译是否通过，缺啥再补）
# target_include_directories(Bentley PRIVATE ${THIRD_PARTY_DIR}/icu4c/...)
# target_link_libraries(Bentley PRIVATE icu4c)

# Windows 特定库
if(BENTLEY_PLATFORM_WINDOWS)
    target_link_libraries(Bentley PRIVATE dbghelp)
endif()

# POSIX 线程库
if(BENTLEY_PLATFORM_LINUX OR BENTLEY_PLATFORM_ANDROID OR BENTLEY_PLATFORM_HARMONYOS)
    find_package(Threads REQUIRED)
    target_link_libraries(Bentley PRIVATE Threads::Threads)
endif()

# ============================================================
#  输出信息
# ============================================================
message(STATUS "  [Bentley] 静态库目标已创建 (源文件: ${BENTLEY_ALL_SOURCES})")
message(STATUS "  [Bentley] 公开头文件: ${CMAKE_CURRENT_SOURCE_DIR}/PublicAPI")

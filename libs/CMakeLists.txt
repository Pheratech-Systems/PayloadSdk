cmake_minimum_required(VERSION 3.10)

project (PayloadSDK)

file(GLOB_RECURSE SOURCES 
            ${CMAKE_CURRENT_SOURCE_DIR}/third-party/mavlink/src/*.cpp
            ${CMAKE_CURRENT_SOURCE_DIR}/payloadSdkInterface.cpp 
            )

# PayloadSDK library build
add_library(${PROJECT_NAME} STATIC ${SOURCES}) # Static by default

# Nexus uses -Werror globally; vendor .cpp would fail the build. Silence diagnostics
# only for this static lib (does not affect nexus / vio_interface).
if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    target_compile_options(${PROJECT_NAME} PRIVATE -w)
elseif(MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE /W0)
endif()

target_link_libraries(${PROJECT_NAME})

# Mark PayloadSDK / MAVLink / GStreamer includes as SYSTEM so GCC/Clang do not
# diagnose third-party headers (packed MAVLink structs, vendor macros, etc.).
# INTERFACE propagates to vio_interface / nexus when linked as PUBLIC.
target_include_directories(${PROJECT_NAME} SYSTEM PUBLIC
            ${CMAKE_CURRENT_SOURCE_DIR}
            ${CMAKE_CURRENT_SOURCE_DIR}/third-party/mavlink/include
            ${CMAKE_CURRENT_SOURCE_DIR}/payload-define
            ${CMAKE_CURRENT_SOURCE_DIR}/third-party/mavlink/include/mavlink/v2.0
            ${CF_HEADER_DIRS}
        )

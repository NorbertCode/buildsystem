cmake_minimum_required(VERSION 3.25)
project(buildsystem)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

add_library(warnings INTERFACE)
if (MSVC)
    target_compile_options(warnings INTERFACE /W4 /WX)
else ()
    target_compile_options(warnings INTERFACE -Wall -Wextra -Wpedantic -Werror)
endif ()

add_library(sanitizers INTERFACE)
if (NOT MSVC AND CMAKE_BUILD_TYPE STREQUAL "Debug")
    target_compile_options(sanitizers INTERFACE -fsanitize=address,undefined -fno-omit-frame-pointer)
    target_link_options(sanitizers INTERFACE -fsanitize=address,undefined)
endif ()

add_subdirectory(src)
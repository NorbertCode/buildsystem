include(FetchContent)
FetchContent_Declare(
    googletest
    URL https://github.com/google/googletest/archive/03597a01ee50ed33e9dfd640b249b4be3799d395.zip
    SYSTEM
)

# For Windows: Prevent overriding the parent project's compiler/linker settings
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
set(INSTALL_GTEST OFF CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(googletest)

set_target_properties(gtest gtest_main gmock gmock_main PROPERTIES EXPORT_COMPILE_COMMANDS OFF)

add_executable(
    run_tests
    test.cpp
)

target_link_libraries(
    run_tests PRIVATE
    gtest
    gtest_main
    warnings
    sanitizers
)

include(GoogleTest)
gtest_discover_tests(run_tests)
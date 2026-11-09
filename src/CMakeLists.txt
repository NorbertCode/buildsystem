add_executable(
    buildsystem
    main.cpp
)

target_link_libraries(buildsystem PRIVATE warnings sanitizers)
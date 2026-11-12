# Table of Contents

1. [Overview](#overview)
2. [Configuration Files](#configuration-files)

# Overview

The purpose of the project is to create a tool for building C++ projects. It acts similarly to tools like `CMake`, `Make` or `Cargo`, allowing the user to compile their projects using a single command.

This will be achieved by going through multiple steps. First, the program will read the user's configuration files. Based on them, it will create a directed acyclic graph (DAG) of targets and their dependencies. The graph will then be used to decide compilation order and incrementally (only compiling targets which have been modified) build the user's software. 

Our architecture will allow for easily adding support for multiple compilers (by using polymorphism), but we will focus only on the `gcc` compiler. This means that on Windows the tool will require `MinGW` to be installed. 

# Configuration Files and Launching

Configuration files will provide the following functionality:

- Defining projects - The user will define the project's name and C++ standard (we do not plan on supporting building library files, only executables)
- Defining the project's structure - The user will define targets and their properties: source files, include directories and dependencies (other targets like user-defined libraries, or system libraries - we do not plan supporting static or dynamic libraries)
- Setting compilation flags - The user will be able to define compilation flags for targets

Configuration files will be written in Python and use bindings (`pybind11`) to call C++ code. We also considered using `toml`, however we decided that Python bindings will give the user more flexibility. This will require creating an interface which Python will be able to call.

An example configuration file could look like this:

```python
import buildsystem as bs

bs.create_project(name="foo", standard=20)

bs.create_library(
    name="mathlib",
    sources=[
        "src/mathlib/mathlib.cpp",
        "src/mathlib/matrix.cpp"
    ],
    includes=[
        "src/mathlib"
    ]
)

bs.create_executable(
    name="bar",
    sources=["main.cpp"],
    dependencies=["mathlib"],
    system_libraries=["ncurses"], # Because ncurses can be installed as a system library, adding it boils down to adding a -lncurses flag
    flags=[ "Werror", "Wall" ]
)

bs.build()
```

The user would then call the script `py build.py` and our build system would generate the executable.
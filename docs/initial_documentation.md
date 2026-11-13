# Table of Contents

1. [Overview](#overview)
2. [Configuration Files](#configuration-files)
3. [Dependency Graph](#building-dependency-graph)
4. [Compilation Commands](#generating-compilation-commands)

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

# Building Dependency Graph

Upon receiving configuration files, the program will start to build a dependency graph from listed files. The process of building such graph will feature:

- Finding listed files in directories given in configuration
- Creating list of dependencies
- Created list shall be sorted via topological sorting (e.g Kahn's algorithm) - it will order all files accordingly for compilation.
- Using topological sorting will provide assuring that created graph is of DAG class - it will detect any cycles.

It was debated what type of topological sorting algorithm will be used in our project. There were considered Kahn's algorithm, Coffmann-Graham algorithm and mixture of them. The Kahn's algorithm is one of the simplest of them, but very effective and allows for simultaneous compilation of non-dependent files. Coffmann-Graham is as efective as Kahn's, with added bonus of ordering the files in such a way, that it will allow for the most efficient job distribution. However, there is a caveat that the algorithm treats every file compilation as if they take the same amount of time to process.

The best solution and one that is used by modern build systems, such as `CMake`, `Cargo` or `Ninja` is a mixture of their best features. However, for our project we will use Kahn's algorithm, as it is the simplest, yet effective way of building the dependency graph.

Resulting dependency graph will be a primary input for the next stage of building compilation commands list.

# Generating Compilation Commands

With the dependency graph built, the next step is to create a list of commands to execute, suited for a compiler given in the configuration. We debated two possible ways of dealing with it:

- Use of the `Make` software and its template.
- Executing commands live via something like `std::system()`.

Both solutions have their reasons to be chosen. With `Make`, we can operate on a solid and documented infrastructure that handles detecting changes in files, allows for creating specific builds of the application (e.g. deployment or tests), handles simultaneous compilation and more in one, organized, single file. With live compilation, the main caveat is the need of implementation of some basic features that `Make` implements, such as out-of-date checking and simultaneous execution. However, we will gain freedom of implementation of these features. 

In the end, we decided to use `Make` for its defined structure and convenient updated files checking. However, regardless of chosen method we will have same features to implement. `Make` makes it a little easier, especially in organisation department, but not that much than generating raw scripts.

After the `makefile` is prepared, the program will automatically use `Make` via `std::system()` to start compiling the project and return created binary files for the project.
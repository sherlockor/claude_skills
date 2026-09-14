---
name: cpp-embedded-develop
description: Develop or modify embedded C++ modules for ARM Linux robotics projects that use ROS2 Humble as a dependency and generic CMake instead of a ROS2 workspace. Use for C++ code generation, CMake integration, ARM/aarch64 cross-compiling, sysroot/toolchain work, rclcpp nodes, lifecycle nodes, launch-related integration, and resource-constrained no-exceptions runtime code.
---

# cpp-embedded-develop

## Core Workflow

1. Inspect the repository shape before editing: top-level `CMakeLists.txt`, `modules/`, `lib/3rd/<platform>/include`, `lib/3rd/<platform>/lib`, and `lib/product/<platform>/lib`.
2. Keep changes inside the requested module boundary unless the existing dependency direction requires a shared interface change.
3. Use generic CMake patterns, not `ament` or `colcon`, unless the repository already requires them.
4. Prefer repository-provided libraries under `lib/3rd` and existing wrappers before adding direct OS or STL-heavy logic.
5. Preserve no-exceptions behavior on runtime paths: return error codes, `std::optional`, or the project `Status`/`Result` type.

## Build And CMake

- Treat ROS2 Humble as an installed dependency. Prefer explicit include/library paths and `target_link_libraries()` consistent with the local build instead of introducing `find_package(rclcpp REQUIRED)` when the project avoids it.
- Use `make <module>` for the ARM build convention, and use `./local_build.sh <module>` for the x86 build convention. Compile only for the ARM platform unless otherwise specified.
- Install targets to the existing `lib/product/${BUILD_PLATFORM}` layout and copy scripts/configs using the repository's current `install()` or `file(COPY ...)` style.
- Keep `target_include_directories`, `target_compile_definitions`, and link dependencies target-scoped where the existing CMake permits it.

## C++ Rules

- Follow the repository's C++ standard and local naming. For new code, use include guards, concise public interfaces, and RAII for ownership.
- Keep constructors lightweight. Move blocking work, IO, thread startup, and long initialization into explicit `start()` or initialization methods.
- Avoid dynamic allocation on periodic callbacks or control paths. Reserve containers or reuse message objects when the code already has a safe lifetime model.
- Do not create ROS publishers or subscribers as local temporaries. Store them as members so their lifetime covers the node lifetime.
- Keep callback work bounded. Use queues or cached state when IO, allocation, or cross-thread work would make callback timing unpredictable.

## Output Style

- Respond in Chinese by default, using standard C++/ROS2/CMake terminology where useful.
- Keep explanations short. Report the files changed, the important design decision, and the verification result.

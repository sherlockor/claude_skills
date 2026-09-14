---
name: data-develop
description: Develop or modify the `modules/data` embedded C++ module in the ARM Linux robotics project. Use for data-layer ROS2 Humble integration, serial protocol handling, CMake changes, publisher/subscriber lifetime fixes, enum-based protocol logic, and module-local code generation that must follow the existing data module style.
---

# data-develop

## Core Workflow

1. Work primarily under `modules/data/`; inspect nearby files before adding new classes or directories.
2. Check shared dependencies under `lib/3rd/<platform>` and generated/product libraries under `lib/product/<platform>` before adding new dependencies.
3. Refer to `modules/data/docs/串口通信协议.xlsx` when the task involves serial protocol fields, commands, or frame semantics.
4. Keep ROS2 Humble as a dependency of the generic CMake build; do not convert the project to a ROS2 workspace.
5. Verify with the module build convention `make data` when build verification is needed and allowed.

## Module Conventions

- Match the existing `modules/data` file layout, logging macros, message naming, and error handling.
- Use enum values for type or command checks. Avoid comparing protocol or message types to raw numbers when an enum exists or should be introduced.
- Do not use `topic_def.h` macros for new publisher/subscriber topic names. Pass the static topic string directly at creation sites.
- Store ROS publishers and subscribers as members; local publisher/subscriber variables are invalid for persistent communication.
- Keep callback processing small and deterministic. Move parsing, validation, and state transitions into testable helpers when callbacks grow.

## C++ Rules

- Default to no exceptions on runtime paths. Use project status/error types, `enum class`, or `std::optional` for failures.
- Avoid repeated initialization outside the proper startup path. Initialize cached protocol state once, then update it explicitly.
- Prefer existing utility libraries from `lib/3rd` before using lower-level OS APIs.
- Keep new functions short and cohesive; split parsing, validation, and side effects into separate helpers.

## Logging

- Log key input parameters at externally visible API boundaries when they affect behavior.
- For enum logs, print both the numeric value and its meaning, for example `type=2(delete)`.
- Avoid logging high-frequency callback data unless the value changes or the log is needed to diagnose a failure path.

## Output Style

- Respond in Chinese by default.
- Keep the final summary to changed files, important behavior, and build or test result.

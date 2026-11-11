---
name: logic-plugin-develop
description: Develop or modify `modules/logic_plugin` task plugins for the embedded C++ robotics project. Use for MissionTask or MissionRequestTask implementations, task registration, Work Ease context parameter handling, zmq publisher/subscriber integration inside tasks, generic CMake changes, and behavior coordinated with `modules/logic_msg`.
---

# logic-plugin-develop

## Core Workflow

1. Work primarily under `slamMower/src/modules/logic_plugin/`; inspect neighboring tasks before adding or changing a task.
2. Use `slamMower/src/modules/logic_msg/` for shared data types and keep cross-module contracts stable.
3. Register every new task once in `register_task.cpp`.
5. Keep task behavior compatible with Work Ease script execution and context parameter passing.

## Task Model

- Sync tasks inherit from `MissionTask`.
- Sync request tasks inherit from `MissionRequestTask`.
- Sync request task call order is `onStart`/`onStartUp` -> `onDone`.
- `onDone` is always called once at the end of an execution with the task result.
- Initialize per-run task data in `onStartUp`; avoid redundant initialization checks in later callbacks unless state can be changed by another thread.

## Code Rules

- Match the style of existing tasks, including file layout, naming, logging macros, and context access patterns.
- Use enums for task or message type checks instead of raw numbers.
- Keep `onRunning` deterministic and short. Return `running` without logging every cycle unless a state value changes.
- Default to no exceptions on runtime paths; use existing status/error conventions.

## Logging

- Log at the start of triggered task hooks such as pause, resume, cancel, timeout, reactive handling, and done.
- If `onRunning` returns success or failure, log the concise reason. If it returns running, avoid repeated logs.
- For enum logs, include both value and meaning, for example `result=1(success)`.

## Output Style

- Respond in Chinese by default.
- Summarize changed task files, registration changes, context parameters, and verification result.

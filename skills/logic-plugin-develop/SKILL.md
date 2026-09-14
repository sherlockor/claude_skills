---
name: logic-plugin-develop
description: Develop or modify `modules/logic_plugin` task plugins for the embedded C++ robotics project. Use for MissionTask or MissionRequestTask implementations, task registration, Work Ease context parameter handling, ROS2 publisher/subscriber integration inside tasks, generic CMake changes, and behavior coordinated with `modules/logic_msg`.
---

# logic-plugin-develop

## Core Workflow

1. Work primarily under `modules/logic_plugin/`; inspect neighboring tasks before adding or changing a task.
2. Use `modules/logic_msg/` for shared data types and keep cross-module contracts stable.
3. Register every new task once in `register_task.cpp`.
4. Verify with `make logic_plugin` when build verification is needed and allowed.
5. Keep task behavior compatible with Work Ease script execution and context parameter passing.

## Task Model

- Sync tasks inherit from `MissionTask`.
- Sync request tasks inherit from `MissionRequestTask`.
- Tasks in the same script execute on one task thread. ROS callbacks run on separate ROS execution threads, so shared state must be synchronized or updated predictably.
- Sync task call order is `onStart`/`onStartUp` -> repeated `onRunning` -> optional `onPause`/`onResume`/`onCancel`/`onTimeout`/`onReactive` -> `onDone`.
- Sync request task call order is `onStart`/`onStartUp` -> `onDone`.
- `onDone` is always called once at the end of an execution with the task result.
- Initialize per-run task data in `onStartUp`; avoid redundant initialization checks in later callbacks unless state can be changed by another thread.

## Code Rules

- Match the style of existing tasks, including file layout, naming, logging macros, and context access patterns.
- Do not use `topic_def.h` macros for new ROS topic names. Pass the static topic string directly when creating publishers/subscribers.
- Store ROS publishers and subscribers as members. Do not create them as local temporaries.
- Use enums for task or message type checks instead of raw numbers.
- Keep `onRunning` deterministic and short. Return `running` without logging every cycle unless a state value changes.
- Default to no exceptions on runtime paths; use existing status/error conventions.

## Logging

- Log at the start of triggered task hooks such as pause, resume, cancel, timeout, reactive handling, and done.
- If `onRunning` returns success or failure, log the concise reason. If it returns running, avoid repeated logs.
- For enum logs, include both value and meaning, for example `result=1(success)`.
- In ROS callbacks, log only when values change or errors occur.

## Output Style

- Respond in Chinese by default.
- Summarize changed task files, registration changes, context parameters, and verification result.

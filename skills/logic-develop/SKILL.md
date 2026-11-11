---
name: logic-develop
description: Develop or modify the `src/modules/logic` embedded C++ module and related shared logic interfaces. Use for logic scheduling, protocol parsing, state-machine changes, Work Ease script integration, ZMQ communication, generic CMake changes, and code that coordinates `src/modules/logic`, `src/modules/logic_msg`, and `src/modules/logic_plugin`.
---
# logic-develop

## Core Workflow

1. Work primarily under `slamMower/src/modules/logic/`; inspect `slamMower/src/modules/logic_msg/` when shared data types or message contracts are involved.
2. Treat `slamMower/src/modules/logic_plugin/` as task implementation code invoked by logic scripts; do not change plugin task behavior from `logic` unless the request requires cross-module integration.
3. Follow the repository's `slamMower/deps_gcc10.3/`layout.
4. Keep interface changes narrow because `logic_msg` is shared by logic and plugin modules.

## Module Model

- `logic` handles protocol parsing, task scheduling, state transitions, and task orchestration scripts.
- `logic_msg` contains shared data types used by both `logic` and `logic_plugin`.
- `logic_plugin` contains executable tasks; tasks exchange data through context parameters.
- State scheduling uses `tinyfsm`. A state transition runs current-state `exit`, then next-state `entry`; event handling belongs in `react`.
- Work Ease scripts use the `work_ease` library under `slamMower/deps_gcc10.3/aarch64/work_ease/include/`.

## Code Rules

- Match local naming, include guards, logging macros, and existing C++ standard.
- Use enum values for type checks instead of raw numbers.
- Keep callbacks bounded; offload complex scheduling, parsing, or IO to helpers or existing scheduler paths.
- Default to no exceptions on runtime paths; use existing status/error conventions.

## Logging

- Log key external inputs and state transitions, but keep content concise.
- For enum logs, include both the value and meaning, for example `state=3(running)`.
- Do not repeat topic names in every publish log when the log message already identifies the action.

## Output Style

- Respond in Chinese by default.
- Keep explanations practical: what changed, why it matches the logic/module boundary, and whether verification passed.

---
name: map-manager-develop
description: Develop or modify the `modules/map_manager` embedded C++ module. Use for map persistence, multi-map management, boundary/island/channel/area data handling, ROS2 map services or topics, generic CMake changes, and code that mediates map data between logic and algorithm modules.
---

# map-manager-develop

## Core Workflow

1. Work primarily under `modules/map_manager/`; inspect the existing layer where the change belongs before editing.
2. Use shared message/data definitions from `modules/logic_msg/` when map interfaces cross module boundaries.
3. Keep ROS2 Humble as a generic CMake dependency; follow the repository's `lib/3rd` and `lib/product` layout.
4. Use `make map_manager` for module verification when build verification is needed and allowed.
5. Preserve `components/` and `main.cpp` unless the user explicitly asks to change startup or compression component behavior.

## Domain Model

- A map contains one or more boundaries. If the last boundary is deleted, the map is deleted.
- A map can contain zero or more islands and channels.
- User regions or PNC partitions should be named `area` in generated code. Areas belong to boundaries, not directly to maps.
- Area data depends on its boundary. Deleting a boundary must delete its areas and related boundary-scoped island data.
- The module supports at most three maps, with `map_id` in `1..3`.
- Only one map is loaded in memory at a time. Other maps exist only as persisted files.
- Creating or switching maps updates the current `map_id` and persists that value immediately.

## Module Layers

- ROS communication layer: `ROSInterface` creates ROS services/topics and delegates business behavior to services.
- Service layer: `MapService` and similar classes hold business operations and call repositories/utilities for data work.
- Data layer: repository classes own persistence and load map data from files during startup.
- Utilities: `CommonUtils`, `DataConvert`, and BO objects hold non-business helpers, ROS-message/JSON conversion, and module data wrappers.
- Keep headers and sources separated according to the existing module layout.

## ROS Interfaces

- Subscriber QoS for map source topics should be reliable, keep-last depth 2, and transient-local when matching existing behavior.
- Publisher QoS for exposed map topics should be reliable, keep-last depth 2, and transient-local.
- Source map topic: `/slam/mult_map`, message `common::msg::MultMap`.
- Source partition topic: `/pnc/multmapregion`, message `common::msg::MultMapRegion`.
- Source island topic: `/pnc/island_map`, message `common::msg::PncMap`.
- PNC partition command topic for new-map operation: `/logic/department`, message `common::msg::LogicDepartment`, with `department` fixed to `1`.
- PNC parition command topic for user-initiated opertaion: publish data `/logic/mult_user_regions` first, message `PncMultRegionData.msg`. follow by command `/logic/app_partition_set`, message `std_msgs::Bool`, with `data` set to `true`.

- When service logic publishes a command and waits for a ROS callback, use a wait-set or equivalent pattern that does not block the ROS executor thread.

## Exposed Operations

- `/map/get_map`: compress map data and return the saved path.
- `/logic/multi_map`: publish in-memory `common::msg::MultMap` when available.
- `/map/save_map`: persist map and element data using the existing file utility style.
- `/map/pnc_region_pub`: publish in-memory partition data when available.
- `/map/delete_map`: delete map, boundary, island, or channel data according to request type.
- `/map/control_pub_grid_map`: publish selected `MultMap`, `MultMapRegion`, or `PncMap` data.
- `/map/get_current_map`: return current map information.
- `/map/region_operation_save`: for type `3`, trigger new-map PNC partition flow; the logic of type `2` is consistent with type `3`; for type `4`, trigger user-initiated PNC partition; leave other types as existing behavior or TODO if still unimplemented.
- `/map/control_map_path`: control path data, include start record point data; pause record point data; clear recorded point data; save point data to file. Path data consists of a set of points.
- `/map/persist_path`: save recorded point data (path data) to file in accordance with the specified protocol. Save the map data at the same time. Return the storage path and map.
- `/map/get_path`: get recorded point data (path data).

## Code Rules

- Match `modules/map_manager` style and use the existing `utils` library under `lib/3rd/arm/include/utils`.
- Store ROS publishers and subscribers as members. Do not create them as local temporaries.
- Do not use `topic_def.h` macros for new ROS topic names. Pass the static topic string directly at creation sites.
- Use enums for type checks instead of raw numbers.
- Default to no exceptions on runtime paths. Return project status/error values.
- Keep persistence and conversion logic out of ROS callbacks and service entry points when it belongs in repositories or utilities.

## Logging

- Log key service requests and persistence results.
- For enum logs, include both value and meaning, for example `delete_type=2(boundary)`.
- Avoid logging high-frequency map source callbacks unless data changes or an error occurs.

## Output Style

- Respond in Chinese by default.
- Summarize changed layer, ROS interface impact, persistence behavior, and verification result.

## Map Version
- Map elements include: boundary, channel, island and area(also refered to as region or partition);
- Map version: multmap_time field in /slam/mult_map. Need update when map version received.
- Boundary version: generated by map_manager. Set boundary version to current timestamp when receive /slam/mult_map.
- Channel version: generated by map_manager. Set channel version to current timestamp when receive /slam/mult_map.
- Island version: generated by map_manager. Set island version to current timestamp when receive /pnc/island_map.
- Area version: 
  - Only one version is required for partitions within a single boundary.
  - Set area version to 0, when publish /logic/department and receive /pnc/multmapregion.
  - Set area version to request->regionver, when publish /logic/app_partition_set and receive /pnc/multmapregion.
# Work Ease Reference

## Keywords

| Keyword | Purpose |
| --- | --- |
| `var` | Define a task variable bound to a registered C++ task type. |
| `if/else` | Conditional branch. |
| `while` | Loop while a task or group condition succeeds. |
| `repeat` | Repeat a body a fixed number of times, stopping on failure. |
| `retry` | Retry a body after failure, stopping on success. |
| `reactive` | Run a body while continuously checking a condition. |
| `parallel` | Tick child tasks together. |
| `force_pass` | Execute body and always report success. |
| `force_fail` | Execute body and always report failure. |
| `function` | Define reusable script logic. |
| `import` | Load task variables/functions from another script. |
| `print` | Print log text. |
| `sleep` | Sleep. |
| `pause` | Pause. |
| `cancel` | Cancel. |

## Operators

| Operator | Purpose |
| --- | --- |
| `!` | Logical not. |
| `and` | Logical and. |
| `or` | Logical or. |
| `=` | Assignment in `var` declarations. |
| `""` | String literal. |

## Syntax Rules

- Scripts are line-oriented: one statement per line.
- Blocks use `{}` and are required even when the block has one statement.
- Comments begin with `#`.
- Task variables execute by writing the variable name as a statement.
- Sequential execution stops on failure unless control-node semantics override that result.

## Task Declaration

```python
var navigation_task = NavigationTask("navigation_task")
navigation_task
```

`NavigationTask` must be a registered C++ task type. `"navigation_task"` is the instance name passed to the task constructor.

## Branching

```python
var battery_check = BatteryLevelCheck("battery_check")
var navigation_task = NavigationTask("navigation_task")
var docking_task = DockingTask("docking_task")

if(battery_check)
{
    navigation_task
}
else
{
    docking_task
}
```

The condition is a task or task group. Success enters the `if` body; failure enters `else`.

## Loops

```python
while(battery_level_check)
{
    navigation_task
}
```

`while` exits only when the condition finishes with `Failure`. A body failure does not by itself terminate the loop according to the original documentation.

```python
repeat(3)
{
    navigation_task
}
```

`repeat(n)` repeats after success and stops on first failure.

```python
retry(3)
{
    navigation_task
}
```

`retry(n)` retries after failure and stops on first success.

## Reactive

```python
reactive(battery_level_check)
{
    navigation_task
}
```

`reactive` continuously checks the condition while ticking the body:

- If the condition returns `Success`, the body result determines the final result.
- If the condition returns `Failure`, the running child receives `onReactive()`.
- `onReactive()` returning `true` ignores this reactive event and continues ticking.
- `onReactive()` returning `false` halts the child through `onHalt()` and the reactive node reports `Failure`.

Use this for safety or precondition monitoring during long-running robot actions.

## Parallel

```python
parallel
{
    navigation_task
    screen_return_zero_task
}
```

All children tick together. The group succeeds only when every child succeeds. If one or more children fail, the group fails and remaining running children receive `onHalt()`.

## Force Result

```python
force_pass
{
    navigation_task
    docking_task
}
```

Use `force_pass` when the outer flow must continue regardless of inner failures, often for cleanup or recovery. `force_fail` mirrors this but always reports failure.

## Functions

```python
function run()
{
    navigation_task
    docking_task
}

run
```

Rules:

- Define a function before calling it.
- Function execution follows normal sequential script semantics.
- Function body task states are not reset after function completion.

## Import

```python
import test_run_simple_script

test_run_simple_script.run_docker
```

Rules:

- Import paths may be absolute or relative, with or without `.bt`; missing suffix is added by the parser.
- Import loads task instances declared with `var` and functions declared with `function`.
- Import does not load or execute the imported script's top-level logic.
- Access imported instances/functions using the script-name namespace.

## C++ Integration Pattern

```cpp
#include "scripting/script_interpreter.h"
#include "types/reflex.h"

class NavigationTask : public we::task::SyncTask
{
public:
    explicit NavigationTask(const std::string &name)
        : we::task::SyncTask(name)
    {
    }

private:
    bool onStart() override
    {
        EASE_INFO("name: %s, on start", task_name_.c_str());
        start_time_ = we::utils::DateTime::currentSecond();
        return true;
    }

    we::task::TaskStatus onRunning() override
    {
        const int64_t now = we::utils::DateTime::currentSecond();
        if (now - start_time_ <= cost_time_)
        {
            return we::task::Running;
        }
        EASE_INFO("name: %s, done", task_name_.c_str());
        return we::task::Success;
    }

    void onHalt() override
    {
        EASE_INFO("name: %s, on halt", task_name_.c_str());
    }

private:
    int cost_time_{3};  // seconds
    int64_t start_time_{};
};

REGISTER_TASK_TYPE(NavigationTask)
```

Interpret and run:

```cpp
ScriptInterpreter script_run;
if (auto scheduler = script_run.interpret(code))
{
    scheduler.value()->run();
}
else
{
    const ErrorInfo error_info = script_run.getErrorInfo();
    printf("%s\n%d:%d, %s\n",
           error_info.line_code.c_str(),
           error_info.line,
           error_info.column,
           error_info.error_message.c_str());
}
```

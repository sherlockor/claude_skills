# 使用并行执行的编排策略

场景：

1. 控制机器人一边导航一边控制屏幕归位；
2. 同时执行导航任务和屏幕归位任务；

## 1、编写task

**ScreenReturnZeroTask** 屏幕归位任务

```c++
class ScreenReturnZeroTask: public we::task::SyncTask
{
public:
    explicit ScreenReturnZeroTask(const std::string &name)
        : we::task::SyncTask(name)
    {
    }
    ~ScreenReturnZeroTask()
    {
    }

    bool onStart() override
    {
        EASE_INFO("name: %s, on start", task_name_.c_str());
        start_time_ = we::utils::DateTime::currentSecond();
        return true;
    }
    we::task::TaskStatus onRunning() override
    {
        int64_t now = we::utils::DateTime::currentSecond();
        if (now - start_time_ <= cost_time_)
        {
            return we::task::Running;
        }
        EASE_INFO("name: %s, done", task_name_.c_str());
        return we::task::Success;
    }
    bool onPause() override
    {
        EASE_INFO("name: %s, on pause", task_name_.c_str());
        return true;
    }
    bool onResume() override
    {
        EASE_INFO("name: %s, on resume", task_name_.c_str());
        return true;
    }
    void onCancel() override
    {
        EASE_INFO("name: %s, on cancel", task_name_.c_str());
    }

private:
    int cost_time_{1};//s
    int64_t start_time_{};
};
```

**NavigationTask**， 导航任务

```c++
class NavigationTask: public we::task::SyncTask
{
public:
    explicit NavigationTask(const std::string &name)
        : we::task::SyncTask(name)
    {
    }
    ~NavigationTask()
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
        int64_t now = we::utils::DateTime::currentSecond();
        if (now - start_time_ <= cost_time_)
        {
            return we::task::Running;
        }
        EASE_INFO("name: %s, done", task_name_.c_str());
        return we::task::Success;
    }
    bool onPause() override
    {
        EASE_INFO("name: %s, on pause false", task_name_.c_str());
        return true;
    }
    bool onResume() override
    {
        EASE_INFO("name: %s, on resume", task_name_.c_str());
        return true;
    }
    void onCancel() override
    {
        EASE_INFO("name: %s, on cancel", task_name_.c_str());
    }
    void onHalt() override
    {
        EASE_INFO("battery is less 30, stop navigation");
        EASE_INFO("name: %s, on halt", task_name_.c_str());
    }

private:
    int cost_time_{3};//s
    int64_t start_time_{};
};
```

## 2、注册task类型

```c++
#include "types/reflex.h"

REGISTER_TASK_TYPE(BatteryLevelCheck)
REGISTER_TASK_TYPE(ScreenReturnZeroTask)
```

## 3、编写策略编排脚本

```c++
const char *code = R"(
    var navigation_task = NavigationTask("navigation_task")
    var screen_return_zero_task = ScreenReturnZeroTask("screen_return_zero_task")

    parallel
    {
        navigation_task
        screen_return_zero_task
    }
)";
```

# 4、解释代码并运行

```c++
#include "scripting/script_interpreter.h"

int main()
{
    ScriptInterpreter script_run;
    if (auto scheduler = script_run.interpret(code))
    {
        printf("%s\n", scheduler.value()->toString().c_str());
        scheduler.value()->run();
    }
    else
    {
        printErrorInfo(script_run.getErrorInfo());
    }   
    return 0;
}
```

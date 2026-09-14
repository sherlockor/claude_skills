# 使用循环的编排策略

场景：

1. 检查电池电量是否高于30%；
2. 如果电量>=30%，则导航循环；
3. 电量<30%，则停止；

## 1、编写task

**BatteryLevelCheck** ：电池电量检查

```c++
class BatteryLevelCheck: public we::task::SyncRequestTask
{
public:
    explicit BatteryLevelCheck(const std::string &name)
        : we::task::SyncRequestTask(name)
    {
    }
    ~BatteryLevelCheck() override
    {
    }
    bool onStart() override
    {
        static int count = 0;
        if (count++ < 3)
        {
            EASE_INFO("name: %s, battery more than 30", task_name_.c_str());
            return true;
        }
        else
        {
            EASE_INFO("name: %s, battery less than 30", task_name_.c_str());
            return false;
        }
    }
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
REGISTER_TASK_TYPE(NavigationTask)
```

## 3、编写策略编排脚本

```c++
const char *code = R"(
    var navigation_task = NavigationTask("navigation_task")
    var battery_level_check = BatteryLevelCheck("battery_level_check")

    #while测试
    while(battery_level_check)
    {
        navigation_task
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

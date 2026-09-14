# 函数

语法如下：
```bash
function 函数名()
{
  #函数体
}
```
1. 函数和正常编程语言一样，用于一定组合的封装，定义函数后，使用函数名开始调用，执行逻辑同顺序执行；
2. 函数定义必须在调用之前，否则会出现找不到变量的错误；
3. 函数执行完成后，函数体内task的状态不会重置；

# 代码示例
## 1、编写task

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

**DockingTask**， 回充任务
```c++

class DockingTask: public we::task::SyncTask
{
public:
    DockingTask(const std::string &name)
        : we::task::SyncTask(name)
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
        EASE_INFO("name: %s, running done", task_name_.c_str());
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

REGISTER_TASK_TYPE(NavigationTask)
REGISTER_TASK_TYPE(DockingTask)
```

## 3、编写策略编排脚本
```c++
const char *code = R"(
    var navigation_task = NavigationTask("navigation_task")
    var docking_task = DockingTask("docking_task")

    function run()
    {
        navigation_task
        docking_task
    }

    docking_task
    run
)";
```
代码说明：
1. 定义函数run，函数体表示，按顺序执行navigation_task和docking_task；
2. 整体脚本运行，是先执行docking_task，再执行函数run，是先执行docking_task -> navigation_task ->docking_task；
3. 执行遵循顺序原则，即其中任何一个task fail，整体运行中止，后续task不再执行；

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

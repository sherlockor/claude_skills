# 如何使用脚本来创建调度策略

以最简单的单任务示例：
1. 编写task；
2. 注册task类型；
3. 编写策略脚本代码；
4. 解释代码并运行；

## 1、编写task

自定义task需要继承 SyncTask 或者 SyncRequestTask

```c++
class NavigationTask: public we::task::SyncTask
{
public:
    explicit NavigationTask(const std::string &name)
        : we::task::SyncTask(name)
    {
    }
    ~NavigationTask() = default;

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
```

## 3、编写策略脚本代码

```c++
const char* code = R"(
    var navigation_task = NavigationTask("navigation_task")
    navigation_task
)";
```

## 4、解释代码并运行

```c++
ScriptInterpreter script_run;
if (auto scheduler = script_run.interpret(code))
{
    scheduler.value()->run(); //运行
}
else
{
    //错误处理
    ErrorInfo eror_info = script_run.getErrorInfo();
    printf("%s\n%d:%d, %s\n\n", error_info.line_code.c_str(), error_info.line, error_info.column, error_info.error_message.c_str());
}
```
运行输出：
```bash
name: navigation_task, on start
name: navigation_task, on running pass
name: navigation_task, done
```

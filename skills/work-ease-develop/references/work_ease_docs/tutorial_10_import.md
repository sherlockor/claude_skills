# 调用其他脚本

语法如下：
```bash
import docker_script.bt
import docker_script
import ./docker_script
import ../docker_script
import ../script./docker_script
```
1. import 关键字后传入希望调用的脚本路径，可使用绝对路径，或相对路径；
2. 传入的脚本可以带.bt后缀，也可以不带。如果不带.bt后缀，则脚本解析时自动添加；
3. import 导入其他脚本后，当前脚本只会加载导入脚本的的实例（var定义的task和function定义的函数），而不会加载逻辑；
4. import 其他脚本后，如果需要使用其他脚本的实例，需要使用命名空间指定（脚本名）；

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
主脚本: test_run_import.bt：
```
import test_run_simple_script #import脚本test_run_simple_script

function run_navi()
{
    var navigation_task = NavigationTask("import_navigation_task")
    navigation_task
}

run_navi

#执行test_run_simple_script.bt中的run_docker函数，则使用test_run_simple_script指定
test_run_simple_script.run_docker
```
被调用脚本: test_run_simple_script.bt：
```
var navigation_task = NavigationTask("navigation_task")

function run_docker()
{
    var docking_task = DockingTask("docking_task")
    navigation_task
    docking_task
}


navigation_task
run_docker

```

# 4、解释代码并运行

```c++
#include "scripting/script_interpreter.h"

int main()
{
    std::string path = "./script/test_run_import.bt";
    script::ScriptInterpreter script_interpreter;
    std::optional<we::scheduler::AbstractSchedulerConstPtr> scheduler = script_interpreter.interpretSourceFile(path);
    if (!scheduler)
    {
        const ErrorInfo &error_info = script_interpreter.getErrorInfo();
        printErrorInfo(error_info);
        FAIL();
    }
    printf("interpreter pass\n");
    scheduler.value()->run();
}
```

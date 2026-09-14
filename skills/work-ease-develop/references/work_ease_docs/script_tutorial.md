# 1、脚本语法说明

关键字：

| key        | description |
|------------|-------------|
| var        | 定义变量        |
| if/else    | if-else条件   |
| repeat     | 重复执行        |
| retry      | 重试执行        |
| while      | 循环          |
| reactive   | 响应式         |
| parallel   | 并行          |
| force_pass | 强制成功        |
| force_fail | 强制失败        |
| function   | 定义函数        |
| import     | 引入其他脚本      |
| print      | 打印日志        |
| sleep      | 休眠          |
| pause      | 暂停          |
| cancel     | 取消          |

操作符号

| operator | description |
|----------|-------------|
| !        | 非           |
| and      | 与           |
| or       | 或           |
| =        | 变量赋值        |
| ""       | 字符串         |

# 2、语句说明

1. 代码以行为单位，即一行为一条语句；
2. 组合代码使用 **{}** 包裹，如if/while的执行体等，即使组合代码只有一行，也必须用 **{}** 包裹；
3. 是用 **#** 作为注释标记，其后所有内容都作为注释

# 3、关键字说明

## 3.1、var

- 用于定义任务变量

- 声明的任务变量在后面代码中可用于执行任务，任务变量执行返回成功或者失败

- 任务变量的声明必须指定任务类型

示例：

```python
#声明任务变量get_pose，类型是GetPose，传入了"get_pose_name"作为参数，后面可以执行任务变量get_pose
var get_pose = GetPose("get_pose_name")
get_pose #执行任务
```

## 3.2、if-else

用于条件判断语句

示例：

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

## 3.3、while

- 用于根据条件，循环执行
- 条件和执行体都是task或task组
- while 循环退出的唯一条件是，循环条件结束是 Failure，否则，即使内部执行体返回Failure，循环也不会退出

示例：

```python
var battery_level_check = BatteryLevelCheck("battery_level_check")
var navigation_task = NavigationTask("navigation_task")

while(battery_level_check)
{
    navigation_task
}
```

## 3.4、repeat

- 用于重复执行，需要传入重复次数
- 任务变量执行成功后，会重复，执行失败则跳出重复

示例：

```python
var navigation_task = NavigationTask("navigation_task")

#重复执行三次导航任务，如果其中有失败的，则立即停止
repeat(3)
{
    navigation_task
}
```

## 3.5、retry

- 用于重试执行，需要传入重试次数
- 任务变量执行失败后，会重试，执行成功则跳出重试

示例：

```python
var navigation_task = NavigationTask("navigation_task")
retry(3)
{
    navigation_task
}
```

## 3.6、reactive

- 用于响应式执行
- reactive 的使用场景是：在阻塞或非阻塞的task或task组中，不断检查reactive条件，如此，当reactive条件不满足时，task或task组会立即处理；

**执行逻辑：**

- 如果reactive条件执行结束是Failure（reactive条件触发），其内部正在执行子task会触发 ```onReactive()``` 回调, ```onReactive()``` 函数需要一个bool型返回值：
    - 如果返回true，则继续tick，这意味着本次忽略 reactive 条件事件;
    - 如果返回false，则当前task停止tick， ```onHalt()``` 回调会被执行，且当前task的执行结果是Failure；
- 如果reactive条件执行结果是Success（reactive条件未触发），则reactive语句的最终结果由执行体执行结果决定；

> 和while的区别是，在执行体执行过程中，循环检查条件，while是执行体执行完成，再检查条件：

## 3.7、parallel

- 用于同步执行，内部多任务同时开启执行
- 内部任务全部Success，则整个parallel执行结果为Success
- 内部任务出现一个或多个Failure，则整个parallel执行结果为Failure
- 当内部内部出现Failure，其余任务会触发```onHalt()``` 回调

parallel和reactive区别：

- reactive: 在执行体tick时，不断tick条件，且当条件执行完成后，再tick执行体
- parallel: 内部任务全部同步tick，且不重复执行

示例：

```python
var battery_check = BatteryLevelCheck("battery_check")
var navigation_task = NavigationTask("navigation_task")
parallel
{
    battery_check
    navigation_task
}
```

## 3.8、force_pass

- 用于强制成功，内部多任务无论结果如何，最后都会将结果变成成功

示例：

```python
var battery_check = BatteryLevelCheck("battery_check")
var navigation_task = NavigationTask("navigation_task")
force_pass
{
    battery_check
    navigation_task
}
```

## 3.9、force_fail

- 用于强制失败，内部多任务无论结果如何，最后都会将结果变成失败

示例：

```python
var battery_check = BatteryLevelCheck("battery_check")
var navigation_task = NavigationTask("navigation_task")
force_fail
{
    battery_check
    navigation_task
}
```

## 3.10、function

函数用于脚本中特定策略的封装，便于复杂脚本阅读，使用时，有以下几点需要注意：

- 定义函数，需要指定函数名，使用函数时，需要通过函数名执行
- 函数内部执行，和脚本执行策略相同
- 函数需要经过调用才会执行
- 函数执行后，内部task不会还原状态

示例：

```python
var navigation_task = NavigationTask("navigation_task")
var docking_task = DockingTask("docking_task")

function run()
{
    navigation_task
    docking_task
}

docking_task
run
```

如上脚本示例，执行顺序是：docking_task -> navigation_task -> docking_task

## 3.11、print

```print``` 用于在脚本中打印日志，使用方式如下：

- print语句需要传入一个字符串作为参数；
- print语句一定返回Success；

```python
print("Hello World")
```

## 3.12、sleep

```sleep``` 用于在脚本中休眠一段时间，使用时需要传入一个uint64的休眠时间，单位：ms

- 休眠时间不能小于0;
- 休眠时可以触发暂停/恢复/取消等事件;
- 休眠时如果暂停，休眠计时器不会停止tick;
- 休眠语句一定返回Success；

使用方式如下：

```python
sleep(10000) #休眠10s
```

## 3.13、pause/cancel

用于在脚本中暂停/取消整条任务链

使用方式如下：

```python
pause()
cancel()
```

# 4、执行逻辑

和常见的脚本语法不同，bt脚本前后语句执行需要遵循条件顺序，及A成功，B才会执行

示例：

```python
var navigation_task = NavigationTask("navigation_task")
var docking_task = DockingTask("docking_task")

function run()
{
    navigation_task
    docking_task
}

#以下为逻辑语句
docking_task
#docking_task执行成功，run才会执行，docking_task如果失败，则run不再执行，所有逻辑结束
run
```

下面两段脚本，运行逻辑相同

```python
docking_task #docking_task执行成功，run才会执行
run
```

逻辑效果等同于：

```js
if(docking_task)
{
    run
}

```

# 5、内置数据

## 5.1、WorkEase内置部分通用task类型，如下：

| 类型                     | 说明                     |
|------------------------|------------------------|
| SuccessSyncRequestTask | 此类型创建的task不执行任务操作，一定成功 |
| FailureSyncRequestTask | 此类型创建的task不执行任务操作，一定失败 | 
| PrintSyncRequestTask   | 打印字符串，结果是成功            |
| SleepSyncTask          | sleep任务                |
| PauseSyncRequestTask   | 暂停任务链                  |
| ResumeSyncRequestTask  | 恢复任务链                  |
| CancelSyncRequestTask  | 取消任务链                  |

## 5.2、内置变量（注意：同名变量不能重复创建）：

| 变量                          | 类型                     | 说明   |
|-----------------------------|------------------------|------|
| __success_sync_request_task | SuccessSyncRequestTask | 执行成功 |
| __failure_sync_request_task | FailureSyncRequestTask | 执行失败 |
| __print_sync_request_task   | PrintSyncRequestTask   | 打印日志 |

## 5.3、内置上下文参数

| key                             | 类型              | 说明                 |
|---------------------------------|-----------------|--------------------|
| _work_ease_CANCEL_FLAG          | bool            | 当前任务链是否执行了cancel   |
| _work_ease_TASK_CHAIN_STATUS    | TaskChainStatus | 当前任务链状态            |
| _work_ease_SKIP_FINISH_CALLBACK | bool            | 是否需要忽略结束回调函数       |
| _work_ease_START_TIMESTAMP      | uint64_t        | 任务链开始时间戳, 单位:s     |
| _work_ease_PAUSE_TIMESTAMP      | uint64_t        | 任务链最近一次暂停时间戳, 单位:s |
| _work_ease_PAUSE_TIME           | uint64_t        | 任务链总暂停时长, 单位:s     |
| _work_ease_pause_num            | uint32_t        | 任务链暂停次数            |


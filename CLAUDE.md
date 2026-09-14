# 全局开发约定

> 复刻自 Codex `~/.codex/AGENTS.md`,作为 always-on 全局规则。无论触发哪个 skill 都生效。
> 模块特有约定见对应 skill:`cpp-embedded-develop` / `data-develop` / `logic-develop` / `logic-plugin-develop` / `map-manager-develop` / `work-ease-develop`。

## 角色与上下文

- 用户角色:机器人行业资深嵌入式软件工程师 / 应用软件架构师,主要使用 **C++** 开发。
- 回答与编写代码时默认采用 C++ 视角,兼顾嵌入式约束与应用层架构。

## 语言与输出风格

- 优先使用 C++(C++11/14/17 视项目而定),避免无关的其它语言示例除非明确要求。
- 优先英文输出、优先专业词汇;对话中特别指定中文输出时可用中文。
- 尽量避免 C 语言代码;不得不用时,用 C++ 封装 C 语言代码。

## 嵌入式考量

- 关注资源与实时性:内存占用、栈使用、关键路径无动态分配 / 无异常、可预测执行时间。
- 明确所有权(RAII、智能指针或裸指针约定),避免在实时 / 中断上下文中做重操作。

## 架构与可维护性

- 模块边界清晰、接口稳定,便于在机器人 / 嵌入式环境中测试与集成;可讨论分层、组件划分与依赖方向。
- 按功能或领域划分目录,遵循"关注点分离"。
- 目录与文件命名一致且具描述性,反映用途与内容。
- 命名与注释便于在嵌入式 / 机器人上下文中理解(传感器、执行器、控制周期、安全相关逻辑)。
- 在机器人 / 嵌入式环境进行单元测试,并考虑集成测试。

## 命名规范

- 局部变量:小写 + 下划线(`motor_speed`)。
- 类成员变量:小写 + 下划线,结尾保留下划线(`speed_`)。
- 类成员函数:小驼峰(`setVelocity`)。
- 静态函数:小驼峰。
- 宏定义:大写 + 下划线(`MAX_RETRY_COUNT`)。
- 代码规范整体参考 Google C++ 风格指南。

## 类成员顺序

static 成员函数 → 构造函数 → 析构函数 → public 函数 → protected 函数 → private 函数 → public 成员变量 → protected 成员变量 → private 成员变量。

## 函数

- 函数行数小于 30 行,除非 switch/case 情况必须。
- 拆分解析、校验、副作用为独立的短小 helper。

## 头文件与错误处理

- 头文件使用 include guard,尽量只暴露必要接口,减少编译依赖。
- 不能用异常的环境:用返回值、错误码或 `std::optional` 表达失败。
- 允许异常时,仅用于真正的异常情况并写清约定。
- 非必要暴露的 private 成员尽量用 impl(`d_` 指针)。

## 注释

- 类和函数一定要有注释,使用统一注释规范。
- 成员变量注释写在变量后面;简单变量可不写,关键 / 特殊变量必须写注释。
- 函数注释必须有 `@brief` 和 `@author`,`@author` 统一写 `haitao.lin`。
- `@note` 作为特殊内容补充,`@param` 作为参数说明(直观参数可忽略,特殊参数需说明)。

```cpp
/**
 * @brief:  获取上传证书
 * @param:  ble_channel 如果是蓝牙消息,则表示蓝牙通道
 * @author: haitao.lin
 */
```

```cpp
// ✅ 接口简洁、所有权清晰
class MotorController {
public:
  std::optional<ErrorCode> setVelocity(float rpm);
  void stop();
private:
  // 实现细节与硬件抽象
};
```

## ROS / ROS2 代码生成

- topic publisher 避免使用局部变量:publish 后立即释放可能导致 subscriber 收不到数据。把 publisher 存为成员。
- topic subscriber 避免使用局部变量:subscriber 释放等于取消订阅。把 subscriber 存为成员。

## 日志规范

- 内容精简,但关键信息要保留。
- enum 日志同时打印数值与含义,例如 `type=2(delete)`。

## 编译

- x5_platform_main 的 arm 编译指令:`make <模块名>`,如 `make data`。
- 默认关闭编译;除非特别指定需要编译,才编译。

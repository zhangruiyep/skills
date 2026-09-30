---
name: "c-coding-standard"
description: "C语言编程规范V1.1（编码规范/代码风格/命名规范/注释规范）。所有涉及C/C++代码的任务都必须遵循本规范。Invoke when: (1) generating/creating any new C/C++ source file or code (write C code, implement, add function, create header, 新增C代码、写C程序、实现C函数、添加头文件); (2) modifying/editing existing C/C++ code (change, fix, refactor, update function, macro, variable, comment, header, 修改C代码、改C语言、调试修复C/C++、重构); (3) user asks to follow coding standards/style/comment/naming conventions (编码规范、编程规范、代码规范、代码风格、命名规范、注释规范、遵守规范、code style, style guide); (4) reviewing or auditing C/C++ code (code review, 代码审查、代码评审、检查代码规范); (5) any embedded/RTOS/SDK development on this project writing C code. Covers: English comments, indentation, Linux-style naming, macro/enum uppercase, typedef suffix, variable init before use, function <200 lines & nesting <=4, static globals, include guards, no magic numbers, input validation, macro-guarded code. | C语言编程规范V1.1，适用于本工程全部C/C++代码：生成新代码、修改现有代码、代码审查/重构，以及用户要求遵循编码/命名/注释规范时，必须先加载本技能并按规范执行。"
---

# C语言编程规范 V1.1

本规范适用于本项目所有C/C++代码。生成新代码或修改现有代码时，必须遵守以下规则。

## 生效条件 / Invocation Conditions

满足以下任一条件即应加载本技能（自动触发，无需用户显式指定）：

- **条件 A（生成新代码）**：编写/创建任何 C/C++ 文件（`.c` / `.h` / `.cpp` / `.hpp`）或新增函数、宏、结构体、变量、注释等代码。
- **条件 B（修改现有代码）**：修改、修复、重构、优化任何现有 C/C++ 代码，包括重命名、调整缩进/注释、拆分函数、改头文件等。
- **条件 C（用户显式要求）**：用户提到“代码规范/编码规范/编程规范/代码风格/命名规范/注释规范/遵守规范”或英文 `coding standard / code style / style guide / naming convention` 等关键词。
- **条件 D（代码审查）**：对 C/C++ 代码进行 code review、代码审查、代码评审、规范检查。
- **条件 E（本工程开发）**：任何涉及本嵌入式/RTOS/SDK 工程的 C 代码开发任务。

> 一句话：只要本次任务**接触或产生 C/C++ 代码**，就先加载本技能。

## 1. 注释规范

- 代码注释必须使用**英文**。
- **单行注释**：使用 `//`，放在代码**右侧**。
- **多行注释**：使用 `/* */`，放在代码**上一行**。

```c
// Good
int ret = do_something();  // do something and get result

/*
 * This is a multi-line comment
 * explaining the following logic.
 */
void func(void) { ... }
```

## 2. 缩进与换行

- 代码换行时增加**一级缩进**（一个TAB）。
- **双目操作符**两边必须加空格。
- 对于**独立文件或独立函数**：严格按照此标准执行。
- 对于**修改原有第三方代码**：与原代码格式保持一致。

```c
// Good
int result = a + b;
int value = (condition_a && condition_b)
	|| (condition_c && condition_d);
```

## 3. 命名规范

### 3.1 函数与变量（Linux风格）
- 全部**小写**，单词之间用**下划线**分隔。
- 独立文件/函数按此标准执行；修改第三方代码时保持原风格。

```c
// Good
void uart_send_data(uint8_t *buf, uint32_t len);
int receive_count = 0;
```

### 3.2 宏与枚举
- 全部**大写**，单词之间用**下划线**分隔。

```c
// Good
#define MAX_BUFFER_SIZE  256
#define GPIO_PIN_NUM     5

enum {
	STATE_IDLE,
	STATE_RUNNING,
	STATE_ERROR,
};
```

### 3.3 Typedef 类型
- 以大写 `_T` 结尾，或小写 `_t` 结尾。

```c
// Good
typedef struct {
	int x;
	int y;
} POINT_T;

typedef uint32_t tick_t;
```

## 4. 变量使用规范

### 4.1 最小取值范围原则
- 能用 `bool` 或 `enum` 时，尽量不用 `int`。

```c
// Good
bool is_ready = false;
enum state_type state = STATE_IDLE;

// Bad
int is_ready = 0;
int state = 0;
```

### 4.2 变量/指针/结构体初始化
- 变量、指针、结构体在**使用之前必须赋值**（初始化）。

```c
// Good
int count = 0;
char *buf = NULL;
struct config_t cfg = {0};
```

## 5. 函数设计规范

### 5.1 代码块嵌套深度
- 新增函数的代码块嵌套**不超过4层**。

### 5.2 函数行数
- 一个函数的总行数控制在**200行以内**。
- 超过200行的函数，需要**至少2人评审通过**才可以入库。

## 6. 全局变量规范

- 全局变量必须加 `static`，限制在**本文件内使用**。
- **模块之间的数据传递**必须使用函数方式。

```c
// Good
static int g_local_counter = 0;

int get_counter(void) {
	return g_local_counter;
}
```

## 7. 头文件规范

### 7.1 包含保护符
- 头文件必须编写 `#define` 包含保护符（include guard）。

```c
// Good
#ifndef __UART_DRIVER_H__
#define __UART_DRIVER_H__

// ... header content ...

#endif  // __UART_DRIVER_H__
```

### 7.2 头文件精简
- 头文件应尽量精简，仅包含本模块**对外提供的接口或定义**。
- 内部使用的宏、类型、函数声明不要暴露在头文件中。

## 8. 禁止魔鬼数字

- 不允许直接使用魔鬼数字（magic number），必须定义为有意义的宏或枚举。

```c
// Good
#define RETRY_MAX_COUNT  3
for (int i = 0; i < RETRY_MAX_COUNT; i++) { ... }

// Bad
for (int i = 0; i < 3; i++) { ... }
```

## 9. 用户输入合法性检查

- 对**用户输入**必须进行合法性检查。

```c
// Good
if (buf == NULL || len == 0) {
	return -1;
}
```

## 10. 宏控制代码规范

- 使用宏控制的代码，必须确保**宏打开或关闭均可以编译通过**。
- 除BSP模块外，其他模块**禁止直接使用项目级别的宏**控制代码。

```c
// Good - 宏打开/关闭均能编译
#if CONFIG_FEATURE_ENABLED
	do_feature();
#endif

// Bad - 只在BSP外使用项目宏
#if PROJECT_X_ENABLED  // Not allowed outside BSP
	...
#endif
```

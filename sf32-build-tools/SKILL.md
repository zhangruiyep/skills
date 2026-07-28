---
name: "sf32-build-tools"
description: "SF32 系列芯片（SF32LB52x/55x/56x/58x）编译、下载与编译报错检查工具。当项目使用 SF32 系列芯片且 Solution 版本号 >= 2.0 时自动生效。用于执行 scons 编译、检查编译报错、烧录固件等任务。"
---

# SF32 编译与下载工具

## 触发条件

当以下条件全部满足时，本 Skill 自动生效：

1. 项目使用思澈科技（SiFli）SF32 系列芯片：SF32LB52x / SF32LB55x / SF32LB56x / SF32LB58x
2. Solution 版本号 >= 2.0（检查 `solution/.version` 中的 `MAJOR` 字段）
3. 用户需要执行编译

> 当前项目 Solution 版本可通过读取 `solution/.version` 文件确认。

---

## 项目架构概述

SF32 Solution 项目采用**双核架构**：
- **HCPU**（大核 Cortex-M33）：主应用处理器，运行 RT-Thread + LVGL
- **LCPU**（小核 Cortex-M33）：低功耗处理器，运行 BLE 协议栈

构建系统使用 **SCons**（基于 RT-Thread 构建框架）+ Kconfig 配置系统。

---

## 一、环境准备

### 1.1 初始化环境变量

在项目根目录执行：

```powershell
.\set_env.bat          # 默认使用 Keil 编译器
```

支持的工具链参数：
- `set_env.bat keil` — Keil MDK (ARMCC)，默认
- `set_env.bat gcc`  — GNU Arm GCC
- `set_env.bat iar`  — IAR EWARM

### 1.2 Kconfig 配置（菜单配置）

```powershell
scons --menuconfig     # 在 hcpu 项目目录下执行
```

配置修改后会自动更新 `rtconfig.h`。

---

## 二、编译

### 2.1 方式一：SCons 直接编译（不推荐）

在 HCPU 项目目录（如 `solution\examples\watch\project\fy6718\hcpu\`）下执行：

```powershell
cd solution\examples\watch\project\fy6718\hcpu
scons                   # 编译 HCPU
```

**常用参数：**

| 命令 | 说明 |
|---|---|
| `scons` | 编译 HCPU 固件 |
| `scons -jN` | 多线程编译（N 为线程数，如 `-j8`） |
| `scons --clean` | 清理编译产物 |
| `scons --menuconfig` | 打开 Kconfig 配置菜单 |
| `scons --resource` | 仅编译资源文件（字符串、语言包等） |
| `scons --resource --lang=<name>` | 指定语言包编译 |

**编译输出（在 `build/` 目录下）：**
- `build/bf0_ap.axf` — ELF 可执行文件（含调试信息）
- `build/bf0_ap.bin` — 烧录用二进制文件
- `build/bf0_ap.hex` — Intel HEX 格式（JLink 烧录用）
- `build/bf0_ap.map` — 链接映射文件

### 2.2 方式二：Butterfli 工具编译（集成资源打包 + 编译 + 烧录，推荐优先使用）

```powershell
cd solution\tools\sifli_develop\Butterfli
Butterfli --group2 watch fy6718 --binpath <备份路径> --logpath <日志路径> --hide 1
```

参数说明：
- `--group2`：项目类型名和项目名，如 `watch fy6718`
- `--binpath`：bin 备份目录
- `--logpath`：日志输出路径
- `--hide`：隐藏界面（0 显示，1 隐藏）

也支持使用脚本调用（如项目根目录下的 `fy6718_build.bat`）。使用bat脚本时，不需要设置环境变量。直接在windows cmd.exe环境执行即可。

**Butterfli只支持一个实例，当系统中已经有Butterfli程序正在执行，则不要再次启动，需等待当前Butterfli执行结束。绝对不能强制停止正在后台运行的Butterfli程序。**

编译前，先删除已经存在的编译日志文件。完整编译大概需要十几分钟。等待Butterfli后台执行结束后，再查看日志，确认编译是否成功。

使用脚本调用（如项目根目录下的 `fy6718_build.bat`）编译时，会自动更新缓存，不需要手动删除.o、.lib等目标文件。

**编译成功后，会在bin_bak目录下生成多个bin文件（如ER_IROM1.bin），这些文件可以在编译日志中确认是否正确生成了。**

### 2.3 编译新项目（当前项目不匹配时）

如果当前项目名（如 fy6718）与用户要求的不同，需要找到对应项目的 HCPU 目录：
```
solution\examples\<项目类型>\project\<项目名>\hcpu\
```

例如：
- 手表项目：`solution\examples\watch\project\<项目名>\hcpu\`
- Grid View：`solution\examples\grid_view\project\<项目名>\hcpu\`
- Hello World：`solution\examples\hello_world\project\<项目名>\hcpu\`

---

## 三、编译报错检查与处理

### 3.1 执行编译并捕获报错

编译流程：
1. 参考2.2章节的编译流程
2. 等待Butterfli后台编译完成
3. 检查退出码：`0` 表示成功，非 `0` 表示失败

### 3.2 常见报错类型及处理

| 报错类型 | 示例 | 常见原因 | 处理方式 |
|---|---|---|---|
| **头文件找不到** | `fatal error: xxx.h: No such file or directory` | include 路径缺失 | 检查 `SConscript` 中是否添加了对应路径 |
| **符号未定义** | `undefined reference to 'xxx'` | 缺少源文件或库链接 | 确认 `SConscript` 中是否包含对应 `.c` 文件 |
| **重定义** | `multiple definition of 'xxx'` | 同一符号在多个 `.o` 中定义 | 检查是否有重复的源文件或全局变量定义 |
| **语法错误** | `error: expected ';' before 'xxx'` | C/C++ 语法错误 | 检查对应行代码语法 |
| **类型不匹配** | `warning: incompatible pointer type` | 类型转换问题 | 确认指针类型是否匹配 |
| **链接脚本错误** | `section xxx overlaps` | 内存布局溢出 | 检查链接脚本和内存配置 |
| **段溢出** | `section xxx will not fit in region xxx` | Flash/RAM 空间不足 | 优化代码或调整分区 |
| **prebuild.bat 失败** | prebuild 阶段找不到文件 | LCPU 固件路径错误 | 检查 BSP 板级配置 |

### 3.3 报错分析流程

1. **定位报错文件**：从编译器输出中提取报错文件路径（相对于 SDK/Solution 根目录）
2. **阅读相关代码**：使用 Read 工具查看报错文件的相关上下文
3. **分析根因**：
   - 如果是代码逻辑错误 → 直接修复
   - 如果是配置/路径问题 → 检查 SConscript、rtconfig.h、.config
   - 如果是链接问题 → 检查 SConscript 是否包含所需源文件
4. **修复后重新编译验证**

### 3.4 编译产物验证

编译成功后检查项目对应的 `bin_bak` 目录下是否生成了 `.bin` 文件：
```
ls bin_bak/ER_IROM1.bin
```

---

## 四、固件下载/烧录

### 4.1 UART 串口烧录（主要方式）

```powershell
cd solution\tools\sifli_develop\Butterfli
ImgDownUart.exe --func 0 --port <COM端口> --baund 6000000 --loadram 1 --postact 1 --device <设备类型> --file "ImgBurnList0.txt" --log ImgDownUart_log.txt --compare --verify
```

参数说明：
- `--port`：串口号，如 `COM18`
- `--baund`：波特率，如 `6000000`
- `--device`：设备类型，如 `SF32LB52X_NAND`、`SF32LB52X_NOR`、`SF32LB56X_NAND` 等
- `--loadram 1`：先加载 RAM 程序再烧录
- `--postact 1`：烧录后启动固件
- `--compare` / `--verify`：烧录后校验

使用脚本：项目根目录下的 `fy6718_uart_dl.bat <COM端口>`，需传入串口号作为参数。例如 `fy6718_uart_dl.bat COM4`。不传参数时会提示用法并退出。

如果进入调试模式失败，提示用户检查：
- 确认串口号是否正确
- 确认串口是否可用，是否被其他程序占用
- 确认设备是否已在正常工作模式，不能在休眠模式下

### 4.2 JLink SWD 烧录

脚本位于 `sdk\tools\segger\`：

| 脚本 | 目标芯片 |
|---|---|
| `download_pro.bat <bin_path>` | SF32LB58X |
| `download_a0.bat <bin_path>` | SF32LB55X |
| `download_lite.bat <bin_path>` | BUTTERFLITE |

使用方式：
```powershell
cd sdk\tools\segger
.\download_pro.bat <build目录路径>
```

JLink 连接命令（调试用）：
```powershell
jlink.exe -device SF32LB58X -if SWD -speed 10000 -autoconnect 1
```

### 4.3 设备类型对照

| 芯片 | NAND Flash | NOR Flash | SD/eMMC |
|---|---|---|---|
| SF32LB52X | `SF32LB52X_NAND` | `SF32LB52X_NOR` | `SF32LB52X_SD` |
| SF32LB55X | `SF32LB55X_NAND` | `SF32LB55X_NOR` | — |
| SF32LB56X | `SF32LB56X_NAND` | `SF32LB56X_NOR` | `SF32LB56X_SD` |
| SF32LB58X | `SF32LB58X_NAND` | `SF32LB58X_SD` | — |

---

## 五、常用操作流程

### 5.1 标准编译流程

参考2.2章节的编译流程。

### 5.2 用户要求编译时应做的事情

1. 确定目标项目（默认 fy6718，或用户指定）
2. 参考2.2章节的编译流程
3. 分析输出：
   - 编译成功 → 报告成功，列出产物路径
   - 编译失败 → 提取报错信息，分析原因，定位代码位置，提供修复建议
4. 如用户要求下载，按上述烧录方式执行

---

## 六、关键文件路径速查

| 用途 | 路径 |
|---|---|
| Solution 版本号 | `solution\.version` |
| SDK 版本号 | `sdk\.version` |
| 产品配置 | `solution\.product` |
| 环境设置 | `set_env.bat`（根目录）+ `sdk\set_env.bat` |
| HCPU SConstruct | `solution\examples\<type>\project\<name>\hcpu\SConstruct` |
| Butterfli 工具 | `solution\tools\sifli_develop\Butterfli\Butterfli.exe` |
| Butterfli 配置 | `solution\tools\sifli_develop\Butterfli\configure\CompileBurnUser.ini` |
| JLink 下载脚本 | `sdk\tools\segger\download_*.bat` |
| UART 下载工具 | `solution\tools\sifli_develop\Butterfli\ImgDownUart.exe` |
| 构建配置 | HCPU 目录下的 `.config` 和 `rtconfig.h` |
| 编译输出 | HCPU 目录下的 `build/` |
| SDK 中间件版本 | `sdk\middleware\include\sdk_ver.h` |

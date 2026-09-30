---
name: "release-notes-updater"
description: "根据 git commit 时间范围，把功能修改点更新到仓库的 ReleaseNotes.md。当用户要求更新/维护 release notes、只提供 git commit 时间范围（如“8月7日以后”“从X到X”）时触发。"
---

# Release Notes 更新

把用户指定时间范围内的 git 功能修改点，追加到主仓库的 `ReleaseNotes.md` 顶部。主仓库不固定，适用任意仓库。用户下次只需给出 git commit 的时间范围。

## 触发条件

- 用户要求更新/维护/整理 release notes，并给出（或只给出）git commit 的时间范围。
- 用户提到「更新 ReleaseNotes.md」「把这几次修改写到 release notes」「整理修改记录」等。

## 路径（主仓库不固定）

- **主仓库**：用当前工作目录；若用户指定了仓库/目录，以用户指定为准。
- **release notes 文件**：主仓库根目录下的 `ReleaseNotes.md`（文件名或位置不确定时问用户）。
- **gitee 仓库**（合入复云代码时取记录）：默认 `E:\work\gitee\w6069`；若主仓库不是 W6069 或路径不符，问用户对应的 gitee 仓库路径。

## 工作流程

### Step 1：解析时间范围并拉取 commit

用户可能说「8月7日以后」「8月7日之后」「从 2026-08-07 到 2026-09-10」等。先转成具体日期（当前年份缺省时用当前年），再用带日期的 git log 拉取 commit：

```powershell
git log --pretty=format:"%h | %ad | %s" --date=short -N
```

只处理时间范围内的 commit。

### Step 2：过滤 commit

- **跳过版本号 commit**：subject 含 `modify version to`、`修改版本`、`mcu version to` 等纯版本号修改，不写成修改点，仅用于确定版本号。
- **跳过合并记录 commit**：subject 形如 `Merge "..." into watch`（Gerrit 的 merge 记录），真正的改动在它引用的那个 commit 里，不重复写。
- 其余为功能 commit。

### Step 3：逐条转写为修改点

规则：

1. **只写功能修改，不写版本号修改。**
2. **中文 commit 直接写；英文 commit 翻译成中文。**
3. **一条 commit 有多个修改，拆分成多个修改点**（用「1. 2. 3.」或子项「（1）（2）（3）」）。
4. **每条修改点带模块名称**，格式对齐 VJ9.3.7：`编号.模块: 描述`，例如 `1. BT: ...`、`4. BATTERY: ...`。
5. **描述要简短**，每条修改点控制在 **30 个汉字以内**，只说明改了什么，不展开实现细节。
6. **不要出现代码中的宏、常量、变量名、函数名、结构体名、字段名等标识符**（如 `PROTOCOL_T_U_HR_ANALYSIS`、`IWAPTK`、`alert_mode`），一律用中文业务语义描述（如「平均心率与心电图结果上报」「AI 心脏评估协议」「报警模式字段」）。

### Step 4：处理「合入 gitee 代码」

若 commit subject 含 `merge gitee code` / `合入 gitee`，去 gitee 仓库（默认 `E:\work\gitee\w6069`，见「路径」）取该时间点附近的 gitee 修改记录（通常是当天及上次合入之后的 `【MCU】`、`【MDM】` 提交）。

- gitee 仓库的 commit 消息是 **UTF-8 编码**，PowerShell 默认按 GBK 解码会乱码，必须显式用 UTF-8 读取：

```powershell
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
$pinfo = New-Object System.Diagnostics.ProcessStartInfo
$pinfo.FileName = "git"
$pinfo.Arguments = "log -1 --format=%B <commit_hash>"
$pinfo.RedirectStandardOutput = $true
$pinfo.StandardOutputEncoding = [System.Text.Encoding]::UTF8
$pinfo.UseShellExecute = $false
$p = [System.Diagnostics.Process]::Start($pinfo)
$p.WaitForExit()
$p.StandardOutput.ReadToEnd()
```

- gitee 记录用「合入复云代码」小节展示，按 `【MCU】`、`【MDM】` 分组。

### Step 5：确定版本号与时间

- **版本号**：取时间范围内最后一次 `modify version to X` 的版本号 `X`，前缀按该仓库现有 release notes 的写法（W6069 为 `W6069_V` + X，如 `W6069_VJ9.4.3`；其它仓库沿用其历史前缀）。
- **时间**：用该版本号 commit 的日期；若拿不准（例如现有 release notes 日期晚于 commit 日期），**问用户**。
- 时间范围内没有版本号 commit 时，问用户用哪个版本号。

### Step 6：写入 release notes（只新增，不改旧）

在 `## 修改记录` 下第一个 `----------` 之后、现有最新条目之前，插入新条目。**已有条目一律不改，只新增新条目。**

> 注意：**一个版本的条目可能包含多个仓库的修改点**（例如 MCU、MDM、gitee 复云代码等）。更新时若发现该版本条目已存在（其它仓库已先写入部分修改点），**不得覆盖或重写**已有修改点，只能在该条目的对应位置**追加/合并**本仓库的修改点：缺的模块分组（如 `【MDM】`、`【MCU】`）补上，已存在的修改点保持原样。

## 模块名称约定（参考历史）

| 模块 | 场景 |
|------|------|
| `BT` | 蓝牙心电贴 / 蓝牙连接 / 蓝牙名称 / 蓝牙扫描 |
| `BATTERY` | 电池电压 / 关机电压 / 电量 |
| `UI` | 表盘界面 / 页面显示 / UI 文案 |
| `MDM` | 数据协议 / dal / 上报 / modem 通信 |
| `MCU` | 手表端代码（主要出现在「合入复云代码」的【MCU】分组） |
| `OTA` | 升级相关 |
| 其他 | STEP / HR / SOS / SMS / TEMP / GPS / SYSTEM / CYWEE / ALGO / FACTORY 等，按历史语义选择 |

心电贴（蓝牙 ECG）整体归 `BT`，与 VJ9.3.7 保持一致。

**特例：ASR1603 / EC718 仓库** —— 若主仓库为 ASR1603 或 EC718 项目（路径/项目名/提交信息含 `ASR1603` 或 `EC718`），所有修改点的模块分类**统一使用 `MDM`**，不再细分 BT/UI/BATTERY 等。

## 格式模板（严格 Markdown，用空格缩进，禁止用 Tab）

```markdown
* 版本号：W6069_VJ9.4.3
* 时间：2026-09-10

* 主要修改点：
  * 修改的功能或需求：
    1. BT: 心电贴BUG修改：
       （1）修改设置蓝牙和心电贴界面蓝牙连接统一息屏时间改为15s；
       （2）解决偶现开启测量后产生上报无效数据包问题；
    2. BATTERY: 修改关机电压为3.5V；
    3. UI: 解决表盘界面上下滑偶现失效问题；
  * 合入复云代码：
    * 【MCU】
      1. 新增定位与海拔tlv和自动定位功能；
    * 【MDM】
      1. BPCF协议新增手表自动定位配置LOC，新增震动配置VIB；
```

要点：

- 顶层键值/标题用 `* `（无序列表）。
- 嵌套层级用 **2 空格** 递增缩进。
- 有序列表用 `1. `（`1.` 后必须带空格才是合法列表标记，不能写 `1.BT:`）。
- 子项 `（1）（2）` 用缩进段落，缩进对齐到父列表项内容位置。
- 键值分隔用空格，不用 Tab。
- 每个新条目结束后用一行 `----------` 与下一条目分隔。
- 版本号前缀按仓库历史写法（模板中 `W6069_V` 仅适用 W6069 仓库，其它仓库按各自历史调整）。

## 其它

- 不确定的点（版本号、日期、模块归属）**优先问用户**，不臆测。
- 完成后简要汇报：新增条目版本号/时间、各修改点来源与模块归类，便于用户复核。
- 不要修改 `ReleaseNotes.md` 中已有条目的格式或内容。

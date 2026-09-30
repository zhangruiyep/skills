---
name: gerrit-gitee-code-sync
description: 在 gerrit 与 gitee 之间同步指定仓库的代码（仅限 W6069 / FY6718 项目）。只有源仓库或目标仓库含 w6069、fy6718、SF32LB56x、SF32LB52x、ASR1603 中任意一个时才触发，其他仓库一律不生效。不用于普通两个目录之间的合并，那类需求走 dir-code-merge。
---

# Gerrit 与 Gitee 代码同步

把**一个仓库**的代码改动同步到**另一个仓库**，只在 gerrit 与 gitee 两侧的指定仓库之间进行。

## 触发条件

**只有源仓库或目标仓库命中以下任一关键词时才生效**：

`w6069`、`fy6718`、`SF32LB56x`、`SF32LB52x`、`ASR1603`

即：用户要求同步/搬运/对齐代码，且源仓库或目标仓库的路径或仓库名里含上面任意一个关键词。

以下情况**一律不生效**：

- 源仓库与目标仓库都不含上述任一关键词
- 两个普通目录/文件夹之间的代码合并，走 `dir-code-merge`

## 仓库位置与配对

- gitee 侧仓库根：`E:\work\gitee\<repo>`
- gerrit 侧仓库根：`E:\work\gerrit\<repo>`
- 只处理「同步规则」一节中列出的仓库对；未列出的仓库一律不碰，需要时先问用户。

## 核心原则

1. **先确认再动手**：同步方向、范围、路径映射拿不准就先问用户。
2. **只动指定仓库对**：不推断、不扩展到大盘其他仓库。
3. **区分文本与二进制**：release 包 → gitee release 仓时，文本与二进制一律整文件覆盖复制；只有两侧都是 git 仓库、且目标有需保留的自有改动时，文本才用 `git diff` 只合并差异。
4. **不自动 commit / push**：改动留在工作区，交给用户 review。
5. **目标有未提交改动先提醒**：存在被覆盖风险时停下确认。
6. **库文件范围不提供源码**：命中 `dal_sdk_release.py` 范围的改动不按源码同步，改同步 `lib_dir` 下的 `.lib` / 保留头文件，见 Step 3 与「不允许源码同步的范围」。
7. **只同步指定范围内的文件**：范围外的文件一律不动，不顺手扩大。
8. **源有、目标无就直接复制**：范围内源里存在、目标里没有的文件（含目标整块目录都不存在），直接建目录整文件复制，不要因为「目标没这目录」而跳过。

## 工作流程

### Step 1：确认仓库对与方向

- 源仓库（source）与目标仓库（target），以及方向（gitee→gerrit / gerrit→gitee）。
- 方向**每次由用户指定，单向执行**；用户没说就先问。
- 用户只给一侧时，按「同步规则」查配对，查不到就追问。

### Step 2：确认同步范围

```powershell
git -C <src> rev-parse --show-toplevel   # 确认仓库根，可能不是用户给的目录本身
git -C <src> status --short              # 工作区未提交的改动
git -C <src> diff --name-status          # 未提交的改动文件清单
git -C <src> log --oneline -N            # 最近提交
git -C <src> show --name-status <commit> --no-renames   # 某次提交改了哪些文件
```

`--name-status` 中 `A`=新增、`M`=修改、`D`=删除、`R`=重命名。

范围**每次由用户指定**；用户没给范围就先问。用上面命令在**源仓库**查出改动文件清单，这份清单就是本次要同步的文件集合，只处理这些文件。

**源侧可能根本没有 git 记录**：gitee 侧的 MCU 仓常以 release 包形式拿到（如 `E:\work\gitee\fy6718_SDK_20260921`，解压出来的目录不是 git 仓库）。这时范围改到**配对仓库**去查（本项目就是配对的 gerrit 仓，如 `E:\work\gerrit\SF32LB52x`）：

```powershell
git -C <配对仓库> log --oneline --since=<起始日期> --until=<结束日期>   # 按日期找区间端点提交
git -C <配对仓库> log --oneline -N                                      # 或先看有哪些提交
git -C <配对仓库> diff --name-status --no-renames <c1> <c2>             # 区间改动文件清单
```

区间端点用提交日期对齐（用户说的「9月4日以后」就落到当天那个 merge commit）。**源包只用来取文件内容，范围一律以配对仓库的 git 记录为准**。

### Step 3：排除不允许源码同步的内容（库文件）

MCU 仓库同步到 gitee 用的是 **release 包**，一部分代码在 gitee 侧是 **库文件（`.lib`）** 而不是源码，因此 gerrit 源码仓库与 gitee 目录**不是固定的一一对应**。

范围以仓库里的 `dal_sdk_release.py` 为准，**每次同步前重新读它**（位置不固定，先用 `Glob` 搜 `**/dal_sdk_release.py` 定位；「不允许源码同步的范围」一节里的清单只是快照，可能已过期）：

- `dal_src_dirs`：这些目录下的 `.c` / `.h` 在 release 时会被删除，**不得以源码形式同步到 gitee**。
- `dal_header_files`：例外，仍随 release 提供，可以正常同步。
- `dal_lib_files` 与 `lib_dir`：对应代码在 gitee 侧以 `.lib` 存在；改了这些源码后需要构建出 lib，再同步 lib。

**命中库文件范围时怎么同步**：范围内的文件若落在 `dal_src_dirs` 里，**不要同步源码路径**（源包里这些文件通常已被删除，压根不存在），改为同步 `lib_dir` 下的对应产物——源、目标都指向同一份 lib 目录文件：

- 头文件：`dal_header_files` 列出的文件在 `lib_dir` 下同名存放（如 `ec718.h`、`dal_mdm_service_api.h`、`dal_mdm_service_msg.h`、`dal_mdm_service_timer.h`），按 `lib_dir\<同名文件>` 同步。
- 源码：按代码归属映射到 `dal_lib_files` 里对应的 `.lib`，按 `lib_dir\<该 lib>` 同步。例如 FY6718：`sdk\customer\peripherals\modem\ec718\` 下的 `.c` → `dal_ec718.lib`；`solution\framework\service\srv_dal\modem\` 下的 `.c` → `dal_mdm_service.lib`。

识别方法：比对时源包里这些路径报「源不存在」就是命中了库文件范围，属正常现象，**不要当成错误**，也不要因此跳过——要转成上面的 lib 路径去判断和同步。

### Step 4：确定路径映射

两侧目录结构通常不一致，先建立源路径到目标路径的映射：

- 常见形式是共享同一子目录前缀，或多出/缺少一层前缀（如源 `mcu/xxx/...` → 目标 `xxx/...`）。
- 用 `LS` / `Glob` 对比两侧结构确认；`git show` 输出的是相对仓库根的路径，据此拼真实绝对路径。
- 拿不准就问用户，不要猜。

**源是 release 包时是 1:1 映射**：源为 `<project>_SDK_<日期>` 解压目录、目标为 gitee 的 release 仓时，两侧结构一致（顶层同为 `sdk`、`solution`），命中范围的相对路径**直接照搬**，不需要增删前缀。

### Step 5：检查目标仓库状态（防丢失）

```powershell
git -C <target> status --short
```

确认目标没有会被覆盖的未提交改动；有则先提醒用户。

### Step 6：执行同步

按文件类型分别处理：

- **源有、目标无的文件**：直接创建目录并整文件复制（哪怕是整块目录在目标都不存在，也照建照拷）。

  ```python
  import os, shutil

  s = os.path.join(src_base, rel.replace('/', os.sep))
  d = os.path.join(dst_base, rel.replace('/', os.sep))
  os.makedirs(os.path.dirname(d), exist_ok=True)
  shutil.copy2(s, d)
  ```

- **修改文件（`M`）**：源是 release 包、目标是配对的 gitee release 仓时，**直接整文件覆盖复制**（release 包就是发布产物，目标侧不存在需要保留的独立改动），比 patch 更省事也更可靠。

  只有两侧都是 git 仓库、且目标侧有源提交未触及的自有改动时，才用 patch 只合并差异：

  ```powershell
  git -C <src> diff <commit>^ <commit> -- <源相对路径> > patch.diff
  git -C <target> apply -pN patch.diff     # -pN 去掉源路径多出的目录层级
  ```

  冲突或上下文不匹配时不强行覆盖：提示用户，或退回逐段用 Edit 合并。

- **二进制文件**（png/lib/gif/xlsx 等）：整文件字节级覆盖复制。

覆盖前先比对内容（如 MD5），**只对确实不同或目标不存在的文件动手**；相同的跳过，避免无意义的改动噪声。

### Step 7：验证并汇报

复制完后**逐文件比对源/目标 MD5**，确认全部一致；再用 `git status` 核对落盘结果：

```powershell
git -C <target> status --short
git -C <target> status --short --untracked-files=all   # 展开未跟踪目录，核对新增文件数
```

汇报覆盖更新了哪些、新增了哪些、按模块分类的统计、使用的路径映射，并提醒改动未提交、等待 review。

## 同步规则

只适用于 **W6069**（MCU + MDM）和 **FY6718**（只有 MCU）两个项目。其他项目、其他仓库一律不处理。

### W6069

| 仓库 | 仓库名 | gitee 侧 | gerrit 侧 |
|------|--------|----------|-----------|
| MCU | `SF32LB56x` | `E:\work\gitee\w6069\mcu` | `E:\work\gerrit\SF32LB56x` |
| MDM | `ASR1603` | `E:\work\gitee\w6069\mdm` | `E:\work\gerrit\ASR1603` |

注意：`E:\work\gitee\w6069\mcu` 与 `mdm` 都**不是**独立 git 仓库，它们同属 git 根 `E:\work\gitee\w6069`。与 gerrit 单仓库（`SF32LB56x` / `ASR1603`）对应时，路径关系是：

- MCU：`E:\work\gitee\w6069\mcu\...` ↔ `E:\work\gerrit\SF32LB56x\...`
- MDM：`E:\work\gitee\w6069\mdm\...` ↔ `E:\work\gerrit\ASR1603\...`

即 gitee 侧要去掉一层 `mcu\` 或 `mdm\` 前缀再映射到 gerrit 仓库根。

### FY6718

只有 **MCU** 仓库需要同步，MDM 不参与。

| 仓库 | 仓库名 | gitee 侧 | gerrit 侧 |
|------|--------|----------|-----------|
| MCU | `SF32LB52x` | `E:\work\gitee\fy6718` | `E:\work\gerrit\SF32LB52x` |
| MDM | 不需要同步 | — | — |

`E:\work\gitee\fy6718` 自身即 git 根（含 `sdk`、`solution` 等子目录），与 gerrit 仓库 `SF32LB52x` 根直接对应，无需去前缀。

同步时源常是解压出来的发布包 `E:\work\gitee\fy6718_SDK_<日期>`（非 git 仓库），与目标 `E:\work\gitee\fy6718` 结构一致，相对路径 **1:1 对应**；范围则回到 gerrit 仓 `E:\work\gerrit\SF32LB52x` 查（见 Step 2）。

### 同步方向

**每次同步只做单向，方向由用户当次指定**（gitee→gerrit 或 gerrit→gitee）。用户没指明方向时必须先问，不要自行推断，也不要顺手做反向同步。

### 仓库识别（remote 对照）

| 本地路径 | remote |
|----------|--------|
| `E:\work\gitee\w6069` | `git@gitee.com:dal_liuxun/w6069.git` |
| `E:\work\gitee\fy6718` | `git@gitee.com:dal_liuxun/fy6718.git` |
| `E:\work\gerrit\SF32LB56x` | `ssh://leon@DAL-Server-2:29418/SF32LB56x` |
| `E:\work\gerrit\ASR1603` | `ssh://leon@DAL-Server-2:29418/ASR1603` |
| `E:\work\gerrit\SF32LB52x` | `ssh://leon@DAL-Server-2:29418/SF32LB52x` |

### 不允许源码同步的范围（库文件）

MCU 仓库发布到 gitee 用的是 release 包：脚本会把下列源码目录里的 `.c` / `.h` 删掉，改为只提供 `.lib` 与少数保留的头文件。**范围以各仓库当前的 `dal_sdk_release.py` 为准**，下表只是快照，同步前先重新读脚本确认。

#### SF32LB56x（W6069 MCU）

脚本位置：`tools\dal_build_tool\dal_sdk_release.py`（仓库根下 `w6069_sdk_release.bat` 调用，project 为 `fosun_w6069_v2`）

| 类别 | 内容 |
|------|------|
| 不提供源码的目录 | `sdk\customer\peripherals\asr1603`、`watch\service\dal_mdm_service`、`watch\customize\dal_algo\dal_raise_hands` |
| 例外保留的头文件 | `sdk\customer\peripherals\asr1603\asr1603.h`、`watch\service\dal_mdm_service\` 下的 `dal_mdm_service_api.h`、`dal_ril.h`、`dal_modem_net_handle.h` |
| 库文件 | `dal_asr1603.lib`、`solution_mdm_service.lib` |
| 库文件落点 | `watch\lib\dal_lib` |

#### SF32LB52x（FY6718 MCU）

脚本位置：`solution\tools\dal_build_tool\dal_sdk_release.py`

| 类别 | 内容 |
|------|------|
| 不提供源码的目录 | `sdk\customer\peripherals\modem\ec718`、`solution\framework\service\srv_dal\modem`、`solution\components\sensor\dal_algo\dal_raise_hands` |
| 例外保留的头文件 | `ec718.h`、`dal_mdm_service_api.h`、`dal_mdm_service_msg.h`、`dal_mdm_service_timer.h`、`dal_mdm_period_meas_handle.h`、`dal_mdm_peripheral_handle.h` |
| 库文件 | `dal_ec718.lib`、`dal_mdm_service.lib` |
| 库文件落点 | `solution\framework\dal_lib` |

不在以上范围的改动，按正常源码同步。

### 同步范围

**每次同步都必须由用户指定范围，只同步落在范围内的文件**；用户没给范围就先问，不要自行扩大或缩小。

确定范围的方式：到**源仓库**里用 git 查出有哪些改动文件，同步清单就以这批文件为准；**源侧不是 git 仓库时（release 包）改到配对仓库查**，详见 Step 2。

```powershell
git -C <src> status --short                          # 工作区未提交的改动
git -C <src> diff --name-status                      # 改动文件清单（未提交）
git -C <src> diff --name-status <commit>^ <commit>   # 某个 commit 的改动文件
git -C <src> diff --name-status <c1> <c2>            # 一段 commit 区间的改动文件
git -C <src> log --oneline -N                        # 先看有哪些提交，再决定区间
```

范围可以是以下任一种（或组合）：

- 源仓库当前工作区未提交的改动文件
- 某个 commit，或一段 commit 区间的改动文件
- 用户直接指定的若干文件 / 目录

范围确定后，**只同步这些文件**；清单外的文件一律不动，也不要顺手带上相邻改动。

### 固定不同步的文件

以下文件**即使落在范围内也不同步**：

- **语言资源 json**：`solution\examples\watch\resource\langs\en_us.json` 一律不同步（用户确认）。该文件由 `multi_language_table.xlsx` 生成 / 单独维护，只同步 `multi_language_table.xlsx`。
- **构建指纹与缓存**：`build_info.h`（记录 SOLUTION_BUILD 哈希，只对应当次构建的 commit）、`__pycache__\*.pyc` 等构建产物。

## 注意事项 / 坑

1. **并行改同一文件会互相覆盖**：同一文件的多次修改串行处理，或直接整文件复制。
2. **二进制文件**必须字节复制，不能用文本方式读写。
3. **Windows 路径分隔符**：用 `os.path.join` 配合 `rel.replace('/', os.sep)`。
4. **仓库根 ≠ 用户给的目录**：`git -C <dir>` 会向上查找仓库根，`git show` 路径相对仓库根。
5. **两侧仓库历史不同**：不要 cherry-pick，除非确认能 fast-forward。
6. **源里 `A`（新增）的文件在目标可能已存在**：复制后表现为 `M`（覆盖），属正常。
7. **范围取 gerrit、内容取源包**：范围用配对 gerrit 仓的 git 记录确定，文件内容从 release 包里取。**不要把 gerrit 的源码直接拷进 gitee**——gitee 侧是 release 形态，源码与库的取舍由源包决定。
8. **源包是 release 形态**：`<project>_SDK_<日期>` 里 `dal_src_dirs` 覆盖的源码已被删除，比对这些路径会报「源不存在」，属正常，需转成 `lib_dir` 路径处理（见 Step 3）。

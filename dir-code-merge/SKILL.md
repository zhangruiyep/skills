---
name: "dir-code-merge"
description: "在两个目录/文件夹/代码仓库之间合并或同步代码（源目录 → 目标目录）。当用户要求合并代码、同步代码、把某目录/仓库的最近改动搬到另一个目录，或 merge/sync code between two directories 时触发。"
---

# 目录代码合并 (Directory Code Merge)

把**一个目录/仓库**里的代码改动，合并/同步到**另一个目录/项目**。

## 触发条件

满足以下任一即生效：

- 用户要求「把 A 目录的代码合并到 B 项目」
- 用户要求「把某仓库最近一次 git 修改的文件内容同步/合并到本项目」
- 用户提到 merge / sync / 合并 / 同步 / 搬代码，且涉及两个目录、文件夹或仓库

## 核心原则

1. **先确认，再动手**：合并范围、路径映射、覆盖风险，拿不准就先问用户。
2. **只做文件级复制/覆盖**：两个仓库历史不同时，不做 git 三方合并或 cherry-pick。
3. **区分文本与二进制**：文本修改文件用 `git diff` 只合并差异；新增文件与二进制文件（png/gif/xlsx/json 等）用字节级复制。
4. **只动本次涉及的文件**：不碰目标里其他文件。
5. **不自动 commit**：改动留在工作区，交给用户 review。

## 工作流程

### Step 1：明确源目录与目标目录

- 源目录（source）：改动来源
- 目标目录（target）：改动要合入的项目
- 用户只给一个目录时，追问另一个。

### Step 2：确定合并范围

先判断源目录是不是 git 仓库：

```powershell
git -C <src> rev-parse --show-toplevel   # 确认仓库根（可能不是用户给的目录本身）
git -C <src> status                      # 是否有未提交改动
git -C <src> log --oneline -N            # 最近提交
git -C <src> show --name-status <commit> --no-renames   # 某次提交改了哪些文件
```

`--name-status` 里 `A`=新增、`M`=修改、`D`=删除。

常见范围：最近一次提交、最近 N 次提交、指定提交、未提交的工作区改动、用户指定的若干文件。

### Step 3：确定路径映射

源与目标目录结构通常不完全一致，需要建立映射：

- 最常见：两边共享同一个子目录前缀。例如源 `mcu/watch/...` → 目标 `watch/...`（去掉 `mcu/` 前缀）。
- 用 `LS` / `Glob` 对比两边目录结构确认映射。
- 注意：`git show` 里的路径是**相对仓库根**的，要据此拼出真实源文件绝对路径。
- 拿不准就问用户。

### Step 4：检查目标目录状态（防丢失）

```powershell
git -C <target> status --short
```

确认目标没有未提交的重要改动会被覆盖；有则先提醒用户。

### Step 5：合并文件（按改动类型区分）

先按 `git show --name-status <commit>` 的结果，把文件分成两类处理。

#### A. 新增文件（状态 `A`）

源里没有修改记录、本次新增的文件，直接整文件复制到目标：

```python
import os, shutil

s = os.path.join(src_base, rel.replace('/', os.sep))
d = os.path.join(dst_base, rel.replace('/', os.sep))
os.makedirs(os.path.dirname(d), exist_ok=True)
shutil.copy2(s, d)
```

#### B. 修改文件（状态 `M`）

按文件用 `git diff` 生成 patch，再 apply 到目标目录里对应的文件上，**只合并差异、不要整体覆盖**，以保留目标里源提交未触及的改动。

1. 对每个修改文件生成 patch：

   ```powershell
   git -C <src> diff <commit>^ <commit> -- <源相对路径> > patch.diff
   # 未提交的工作区改动则用：
   git -C <src> diff -- <源相对路径> > patch.diff
   ```

2. 把 patch 应用到目标里对应的文件（**注意路径对应关系**，源/目标路径前缀不同时要调整）：

   ```powershell
   git -C <target> apply -pN patch.diff
   ```

   - `-pN`：去掉 patch 路径前 N 级目录。源路径比目标多几级就去掉几级，例如源 `a/mcu/watch/...` → 目标 `watch/...`，用 `-p2`。
   - 先确认每个源文件在目标里的对应路径，再决定用 `-pN` 或 `--directory=`。

3. 应用冲突时不要强行覆盖，提示用户手工解决；若上下文不匹配导致 `git apply` 失败，退回逐处用 Edit 合并差异块。

#### C. 二进制文件（git 无法查看差异）

png/gif/xlsx/json 等 git 无法生成文本 diff 的文件，直接整文件覆盖复制到目标（同 A 的复制方式）。

### Step 6：验证并汇报

```powershell
git -C <target> status --short
```

- `M` = 被覆盖修改；`??` = 新增文件
- 汇总：覆盖更新了哪些、新增了哪些、路径映射规则
- 提醒：改动未提交，等待 review

## 注意事项 / 坑

1. **并行 Edit 会互相覆盖**：对同一文件的多次修改要串行，或直接整文件复制。
2. **二进制文件**（png/gif/xlsx/json 等）必须字节复制，不能用 Read/Write 文本方式处理。
3. **Windows 路径分隔符**：`os.path.join` + `rel.replace('/', os.sep)` 最稳妥。
4. **仓库根 ≠ 用户给的目录**：`git -C <dir>` 会向上查找仓库根，`git show` 路径相对仓库根。
5. **两仓库历史不同**：不要尝试 cherry-pick，除非确认能 fast-forward。
6. **源里 `A`（新增）的文件在目标可能已存在**：复制后表现为 `M`（覆盖），属正常现象。

## 示例

把 `E:\work\gitee\w6069` 最新提交 `2619066d` 改动的文件合并到 `e:\work\gerrit\SF32LB56x`：

1. `git -C "E:\work\gitee\w6069" show --name-status 2619066d` 得到 47 个文件。
2. 映射：源 `mcu/watch/X` → 目标 `watch/X`（去掉 `mcu/` 前缀）。
3. 用 Python 脚本把这 47 个文件从 `E:\work\gitee\w6069\mcu\watch\...` 复制到 `e:\work\gerrit\SF32LB56x\watch\...`。
4. `git status` 验证：17 个 `M` + 30 个新增文件。

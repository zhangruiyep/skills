---
name: release-version-bump
description: 递增软件版本号，用于准备 release 版本，仅对 ASR1603、EC718、SF32LB52x、SF32LB56x 四个仓库生效。当用户要求升版本、版本号递增、准备/发布 release 版本时触发。
---

# 版本号递增（Release 版本准备）

把指定仓库的软件版本号递增一级，然后提交并推送，用于准备 release。只作用于下列仓库，其它仓库一律不生效。

## 触发条件

用户要求升版本 / 版本号递增 / 准备 release 版本 / 发版本，且目标仓库命中以下任一关键词：

`ASR1603`、`EC718`、`SF32LB52x`、`SF32LB56x`

必须同时满足「要求升版本」与「仓库命中关键词」才生效。以下情况一律不生效：

- 目标仓库不在上述四个之内
- 只是查版本号 / 看版本号，不要求递增

## 核心原则

1. **先确认再动手**：仓库路径、版本号定义位置、递增规则拿不准就先问用户。
2. **只动指定仓库**：不推断、不扩展到其它仓库。
3. **改动最小**：只改版本号相关字段，不顺手改其它内容。
4. **拉取和推送用现成脚本**：`~/bin/gerrit_pull.sh` 和 `~/bin/gerrit_push.sh`，不自己拼 `git pull` / `git push`。
5. **提交与推送按本技能执行**：提交信息沿用仓库历史版本号提交的写法；推送走 Gerrit 评审（`refs/for`），**禁止 force push**。

## 工作流程

### 拉取 / 推送脚本

用户已提供两个 Git Bash 脚本，位于 `~/bin`（`C:\Users\zhang\bin`）：

| 脚本 | 内容 |
|------|------|
| `~/bin/gerrit_pull.sh` | `git stash` → `git pull --rebase` → `git stash pop` |
| `~/bin/gerrit_push.sh` | `git push origin HEAD:refs/for/<当前分支>`，即推到 Gerrit 评审队列 |

脚本**按当前目录（cwd）操作仓库，不支持 `-C`**，所以要先 `cd` 到仓库根。从 PowerShell 调 Git Bash：

```powershell
& "D:\Program Files\Git\bin\bash.exe" -lc "cd /e/work/gerrit/ASR1603 && ~/bin/gerrit_pull.sh"
& "D:\Program Files\Git\bin\bash.exe" -lc "cd /e/work/gerrit/ASR1603 && ~/bin/gerrit_push.sh"
```

路径写成 Git Bash 形式：`E:\work\gerrit\ASR1603` → `/e/work/gerrit/ASR1603`。

### Step 1：从远端更新代码

先确认仓库状态，再用 `gerrit_pull.sh` 拉取：

```powershell
git -C <repo> rev-parse --show-toplevel          # 确认仓库根
git -C <repo> rev-parse --abbrev-ref HEAD        # 当前分支
git -C <repo> status --short                     # 工作区是否干净
```

```powershell
& "D:\Program Files\Git\bin\bash.exe" -lc "cd <repo 的 Git Bash 路径> && ~/bin/gerrit_pull.sh"
```

- 拉取时版本号还没开始改，工作区应保持干净；脚本里的 `stash` / `stash pop` 只是兜底。若本来就有未提交改动，**先停下告诉用户**，不要擅自 stash / reset。
- 脚本用的是 `pull --rebase`：本地未推送的提交会被 rebase 到远端最新之上。出现 rebase 冲突或 `stash pop` 冲突时**停下让用户处理**，绝不自行 `reset` / `--force`。
- 拉取后确认本地已是最新：`git log --oneline -3`。

### Step 2：读取当前版本号并递增

**优先用仓库自带的版本脚本，不要手改版本号文件。** 先在仓库里 `Glob` 搜 `**/dal_ver_inc.py`（以及同目录的 `dal_config.py`、`dal_ver_set.py`），有就直接调用；下表只是快照，可能已过期。

| 仓库 | 脚本 | 调用方式（cwd 见备注） |
|------|------|------------------------|
| ASR1603 | `SDK\DM_LTEGSM_SDK_1_011_110\dal\py\dal_ver_inc.py` | 在 `SDK\DM_LTEGSM_SDK_1_011_110` 下执行 `python dal\py\dal_ver_inc.py w6069` |
| EC718 | `dal_tools\dal_ver_inc.py` | 在**仓库根**执行 `python dal_tools\dal_ver_inc.py <project> <version_macro>` |
| SF32LB52x | `solution\tools\dal_build_tool\dal_ver_inc.py` | 在**仓库根**执行 `python solution\tools\dal_build_tool\dal_ver_inc.py <project>` |
| SF32LB56x | `tools\dal_build_tool\dal_ver_inc.py` | 在**仓库根**执行 `python tools\dal_build_tool\dal_ver_inc.py <project>` |

各仓库改动的文件与版本宏：

| 仓库 | 版本号文件与宏 | 同步的 OTA 版本 |
|------|----------------|-----------------|
| ASR1603 | `dal\project\dal_conf_w6069.h` 的 `DALWATCH_SV_CODE`（形式 `"V9.4.9"`） | `pcac\fota\src\cm\cmiot_src\oneos_config.h` 的 `CMIOT_FIRMWARE_VERSION` |
| EC718 | `PLAT\project\DW6S\ap\apps\<PROJECT>\common\inc\dal_proj_conf.h` 的 `DAL_FY6718_SV_CODE` / `DAL_DW6S_SV_CODE` / `DAL_SY12_SV_CODE` | `PLAT\middleware\thirdparty\ota\oneos_<proj>_config.h` |
| SF32LB52x | `solution\examples\watch\project\<project>\hcpu\.config` 的 `CONFIG_DAL_SW_VER` | 脚本本身会处理 |
| SF32LB56x | `watch\sifli\project\<project>\hcpu\.config` 的 `CONFIG_DAL_SW_VER` | 脚本本身会处理（同步 `watch\component\oneos\oneos_<project>_config.h` 的 `CMIOT_FIRMWARE_VERSION`；该头文件不存在时脚本会跳过并告警） |

递增规则（版本号格式一般是 `[字母]X.Y.Z`）：

- **ASR1603（w6069）**：最低位 **+2**，满 10 按十进制进位。
- **EC718 / SF32LB52x**：最低位 **+1**，满 10 进位。
- **SF32LB56x**：`dal_ver_inc.py` 内固定——Project 为 `fosun_w6069` / `fosun_w6069_v2` 时最低位 **+2**，其余 Project **+1**；满 10 进位，**最高位不再向上进位、允许超过 9 并用两位数表示**（如 `J9.9.9` → `J10.0.1`）。版本格式为单字母前缀 `[字母]X.Y.Z`（如 `J9.4.9` / `E8.8.6`），也兼容无字母的 `X.Y.Z`（如 `1.0.8`）。
- 字母前缀（如 `V` / `J` / `E`）一般不变；要改系列字母时必须问用户。

EC718 与 SF32LB52x 另外自带一键包装脚本 `dal_ver_inc.bat`（仓库根，参数分别是 `<FY6718|DW6S|SY12>` 和 `<project>`），它会自己完成 stash/pull/rebase/commit/push。若直接用 bat，就**不要再跑** `gerrit_pull.sh` / `gerrit_push.sh`，提交信息也由 bat 固定。

### Step 3：检查版本号并本地提交

- 回读改动，确认新版本号写对了（位数、格式、前缀、原有其它字段都没被破坏），且 OTA 版本已同步一致。
- 汇报「原版本号 → 新版本号」与改动文件，再提交。
- 提交信息沿用仓库历史版本号提交的写法（先 `git log --oneline -20 --grep=version -i` 看历史），**不要自创格式**。已知写法：
  - SF32LB56x：`w6069-<系列字母小写>: modify version to <版本号>`，如 `w6069-j: modify version to J9.4.9`
  - EC718（bat）：`Modify <target> version`
  - SF32LB52x（bat）：`Modify version for project <project> to <ver>`
  - ASR1603：按仓库 `git log` 里的历史版本号提交写法

```powershell
git -C <repo> diff                                # 确认只改了版本号相关文件
git -C <repo> add <版本号相关文件>
git -C <repo> commit -m "<沿用历史写法的提交信息>"
```

### Step 4：推送到远程（Gerrit 评审）

```powershell
& "D:\Program Files\Git\bin\bash.exe" -lc "cd <repo 的 Git Bash 路径> && ~/bin/gerrit_push.sh"
```

- 脚本内部是 `git push origin HEAD:refs/for/<当前分支>`，即**推到 Gerrit 评审队列（`refs/for`）**，不是直接推分支；评审通过后才落到分支上。
- 推送前确认：当前分支正确、工作区只剩版本号提交、没有多余改动混进来。
- 脚本不带 `--force`，**不要自行改成 force push**。
- 推送成功后 Gerrit 会返回 change 链接（若有），一并汇报给用户。

### Step 5：汇报

汇报：仓库、分支、原版本号 → 新版本号、改动的文件、commit 哈希、Gerrit change 链接与推送结果。

## 注意事项 / 坑

1. 仓库不在白名单内时**不要执行**，直接说明该技能不适用。
2. 脚本找不到、或跳到脚本没覆盖的仓库时，**先与用户确认递增规则**（第几位 +N、是否进位清零、字母前缀怎么变），不要臆测。
3. 一个仓库可能有多个版本号定义位置（如 MCU/MDM 各一处），确认是否都要改。
4. 推送前先确认分支正确，别把版本号提交推到错误分支。
5. 版本号提交一般是独立提交，**不要和其它改动混在一个 commit 里**。
6. 两个脚本针对 **gerrit 侧仓库**（`E:\work\gerrit\...`）；`gerrit_push.sh` 推的是 `refs/for` 评审队列，不是直接落分支，别误以为推完就发布完成。
7. 脚本必须在 **Git Bash** 里跑（PowerShell 直接执行 `.sh` 无效），且要 `cd` 到仓库根，脚本不接受路径参数。
8. 版本脚本用 `os.getcwd()` 定位版本文件，**cwd 必须是上表指定的目录**（ASR1603 是 `SDK\DM_LTEGSM_SDK_1_011_110`，其余是仓库根），不能换目录或复制脚本单独跑；`dal_config.py` 与脚本同目录，靠 Python 自动把脚本目录加进 `sys.path` 才能 import。
9. 脚本执行完会自己打印「旧版本 → 新版本」，用这句输出核对结果；不要凭猜写提交信息里的版本号。


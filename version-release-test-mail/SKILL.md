---
name: version-release-test-mail
description: 发送版本转测试通知邮件（版本已发布，请安排测试），走企业微信邮件接口并含正文模板与踩坑清单。当用户要求发版本发布通知、版本转测试邮件、发版通知测试组时触发。不负责升版本号、写 Release Notes 或日程会议邀约。
---

# 版本转测试邮件（企业微信）

版本已发布后，通知测试对接人及相关同事安排测试。产出：一封通过企业微信邮件接口发出的通知邮件。

## 适用与不适用

- **适用**：版本已发布 → 发通知给测试/相关同事，安排测试（"发版本转测试邮件""版本发布了通知测试""发版通知"）。
- **不适用**：
  - 递增版本号 → `release-version-bump`
  - 更新 ReleaseNotes → `release-notes-updater`
  - 日程邀约 / 会议邮件 → `wecomcli-calendar` / `wecomcli-meeting`

## 前置必读

执行任何 `wecom-cli` 命令前，先读这两个技能的文档，禁止凭记忆拼参数：

1. `wecomcli-shared` — 公共前置检查（CLI 安装、版本 ≥ 1.1.0、授权状态）
2. `wecomcli-email` 及其 `references/send-mail.md` — 发送接口参数与流程

```powershell
wecom-cli --version; wecom-cli auth show --status
```

输出 `authorized` 且版本 ≥ 1.1.0 才继续。

## 邮件要素

先收集齐以下信息，缺什么就用自然语言追问用户，**禁止猜测或编造**：

| 要素 | 说明 |
|------|------|
| 项目名 / 版本 | 如 FY6718 / R9.7.2；**版本本身就由字母、数字、小数点组成（如 `R9.7.2`），原样使用，不要自行加前缀**；同时确认「版本号」字段（如 `FY6718_J9.7.2`）与发布日期 |
| 收件人 | 一般是对接测试的人，如 Iris |
| 抄送 | 相关同事；`fosun.com` 那 5 位**仅 W6069 / FY6718 项目才加** |
| 测试对接人称呼 | 正文第一行 `Hi <名字>,` |
| 固件路径 | MCU 与 MDM 两条 UNC 路径；**发送前必须确认路径真实存在** |
| 主要修改点 | **从固件包里的 `ReleaseNotes.md` 读取**（一般在 MCU 版本固件包目录下），按「修改的功能或需求」「合入复云代码」两组整理 |

### 默认收件人 / 抄送（每次发送前仍需与用户核对一次）

收件人：

- Iris `iris@deepailink.com`
- 刘坪崎 `liupingqi@deepailink.com`
- monica `monica@deepailink.com`

抄送（`deepailink.com`，所有项目都抄送）：

- Luke `luke@deepailink.com`
- zoomonkey `zoomonkey@deepailink.com`
- Jhon `liuzhanwang@deepailink.com`
- liuxun `liuxun@deepailink.com`
- Leon `leon@deepailink.com`
- 孟子天宇 `mengzitianyu@deepailink.com`
- 常佳伟 `changjiawei@deepailink.com`
- 吕豪 `lvhao@deepailink.com`
- peter `peter@deepailink.com`

抄送（`fosun.com`，**只有 W6069 和 FY6718 两个项目才抄送**，其他项目一律不加）：

- fenggx `fenggx@fosun.com`
- wangwl1 `wangwl1@fosun.com`
- v_xinzb `v_xinzb@fosun.com`
- v_xuhs `v_xuhs@fosun.com`
- v_masj `v_masj@fosun.com`

> 项目名不是 W6069 / FY6718 时，抄送里**不要出现任何 `@fosun.com` 地址**，只抄送 `deepailink.com` 的名单。

## 主题与正文模板

### 主题

```
[<项目名>] <版本>版本发布，请安排测试
```

例：`[FY6718] R9.7.2版本发布，请安排测试`

### 正文模板

```markdown
Hi <测试对接人>,

<项目名> <版本>版本已发布，请安排测试。

**MCU版本：**
`\\192.168.1.201\shared\firmware\<项目名>\MCU\<项目名>_V<版本>`

**MDM版本：**
`\\192.168.1.201\shared\firmware\<项目名>\MDM\<项目名>_V<版本>`

----------

- 版本号：<项目名>_<版本号后缀>
- 时间：<YYYY-MM-DD>

- 主要修改点：
  - 修改的功能或需求：
    1. …
  - 合入复云代码：
    1. …
```

要点：

- UNC 路径必须包在反引号里，否则 Markdown 会吞掉反斜杠。
- **版本原样使用**：版本由字母、数字、小数点组成（如 `R9.7.2`），直接把 `<版本>` 替换成用户给的值，不要自行加前缀或改字母。
- 「版本号」字段（`FY6718_J9.7.2`）与主题里的版本（`R9.7.2`）不同，按用户给的原文照抄，不确定就问用户。
- 修改点按「修改的功能或需求」「合入复云代码」两组分列，逐条编号。
- **修改点来源**：从 MCU 固件包里的 `ReleaseNotes.md` 读取（一般在 MCU 版本固件包目录下，即 `<MCU 路径>\ReleaseNotes.md`），按其中内容整理成上面两组；读取失败或文件缺失就问用户要，不要编造。

### 实例（FY6718 R9.7.2，可作为格式参考）

> 下面的「主要修改点」就是从该版本 MCU 固件包里的 `ReleaseNotes.md` 整理出来的。

```markdown
Hi Iris,

FY6718 R9.7.2版本已发布，请安排测试。

**MCU版本：**
`\\192.168.1.201\shared\firmware\FY6718\MCU\FY6718_VR9.7.2`

**MDM版本：**
`\\192.168.1.201\shared\firmware\FY6718\MDM\FY6718_VR9.7.2`

----------

- 版本号：FY6718_J9.7.2
- 时间：2026-09-30

- 主要修改点：
  - 修改的功能或需求：
    1. UI：新增瑞金渠道切换表盘，修改 03 与瑞金表盘名称。
    2. UI：更新多语言表，新增瑞金医院小程序模块及配套图标资源。
    3. ECG：新增瑞金 AI 心电（AI-HEART）主流程界面。
    4. MDM：协议新增 AI 心电任务下发、结果上报与状态处理。
  - 合入复云代码：
    1. 优化手表绑定机制，防止刚开机渠道和实际不一致造成误绑定；
    2. 新增震动接口，支持指定时长和次数，并调整健康预警震动模式；
    3. 调整开机动画调用接口，优化宏控制播放哪个开机动画；
    4. 删除无效函数声明；
    5. 瑞金渠道化定制开发；
    6. 更新消息通知中心通知类图标，删除无效图标。
```

## 发送流程

1. **收集要素**：按上表逐项确认，缺项追问用户。
2. **确认固件路径存在**：用 `Test-Path` 校验 MCU / MDM 两条 UNC 路径都返回 `True` 才继续；任一条为 `False` 先跟用户核对项目名/版本再发。
3. **读取修改点**：读 MCU 固件包目录下的 `ReleaseNotes.md`，按「修改的功能或需求」「合入复云代码」两组提取本次版本的修改点。
4. **写正文文件**：用 Write 把正文写成 `.md`（如 `%TEMP%\mail_body_<项目>_<版本>.md`）。
5. **展示预览**：把主题、收件人、抄送、正文在对话里完整呈现给用户确认。
6. **dry-run 校验**：加 `--dry-run` 跑一次，确认 payload 里 to / cc / subject / file_path 都正确。
7. **正式发送**：去掉 `--dry-run` 发送。
8. **回报**：告知"邮件已成功发送"，只列收件人与主题。

```powershell
Test-Path '\\192.168.1.201\shared\firmware\<项目名>\MCU\<项目名>_V<版本>'
Test-Path '\\192.168.1.201\shared\firmware\<项目名>\MDM\<项目名>_V<版本>'
Test-Path '\\192.168.1.201\shared\firmware\<项目名>\MCU\<项目名>_V<版本>\ReleaseNotes.md'
Get-Content -Raw '\\192.168.1.201\shared\firmware\<项目名>\MCU\<项目名>_V<版本>\ReleaseNotes.md'
```

优先用 Read 工具直接读 `ReleaseNotes.md`；读不到时再用上面的 `Get-Content -Raw`。

### Windows PowerShell 传 JSON 的固定写法（关键）

必须使用 `--%` + **外层双引号包裹整个 JSON** + **内层每个引号写成 `\"`**：

```powershell
wecom-cli mail send --% --json "{\"to\":{\"emails\":[\"a@x.com\"]},\"cc\":{\"emails\":[\"b@x.com\"]},\"subject\":\"[FY6718] R9.7.2版本发布，请安排测试\",\"file_path\":\"C:\\Users\\zhang\\AppData\\Local\\Temp\\mail_body.md\",\"content_type\":\"markdown\"}"
```

- 不加 `--%`：PowerShell 会吞掉内层双引号，报 `不是合法的 JSON` 或 `JSON repair error`。
- 不套外层双引号：主题里的空格会把参数截断，报 `unexpected argument`。
- JSON 里路径的反斜杠写 `\\`；命令必须一行写完。
- 非 PowerShell 环境（bash/zsh）用常规单引号包 JSON 即可，不适用上面的写法。

## 踩坑清单（务必遵守）

1. **不支持定时发送、不支持存草稿**。接口只能立即发送；需要定时或草稿，只能在企业微信客户端手动操作。不要答应"排期发送"。
2. **发件人固定显示为机器人名称**（如"Leon的机器人"），并代授权用户执行。接口**没有指定发件人的参数**，改不了；若必须用本人邮箱发，只能改到客户端手动发。
3. **沙箱必须放行 `C:\Users\zhang\.config\wecom\cache`**。该目录被拦截会导致 CLI 写临时文件失败并重试，**曾把同一封邮件重复投递 2 次**。发送前先确认沙箱已放行；若命令输出里出现 `hit restricted` 指向该目录，**立即停止重试**并提示用户自行核对是否重复发送（接口的撤回、搜索邮件能力均不可用）。
4. **发送前必须先 dry-run**，确认 payload 正确后再正式发送，避免参数错误导致反复执行。
5. **正文一律走 `file_path` + `.md`**，`content_type` 固定 `markdown`；不要用 `--content` 直接塞正文。
6. **发送前必须先展示预览**（主题 / 收件人 / 抄送 / 正文）。
7. **内部字段禁止外露**：`mail_id` / `media_id` / `userid` / `errcode` / `next_cursor` 以及 `wecom-cli` 命令本身，都不能出现在给用户的回复中。
8. **附件**：有附件时直接给 `file_path`，CLI 自动上传，不需要先手动上传取 media_id；单封邮件总大小（正文 + 附件）不超过 50MB。
9. **业务约束**：`撤回已发送邮件`、`保存草稿`、`标记已读未读`、`邮件标签写操作` 均不支持，需要时告知用户到客户端处理。

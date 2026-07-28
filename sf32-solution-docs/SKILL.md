---
name: "sf32-solution-docs"
description: "SiFli Solution 2.0 编程指南。当项目使用 SF32 系列芯片（SF32LB52x/55x/56x/58x）时自动生效。提供文档站点导航，AI 通过 WebFetch 在线获取最新文档内容辅助开发。"
---

# SiFli Solution 2.0 编程指南

## 触发条件

当项目使用思澈科技（SiFli）SF32 系列芯片时，本 Skill 自动生效。包括但不限于：
- SF32LB52x
- SF32LB55x
- SF32LB56x
- SF32LB58x

## 核心原则

**文档内容必须在线同步，不得硬编码文档细节。** 本 Skill 仅提供文档站点导航索引，AI 在回答任何具体开发问题时，必须通过 `WebFetch` 工具实时抓取对应文档页面的最新内容，确保信息准确。

## 文档站点

- **文档根地址**：`https://docs.sifli.com/projects/solution/`
- **SDK 文档（辅助参考）**：`https://docs.sifli.com/projects/sdk/latest/en/sf32lb52x/`

---

## 文档导航结构

以下是 Solution 文档的完整章节导航。AI 应根据用户问题，定位到对应文档页面并通过 `WebFetch` 获取内容。

### 0. SiFli Solution 介绍
- 软件架构（RT-Thread + LVGL v8.3）、核心优势、支持的核心功能、示例应用总览
- `https://docs.sifli.com/projects/solution/0.introduction/index.html`

### 1. 快速入门
| 主题 | 文档 URL |
|---|---|
| 开发环境配置 | `https://docs.sifli.com/projects/solution/1.get-started/development_env.html` |
| 运行第一个项目 | `https://docs.sifli.com/projects/solution/1.get-started/first_run.html` |
| PC 仿真环境 | `https://docs.sifli.com/projects/solution/1.get-started/simu.html` |
| Solution 目录结构 | `https://docs.sifli.com/projects/solution/1.get-started/directory.html` |
| 方案选型 | `https://docs.sifli.com/projects/solution/1.get-started/solution.html` |
| 线程及线程消息通信 | `https://docs.sifli.com/projects/solution/1.get-started/thread.html` |
| 内存管理 | `https://docs.sifli.com/projects/solution/1.get-started/memory.html` |
| 字体 | `https://docs.sifli.com/projects/solution/1.get-started/font.html` |
| 图片资源 | `https://docs.sifli.com/projects/solution/1.get-started/image.html` |
| 多语言支持 | `https://docs.sifli.com/projects/solution/1.get-started/multi_lang.html` |
| FLASH 分配 | `https://docs.sifli.com/projects/solution/1.get-started/flash_partition.html` |
| IPC（核间）通信 | `https://docs.sifli.com/projects/solution/1.get-started/interface.html` |
| FlashDB 与 NVM 存储 | `https://docs.sifli.com/projects/solution/1.get-started/flashdb.html` |
| MONKEY 测试 | `https://docs.sifli.com/projects/solution/1.get-started/monkey.html` |

### 2. 应用笔记

#### 2.1 UI / 应用
| 主题 | 文档 URL |
|---|---|
| 总览 | `https://docs.sifli.com/projects/solution/2.application-notes/ui/overall.html` |
| 应用 APP | `https://docs.sifli.com/projects/solution/2.application-notes/ui/app.html` |
| 平铺（TLV） | `https://docs.sifli.com/projects/solution/2.application-notes/ui/tlv.html` |
| 表盘（WF） | `https://docs.sifli.com/projects/solution/2.application-notes/ui/wf.html` |
| 弹窗（POPUP） | `https://docs.sifli.com/projects/solution/2.application-notes/ui/popup.html` |
| 息屏显示（AOD） | `https://docs.sifli.com/projects/solution/2.application-notes/ui/aod.html` |
| 场景动画 | `https://docs.sifli.com/projects/solution/2.application-notes/ui/scene_anim.html` |
| 设置 APP | `https://docs.sifli.com/projects/solution/2.application-notes/ui/setting.html` |
| 主菜单 APP | `https://docs.sifli.com/projects/solution/2.application-notes/ui/menu.html` |
| 数据刷新 | `https://docs.sifli.com/projects/solution/2.application-notes/ui/data.html` |
| 多张图片打包 | `https://docs.sifli.com/projects/solution/2.application-notes/ui/pkg_img_seq.html` |
| 资源目录结构 | `https://docs.sifli.com/projects/solution/2.application-notes/ui/resource.html` |
| 创建一个新 APP | `https://docs.sifli.com/projects/solution/2.application-notes/ui/new_app.html` |
| 删除外置 APP | `https://docs.sifli.com/projects/solution/2.application-notes/ui/del_app.html` |
| 设置默认 APP | `https://docs.sifli.com/projects/solution/2.application-notes/ui/default_app.html` |
| SENSOR 数据对接 | `https://docs.sifli.com/projects/solution/2.application-notes/ui/sensor_data.html` |
| 帧率 | `https://docs.sifli.com/projects/solution/2.application-notes/ui/fps.html` |

#### 2.2 BLE / BT
| 主题 | 文档 URL |
|---|---|
| BLE 通信 | `https://docs.sifli.com/projects/solution/2.application-notes/bluetooth/ble_comm.html` |
| BT 通信 | `https://docs.sifli.com/projects/solution/2.application-notes/bluetooth/bt_comm.html` |

#### 2.3 驱动
| 主题 | 文档 URL |
|---|---|
| I2C | `https://docs.sifli.com/projects/solution/2.application-notes/driver/i2c.html` |
| PWM | `https://docs.sifli.com/projects/solution/2.application-notes/driver/pwm.html` |
| UART | `https://docs.sifli.com/projects/solution/2.application-notes/driver/uart.html` |
| SPI | `https://docs.sifli.com/projects/solution/2.application-notes/driver/spi.html` |

#### 2.4 FLASH 分区 & 文件系统
| 主题 | 文档 URL |
|---|---|
| FLASH 分区 | `https://docs.sifli.com/projects/solution/2.application-notes/flash_map/index.html` |
| FAT 文件系统 | `https://docs.sifli.com/projects/solution/2.application-notes/file_system/FAT.html` |
| LittleFS 文件系统 | `https://docs.sifli.com/projects/solution/2.application-notes/file_system/LittleFS.html` |

#### 2.5 外设与传感器
| 主题 | 文档 URL |
|---|---|
| Sensor 应用指南 | `https://docs.sifli.com/projects/solution/2.application-notes/peripheral/sensor.html` |
| 运动传感器移植 | `https://docs.sifli.com/projects/solution/2.application-notes/peripheral/sensor_acc.html` |
| 心率传感器移植 | `https://docs.sifli.com/projects/solution/2.application-notes/peripheral/sensor_hr.html` |
| 地磁传感器移植 | `https://docs.sifli.com/projects/solution/2.application-notes/peripheral/sensor_mag.html` |
| 光感传感器移植 | `https://docs.sifli.com/projects/solution/2.application-notes/peripheral/sensor_asl.html` |
| 传感器数据融合 | `https://docs.sifli.com/projects/solution/2.application-notes/peripheral/sensor_data.html` |
| Sensor 消息处理 | `https://docs.sifli.com/projects/solution/2.application-notes/peripheral/sensor_msg.html` |
| 应用与 Sensor 通信 | `https://docs.sifli.com/projects/solution/2.application-notes/peripheral/sensor_config.html` |

#### 2.6 电源与电池
| 主题 | 文档 URL |
|---|---|
| 低功耗管理 | `https://docs.sifli.com/projects/solution/2.application-notes/power_manager/index.html` |
| 电池管理概述 | `https://docs.sifli.com/projects/solution/2.application-notes/battery/battery_overall.html` |
| 电池管理入门 | `https://docs.sifli.com/projects/solution/2.application-notes/battery/battery_primer.html` |
| 电池管理进阶 | `https://docs.sifli.com/projects/solution/2.application-notes/battery/battery_advanced.html` |

#### 2.7 OTA 升级
| 主题 | 文档 URL |
|---|---|
| OTA（固件端） | `https://docs.sifli.com/projects/solution/2.application-notes/ota/ota_firmware/index.html` |
| OTA（手机 SDK 总览） | `https://docs.sifli.com/projects/solution/mobile-sdk/ota/ota_v3_overall.html` |
| OTA V3 iOS | `https://docs.sifli.com/projects/solution/mobile-sdk/ota/ota_v3_ios.html` |
| OTA V3 Android | `https://docs.sifli.com/projects/solution/mobile-sdk/ota/ota_v3_android.html` |
| OTA V3 Harmony | `https://docs.sifli.com/projects/solution/mobile-sdk/ota/ota_v3_harmony.html` |
| OTA V3 错误码 | `https://docs.sifli.com/projects/solution/mobile-sdk/ota/ota_v3_error_code.html` |
| OTA（HTTP） | `https://docs.sifli.com/projects/solution/2.application-notes/ota/ota_http/index.html` |

#### 2.8 GUI 工具
| 主题 | 文档 URL |
|---|---|
| GUI_Builder 固件 | `https://docs.sifli.com/projects/solution/2.application-notes/gui_tool/gui_builder/index.html` |
| GUI_Builder OTA 说明 | `https://docs.sifli.com/projects/solution/2.application-notes/gui_tool/gui_builder/ota.html` |
| GUI_Builder 表盘编辑 | `https://docs.sifli.com/projects/solution/2.application-notes/gui_tool/gui_builder/wf_edit.html` |
| GUI_Builder 数据对接 | `https://docs.sifli.com/projects/solution/2.application-notes/gui_tool/gui_builder/data.html` |
| GUI_Builder 事件对接 | `https://docs.sifli.com/projects/solution/2.application-notes/gui_tool/gui_builder/event.html` |
| GUI_Builder 调试 | `https://docs.sifli.com/projects/solution/2.application-notes/gui_tool/gui_builder/debug.html` |
| GUI_Builder 控制命令 | `https://docs.sifli.com/projects/solution/2.application-notes/gui_tool/gui_builder/cmd.html` |
| NXP GUI Guider 集成 | `https://docs.sifli.com/projects/solution/2.application-notes/gui_tool/3rd_tool/nxp/nxp.html` |
| SquareLine Studio 集成 | `https://docs.sifli.com/projects/solution/2.application-notes/gui_tool/3rd_tool/squareline/squareline.html` |

#### 2.9 其他
| 主题 | 文档 URL |
|---|---|
| eMMC | `https://docs.sifli.com/projects/solution/2.application-notes/emmc/index.html` |
| USB | `https://docs.sifli.com/projects/solution/2.application-notes/usb/index.html` |
| SDIO Wi-Fi | `https://docs.sifli.com/projects/solution/2.application-notes/sdio_wifi/index.html` |
| 4G（LTE） | `https://docs.sifli.com/projects/solution/2.application-notes/4g/index.html` |
| 新工程创建 | `https://docs.sifli.com/projects/solution/2.application-notes/custom_project/index.html` |
| QuickJS | `https://docs.sifli.com/projects/solution/2.application-notes/quickjs/index.html` |
| RF 测试 | `https://docs.sifli.com/projects/solution/2.application-notes/rf_test/index.html` |
| 烧录相关 | `https://docs.sifli.com/projects/solution/2.application-notes/burn/index.html` |
| eZIP 工具 | `https://docs.sifli.com/projects/solution/2.application-notes/ezip/index.html` |
| 死机分析 | `https://docs.sifli.com/projects/solution/2.application-notes/crash/index.html` |

### 3. 组件说明
| 主题 | 文档 URL |
|---|---|
| UI 控件使用指南 | `https://docs.sifli.com/projects/solution/3.components/ui_widget/index.html` |
| USB MStorage 指南 | `https://docs.sifli.com/projects/solution/3.components/usb_comm.html` |
| BUTTON 指南 | `https://docs.sifli.com/projects/solution/3.components/button_comm.html` |
| Alipay 指南 | `https://docs.sifli.com/projects/solution/3.components/alipay_comm.html` |
| 动态应用指南 | `https://docs.sifli.com/projects/solution/3.components/dynamic_comm.html` |
| 开关机过程 | `https://docs.sifli.com/projects/solution/3.components/power_on_off.html` |
| 睡眠唤醒过程 | `https://docs.sifli.com/projects/solution/3.components/low_power.html` |

---

## 使用方式

当用户提出与 SiFli Solution 开发相关的问题时，AI 应：

1. **定位文档**：根据问题主题，从上方的导航表中找到对应的文档 URL
2. **在线抓取**：使用 `WebFetch` 工具实时获取该 URL 的最新文档内容
3. **结合代码**：将在线文档内容与用户当前项目代码结合，给出准确回答

### 常见问题到文档的快速映射

| 用户问题关键词 | 应获取的文档 |
|---|---|
| 环境搭建、编译、烧录、scons、Butterfli | 快速入门 → 开发环境配置 / 运行第一个项目 |
| UI、界面、LVGL、界面开发、表盘、平铺、弹窗、AOD、动画 | 应用笔记 → UI/应用 对应子页面 |
| 创建新应用、新建APP、app | 应用笔记 → UI/应用 → 创建一个新APP |
| 蓝牙、BLE、BT、对讲、TWS | 应用笔记 → BLE/BT 对应页面 |
| 传感器、心率、加速度计、陀螺仪、地磁 | 应用笔记 → 外设与传感器 对应页面 |
| 低功耗、睡眠、唤醒、功耗 | 应用笔记 → 电源与电池 / 组件说明 → 睡眠唤醒过程 |
| 开关机、boot | 组件说明 → 开关机过程 |
| OTA、升级、推送 | 应用笔记 → OTA 升级 |
| 字体、多语言、图片、资源、eZIP | 快速入门 → 字体 / 图片资源 / 多语言支持 |
| 文件系统、FLASH分区、LittleFS、FAT | 应用笔记 → FLASH分区 & 文件系统 |
| 内存、heap、mem | 快速入门 → 内存管理 |
| 线程、通信、IPC、核间 | 快速入门 → 线程及线程消息通信 / IPC通信 |
| GUI_Builder、NXP、SquareLine | 应用笔记 → GUI 工具 |
| 死机、crash、异常 | 应用笔记 → 死机分析 |
| QuickJS、JS、脚本 | 应用笔记 → QuickJS |
| 驱动、I2C、SPI、UART、PWM | 应用笔记 → 驱动 |
| USB | 应用笔记 → USB / 组件说明 → USB MStorage |
| WiFi、4G | 应用笔记 → SDIO WiFi / 4G |
| 动态应用、外置应用 | 组件说明 → 动态应用指南 |
| 按钮、key、BUTTON | 组件说明 → BUTTON 指南 |

> 注意：如果某个文档 URL 返回 404 或内容不可用，尝试检查 URL 路径是否正确，或向用户确认。文档站点结构可能随版本更新变化，请以实际可访问的页面为准。

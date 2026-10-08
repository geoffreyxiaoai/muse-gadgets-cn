# Muse Gadgets 中文实战手册

**The Chinese field guide to building your own Muse hardware.**

2026 年 10 月 2 日，Meta 开源了 Muse Gadgets：ESP32 固件 + Linux SDK，让任何人都能自制 Muse 的硬件外设。英文官方文档很全，但中文圈目前只有新闻报道、没有实战教程仓库——这个仓库就是来填这个坑的：官方文档的中文翻译、一步步的实测上手记录、DIY 项目精选（每条带点评和避坑）、官方动态追踪。

> 状态：仓库内容本地已备好，待推送。所有"实测"标记均为本仓库维护者真实验证；标"待实测"的是官方文档翻译、尚未经本仓库亲手验证的步骤——欢迎提 PR 补上你的实测。

## 目录

- [先看一眼：官方资源速览](#先看一眼官方资源速览)
- [快速上手](#快速上手)
- [DIY 项目精选](#diy-项目精选)
- [官方动态追踪](#官方动态追踪)
- [FAQ](#faq)
- [维护计划](#维护计划)

---

## 先看一眼：官方资源速览

| 项目 | 说明 |
|---|---|
| 官方仓库 | [facebookincubator/muse-gadget-sdk](https://github.com/facebookincubator/muse-gadget-sdk)（Apache 2.0，2026-10-02 开源）|
| 仓库结构 | `esp32/`（ESP-IDF 固件）· `linux/`（Python SDK）· `skills/`（43 个设备技能：涂鸦、Hue、石头扫地机、小米系等）|
| 官网 | [gadgets.muse.ai](https://gadgets.muse.ai) —— 申请 SDK Token、看 SDK 条款 |
| 社区 | 官方 Discord（官网有 "Join Discord" 入口） |
| 官方设备 | Muse Home Link：ESP32-C5 的 USB-C 小棒，把 Muse 接入家庭局域网；首批 5000 台，仅限美国 Muse 订阅用户免费申领，10 月发货。注意：**Home Link 只能跑官方固件，不能重刷** |

核心机制一句话：每个 gadget 都需要一个 **SDK Token** 才能和 Muse App 配对；配对走 Muse App → 设置 → 设备，先打开开发者模式，找前缀为 `MuseGadget` 的设备（BLE 配对）。

支持的板子（官方仓库 `esp32/devices/` 共 25 个板型配置，节选对国内玩家友好的）：

- **ESP32-C5 DevKitC-1** —— 官方推荐的最快上手板，自带状态灯 + BOOT 键，开箱即用
- **ESP32-S3-BOX-3**、M5Stack CoreS3 / StickC Plus2 / Cardputer
- **微雪（Waveshare）S3/C6 系列**、立创系 ideaspark、Seeed reTerminal
- **Guition JC3248W535**（淘宝常见的廉价彩屏板，官方直接给了配置）
- **M5Stack StopWatch** —— 社区 PR [#17](https://github.com/facebookincubator/muse-gadget-sdk/pull/17)（chantastic，10-03 已合并）：ESP32-S3R8 + 1.75 寸 466px 圆形 AMOLED 触摸屏，完整 UI（头像、一键通话、电池状态、OTA）。首个社区提交即合并的板型，说明官方对社区板型合并很积极
- Home Assistant Voice、SenseCAP Watcher / Indicator、Seeed ReSpeaker Lite
- **Waveshare ESP32-S3-Touch-AMOLED-2.16** —— 2.16 寸 480×480 圆形 AMOLED 全 UI（10-08 上 main，维护者直推；社区 PR #145 已关闭）。微雪圆屏家族第二块
- **FoloToy AI Passport（ESP32-C3）** —— 已上 main（10-08，维护者直推；社区 PR #104 已关闭）。注意 C3 内存限制：无图片显示、无家庭网络隧道
- **LCD7 7 寸屏** —— Muse 头像 UI 官方支持（PR #113，10-05 合并），补记

---

## 快速上手

三条路线，按成本和折腾程度排序。详细步骤见 `docs/`，下面是中文速览。

### 路线 0：先申请 SDK Token（所有路线必备）

1. 打开 [gadgets.muse.ai](https://gadgets.muse.ai)，登录你的 Muse 账号
2. 进入 **Account → SDK tokens**（或直接访问 `/settings/sdk-tokens`），生成一个 token
3. 先读一遍 [Gadget SDK Terms](https://gadgets.muse.ai/sdk-terms)

> 待实测：Token 申请页面的具体字段、是否需要美国区账号，尚未亲手验证。

### 路线 1：ESP32（推荐新手，成本最低）

需要：一块 ESP32 开发板（推荐 ESP32-C5 DevKitC-1，淘宝/京东可购）、一根**能传数据**的 USB 线、一台 macOS/Linux 电脑、Muse App。

官方最快路径（翻译自官方 README，步骤经官方文档核实、刷机环节待实测）：

```sh
# 1. 装 Muse Code（Meta 官方 coding agent，也可用其他 agent）
curl -fsSL https://dev.meta.ai/install.sh | sh

# 2. 拉仓库，进 esp32 目录，用 agent 读 AGENTS.md 全自动构建
git clone https://github.com/facebookincubator/muse-gadget-sdk
cd muse-gadget-sdk/esp32
muse --disable-sandbox
# 然后对 agent 说：Build this firmware for my ESP32-C5 DevKitC-1 and flash it.
```

刷完后：Muse App → 设置 → 设备 → 打开开发者模式 → 选 `MuseGadget` 前缀的设备配对。

> 待实测：国内网络下载 ESP-IDF 工具链的速度与代理方案；C5 板在国内电商版本是否存在兼容差异。

**没板子也能玩**：`esp32/simulator/` 有基于 LVGL/SDL 的模拟器，可在电脑上先跑通流程再买硬件。

### 路线 2：Linux / 树莓派（功能最强）

需要：带蓝牙的树莓派（3B+/4/5/Zero 2 W）或任意 Linux 机器（Debian 11+/Ubuntu 22.04+）、有 sudo 权限的账号。

```sh
# 在目标机器上，用 Muse 要使用的那个账号执行：
curl -fsSL https://raw.githubusercontent.com/facebookincubator/muse-gadget-sdk/main/linux/install.sh -o install.sh
less install.sh     # 先读一遍再跑，别偷懒
bash install.sh --sdk-token mgst_…
```

装完后 `musegadget` 服务会启动并开放蓝牙配对，然后在 Muse App 里配对即可。注意官方警告：**Muse 会获得和安装账号相同的权限，能 sudo 的账号就意味着 Muse 也能 sudo**。

> 待实测：树莓派国内镜像源下的安装流程；中文语音/文字交互效果。

---

## DIY 项目精选

收录标准：真实存在、有出处；每条标注来源与验证状态。**本仓库维护者尚未亲手复刻，复刻后会把状态改为"已实测"并补充避坑。**

| 项目 | 来源 | 状态 | 一句话点评 |
|---|---|---|---|
| Xteink 墨水屏 → 常亮 Muse 随身屏 | 开发者 Federico Viticci（媒体报道） | 转述，未实测 | 把电子墨水阅读器改成 Muse 状态屏，MagSafe 吸手机背面，翻面即看更新。低功耗常亮是墨水屏的天然优势，适合"看板"类场景 |
| SenseCAP Watcher → 酒窖管家 | 社区开发者（媒体报道，约 $50 设备） | 转述，未实测 | 拍酒标识别酒庄年份，经 Tailscale 同步到个人酒窖系统，搭建不到 1 小时。说明"摄像头 + 本地网络 + Muse"是条很顺的 DIY 路线 |
| M5Stack Core2 移植 | 官方仓库 Issue #9（社区提交） | 进行中，有坑 | 非官方支持板型的移植尝试：触摸按键映射错位导致配对确认按不下去、麦克风 GPIO 冲突采不到音。**避坑**：别拿冷门老开发板开局，先用官方配置清单里的板子 |
| M5Stack StopWatch 板型支持 | 官方仓库 PR #17（社区开发者 chantastic，10-03 已合并） | 转述，未实测 | 首个社区提交即合并的完整板型：`sdkconfig` + `board_m5stack_stopwatch.c` + 工具链别名 + CI 构建矩阵一条龙，作者真机实测配对/Wi-Fi/对话全通。**抄作业价值**：想给自己的板子加官方支持，照这个 PR 的文件清单走 |
| Xingzhi Cube 中文小方屏 | 官方仓库 PR #135（segasonicye，open） | 进行中，未合并 | 低成本 ESP32-S3 板，开箱即配 CJK 字体；引脚来自小智生态。**抄作业价值**：中文小屏板子想进官方支持，照这个 PR 的文件清单走 |
| 墨水屏状态板（1.54 寸 e-paper） | 官方仓库 PR #130（zakcohen-work，open） | 进行中，未合并 | 呼应 Xteink 案例的"常亮看板"路线，但走官方板型支持；驱动复用小智 V2 同款屏 |
| Core2 多 App 演示 UI | 官方仓库 PR #131（hfgong，open） | 进行中，未合并 | 在官方头像 UI 之外加触摸启动器示例，只读状态不碰语音链路。**抄作业价值**：想给板子做自定义界面的起点 |
| "Muse Charm" DIY 复刻（M5StickS3） | hackernoon 文章，开源 github.com/zbruceli/kai（zbruceli） | 转述，未实测 | 48×24×15mm 钥匙扣大小，$21 的板子复刻官方 Charm：**双脑架构**——快路径 Gemini Live（~2 秒语音回答）处理大部分问题，慢路径 Hermes Agent（15–50 秒）做研究/记忆/日程，靠语音模型自己的工具选择做路由。**抄作业价值**：Charm 架构的完整实现参考，功耗/中继/双脑路由四个设计问题写得很透；注意它不完全走官方 SDK，是独立实现 |
| Waveshare ESP32-S3-Touch-AMOLED-2.16 全 UI 板型 | 官方仓库 main（维护者直推，社区 PR #145 已关闭） | 转述，未实测 | 2.16 寸 480×480 圆形 AMOLED 完整 UI，微雪圆屏家族第二块。**抄作业价值**：圆屏 UI 的官方实现参考 |
| 官方灵感清单 | 官方仓库/媒体 | 待有人实测 | 彩色墨水屏晨报看板、HDMI 显示棒、带触摸屏的掌上终端、树莓派"家庭实验室 sysadmin" |

想贡献你的 DIY？看 [projects/](projects/) 目录的投稿格式。

---

## 官方动态追踪

| 日期 | 动态 |
|---|---|
| 2026-10-02 | Meta 开源 `facebookincubator/muse-gadget-sdk`（Apache 2.0）；Nat Friedman 宣布 Muse Home Link 首批 5000 台 |
| 2026-10-03 | 社区 PR #12：语音回复改走文本通道——原因是服务器语音模型下线，gadget 端语音提问会收到 "Sorry, I ran into a problem while responding"，现改为 `output_modality: text` 文字转录+字幕显示。**实测影响**：现阶段别指望 gadget 直接语音播报 Muse 的回答，屏幕字幕是正道 |
| 2026-10-03 | 社区 Issue #9：M5Stack Core2 非官方移植遇到配对/麦克风问题（见上表） |
| 2026-10-03 | 社区 PR [#17](https://github.com/facebookincubator/muse-gadget-sdk/pull/17)：M5Stack StopWatch（完整 UI）板型支持已合并——首个社区提交的板型，维护者 anantn 当天跑完 CI 即合并；作者真机实测配对/Wi-Fi/对话全通（边缘触摸校准一项未测）。（2026-10-06 本仓库核实：PR 评论确认已合并，简报"review 中"已过时） |
| 2026-10-05 | 官方仓库单日新增 4 个 issue（标题与内容 2026-10-06 本仓库已核实，均 open、暂无维护者回复）：[#99](https://github.com/facebookincubator/muse-gadget-sdk/issues/99) iPhone App 搜不到自制板（nRF Connect 正常）、[#101](https://github.com/facebookincubator/muse-gadget-sdk/issues/101) iPhone 配对卡在 `get_device_info` 循环、[#105](https://github.com/facebookincubator/muse-gadget-sdk/issues/105) 回复里的 emoji 无字形显示、[#106](https://github.com/facebookincubator/muse-gadget-sdk/issues/106) 回复里的 Markdown 原样显示。信号：**iOS 端配对/发现问题开始集中出现** |
| 2026-10-05 | 社区 Issue [#14](https://github.com/facebookincubator/muse-gadget-sdk/issues/14) 追问语音回复路线图（接 PR #12，下线服务器语音模型后回复为纯文本）：问 Realtime Voice 或新 TTS 端点是否有计划；截至 2026-10-06 本仓库核实，维护者暂未回复（issue 本体更早，此为简报发现日） |
| 2026-10-05 | 本仓库立项：中文圈首个实战向仓库（此前只有新闻报道） |
| 2026-10-06 | 社区 Issue [#129](https://github.com/facebookincubator/muse-gadget-sdk/issues/129)：**新申请的 SDK Token 调用 `fetch_vms` 被 HTTP 401 拒绝**，旧 token 仍可用（设备 Waveshare ESP32-S3-Touch-LCD-4B，token 格式/请求头经串口日志核实无误）。信号：**服务端 token 签发疑似出回归**——配对前先确认手里的 token 是"能用的旧 token"还是刚申请的新 token，别先怀疑自己的固件 |
| 2026-10-06 | 社区 Issue [#127](https://github.com/facebookincubator/muse-gadget-sdk/issues/127)（已关闭，1 条评论）：Linux SDK 下 `/v1/noise` WebSocket 升级返回 403（`proxy_internal_response`），越南与美国 IP 均复现，约 2 分钟后设备 token 被吊销；作者追问是否存在账号/地区 allowlist。信号：**云端接入可能存在账号或地区侧限制**，国内用户配对异常时注意先排查云端连通性 |
| 2026-10-06 | 社区 Issue [#126](https://github.com/facebookincubator/muse-gadget-sdk/issues/126)（功能提案）：Linux SDK 希望**可关闭内置命令**（`system.run`/`file.write`）并**审计 Muse 实际执行的命令**（`sudo musegadget audit`）。背景正好呼应 FAQ 的安全警告——"If it can use sudo, so can Muse."。目前仅为提案未实现，落地前仍建议用低权限账号跑 `musegadget` |
| 2026-10-06 | main 分支直接落地 Linux `system.run` 管道读取的两处修复（原社区 PR #119 内容，由维护者 shaeberling 直接提交）：等待 shell 本身退出、管道读取加 deadline。信号：Linux SDK 的命令执行健壮性正在快速迭代 |
| 2026-10-06 | 社区 PR [#130](https://github.com/facebookincubator/muse-gadget-sdk/pull/130)（open）：Waveshare ESP32-S3-1.54inch-ePaper V2 做**墨水屏状态板**（作者 zakcohen-work），驱动来自小智 V2 的同款屏驱动——"常亮看板"路线正在走官方板型支持 |
| 2026-10-06 | 社区 PR [#131](https://github.com/facebookincubator/muse-gadget-sdk/pull/131)（open）：M5Stack Core2 **可选多 App 演示 UI**（作者 hfgong，`CONFIG_MUSE_APPS_DEMO` 默认关闭）：触摸启动器 + 三个示例 App，只读 `muse_state`，语音会话不受影响。接 Issue #9 的 Core2 故事线——板子跑通之后，自定义界面示例也来了 |
| 2026-10-07 | 社区 PR [#135](https://github.com/facebookincubator/muse-gadget-sdk/pull/135)（open）：**Xingzhi Cube 1.54TFT WiFi** 板型支持（作者 segasonicye）——低成本 ESP32-S3 小方屏，sdkconfig 直接打开 `CONFIG_MUSE_CJK_FONT`，**中文显示是默认配置**；引脚定义来自小智 xiaozhi-esp32 的同款板型。首个面向中文用户的低成本板型 |
| 2026-10-07 | 社区 PR [#134](https://github.com/facebookincubator/muse-gadget-sdk/pull/134)（open）：**320×240 屏中文标题双行显示修复** + 可选 Noto Sans SC 抗锯齿中文字体（作者 longzhi）。信号：中文显示体验正在被中文社区开发者亲手打磨 |
| 2026-10-07 | 社区 PR [#133](https://github.com/facebookincubator/muse-gadget-sdk/pull/133)（open）：**正点原子 ALIENTEK ATK-DNESP32S3-BOX V1.1** 板型支持（作者 longzhi）——国内品牌开发板首次出现在上游 PR；引脚参考 xiaozhi-esp32 的同名板型 |
| 2026-10-05（补记） | 社区 PR [#98](https://github.com/facebookincubator/muse-gadget-sdk/pull/98)（open，上轮未收录）：Waveshare ESP32-S3-Touch-LCD-1.85B，带**中文 UI（简体，Fusion Pixel 字体）+ 手机配网门户**（作者 gxinxing）；附带一个实测坑：6 设备 I2C 总线用 400kHz 会掉包，降到 100kHz 解决 |
| 2026-10-05（补记） | 社区 PR [#104](https://github.com/facebookincubator/muse-gadget-sdk/pull/104)（已上 main，社区 PR 已关闭）：**FoloToy AI Passport（ESP32-C3）**板型支持（作者 gxinxing），无 PSRAM 的 8MB overlay；因内存限制不支持图片显示与家庭网络隧道——**避坑**：C3 这类小内存板子先看清功能取舍再下单 |
| 2026-10-07 | 社区 Issue [#137](https://github.com/facebookincubator/muse-gadget-sdk/issues/137)（open）：**华为手机（EMUI，如 P40）BLE 配对失败**——GATT 写永远到不了固件：App 能连上 BLE、加密和 MTU 协商都通，但 4 分钟发不出配对握手的第一个字节；**同一份固件在 Honor 400 Pro（MagicOS）上一次配对成功**，排除固件问题。信号：**华为/HarmonyOS 系手机与 Muse App 配对存在兼容坑**——国内玩家配对失败时，先换一部非华为系手机做排除法（或用电脑 Chrome Web Bluetooth 直连测试） |
| 2026-10-07 | 社区 Issue [#143](https://github.com/facebookincubator/muse-gadget-sdk/issues/143)（open）：**树莓派 4 上装 Linux SDK 报错**（`musegadget` 安装过程 Python traceback）。信号：Linux 安装脚本在 Debian arm64 上的健壮性还需打磨，Pi 路线建议等官方修完再跟进 |
| 2026-10-07 | 社区 PR [#132](https://github.com/facebookincubator/muse-gadget-sdk/pull/132)（open，作者 zakcohen-work 即 #130 墨水屏作者）：**缺 SDK Token 的报错做醒目提示**。接 #129 的 401 坑——以后配对失败第一眼就能看到是不是 token 的问题 |
| 2026-10-07 | 社区 PR [#136](https://github.com/facebookincubator/muse-gadget-sdk/pull/136)（open）：**BLE 重组缓冲按消息大小动态分配**。接 #50/#51/#99/#101 的配对链路稳定性故事线——配对握手的可靠性还在被社区一点点修 |
| 2026-10-07 | Linux SDK 大重构 PR 栈（作者 abhi，一次连发 5 个，全部 open）：[#138](https://github.com/facebookincubator/muse-gadget-sdk/pull/138) voice notes + spoken-reply 进 LinkSession、[#139](https://github.com/facebookincubator/muse-gadget-sdk/pull/139) chat 订阅 + 语音流式进 LinkSession、[#140](https://github.com/facebookincubator/muse-gadget-sdk/pull/140) MuseTurn（跟踪一次请求到 Muse 回复）、[#141](https://github.com/facebookincubator/muse-gadget-sdk/pull/141) pairing 移出 CLI 成 `pair.pair`、[#142](https://github.com/facebookincubator/muse-gadget-sdk/pull/142) 给基于 SDK 造 gadget 的开发者加 public hooks。信号：**Linux SDK 正在从"一个 CLI 工具"变成"可被二次开发的库"**——进阶 DIY 玩家的新分水岭 |
| 2026-10-07 | 社区板型 PR 新成员（open）：[#112](https://github.com/facebookincubator/muse-gadget-sdk/pull/112) Waveshare ESP32-S3-Touch-LCD-1.85C、[#116](https://github.com/facebookincubator/muse-gadget-sdk/pull/116) Waveshare ESP32-S3-AUDIO-Board（作者 RogerHao）。微雪系板型持续加码 |
| 2026-10-07 | 语音回复体验打磨（open）：[#115](https://github.com/facebookincubator/muse-gadget-sdk/pull/115) 超长回复被截断时明确标记 incomplete、[#117](https://github.com/facebookincubator/muse-gadget-sdk/pull/117) 语音留言可选"简短回复"偏好。接 #87 空回复、#121 语音留言不到设备的语音链路故事线 |
| 2026-10-07 | 社区 PR [#144](https://github.com/facebookincubator/muse-gadget-sdk/pull/144)（open）：Seeed reTerminal E1002 的 SHT40 温湿度上报进 `device.health` 并启用 OTA。接下条 main 落地——**传感器路线在加码** |
| 2026-10-08 | main 分支：**Linux 安全加固实质落地**，接 Issue #126 提案线——commit `5fb03bb2`：`file.write` 的部分文件保持私有权限、去掉 setuid 位；PR [#125](https://github.com/facebookincubator/muse-gadget-sdk/pull/125)（kartsan03）合并：`file.write` 替换文件时保留原 mode。提案正在变成代码，FAQ 的"Linux 那台机器安全吗"可更新为"加固进行中" |
| 2026-10-08 | main 分支：**语音链路系统性修坑**——PR [#110](https://github.com/facebookincubator/muse-gadget-sdk/pull/110)（ramanxg）合并"语音 turn 保持开启直到 Muse 回复"（直接对 #87 空回复、#121 留言不到设备）、PR [#118](https://github.com/facebookincubator/muse-gadget-sdk/pull/118)（ViSaReVe）合并语音留言上传路径的 host 测试、commit `cafaad48`：被截断的超长语音 turn 明确失败而非静默。语音这块从"能用"走向"可靠" |
| 2026-10-08 | main 分支：**环境传感器全板型上报**——commit `4addfff9`：`sensors.read` 在所有板型上报告环境传感器；reTerminal E100x 的 SHT4x 温湿度接入（PR [#120](https://github.com/facebookincubator/muse-gadget-sdk/pull/120)，bruceburge，已合并）。想做"酒窖管家"这类传感器 DIY 的，API 就绪了 |
| 2026-10-08 | **FoloToy AI Passport 板型已上 main**：维护者直接提交 `85503524`（完整 UI）+ 当天 UI 修复 `62dd77af`（UP 键走菜单上移、关背光 console、收紧超长检查）；社区 PR [#104](https://github.com/facebookincubator/muse-gadget-sdk/pull/104)（gxinxing）已关闭（板型走维护者直推路线落地） |
| 2026-10-08 | **Waveshare ESP32-S3-Touch-AMOLED-2.16 全 UI 板型已上 main**：维护者提交 `dc511680`（完整 UI：2.16 寸 480×480 圆形 AMOLED CO5300 + CST9220 触摸、ES8311 音频）+ 后续修复 `95755cbc`（通话键跟踪、跳过未轮询的 PWR 键）；社区 PR [#145](https://github.com/facebookincubator/muse-gadget-sdk/pull/145)（GEMISIS）已关闭未合并（板型走维护者直推落地） |
| 2026-10-05（补记） | main 分支：**LCD7 7 寸屏支持**已落地（PR [#113](https://github.com/facebookincubator/muse-gadget-sdk/pull/113) 合并：Muse 头像 helper 支持 LCD7、头像画布适配、状态环对齐、硬件端口文档）。此前巡检未收录，补记 |

> 仓库安全提示（第三方技术分析，非官方声明）：固件用仓库自带的 dev key 签名、Secure Boot 默认关闭、无设备 attestation，配对过程理论上可被中间人拦截。结论：**自玩可以，别拿它做正经产品或接敏感设备**。

---

## FAQ

**Q：一定要 Muse 账号吗？国内手机号能注册吗？**
A：申请 SDK Token 需要登录 gadgets.muse.ai，即需要 Muse 账号。国内注册流程尚未验证，待实测，有结果会更新在这里。

**Q：Home Link 国内能申领吗？**
A：不能。官方口径：仅限美国地区 Muse 订阅用户，每人限领一台，先到先得。国内玩家走 DIY 路线自己造，功能对等（Home Link 本身就是基于开源 ESP32 SDK 的）。

**Q：gadget 能说中文吗？**
A：官方文档未明确说明中文语音/文字支持情况，待实测。这是中文社区最该验证的一件事，欢迎第一个跑通的人来更新。另：社区 Issue [#14](https://github.com/facebookincubator/muse-gadget-sdk/issues/14) 在追问语音回复路线图（PR #12 下线服务器语音模型后回复为纯文本），维护者暂未回复——语音这事官方还没表态。

**Q：华为手机搜得到设备但配对不上，是什么情况？**
A：社区 Issue [#137](https://github.com/facebookincubator/muse-gadget-sdk/issues/137)：华为 P40（EMUI）上 Muse App 能连 BLE、加密和 MTU 都正常，但配对握手 4 分钟发不出一个字节；同一份固件在 Honor 400 Pro（MagicOS）上一次成功。先换一部非华为系手机做排除法——**这是国内玩家配对的第一个坑**。

**Q：SDK Token 刚申请，设备却报 401 / 配对时连不上云，是什么情况？**
A：先别急着重刷固件——社区 10-06 有两例服务端侧异常：Issue [#129](https://github.com/facebookincubator/muse-gadget-sdk/issues/129) 新 token 在 `fetch_vms` 被 401 拒绝（旧 token 可用）；Issue [#127](https://github.com/facebookincubator/muse-gadget-sdk/issues/127) `/v1/noise` 升级 403 且 2 分钟后 token 被吊销（越南/美国 IP 均复现，作者怀疑地区 allowlist）。排查顺序：先确认 token 本身有效、再看云端连通性，最后才动本地配置。国内用户尤其注意云端连通这一环。另：社区 PR [#132](https://github.com/facebookincubator/muse-gadget-sdk/pull/132) 正在做"缺 token 报错醒目提示"（未合并），以后这类问题第一眼就能定位。

**Q：刷机会变砖吗？**
A：官方原话：副作用可能包括 bricked boards、voided warranties、brownouts。备好救砖方案（USB 转串口、按住 BOOT 进下载模式），别拿唯一的一块板子做实验。

**Q：Linux 那台机器安全吗？**
A：Muse 拿到的是安装账号的完整权限。建议单独建一个低权限账号跑 `musegadget`，别直接用你的主力 sudo 账号。另：社区 Issue [#126](https://github.com/facebookincubator/muse-gadget-sdk/issues/126) 正在提案"可关闭内置命令 + 审计日志"的安全加固方案（未实现），落地前仍建议单独建低权限账号跑 `musegadget`。另：2026-10-08 main 已实质落地加固——`file.write` 的部分文件保持私有权限并去掉 setuid 位（commit `5fb03bb2`），PR [#125](https://github.com/facebookincubator/muse-gadget-sdk/pull/125) 合并（替换文件保留原 mode）。Issue #126 的"可关闭内置命令 + 审计日志"提案正在变成代码，落地前仍建议低权限账号。

**Q：这个仓库和官方是什么关系？**
A：无任何官方关系。纯社区中文实战手册，内容错误请提 Issue/PR。

---

## 维护计划

- 每周巡检：官方仓库的新提交/PR/Issue、Discord 与媒体上的新 DIY 案例
- 新增案例必须标注来源与验证状态，不编造未经核实的步骤
- 链接全部定期做死链检查

## License

本仓库文档内容采用 CC BY 4.0；引用/翻译的官方仓库代码遵循其 Apache 2.0 许可。

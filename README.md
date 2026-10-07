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
| 2026-10-05（补记） | 社区 PR [#104](https://github.com/facebookincubator/muse-gadget-sdk/pull/104)（open，上轮未收录）：**FoloToy AI Passport（ESP32-C3）**板型支持（作者 gxinxing），无 PSRAM 的 8MB overlay；因内存限制不支持图片显示与家庭网络隧道——**避坑**：C3 这类小内存板子先看清功能取舍再下单 |

> 仓库安全提示（第三方技术分析，非官方声明）：固件用仓库自带的 dev key 签名、Secure Boot 默认关闭、无设备 attestation，配对过程理论上可被中间人拦截。结论：**自玩可以，别拿它做正经产品或接敏感设备**。

---

## FAQ

**Q：一定要 Muse 账号吗？国内手机号能注册吗？**
A：申请 SDK Token 需要登录 gadgets.muse.ai，即需要 Muse 账号。国内注册流程尚未验证，待实测，有结果会更新在这里。

**Q：Home Link 国内能申领吗？**
A：不能。官方口径：仅限美国地区 Muse 订阅用户，每人限领一台，先到先得。国内玩家走 DIY 路线自己造，功能对等（Home Link 本身就是基于开源 ESP32 SDK 的）。

**Q：gadget 能说中文吗？**
A：官方文档未明确说明中文语音/文字支持情况，待实测。这是中文社区最该验证的一件事，欢迎第一个跑通的人来更新。另：社区 Issue [#14](https://github.com/facebookincubator/muse-gadget-sdk/issues/14) 在追问语音回复路线图（PR #12 下线服务器语音模型后回复为纯文本），维护者暂未回复——语音这事官方还没表态。

**Q：SDK Token 刚申请，设备却报 401 / 配对时连不上云，是什么情况？**
A：先别急着重刷固件——社区 10-06 有两例服务端侧异常：Issue [#129](https://github.com/facebookincubator/muse-gadget-sdk/issues/129) 新 token 在 `fetch_vms` 被 401 拒绝（旧 token 可用）；Issue [#127](https://github.com/facebookincubator/muse-gadget-sdk/issues/127) `/v1/noise` 升级 403 且 2 分钟后 token 被吊销（越南/美国 IP 均复现，作者怀疑地区 allowlist）。排查顺序：先确认 token 本身有效、再看云端连通性，最后才动本地配置。国内用户尤其注意云端连通这一环。

**Q：刷机会变砖吗？**
A：官方原话：副作用可能包括 bricked boards、voided warranties、brownouts。备好救砖方案（USB 转串口、按住 BOOT 进下载模式），别拿唯一的一块板子做实验。

**Q：Linux 那台机器安全吗？**
A：Muse 拿到的是安装账号的完整权限。建议单独建一个低权限账号跑 `musegadget`，别直接用你的主力 sudo 账号。另：社区 Issue [#126](https://github.com/facebookincubator/muse-gadget-sdk/issues/126) 正在提案"可关闭内置命令 + 审计日志"的安全加固方案（未实现），落地前仍建议单独建低权限账号跑 `musegadget`。

**Q：这个仓库和官方是什么关系？**
A：无任何官方关系。纯社区中文实战手册，内容错误请提 Issue/PR。

---

## 维护计划

- 每周巡检：官方仓库的新提交/PR/Issue、Discord 与媒体上的新 DIY 案例
- 新增案例必须标注来源与验证状态，不编造未经核实的步骤
- 链接全部定期做死链检查

## License

本仓库文档内容采用 CC BY 4.0；引用/翻译的官方仓库代码遵循其 Apache 2.0 许可。

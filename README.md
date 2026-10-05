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
| 官方灵感清单 | 官方仓库/媒体 | 待有人实测 | 彩色墨水屏晨报看板、HDMI 显示棒、带触摸屏的掌上终端、树莓派"家庭实验室 sysadmin" |

想贡献你的 DIY？看 [projects/](projects/) 目录的投稿格式。

---

## 官方动态追踪

| 日期 | 动态 |
|---|---|
| 2026-10-02 | Meta 开源 `facebookincubator/muse-gadget-sdk`（Apache 2.0）；Nat Friedman 宣布 Muse Home Link 首批 5000 台 |
| 2026-10-03 | 社区 PR #12：语音回复改走文本通道——原因是服务器语音模型下线，gadget 端语音提问会收到 "Sorry, I ran into a problem while responding"，现改为 `output_modality: text` 文字转录+字幕显示。**实测影响**：现阶段别指望 gadget 直接语音播报 Muse 的回答，屏幕字幕是正道 |
| 2026-10-03 | 社区 Issue #9：M5Stack Core2 非官方移植遇到配对/麦克风问题（见上表） |
| 2026-10-05 | 本仓库立项：中文圈首个实战向仓库（此前只有新闻报道） |

> 仓库安全提示（第三方技术分析，非官方声明）：固件用仓库自带的 dev key 签名、Secure Boot 默认关闭、无设备 attestation，配对过程理论上可被中间人拦截。结论：**自玩可以，别拿它做正经产品或接敏感设备**。

---

## FAQ

**Q：一定要 Muse 账号吗？国内手机号能注册吗？**
A：申请 SDK Token 需要登录 gadgets.muse.ai，即需要 Muse 账号。国内注册流程尚未验证，待实测，有结果会更新在这里。

**Q：Home Link 国内能申领吗？**
A：不能。官方口径：仅限美国地区 Muse 订阅用户，每人限领一台，先到先得。国内玩家走 DIY 路线自己造，功能对等（Home Link 本身就是基于开源 ESP32 SDK 的）。

**Q：gadget 能说中文吗？**
A：官方文档未明确说明中文语音/文字支持情况，待实测。这是中文社区最该验证的一件事，欢迎第一个跑通的人来更新。

**Q：刷机会变砖吗？**
A：官方原话：副作用可能包括 bricked boards、voided warranties、brownouts。备好救砖方案（USB 转串口、按住 BOOT 进下载模式），别拿唯一的一块板子做实验。

**Q：Linux 那台机器安全吗？**
A：Muse 拿到的是安装账号的完整权限。建议单独建一个低权限账号跑 `musegadget`，别直接用你的主力 sudo 账号。

**Q：这个仓库和官方是什么关系？**
A：无任何官方关系。纯社区中文实战手册，内容错误请提 Issue/PR。

---

## 维护计划

- 每周巡检：官方仓库的新提交/PR/Issue、Discord 与媒体上的新 DIY 案例
- 新增案例必须标注来源与验证状态，不编造未经核实的步骤
- 链接全部定期做死链检查

## License

本仓库文档内容采用 CC BY 4.0；引用/翻译的官方仓库代码遵循其 Apache 2.0 许可。

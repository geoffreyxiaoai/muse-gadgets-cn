# ESP32 上手指南（中文翻译 + 实测记录）

> 翻译自官方仓库 `esp32/README.md`（2026-10-02 版本，已核实）。
> 凡标"待实测"的步骤为本仓库维护者尚未亲手验证。

## 你需要

- **一块 ESP32 开发板**。最快上手：**ESP32-C5 DevKitC-1**（自带状态灯 + BOOT 键，开箱即用）
- 一根**能传数据**的 USB 线（只充电的线不行，这是新手第一坑）
- 一台 macOS 或 Linux 电脑
- 一个 [SDK Token](token.md)
- 手机上的 Muse App

## 用 Muse Code 全自动构建（官方推荐路径）

```sh
# 1. 安装 Muse Code
curl -fsSL https://dev.meta.ai/install.sh | sh

# 2. 拉官方仓库，进 esp32 目录启动 agent
git clone https://github.com/facebookincubator/muse-gadget-sdk
cd muse-gadget-sdk/esp32
muse --disable-sandbox
```

第一次启动 agent 会要求信任工作区并登录。`--disable-sandbox` 是为了让它能访问 USB 串口、下载 ESP-IDF 工具链（每条命令仍需你手动批准）。

然后直接对 agent 说：

> Build this firmware for my ESP32-C5 DevKitC-1 and flash it.

agent 会读 `AGENTS.md`（官方给 coding agent 写的构建手册），自动完成工具链安装、编译、烧录、看日志。继续可以说：

> Watch the serial log and tell me when it's ready to pair.
> I have a Waveshare ESP32-S3 AMOLED board. Build the UI for it.

## 配对

1. Muse App → 设置 → 设备
2. 打开**开发者模式**
3. 选择前缀为 `MuseGadget` 的设备，完成配对

## 没硬件？先用模拟器

`esp32/simulator/` 有基于 LVGL + SDL 的模拟器，可以在电脑上先跑通流程、验证 UI，再决定买哪块板。（待实测：模拟器的构建步骤）

## 已验证支持的板型（`esp32/devices/` 共 25 个配置）

ESP32-C5 DevKitC-1、ESP32-S3-BOX-3、M5Stack（CoreS3 / StickC Plus2 / StickS3 / Cardputer / Core2*）、微雪 S3/C6 系列、Guition JC3248W535、ideaspark、reTerminal E1001/E1002、SenseCAP Watcher/Indicator、Seeed ReSpeaker Lite、Home Assistant Voice 等。

\* Core2 为社区移植（Issue #9），有已知坑，见 README 的 DIY 精选。

## 待实测清单

- [ ] ESP32-C5 DevKitC-1 国内电商版本全流程烧录
- [ ] 国内网络下载 ESP-IDF 工具链（是否需要镜像/代理）
- [ ] 模拟器在 macOS 上的构建
- [ ] 中文语音/文字交互效果

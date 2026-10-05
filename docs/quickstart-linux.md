# Linux / 树莓派上手指南（中文翻译 + 实测记录）

> 翻译自官方仓库 `linux/README.md`（2026-10-02 版本，已核实）。
> 凡标"待实测"的步骤为本仓库维护者尚未亲手验证。

## 你需要

- 带蓝牙的树莓派：3B+、4、5 或 Zero 2 W（其他 Linux 电脑也行，要有 BLE）
- 系统：Raspberry Pi OS Bullseye 或更新 / Debian 11+ / Ubuntu 22.04+（32/64 位都行），已联网
- 机器上一个**有 sudo 权限**的账号
- 一个 [SDK Token](token.md)
- 手机上的 Muse App

## 安装

在目标机器上，用**你希望 Muse 使用的那个账号**执行：

```sh
curl -fsSL https://raw.githubusercontent.com/facebookincubator/muse-gadget-sdk/main/linux/install.sh -o install.sh
less install.sh     # 先读一遍脚本，确认它要干什么
bash install.sh --sdk-token mgst_…
```

安装器会检查系统、把文件装到 `/opt/musegadget`、启动 `musegadget` 服务。交出账号权限前它会先征求你同意，并告诉你这个账号能不能 sudo。然后开放蓝牙配对。

## 配对

安装器提示配对开放后：Muse App → 设置 → 设备 → 打开开发者模式 → 选 `MuseGadget` 前缀的设备。

## 能做什么

Muse 可以在这台机器上跑命令、读写文件、查 uptime/温度/磁盘占用；自定义命令、接 webhook、桥接 Home Assistant 都在官方示例里。

## 安全警告（官方原话的中文转述）

**Muse 获得和安装账号完全相同的权限**——能 sudo 的账号，就意味着 Muse 也能 sudo。建议：

1. 单独建一个低权限账号跑 `musegadget`，别用主力账号
2. 别把生产环境机器 / 存敏感数据的机器拿来当 gadget

## 待实测清单

- [ ] 树莓派国内镜像源下的完整安装流程
- [ ] 自定义命令的中文示例
- [ ] Home Assistant 桥接实测

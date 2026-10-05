# SDK Token 申请指南

> 依据：官方仓库 `esp32/README.md` / `linux/README.md`（2026-10-02 版本，已核实）。
> 本页未亲手走完流程的部分标有"待实测"。

## 为什么需要 Token

每个 gadget（无论 ESP32 还是 Linux）都需要一个 SDK Token 才能和 Muse App 配对，包括你自己做的设备。这是 Meta 把设备绑定到 Muse 账号的机制。

## 步骤

1. 打开 [gadgets.muse.ai](https://gadgets.muse.ai)，登录 Muse 账号
2. 进入 **Account → SDK tokens**（直接访问 `https://gadgets.muse.ai/settings/sdk-tokens`）
3. 生成 token，妥善保存（格式形如 `mgst_…`）
4. 使用前先读 [Gadget SDK Terms](https://gadgets.muse.ai/sdk-terms)

## Token 的用法

- **ESP32**：烧录/配对流程中按官方指引填入（具体字段见 `docs/quickstart-esp32.md`）
- **Linux**：安装脚本参数 `--sdk-token mgst_…`

## 待实测

- [ ] 申请页面是否需要美国区账号 / 美国手机号
- [ ] 国内网络访问 gadgets.muse.ai 是否正常
- [ ] 一个账号可生成几个 token、有效期与吊销方式

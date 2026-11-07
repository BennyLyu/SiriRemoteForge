# SiriRemoteForge 当前交接

更新日期：2026-11-07

本文记录继续开发所需的当前状态。产品与配置说明见 [`README.md`](README.md)；麦克风协议调查见 [`docs/mic-reverse-engineering.md`](docs/mic-reverse-engineering.md)。旧版逐日开发流水已精简，不再作为当前事实来源。

## 项目状态

- 项目是面向 macOS 的第三代 Siri Remote 工具，当前公开版本为 `v0.2.0-beta.9`。应用内部 bundle 名仍为 `HyperVibe`，产品名为 siriRemote。
- 主应用以 Swift 构建；`SiriRemoteCore/` 提供配置和映射逻辑。`mic/` 包含虚拟 CoreAudio 麦克风、BLE 语音路由器和按需采集服务；较早的 `driverkit/` 方案仅供参考。
- 麦克风流依赖 PacketLogger 和未公开的 Apple 蓝牙接口。采集服务按需启动 PacketLogger 与 `srm_router`，路由器解析 Siri Remote 的 BLE 语音通知并向虚拟麦克风提供音频。PacketLogger 不在仓库中分发。
- beta.9 已验证 A2854 及不同 ATT 语音句柄布局；路由器只接受通过方向、通知类型、长度、序号和 Opus 帧校验的数据。

## 当前用户工作方式

- 用户使用 Apple silicon Mac mini、macOS 27.2、第三代 USB-C Siri Remote（A2854）；系统蓝牙中遥控器以序列号 `C08LVYL72330` 显示为已连接，macOS 自身可使用其麦克风。
- HyperVibe `0.2.0-beta.9` 曾停留在“等待遥控器”。用户提供的 `srm_captured` 日志显示采集管线可启动，并出现 `voice stream .../ATT-0x0036`，说明语音通知确实到达路由器。设备发现问题后来已解决。
- 用户要用 Siri Remote 控制 ChatGPT 桌面版自己的 Dictation，不需要接入 OpenAI 或 DeepSeek API。ChatGPT 没有可供 HyperVibe 读取该偏好设置的公开接口，用户需在 ChatGPT 和 HyperVibe 中各设一次相同的快捷键。ChatGPT 的 **Single-tap shortcut** 是开/关切换；将 Siri 按键映射到 `pushToTalk` 后，实体按下和松开会分别发送一次快捷键，开始并结束听写。ChatGPT 的“按住说话”快捷键则要求真实的键盘按下/松开状态，不能误用切换式绑定。
- 过去试验过的 AX Dictation 控制代码已从工作改动中移除；除非用户明确要求，不要恢复该实验。

## 本分支待提交改动

- `app/RemoteDetector.swift`：调整 HID 管理器初始化顺序，并按已知遥控器产品 ID 匹配接口，避免仅依赖易变化的设备名称造成漏检。
- 项目自有的英文 Markdown 文档正在翻译为简体中文。`HANDOFF.md` 已改为精简的当前交接文档。
- 许可证文件保留原文；不要改写第三方许可条款。

## 开发与维护约束

- 先阅读仓库根目录 [`AGENTS.md`](AGENTS.md)，遵循其中的本地签名、安装和运行要求。尤其不要把临时 ad-hoc 签名构建覆盖安装的稳定签名应用。
- 标准入口：`cd SiriRemoteCore && swift test`、`cd app && ./build.sh`。发行包脚本位于 `dist/`。如用户未要求验证，不要擅自增加或运行测试。
- 调试日志通常写入 `/tmp/hypervibe.log`；麦克风采集服务日志为 `/var/log/srm_captured.log`。日志可能包含设备或运行状态信息，公开前先检查并去除隐私数据。
- 项目采用 GPL-3.0-or-later。上游 MIT 版权声明保存在 `NOTICE`，完整许可证文本保存在 `LICENSE`。

## 后续事项

1. 完成剩余项目文档的中文化，保留命令、代码、配置键、API 标识、链接和许可证原文。
2. 将 HID 发现修复与文档翻译分别提交，推送到 BennyLyu 的 Fork 功能分支；不直接改写 Fork 的 `main`。
3. 后续若继续改遥控器按键映射，保留 ChatGPT Single-tap shortcut 与 `pushToTalk` 的按下/松开配对，并在文档中说明两边快捷键需由用户手动保持一致。

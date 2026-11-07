# SiriRemoteForge 当前交接

更新日期：2026-11-07

本文记录继续开发所需的当前状态。产品与配置说明见 [`README.md`](README.md)；麦克风协议调查见 [`docs/mic-reverse-engineering.md`](docs/mic-reverse-engineering.md)。旧版逐日开发流水已精简，不再作为当前事实来源。

## 项目状态

- 项目是面向 macOS 的第三代 Siri Remote 工具，准备发布的版本为 `v0.2.0-beta.10`。应用内部 bundle 名仍为 `HyperVibe`，产品名为 siriRemote。Beta 发布目标为 BennyLyu/SiriRemoteForge Fork。
- 主应用以 Swift 构建；`SiriRemoteCore/` 提供配置和映射逻辑。`mic/` 包含虚拟 CoreAudio 麦克风、BLE 语音路由器和按需采集服务；较早的 `driverkit/` 方案仅供参考。
- 麦克风流依赖 PacketLogger 和未公开的 Apple 蓝牙接口。采集服务按需启动 PacketLogger 与 `srm_router`，路由器解析 Siri Remote 的 BLE 语音通知并向虚拟麦克风提供音频。PacketLogger 不在仓库中分发。
- 已验证 A2854 及不同 ATT 语音句柄布局；路由器只接受通过方向、通知类型、长度、序号和 Opus 帧校验的数据。

## 当前用户工作方式

- 用户使用 Apple silicon Mac mini、macOS 27.2、第三代 USB-C Siri Remote（A2854）；系统蓝牙中遥控器以序列号 `C08LVYL72330` 显示为已连接，macOS 自身可使用其麦克风。
- 用户使用 Apple silicon Mac mini、macOS 27.2、第三代 USB-C Siri Remote（A2854）；系统蓝牙显示遥控器 `C08LVYL72330` 已连接，macOS 可使用其麦克风，HyperVibe 的发现问题已解决。
- 用户要用 Siri Remote 控制 ChatGPT 桌面版 Dictation。用户需在 ChatGPT 的 **Single-tap shortcut** 和 HyperVibe Siri 轻点映射中设置同一快捷键；长按动作 `chatGPTDictation` 自动在阈值触发和松开时发送该快捷键。仅 ChatGPT 在前台时长按才触发。无需 API 密钥。
- 常驻层级状态卡可通过菜单栏的**显示层级**切换；ChatGPT 听写期间自动隐藏相关 HUD，松开后恢复。
- beta.10 的新安装包使用 BennyLyu/SiriRemoteForge 更新源。旧发布渠道的安装包内置原更新 URL，必须先从 Fork 手动安装一次 beta.10，后续应用内检查更新才会转向 Fork。
- 过去试验过的 AX Dictation 控制代码已从工作改动中移除；除非用户明确要求，不要恢复该实验。

## 发布提示

- 发布标签：`v0.2.0-beta.10`；目标仓库：`BennyLyu/SiriRemoteForge`。
- 更新源、Appcast 资产链接和已发布更新校验脚本必须统一指向该 Fork。
- Fork 的 Sparkle 私钥使用登录钥匙串账户 `hypervibe-benny-fork`；公钥固定在应用 Info.plist 中。私钥须另存加密离线备份，不得提交到 Git。
- 许可证文件保留原文；不要改写第三方许可条款。

## 开发与维护约束

- 先阅读仓库根目录 [`AGENTS.md`](AGENTS.md)，遵循其中的本地签名、安装和运行要求。尤其不要把临时 ad-hoc 签名构建覆盖安装的稳定签名应用。
- 标准入口：`cd SiriRemoteCore && swift test`、`cd app && ./build.sh`。发行包脚本位于 `dist/`。如用户未要求验证，不要擅自增加或运行测试。
- 调试日志通常写入 `/tmp/hypervibe.log`；麦克风采集服务日志为 `/var/log/srm_captured.log`。日志可能包含设备或运行状态信息，公开前先检查并去除隐私数据。
- 项目采用 GPL-3.0-or-later。上游 MIT 版权声明保存在 `NOTICE`，完整许可证文本保存在 `LICENSE`。

## 后续事项

1. 完成并发布 beta.10，验证 Fork 的更新源仅提供最新 beta.10。
2. 后续若继续改遥控器按键映射，保留 ChatGPT Single-tap shortcut 与 `chatGPTDictation` 的快捷键配对，并说明两边快捷键需由用户手动保持一致。

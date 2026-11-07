# SiriRemoteForge v0.2.0-beta.3

本版完善 HyperVibe 常驻状态界面，并将设置统一迁移到可移植、热重载的 JSON 配置。小组件在各种状态下都能清楚标识当前图层；动画衔接 macOS 原生材质，并根据底层外观调整对比度，无需屏幕录制权限。

## 选择一个下载包

- **仅应用：** `HyperVibe-0.2.0-beta.3-macOS-arm64.zip`。解压后将 `HyperVibe.app` 移至“应用程序”，然后右键点按并选择一次“打开”。
- **完整安装（进阶）：** `HyperVibe-Full-Setup-0.2.0-beta.3-arm64.zip`。包含 HyperVibe、Siri Remote Mic、按需语音采集服务和 `HyperVibe Uninstall.app`。安装需管理员授权，会短暂重启系统音频，并通过 `coreaudiod` 检查及自动回滚验证插件。

两个版本均为 ad-hoc 签名，未经 Apple 公证。需要 macOS 13 或更高版本、Apple 芯片和第三代 USB-C Siri Remote。

## 主要变化

### 更清楚、连贯的状态小组件

- 闲置、动作、长按和语音状态都显示与图层关联的边框颜色，方便识别当前图层。
- 卡片物理尺寸保持不变；过渡只重建图标和文字，避免图层光晕相对材质漂移或缩放。
- 图标作为整体绕垂直轴翻转，文字使用连续字形动画和水平折叠效果；临时动画快照与 AppKit 常驻标签同步，消除动画结束时的单帧跳动。
- 根据 `NSVisualEffectView.interiorBackgroundStyle`、语义文字颜色和当前外观，自动确保浅色、深色背景下可读；不使用屏幕录制或采样。
- 保留红色录音指示点，同时让语音文字和波形强调色适应材质对比度。

### `config.jsonc` 管理所有正式设置

- 新增 JSON 设置：`interfaceLanguage`、`launchAtLoginEnabled`、`menuBarIconEnabled`、`layerHUDEnabled`、`dragIndicatorEnabled` 和 `showSetupWizardOnFirstLaunch`。
- 设置界面语言、登录项注册和各状态面板的显隐都会立即生效，并通过同一原子写入路径更新配置。
- 只有与本机有关的状态留在可移植配置之外：拖动后的小组件位置（包含显示器 ID）和此 Mac 是否完成首次设置。
- 不接受未知的界面语言值，避免静默回退。

## 麦克风说明

遥控器蓝牙语音采集还需要从 [Xcode Additional Tools](https://developer.apple.com/download/all/?q=Additional+Tools+for+Xcode) 获取 Apple 的 PacketLogger。项目不附带 PacketLogger；没有它时，主应用和内置麦克风回退仍可使用。

## 验证与下载校验

- SiriRemoteCore 的 125 项测试通过，包含新界面设置的 JSON 默认值、覆盖、校验和稳定写回。
- arm64 macOS 应用构建通过。发布包已校验校验和、签名和架构，并检查是否包含私有路径、设备标识、个人配置、PacketLogger 或正式媒体素材。

使用随附的 `SHA256SUMS.txt` 校验下载文件：

```sh
shasum -a 256 -c SHA256SUMS.txt
```

这是预发布软件，使用了 macOS 私有框架和未公开的蓝牙语音通道。反馈可复现问题时，请附上 macOS 版本、遥控器固件版本和相关日志。

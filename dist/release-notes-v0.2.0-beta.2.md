# SiriRemoteForge v0.2.0-beta.2

本版让设置界面更安全、快捷：每次保存都有明确状态，快捷键动作可以直接通过实体键盘录入，不必手工填写配置标记。

## 选择一个下载包

- **仅应用：** `HyperVibe-0.2.0-beta.2-macOS-arm64.zip`。解压后将 `HyperVibe.app` 移至“应用程序”，然后右键点按并选择一次“打开”。
- **完整安装（进阶）：** `HyperVibe-Full-Setup-0.2.0-beta.2-arm64.zip`。包含 HyperVibe、Siri Remote Mic、按需语音采集服务和 `HyperVibe Uninstall.app`。安装需管理员授权，会短暂重启系统音频，并通过 `coreaudiod` 检查及自动回滚验证插件。

两个版本均为 ad-hoc 签名，未经 Apple 公证。需要 macOS 13 或更高版本、Apple 芯片和第三代 USB-C Siri Remote。

## 主要变化

### 直接用键盘录入快捷键

- Keystroke、Push-to-talk 和 Repeat Key 动作现在使用原生 macOS 快捷键录制器。点按输入框并按下组合键，即可立即保存标准化快捷键。
- 保留左右 Command、Control、Option、Shift 的区别，支持单独修饰键、修饰键组合、Fn、Fn 组合、导航键、标点、Forward Delete、Help 和 F1–F20。
- Escape 取消录制；单独按 Delete 可清除内容；铅笔按钮可切换到高级文本输入，以粘贴或编辑组合键。

### 清楚显示配置保存状态

- 设置页标题栏显示“正在保存… / 已自动保存 / 保存失败”；点击失败状态可查看具体错误。
- Tuning 页仍采用短暂防抖并实时生效，Layout 页在控件提交后保存；两者都以 `config.jsonc` 为唯一事实来源并原子写入。
- 现有 JSONC 无效时不会被覆盖，界面会说明拒绝保存的原因。

## 麦克风说明

遥控器蓝牙语音采集还需要从 [Xcode Additional Tools](https://developer.apple.com/download/all/?q=Additional+Tools+for+Xcode) 获取 Apple 的 PacketLogger。项目不附带 PacketLogger；没有它时，主应用和内置麦克风回退仍可使用。

## 验证与下载校验

- SiriRemoteCore 的 124 项测试通过，包含快捷键规范化和实体 Fn 键录入。
- arm64 macOS 应用构建通过；原生 AppKit 录制器桥接通过合成 `NSEvent` 测试，覆盖普通组合键、左右修饰键、Fn+F12、单独 Fn、取消和清除。
- 发布包已校验校验和、签名和架构，并检查是否包含私有路径、设备标识、个人配置、PacketLogger 或正式媒体素材。

使用随附的 `SHA256SUMS.txt` 校验下载文件：

```sh
shasum -a 256 -c SHA256SUMS.txt
```

这是预发布软件，使用了 macOS 私有框架和未公开的蓝牙语音通道。反馈可复现问题时，请附上 macOS 版本、遥控器固件版本和相关日志。

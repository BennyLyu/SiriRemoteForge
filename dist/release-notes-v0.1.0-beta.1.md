# SiriRemoteForge v0.1.0-beta.1

面向 Apple 芯片 Mac 的首个公开测试版。

## 选择一个下载包

- **仅应用：** `HyperVibe-0.1.0-beta.1-macOS-arm64.zip`。解压后将 `HyperVibe.app` 移至“应用程序”，然后右键点按并选择一次“打开”。
- **完整安装（进阶）：** `HyperVibe-Full-Setup-0.1.0-beta.1-arm64.zip`。此外还会安装 Siri Remote Mic 和按需语音采集服务、检查并在失败时自动回滚系统音频插件，并运行 25 秒的 `coreaudiod` 安全检查。

两个版本均为 ad-hoc 签名，未经 Apple 公证。需要 macOS 13 或更高版本以及第三代 USB-C Siri Remote。本测试版仅支持 arm64。

## 主要功能

- 提供 Siri Remote 指针、按键和图层控制，并允许通过配置文件自定义操作。
- 提供可选的 Siri Remote 麦克风采集与内置麦克风回退方案。
- 提供适用于 Apple 芯片 Mac 的原生应用。

## 麦克风说明

使用遥控器蓝牙语音采集还需要从 [Xcode Additional Tools](https://developer.apple.com/download/all/?q=Additional+Tools+for+Xcode) 获取 Apple 的 PacketLogger。项目不附带 PacketLogger；没有它时，应用和内置麦克风回退仍可使用。

## 校验下载文件

使用随附的 `SHA256SUMS.txt`：

```sh
shasum -a 256 -c SHA256SUMS.txt
```

这是预发布软件，使用了 macOS 私有框架和未公开的蓝牙语音通道。反馈可复现问题时，请附上 macOS 版本、遥控器固件版本和相关日志。

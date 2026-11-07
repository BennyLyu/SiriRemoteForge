# SiriRemoteForge v0.2.0-beta.5

本版全面重做 HyperVibe 的安装、首次启动和恢复流程：新增 macOS 原生安装器、实时“系统检查”以及贯穿应用的权限状态提示，减少启动时反复弹出的授权窗口。

## 选择一个下载包

- **原生完整安装器（推荐）：** `HyperVibe-Full-Setup-0.2.0-beta.5-arm64.pkg`。一次管理员授权即可安装 HyperVibe、Siri Remote Mic、按需语音采集服务和卸载器；安装后短暂重启系统音频，监控 `coreaudiod` 25 秒并在失败时回滚，然后打开“系统检查”。
- **仅应用：** `HyperVibe-0.2.0-beta.5-macOS-arm64.zip`。解压后将 `HyperVibe.app` 移至“应用程序”，再按住 Control 点按并选择一次“打开”。
- **旧版完整安装包：** `HyperVibe-Full-Setup-0.2.0-beta.5-arm64.zip`。供无法使用安装器的用户使用，完成后同样进入“系统检查”。

这些测试版未经 Apple 公证；应用为 ad-hoc 签名，原生安装包未签名，因为项目尚无 Developer ID Installer 证书。首次打开时按住 Control 点按并选择“打开”；不要全局关闭 Gatekeeper。需要 macOS 13 或更高版本、Apple 芯片和第三代 USB-C Siri Remote。

## 可恢复的原生安装

- 使用 Apple 标准安装界面，含双语欢迎、安装详情、许可和完成页面。
- 替换软件前验证封装内容和应用内嵌签名。
- 保留现有 `~/.config/siriremote/config.jsonc`；只有当前用户没有配置时才创建默认配置。不跟随配置符号链接，也不递归更改用户主目录内文件的所有权。
- 先备份应用、卸载器、HAL 驱动、麦克风服务和 LaunchDaemon；安装或音频监控失败时恢复完整旧版本。安装器只停止 HyperVibe UI，不会停止 HyperVibe Host。
- 安装诊断写入 `/var/log/hypervibe-install.log`，并在“应用程序”中放置原生卸载器。

## 重做首次设置与恢复界面

- 使用空间充足的原生窗口，默认 760×680 点，可在 700×620 至 1100×900 点之间调整；只有功能列表滚动，避免窗口超出屏幕。
- 设置分为五部分：语言、核心控制权限、语音与高级动作、实时遥控器连接、最终就绪状态；显示分段进度和方向一致的页面转场。
- 使用 SF Symbols、绿/橙/红状态标识、能力卡片和专用操作/设置按钮区分必需与可选能力。
- 汇总安装位置、核心权限、遥控器连接和语音状态，并显示登录时启动控制及注册错误。
- 窗口可关闭或稍后继续；必需设置未完成时不能标为完成。语言和完成状态会被记住，回访用户从相关检查继续。
- 新界面和安装器页面均提供英文与简体中文。

## 实时系统检查

- 将核心权限 Accessibility、Input Monitoring 与按功能启用的 Microphone、Automation 区分开。
- 在一处检查 Siri Remote Mic 组件、Apple 单独分发的 PacketLogger、遥控器连接、安装位置和登录项状态。
- 只有用户点按相应操作时才请求隐私权限；普通启动只做被动检查，不触发无说明的 TCC 请求。
- 从系统设置返回后自动刷新状态；授权 Accessibility 或 Input Monitoring 后立即重新连接受影响的事件监听器或 HID 管理器，无需重启应用。
- 可从菜单栏、设置中的“打开系统检查…”或 `HyperVibe --system-check` 打开；菜单栏持续显示“权限：就绪/需要操作”，需要处理时可点按状态跳转到对应页面。
- 启动时可发现后来被撤销的权限。正常状态轮询始终是被动的，不会自行触发 TCC 对话框。

## 麦克风说明

遥控器蓝牙语音采集还需要从 [Xcode Additional Tools](https://developer.apple.com/download/all/?q=Additional+Tools+for+Xcode) 获取 Apple 的 PacketLogger。Apple 不允许重新分发，因此项目不附带。系统检查会显示其安装状态并提供下载链接；即使没有 PacketLogger，指针、滚动、按键和内置麦克风回退仍可使用。

## 验证与下载校验

- SiriRemoteCore 的 129 项测试通过；arm64 应用、麦克风路由器、HAL 插件和采集服务可在 macOS 13+ 构建。
- 三个下载文件已校验校验和。发布审计展开安装包和压缩包，检查内嵌签名、封装内容、安装器脚本和 XML、应用二进制一致性，并排除个人配置、PacketLogger、设备标识、私有路径和正式媒体素材。
- 单独安装并重启稳定签名的本地构建后，原有 Accessibility/Input Monitoring 身份未改变，HID 检测和媒体键事件监听器均重新连接成功。

使用随附的 `SHA256SUMS.txt` 校验下载文件：

```sh
shasum -a 256 -c SHA256SUMS.txt
```

这是预发布软件，使用了 macOS 私有框架和未公开的蓝牙语音通道。反馈可复现问题时，请附上 macOS 版本、遥控器固件版本和相关日志。

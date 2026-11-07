# SiriRemoteForge v0.2.0-beta.10

本版完善 Siri Remote 对 ChatGPT 桌面版听写的控制，并增加常驻层级提示的显示开关。应用内更新源也已切换到 BennyLyu/SiriRemoteForge。

## 从旧版本升级

- **已安装此前发布渠道版本的用户：**请先从 [beta.10 Releases 页面](https://github.com/BennyLyu/SiriRemoteForge/releases/tag/v0.2.0-beta.10) 手动安装一次本版。旧版本内置的更新源仍指向原仓库，因此无法通过旧版本的“检查更新”发现这个 Fork 的 beta.10。安装后，后续检查更新会使用 BennyLyu/SiriRemoteForge。
- **首次安装或需要安装/修复 Siri Remote 麦克风组件：**使用 `HyperVibe-Full-Setup-0.2.0-beta.10-arm64.pkg`。
- **只更新应用：**使用 `HyperVibe-0.2.0-beta.10-macOS-arm64.zip`。这不会重装或更新虚拟麦克风、路由器和后台采集服务。

需要 Apple 芯片和 macOS 13 或更高版本。Beta 应用使用 ad-hoc 签名，未经 Apple 公证；首次打开时按住 Control 点按并选择“打开”，不要全局关闭 Gatekeeper。安装包不含个人配置或 API 凭据。

## Siri Remote 控制 ChatGPT 听写

- 在 ChatGPT **Settings → Voice → Dictation → Single-tap shortcut** 中设置快捷键；在 HyperVibe 中将 Siri 按钮的轻点绑定设为相同快捷键，并将长按动作选择为 **ChatGPT 听写**。
- 轻点继续按原映射发送快捷键。长按达到阈值时，HyperVibe 自动发送同一快捷键开始听写；松开时再发送一次结束听写。长按动作无需另设快捷键。
- 只有 ChatGPT 位于前台时，长按才控制 ChatGPT 听写。听写期间会隐藏 HyperVibe 的层级状态卡和长按提示，避免遮挡 ChatGPT 的听写界面；松开后恢复。
- 在菜单栏菜单中可切换 **显示层级**，控制常驻的层级状态卡是否显示。该开关会保存设置；隐藏状态卡不会影响遥控器映射或听写。
- HyperVibe 不读取 ChatGPT 的私有快捷键设置；两处快捷键需由用户手动设为一致。本功能不需要 OpenAI 或 DeepSeek API 密钥。

## 下载与完整性

Release 提供应用 ZIP、完整安装 ZIP、原生安装 PKG 和 `SHA256SUMS.txt`。安装前可执行：

```sh
shasum -a 256 -c SHA256SUMS.txt
```

自动更新通过 Sparkle 验证应用归档签名。自动更新只替换 HyperVibe 应用；若要安装或修复 Siri Remote Mic 系统组件，请使用完整安装包。

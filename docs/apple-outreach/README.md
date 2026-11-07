# Apple 沟通材料包

本目录包含一套可供审阅的 HyperVibe 沟通材料。HyperVibe 是 SiriRemoteForge 仓库发布的 macOS 应用。材料将项目介绍为已经公开、非保密的概念验证，并针对 Apple 产品和开发者 API 提供具体反馈；它不是保密的产品提案。

## 建议顺序

1. 按照 [`feedback-assistant.md`](feedback-assistant.md) 分别提交两项功能增强请求。每份报告只写一个技术请求，并将返回的 Feedback ID 加入邮件。
2. 将 `website/apple/` 中的英文演示页发布到公开网址。不要给 Apple 发送仅 Tailnet 可访问的地址。
3. 将 [`email-accessibility.md`](email-accessibility.md) 发送至 [`accessibility@apple.com`](mailto:accessibility@apple.com)，只附一页 PDF，并链接到公开演示、仓库、版本发布页和 Feedback ID。
4. 如有帮助，可通过 [Apple 产品反馈](https://www.apple.com/feedback/) 在 macOS 或 Apple TV 类别下提交精简版。不要在此重复提交两项技术 API 报告。

Apple 明确列出无障碍反馈邮箱，用于功能增强请求和用户故事：[Apple 无障碍支持](https://support.apple.com/en-us/111749)。API 和 SDK 增强请求应提交到 [Feedback Assistant](https://developer.apple.com/feedback-assistant/)。

## 文件

- [`email-accessibility.md`](email-accessibility.md)：主邮件、主题行和较短的后续邮件。
- [`one-page-brief.md`](one-page-brief.md)：单页附件的可编辑源文件。
- [`feedback-assistant.md`](feedback-assistant.md)：两份可单独提交的 API 增强报告。
- [`samples/PublicAPIProbe`](samples/PublicAPIProbe)：可构建的 Game Controller 和 CoreAudio 探测工具，只使用 Apple 公开框架。
- [`demo-script-60s.md`](demo-script-60s.md)：仅动画、60 秒的产品演示脚本。
- [`public-audit.md`](public-audit.md)：已核实的说法、需要限定的说法和最终发送检查清单。
- [`HyperVibe-Apple-Overview.pdf`](../../output/pdf/HyperVibe-Apple-Overview.pdf)：渲染后的一页附件。
- [`website/apple/index.html`](../../website/apple/index.html)：为 Apple 审阅者准备的公开英文产品页。

## 材料中统一使用的事实

- 产品：HyperVibe，由公开的 SiriRemoteForge 仓库分发。
- 主机：Apple 芯片、macOS 13 或更高版本。
- 配件：通过蓝牙配对的第三代 USB-C Siri Remote。
- 核心控制：指针移动、点击、锁定拖动、加速圆环滚动、按键/圆环映射、应用感知配置和可配置图层。
- 手势：轻点、双击、三击，以及最多三个松开选择式长按阶段。
- 反馈：常驻精简状态小组件，以及可选的长按和图层 HUD。
- 语音：实验性的 Siri Remote 麦克风路径，带自动切换到内置麦克风的回退方式；遥控器路径依赖未公开协议和需单独获取的 Apple 诊断工具。
- 配置：一份热重载 JSONC 文件，供图形界面、脚本和编程代理共用。
- 当前证据：公开 Beta 发布版、原生安装器、源码和发布审计；已审阅工作树的核心测试为 130/130 通过，公开 beta.5 发布时对应的测试数则是 129/129。

## 发送前

- 将邮件和 Feedback 报告中的 `PUBLIC_DEMO_URL` 替换为最终公开 HTTPS 地址。
- 提交两份 Feedback 报告，并填入对应 ID。
- 重新确认所记录的 macOS、Xcode 和 SDK 版本，然后将探测工具源码及清理过的输出附到对应报告。
- 确认当前公开版本仍可通过 <https://github.com/HOLODATA-COM/SiriRemoteForge/releases/tag/v0.2.0-beta.5> 获取。
- 使用未登录的浏览器打开 PDF 和所有公开链接。
- 邮件只附 PDF。不要在普通邮件中发送源码归档、数据包日志、设备标识或私人诊断信息。

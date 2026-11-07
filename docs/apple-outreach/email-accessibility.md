# 主邮件

**收件人：**accessibility@apple.com
**主题：**无障碍反馈：使用 Siri Remote 实现完整的单手 Mac 控制

Apple 无障碍反馈团队：

我叫 Wenqian Zhang，开发了 HyperVibe。这是一个公开的 macOS 概念验证，可以把已配对的第三代 Siri Remote 变成 Mac 的完整可配置输入面。

我围绕一个简单的缺口构建了它：语音输入能力越来越强，但以语音为主的编程、演示或行动辅助流程，仍会在用户需要指向、滚动、选择、拖动、切换上下文或确认结果时中断。此时用户不得不重新拿起鼠标或触控板。HyperVibe 将这些操作集中到一台小型控制器上：

- 触控表面移动指针、执行点击，并支持按住拖动；
- 外圈提供加速的竖向或横向滚动；
- 每个按键可配置轻点、双击、三击和分阶段长按；
- 应用 × 图层映射会随当前任务切换控制方式；
- 常驻状态界面显示当前图层、操作、长按进度和实时语音状态；
- 原生图形界面、脚本和编程代理共用一份 JSONC 配置。

这一方案在两种场景中特别有用：站立演示时，演讲者不必回到笔记本旁操作演示；语音辅助编程时，说完提示仍可用同一个控制器继续操作。

该原型也暴露了平台限制。Apple 为 Apple TV 记录了 Siri Remote 游戏输入，但在我们的 macOS 测试中，公开框架无法提供已配对第三代遥控器的完整输入流，也没有提供麦克风路径。因此，当前研究构建依赖私有 MultitouchSupport 行为处理触控，并依赖未公开的蓝牙语音路径。这使它无法成为长期稳定、兼容沙盒的 Mac 输入选项。

我分享的是一个已经公开的项目，以及针对 Apple 产品和开发者 API 的具体无障碍反馈，不是保密产品创意。无障碍、输入或相关开发者框架团队是否愿意查看实际演示，并考虑为以下功能提供受支持的 macOS API？

1. 已配对 Siri Remote 的按键、圆环和触控表面输入；
2. 经用户授权的 Siri Remote 麦克风采集，作为标准音频输入或配件音频流。

演示：PUBLIC_DEMO_URL
源码：https://github.com/HOLODATA-COM/SiriRemoteForge
当前版本：https://github.com/HOLODATA-COM/SiriRemoteForge/releases/tag/v0.2.0-beta.5
Feedback Assistant：INPUT_FEEDBACK_ID 和 AUDIO_FEEDBACK_ID

随信附有一页项目概览。如 Apple 有合适的渠道，我也愿意进行简短的实时演示，或提供最小化诊断项目。

此致，

Wenqian Zhang
GitHub：https://github.com/JamesZwq
邮箱：zhangwenqian6915@gmail.com

---

# 简短跟进邮件

**主题：**跟进：将 Siri Remote 作为受支持的 Mac 无障碍输入设备

Apple 无障碍反馈团队：

我想跟进 DATE 分享的公开 HyperVibe 概念验证。它展示了如何用一台已配对的 Siri Remote 完成指针移动、加速圆环滚动、点击和拖动、上下文按键图层、可见的长按反馈以及语音输入。

两项具体 API 请求已分别提交，编号为 INPUT_FEEDBACK_ID 和 AUDIO_FEEDBACK_ID。60 秒演示位于 PUBLIC_DEMO_URL，源码继续公开在 https://github.com/HOLODATA-COM/SiriRemoteForge。

如果其他 Apple 团队更适合负责这类反馈，烦请转交给相关团队。

此致，

Wenqian Zhang

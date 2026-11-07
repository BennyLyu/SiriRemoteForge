# Feedback Assistant 提交材料

请将以下内容拆成两份独立报告提交。Apple 将 Feedback Assistant 列为提交 SDK、API、工具和服务增强请求的渠道：<https://developer.apple.com/feedback-assistant/>。

类别名称可能随操作系统版本变化。请选择最接近的当前 macOS 开发技术类别，并保留以下标题和技术范围划分。

## 报告 1——macOS 上受支持的 Siri Remote 输入

**建议标题**
增强请求：为已配对 Siri Remote 的按键、圆环和触控输入提供受支持的 macOS API

**分类**
增强 / macOS / 开发者技术 / Game Controller 或 Human Interface Device

**环境**

- macOS：26.5.1 (25F80；请在提交机器上重新确认)
- 硬件：Apple 芯片 Mac
- 配件：通过蓝牙配对的第三代 USB-C Siri Remote
- Xcode/SDK：Xcode 26.6 (17F113)、macOS SDK 26.5（提交时重新确认）
- 示例项目：`PublicAPIProbe/InputProbe`

**问题**

已配对的第三代 Siri Remote 可以作为 Mac 的精巧替代输入设备，用于无障碍操作、站立演示和语音辅助流程。Game Controller 已记录 Siri Remote 配置文件，以及第二代 Siri Remote 在 Apple TV 上的支持情况。但在我们的 macOS 第三代遥控器测试中，现有公开模型无法提供完整交互所需的输入。底层 HID 接口可以部分观测到类似按键的事件，但应用无法通过语义稳定、受支持的公开 API 获取系统使用的完整触控表面数据流。也没有公开的应用路由模型，可让应用明确采集遥控器输入而不与系统行为冲突。

**观察该限制的步骤**

1. 通过蓝牙将第三代 Siri Remote 与 Mac 配对。
2. 运行随附的最小应用，并枚举公开支持的 Game Controller 和 HID 输入。
3. 按下每个实体按键，转动/轻点点击环，并在触控表面上移动一根手指。
4. 将公开事件与实现指针移动、圆形滚动、点击拖动和可靠按键按下/松开状态所需的输入进行对比。

**当前结果**

在已测试的 macOS 环境中，没有公开 API 提供完整且有文档说明的第三代 Siri Remote 输入配置。可运行的概念验证必须组合原始 HID 行为和私有 MultitouchSupport 行为才能获得触控数据，因此不适合沙盒应用或长期维护的产品。

**期望结果**

请在 GameController/GCPhysicalInputProfile 中扩展现有 Siri Remote 模型，使其支持在 macOS 上配对的第三代遥控器；或者提供等效且有文档说明的配件输入框架。建议公开以下能力：

- 连接、断开、稳定设备身份、电量和能力发现；
- 为 Menu/Back、TV、Play/Pause、Mute、Power、音量、圆环方向和中心点击（若硬件支持）提供带时间戳的按下/松开事件；
- 在硬件支持时，提供归一化触摸坐标、触点开始/移动/结束、触点大小或有文档说明的点击/压力信号，以及多点触控能力；
- 明确区分外圈手势与一般表面移动；
- 为前台、后台、独占和系统共享使用提供选择加入式路由策略；
- 提供适用于无障碍输入软件、兼容 App Sandbox 的授权或 entitlement 模型。

**用户影响和使用场景**

随附 HyperVibe 演示使用触控表面执行指针加速、精确和快速圆环滚动、锁定拖动、应用 × 图层命令和可见长按反馈。用户说完语音输入后可以继续操作 Mac；演讲者则可离开桌边控制正在运行的 Mac。问题不在于缺少交互设计，而在于缺少受支持的 API 路径来实现一个已经通过公开概念验证展示可行的交互方式。

**附件**

- 不含个人桌面内容的 60 秒 MP4
- 一页 PDF
- `PublicAPIProbe` 源码和清理后的 `InputProbe` 控制台输出
- 清理后的能力/事件矩阵；省略蓝牙地址、序列号和无关数据包日志

**公开参考**

- 演示：PUBLIC_DEMO_URL
- 源码：https://github.com/HOLODATA-COM/SiriRemoteForge
- 当前版本：https://github.com/HOLODATA-COM/SiriRemoteForge/releases/tag/v0.2.0-beta.5
- Siri Remote 现有模型：https://developer.apple.com/documentation/gamecontroller/gcmicrogamepad
- Apple TV 示例：https://developer.apple.com/documentation/gamecontroller/letting-players-use-their-second-generation-siri-remote-as-a-game-controller

---

## 报告 2——macOS 上受支持的 Siri Remote 麦克风路径

**建议标题**
增强请求：在 macOS 上提供经用户授权的 Siri Remote 麦克风采集

**分类**
增强 / macOS / 开发者技术 / Core Audio 或 Bluetooth

**环境**

- macOS：26.5.1 (25F80；请在提交机器上重新确认)
- 硬件：Apple 芯片 Mac
- 配件：通过蓝牙配对的第三代 USB-C Siri Remote
- Xcode/SDK：Xcode 26.6 (17F113)、macOS SDK 26.5（提交时重新确认）
- 示例项目：`PublicAPIProbe/AudioInputProbe`

**问题**

第三代 Siri Remote 内置近讲麦克风，但已配对的遥控器不会显示为受支持的 macOS 音频输入，也没有公开 API 可申请经过授权的按键通话语音流。因此，标准 CoreAudio/AVAudioEngine 客户端、无障碍工具、听写流程和会议应用都无法在 Mac 上使用它的麦克风。

**观察该限制的步骤**

1. 将 Siri Remote 与 Mac 配对。
2. 运行随附的 `AudioInputProbe`；它只使用公开 API 枚举 CoreAudio 输入设备。
3. 查看报告中的设备名称、UID、制造商、传输类型和标称采样率。
4. 按住遥控器 Siri 键后再次枚举。如果出现新的公开端点，请尝试在标准、经授权的 CoreAudio 采集客户端中选择它。

**当前结果**

没有可用的受支持遥控器麦克风输入或配件音频流。HyperVibe 研究原型只能通过与未公开的蓝牙语音路径交互，并将解码音频路由到虚拟 CoreAudio 设备，来演示语音采集。此实现容易受系统版本变化影响，还依赖无法重新分发的诊断工具。

**期望结果**

请提供标准 CoreAudio 输入端点，或有文档说明的配件音频 API，并具备以下能力：

- 明确的用户授权和清晰的录音指示；
- 有文档说明的按键通话启动和松开生命周期；
- 采样格式、时钟、缓冲区、中断和路由变更语义；
- 连接丢失和回退事件，使应用可切换到 Mac 麦克风，而不会误把遥控器音频继续当作实时音频；
- Siri/系统使用和第三方采集之间的共存规则；
- 兼容 App Sandbox 的 entitlement 和审核流程。

**用户影响和使用场景**

如果提供受支持的路径，一台控制器即可同时承担命令输入和近讲语音输入。在 HyperVibe 的实际界面中，按住侧边键会显示由 25 根柱状条构成的实时语音界面，同时仍可用同一设备控制指针、滚动和执行命令。遥控器音频过期时，应用回退到 Mac 内置麦克风。公开 API 可让普通 Mac 音频客户端以可靠、经授权的方式使用此行为。

**附件**

- 60 秒 MP4 和一页 PDF
- `PublicAPIProbe` 源码和清理后的 `AudioInputProbe` 控制台输出
- 清理后的输入设备列表，以及通过 Feedback Assistant 收集的相关 sysdiagnose
- 不要附加公开数据包采集、凭据、蓝牙地址或序列号

**公开参考**

- 演示：PUBLIC_DEMO_URL
- 源码：https://github.com/HOLODATA-COM/SiriRemoteForge
- 当前版本：https://github.com/HOLODATA-COM/SiriRemoteForge/releases/tag/v0.2.0-beta.5
- 技术背景：https://github.com/HOLODATA-COM/SiriRemoteForge/blob/main/docs/mic-reverse-engineering.md

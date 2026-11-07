# 纯 macOS 的语音管线——Siri Remote Mic

**当前状态：**整条管线已实现并集成。HAL 设备、实时 PacketLogger 路由器、按需采集守护进程和内置麦克风回退功能由 `dist/build-release.sh` 构建，并随高级版“完整安装”资源发布。修复后的 HAL 插件已通过真实主机验证，并由 `coreaudiod` 自动回滚监视器保护。本功能仍属实验性质：它依赖未公开的蓝牙行为和需要单独下载的 Apple 工具。

最终会提供一个全系统可见、名为 **Siri Remote Mic** 的输入设备。按住 Siri 键时，它传送遥控器的语音；当该语音流过期时，设备会平滑切换到 Mac 内置麦克风。不会改动内置设备的驱动、格式或默认设备选择。纯 Mac 方案，无需 ESP32 或 DriverKit 权限。

> 本文件最初是实现过程的实验记录。以下按日期整理的验证和事故记录作为工程证据保留；较早阶段的状态描述仅供历史参考，不代表当前状态。

## 管线

```
① 启用 HCI 日志       defaults write /Library/Preferences/com.apple.MobileBluetooth.debug \
   （需要管理员权限，    HCITraces -dict …HCISkipAuth,RawAudioTrace,HIDTrace… ; killall -30 bluetoothd
    可恢复）            ← 这一步绕过“需要蓝牙描述文件”的限制
② 采集 HCI            packetlogger convert            （实时保存为 .pklg 文件；加 -s 则输出到标准输出）
③ 提取语音            packetlogger audio -i cap.pklg  或手动解析 -f nhdr 流中的报告 0xFA
④ Opus 转 PCM         OpusVoiceDecoder               （连续 PCM；实时设备必须经过此步骤）
⑤ 虚拟麦克风设备       自有 CoreAudio HAL 插件          （“Siri Remote Mic”；闲置时输出静音）
```

## 输出阶段——已决定：方案 B，自建虚拟麦克风（2026-07-23）

不是把语音识别成文本再注入。输出是一个**全系统虚拟输入设备**，名为“Siri Remote Mic”，会出现在所有应用的麦克风选择器中（Zoom、QuickTime、语音备忘录和 macOS 听写都可使用；选择此设备即可让系统听写输入遥控器语音）。之所以不在运行时复用 BlackHole，是因为我们要提供一个带内置回退功能的品牌设备，而不是要求用户运行一个可见的路由器应用。

- **技术方案：**CoreAudio `AudioServerPlugIn`（HAL 插件），安装到 `/Library/Audio/Plug-Ins/HAL/`，由 `coreaudiod` 加载。这是 BlackHole 和 Loopback 使用的机制。关键在于它运行于**用户空间，不需要 DriverKit 权限或配置描述文件**；正是缺少 DriverKit HID 权限导致早期 `SiriRemoteMicDriver` DEXT 无法继续。临时签名可用于本地安装；BlackHole 是现成的实现范例。
- **已实现回退：**发布应用会按需启用内置麦克风馈送器，并写入第二个共享内存环形缓冲区。HAL 插件优先选择新鲜的遥控器语音，否则改用内置麦克风缓冲区，并在每次切换时线性淡入淡出。若两个来源都不可用，则输出静音。
- **进程间通信：**插件运行在 `coreaudiod` 内，采集应用单独运行，因此解码后的 PCM 通过无锁 **POSIX 共享内存环形缓冲区** `/SiriRemoteMicAudio` 传给插件。
- **需求检测：**路由器观察 CoreAudio 属性 `kAudioDevicePropertyDeviceIsRunningSomewhere`。无需插件向外写入反向 IPC 标记。已通过系统设置打开和关闭该设备进行验证。

### 此阶段引入的新难题：抖动缓冲

语音转文字接口（Speech.framework）可以容忍突发的 `append`；但**实时音频设备按照稳定时钟拉取数据**，BLE 语音帧则以约 60 Hz 突发到达。缺少**抖动/环形缓冲区和时钟匹配**会导致声音卡顿、断流。这是 VoIP 的常规做法，但确实涉及 DSP 工程；与方案 A 相比，选择方案 B 的主要工程风险就在这里。

### 系统安装安全边界

2026-07-23 的一次安装测试触发了 CoreAudio 属性通知/协调风暴：`coreaudiod` CPU 使用率超过 100%，Mac 几乎无法操作。事故期间移除了插件，重启后风暴消失；系统音频偏好 plist 仍将 `SiriRemoteMic_UID` 记录为首选输入索引 0，因此有意没有手动改动。

**修复后的系统验证已通过（2026-07-23）。**在自动回滚监视器保护下安装修复后的程序包（`coreaudiod` 持续达到 85% 即自动卸载）。加载协调在约 2 秒内稳定为空闲；之前导致风暴的客户端打开 I/O 路径峰值仅 **6%**；CoreAudio 消费者通过共享缓冲区收到 147,456 个非静音样本；设备没有变成默认输入。测试主机保持稳定（`coreaudiod` 空闲约 3%）。此前的风暴已修复，并非只是被绕开。公开安装通过受监视器保护的“完整安装”应用进行；组件开发使用 `mic/driver/install-watchdog.sh`。

- 虚拟设备是**持久系统组件**（安装需要管理员权限；通过 `killall coreaudiod` 加载，会造成全机短暂音频中断；重启后仍然存在），不是每次使用时才安装。
- 以后进行任何系统级测试，都必须先取得单独明确同意，配备自动回滚/监视器，限定测试时长，并独立监控 `coreaudiod`。

### 当前发布边界

真实采集解析器/解码器/路由器、HAL 合约与 I/O 模拟、实时 PacketLogger 管道、按需监督程序，以及非活动状态下的内置音源均已通过验证。“完整安装”每次安装仍保留 25 秒监视器。后续重点是覆盖更多 macOS 版本和遥控器固件，并加强长时间运行时的时钟漂移处理，以及应对 Apple 私有跟踪格式变化。

## 已完成并验证的部分（2026-07-23，离线）

- **③/④ 编解码路径——`OpusVoiceDecoder.swift`。**基于 Homebrew libopus 1.6.1 的纯解码封装。使用 `./build-test.sh` 运行 `test_decoder.swift`，通过编码→解码往返验证：输出 960 个样本且非静音；同时验证了丢帧隐藏、Speech 可用的 `AVAudioPCMBuffer` 和 WAV 导出。遥控器帧为 CELT-only **宽带（16 kHz）、20 ms、TOC 0xb8**；解码器以 48 kHz 创建，libopus 会在内部升采样，因此每帧输出 960 个可供 Speech.framework 使用的样本。
- **PacketLogger 命令行工具勘察**（Additional Tools 26.6 中的 `packetlogger` 二进制，位于 `/Applications/PacketLogger.app/Contents/Resources/packetlogger`）：
  - 不带 `-i` 的 `convert -o FILE.pklg` 会**实时采集**并写入 pklg；加 `-s` 时输出到标准输出。
  - `-f nhdr` 的字段为 `timestamp · name · connection-handle · direction · raw-bytes`，这正是实时解析器需要的数据形态。（RemotePilot 使用 `convert -s -f nhdr`。）
  - `audio -i FILE.pklg -o DIR -f <freq>` 可直接从 HCI 跟踪文件提取语音；因此阶段③的 HCI/ATT 重组工作，可能可以交给 Apple 工具处理文件模式。

## 端到端试验——已完全确认（2026-07-23）

**整条输入管线已验证，且已确认声音可听。**用户按住 Siri 键并说话；我们采集、解析、解码并播放了语音，用户确认声音清楚，确定继续。一次实时采集中，具体数据如下：

- 遥控器连接句柄为 `0x0406`；按住 Siri 后出现约 111 字节的数据包，时间窗口内收到 **877 个 RECV 数据包**。
- 提取出 **804 个语音帧**（约 16.1 秒、每秒 50 帧、每帧 20 ms），全部通过 `OpusVoiceDecoder` 成功解码（0 错误），RMS 为 3232（非静音）。WAV 文件已写出并播放；用户反馈“很清楚”。

**精确语音帧格式——实时解析器的参考（已通过真实采集验证）：**

- 语音数据是 **ATT Handle-Value-Notification（opcode `0x1B`）**，位于实际观测到的语音属性句柄（`0x0035` 或 `0x0036`），经遥控器动态 ACL 连接句柄到达。对应原始签名为 `04 00 1B 35 00` 和 `04 00 1B 36 00`（L2CAP CID 0x0004 = ATT；句柄采用小端序）。句柄不严格依赖硬件代际：A2854（产品 ID `0x0315`、固件 `0x0033`）已在 macOS 26.6.2 上独立确认使用 `0x0036`。
- ATT 值的布局为 `[4 字节序列号/头部][1 字节 Opus 长度 L][L 字节 Opus 帧]`；Opus 帧以 **TOC `0xB8`** 开始（CELT-only 宽带，20 ms）。
- 每帧以 48 kHz 单声道解码为 960 个样本；约每秒 50 帧。
- **无需发送启用写入：**遥控器正常与 macOS 配对（HyperVibe 只需运行），按住 Siri 键后数据流会自行出现；我们只监听。`packetlogger audio` 无法提取这些数据（它不识别此 GATT 语音），因此需要自建解析器和 `OpusVoiceDecoder`。

额外一次 `0x0036` 采集提取出 619 个语音帧；全部解码成功，无错误且 PCM 非静音（RMS 3774.7，峰值 32767）。没有公开原始采集文件，因为其中含有私人语音和设备标识。

可以用 `tmp/decode_voice.py` 对采集文件离线复现（ctypes 调用 libopus），解析 `mic_raw.txt` 并写出 WAV。该脚本与 `OpusVoiceDecoder.swift` 一起对应阶段②至④，现已通过真实数据验证。

## 虚拟设备验证记录（2026-07-23）

- 早期有界测试已端到端证明共享内存机制可用：**144,384 个样本，RMS 0.162295，峰值 0.25**；源音频 RMS 为 0.178589。
- 早期版本安装测试曾导致严重 CoreAudio 风暴，见上文。后续修复通过离线和有界真实主机验证；HAL 插件属于系统级代码，因此发布安装器仍保留自动回滚。
- 已确认的离线修复包括：仅输入时间戳锚点、UID 限定符检查、静态单设备/无 Box 对象图、拒绝继承来的隐藏/输出对象、属性大小正确性、单声道/仅输入能力声明，以及时钟控制可变性。
- `srm_driver_contract_test.c` 在带伪造主机的进程内加载程序包，并通过 canary、I/O 时间戳和静音情况验证所有公开属性；同时执行 20,000 次稳定协调循环，通知数为零。优化版、ASan 和 UBSan 运行均通过。
- `srm_capture_test.c`、`srm_test_writer.c` 和 `srm_usage_monitor.c` 仍是诊断工具，不能证明修订后的程序包已由真实 CoreAudio 主机安全加载。

### 首次确认机制的方法（2026-07-23，较早记录）

在用户提供管理员密码后，通过无人值守且完全可逆的探测脚本 `tmp/hci_capture_probe.sh` 执行步骤①至②，无需按住 Siri：

- `defaults write …com.apple.MobileBluetooth.debug HCITraces -dict …HCISkipAuth,RawAudioTrace,HIDTrace…` 写入了正确的布尔字典（已读回验证）；`killall -30 bluetoothd` 重新加载设置；`packetlogger convert -o cap.pklg` 生成了非空的 15 KB HCI 跟踪文件，可解码为 219 行。2026-07-20 时同类操作返回 `bluetoothd: "Bluetooth Profile Required"` 且没有数据；现已确认可通过调试默认值（而非 iOS `.mobileconfig`）启用。
- 首次真实看到 `-f nhdr` 行格式：`timestamp · name · handle · direction · raw-hex`（例如广播报告，以及句柄 0x0001 上的 Magic Trackpad）；实时解析器读取的就是这种格式。
- 清理情况已验证：删除了原本不存在的调试域、重新加载 bluetoothd，并确认仍有且只有一个不带参数的 HyperVibe 实例运行。系统已恢复到之前状态。

## 构建依赖

本地组件构建使用 `brew install opus`（BSD 许可，与 GPL-3.0 兼容）。发布构建会下载带固定校验和的官方源码，并按 macOS 13 部署目标编译；路由器将其静态链接，因此目标 Mac 不需要 Homebrew。Opus 二进制分发声明保存在 `router/Opus-LICENSE.txt` 和“完整安装”负载中。

## 注意事项（源自 HANDOFF 的技术限制）

需要管理员权限；依赖 Apple PacketLogger 和未公开的调试偏好设置，系统更新可能改变这些设置；重启 bluetoothd 会短暂断开所有蓝牙连接。这是对**你自己的 Mac 上已配对遥控器**进行特权 HCI 采集，个人使用可行，但向他人发布时必须谨慎评估。

# 第三代 Siri Remote 麦克风逆向记录

本文记录在 macOS 上读取遥控器内置麦克风并将其作为系统音频输入的研究结果。项目目标是让 Siri Remote 的近讲麦克风可供 Mac 应用使用；目前通过 `mic/` 中的蓝牙采集与虚拟音频设备实现。

## 硬件与连接

- 目标设备为 2022 年第三代 USB-C Siri Remote，型号包括 A2843/A2854。HID 产品 ID 为 `0x0315`（789），Apple 厂商 ID 为 `0x004C`。蓝牙设备名称会显示遥控器序列号。
- 遥控器通过 BLE 连接，不提供 A2DP/HFP 等标准蓝牙音频配置，因此它不是普通蓝牙耳机麦克风。
- 它不会出现在 `system_profiler SPAudioDataType` 的 CoreAudio 设备列表中；macOS 通常只列出 Mac 内置麦克风、iPhone 连续互通麦克风等标准设备。
- macOS 会为遥控器暴露七个虚拟 IOHID 接口。音频接口为接口 5，Usage `0x0C/0x04`，`MaxInputReportSize = 209`，并加载 `AppleEmbeddedBluetoothAudio`。接口 2 承载三字节 `0xFB` 按键报告；接口 0（`0xFF00/0x0B`）承载 Apple 设备管理命令；另有两个接口使用 Usage Page `0x20`。
- 普通应用模式会占用遥控器接口。`--direct-ptt` 诊断模式仅独占音频接口，其余接口保持非独占。

## 已确认的协议事实

- 第三代遥控器在线路上使用报告 ID `0xFA` 传输麦克风音频；每帧含 99 字节 Opus 负载。
- 已观测到的编码为 Opus CELT-only 宽带、48 kHz 单声道、每帧 20 ms（960 个样本），TOC 字节为 `0xB8`。
- 第三代固件通过 Feature Report 启用输入，而不是通过 Output Report；启用数据为 `0xAF`。管理 HID 服务暴露可写的 Feature Report 特征。
- 在 HID 输入层，音频接口的主要 Usage 为 `0x0C/0x04`；Input 和 Feature 报告 ID 为 `0xFF`，数据长度 208 字节（加报告 ID 共 209 字节）。最大 Output Report 长度为一字节，但接口没有声明 Output 元素。
- 声音只会在主机完成激活后传输。最初仅按住 Siri 键时，日志出现按键事件，却没有任何大于 6 字节的音频报告；这表明单独按 Siri 不会启动语音流。

## 激活实验与结果

### HID Feature Report 激活

诊断程序曾对所有接口发送 `[0xAF]`，同时注册七个接口的原始输入报告回调。接口 0、2、4、5（音频）、6 和 7 接受 Feature 写入；接口 1（`0x0D/0x01`）返回 `0xE00002BC`（`kIOReturnError`）。接口 5 的 Feature 写入返回成功，Siri 按下也会触发当秒重新激活所有接口，但十秒讲话期间仍没有音频回调。IORegistry 显示音频接口的 `ReportAvailableCalls=0`、`ReportAvailableRuns=0`。这说明 HID 写入被接受，不代表固件已开始流式传输。

### CoreBluetooth 与系统 HID 所有权

当 macOS 已用其 HID 栈连接遥控器时，应用的 CoreBluetooth 会话无法取得该系统连接的 HOGP 外设。公开 CoreBluetooth 路径因此无法直接访问系统已连接遥控器的原始 GATT 服务和特征。研究过程中还测试了解除配对、让应用自行连接，以及显式发现 HID 服务；系统对该服务的可见性受到限制，应用连接并不能绕过所有权边界。

### Apple 原生 PTT 与隐藏报告

- 调用 Apple Bluetooth Remote 驱动的原生 `PushToTalk` 属性，在测试产品 `0x0315` 上返回 `kIOReturnUnsupported`。
- 隐藏 Feature Report `0x99` 也未能启动语音。诊断程序只在管理接口 0 发送报告 ID `0x99`、单字节 `[0x01]`，20 秒后自动发送 `[0x00]` 并在退出时再次释放。该试验仍没有收到接口 5 的音频报告，IORegistry 的可用报告计数也仍为零。
- 音频接口接收端曾报告 `SetReportCnt=1`、`SetReportErrCnt=0`；但试验期间系统日志没有相应的音频驱动或遥控器驱动事件。这些结果进一步说明 HID 层写入成功与远端激活成功是两回事。

### DriverKit 替代驱动探索

曾完成一个本机 HIDDriverKit 替代驱动概念验证。其匹配条件限定为厂商 76、产品 789 和主要 Usage `0x0C/0x04`，以默认匹配类别和 `IOProbeScore=8000` 参与匹配。驱动启动后直接发送 Feature Report `0xFF [AF]`。Xcode 26.6 成功运行 IIG、编译并链接 DriverKit/HIDDriverKit，生成 arm64 DEXT；AppKit 主程序只在用户明确启用或停用时嵌入并提交系统扩展请求。

但本机无法完成实际替代：个人开发团队无法取得所需 DriverKit HID 能力，系统扩展未能通过 AMFI/配置文件验证。放宽平台安全策略也没有形成可维护的部署方式；AMFI 开关状态会分别导致内核签名验证或扩展进程启动失败。当前项目没有依赖该 DEXT，改由 `mic/` 的用户态 PacketLogger 采集路径实现。

### 原始 GATT 地址与 ATT 句柄

早期有一轮试验把 HID Feature 写入结果误当成原始 GATT 激活成功。后续复核发现，设备的语音通知通过 ATT 特征值句柄传输；必须针对正确 GATT 句柄写入/订阅，HID 接口的成功回执不足以证明这一点。通过 USB-C 连接、解除配对、不同 BLE 状态等试验都不能单独证明遥控器固件允许该激活路径。

macOS 控制已连接 HID 设备的 GATT 服务和原始连接，因此公开 CoreBluetooth 不能自由枚举或订阅系统拥有的连接。PacketLogger 可观察 HCI 数据并将 BLE 通知交给用户态路由器；不同固件会使用不同的 ATT 语音值句柄。当前路由器已支持实测的 `0x0035` 和 `0x0036`，仍会校验接收方向、ATT 通知操作码、序号/长度布局和 Opus TOC；未知句柄（例如 `0x0037`）会被拒绝。

## 当前实现与边界

可用方案使用 Apple PacketLogger 提供的蓝牙 HCI 调试数据：按需采集服务启动 PacketLogger，`srm_router` 解析遥控器专有 BLE 语音通知并解码 Opus，随后写入共享内存环形缓冲区。`Siri Remote Mic` CoreAudio HAL 插件从环形缓冲区读取语音，并在遥控器音频不可用时切换到内置麦克风。

这条路径不需要应用自行向遥控器发送未经验证的语音启动命令。遥控器按键或原生 Voice 流程启动采集后，PacketLogger 捕获系统蓝牙栈产生的 HCI 数据，路由器只接受完成帧校验的通知。服务在有麦克风需求时启动，在需求结束时停止。

当前实现具有以下限制：

- 需要用户另行安装 Apple PacketLogger；本项目不会重新分发它。
- 采集依赖 macOS 私有/未公开行为，包括蓝牙 HCI 调试跟踪和 Apple 的遥控器语音协议；系统更新可能改变行为。
- 启用调试跟踪可能需要管理员权限；重启 `bluetoothd` 会短暂断开蓝牙设备。仅在用户控制的 Mac 和已配对遥控器上操作。
- 209 字节报告大小本身不能证明报告一定会作为输入到达；实际是否有音频必须由通知帧和解码结果确认。
- 只有开始接收有效帧后，靠近遥控器播放的声音才是有效的硬件测试信号。它无法自行解除固件侧的激活限制。

## 当前位置与复现入口

- 主应用位于 `app/`。构建入口为 `./build.sh`；遥控器匹配在 `app/RemoteDetector.swift`，HID 打开/独占逻辑位于 `app/RemoteInputHandler.swift`。
- 麦克风组件位于 `mic/`：`mic/captured/` 是按需 root LaunchDaemon，`mic/router/` 是 BLE 路由器，`mic/driver/` 是 HAL 插件。
- 服务日志位于 `/var/log/srm_captured.log`；应用日志通常位于 `/tmp/hypervibe.log`。公开日志前应移除个人路径、设备序列号和私人采集内容。
- 诊断参数曾包括 `--capture-mic`、`--activate-mic`、`--direct-ptt`、`--dump-reports` 和 `--dump-gatt`。这些参数用于调查，不应视为受支持的用户工作流。
- 规范仓库为 [HOLODATA-COM/SiriRemoteForge](https://github.com/HOLODATA-COM/SiriRemoteForge)。目前实际可用实现和安装要求以 [`mic/README.md`](../mic/README.md) 为准；有关遥控器设备发现的开发要求见 [`HANDOFF.md`](../HANDOFF.md)。

# Siri Remote Mic——CoreAudio HAL 虚拟设备（方案 B）

这是一个名为“Siri Remote Mic”的虚拟**输入**设备，制造商为 *Holodata.au*，供应用显式选择。使用者打开设备后，路由器会将解码后的 Siri Remote 语音送入其中；当遥控器语音过期时，设备会切换到发布应用馈送的内置麦克风共享缓冲区。两个音源都不可用时，设备输出静音。它不会替换或重新配置 Mac 内置设备。

这是自建设备方案 B，运行时不会复用 BlackHole。**BlackHole 只作为实现基础**：它是已在 macOS 26 上验证可加载、采用 GPL-3.0 的 AudioServerPlugIn；我们以它为基础，避免从头手写约 2,000 行繁琐的 CoreAudio 样板代码。

## 署名与许可

`vendor/BlackHole.c` 和 `vendor/BlackHole-LICENSE.txt` 是**原样未修改**的 BlackHole（© 2019–2026 Existential Audio Inc.，GPL-3.0）。本项目的改动位于 `SiriRemoteMic.c`；产品配置集中在 `SiriRemoteMic.config.h`，通过 `clang -include` 注入，因此上游源码仍可同步更新。整个仓库采用 GPL-3.0，纳入 GPL-3.0 代码没有许可冲突；集成时请把 BlackHole 版权信息加入 `NOTICE`。不要提交 BlackHole 图标或品牌素材。

## 构建、安装与移除

```sh
./build.sh       # 本地程序包 + 进程内合约测试；安全，不会接触 coreaudiod
./install.sh     # 失败关闭；系统级安装需明确确认和单独批准
./uninstall.sh   # 移除插件并重启 coreaudiod
```

插件不需要外部运行时依赖，完全使用 Apple CoreAudio。（采集管线仍需要 libopus 和 PacketLogger；它们与此设备分开。）

## 状态

- **已在真实主机验证（2026-07-23，修复后）。**在自动回滚监视器保护下安装了修复版（若 `coreaudiod` 持续达到 85%，即移除插件并执行 `killall coreaudiod`）。之前两条可能导致风暴的路径现已正常：**加载/协调**约 2 秒内回到空闲（单样本 87% 瞬时峰值属于正常的 `coreaudiod` 重启，并非风暴）；**客户端打开 I/O**峰值仅 **6%**（之前事故超过 100%）。CoreAudio 消费者打开设备后，测试生产者通过共享内存送达 **147,456 个非静音样本**（“PASS——共享内存音频已到达 CoreAudio 消费者”）。设备为单声道、48 kHz、仅输入；没有接管默认输入（内置麦克风仍为默认输入，`kCanBeDefaultDevice=false` 生效）。此后 `coreaudiod` 空闲时约 3%。
- **发布集成：**“完整安装”以相同的 25 秒监视器安装修复版，保留旧插件供回滚，并附带专用卸载器。
- **早期风暴（历史记录）：**未修复版本曾使 `coreaudiod` CPU 使用率超过 100%，需要重启；这促使我们实施下文的 10 项 HAL 修复。问题已修复，不只是绕过。
- **机制已验证：**共享内存 IPC 传输可用（此前也独立送达过 144,384 个样本）。
- **修订版 HAL 离线验证通过：**一个静态的 48 kHz 单声道输入设备；没有 Box、镜像设备、输出流或缺失图标；禁用默认设备资格。伪主机进程通过 UID 转换、时间戳、所有公开属性、精确缓冲区大小/canary、无生产者静音和 20,000 次协调循环验证。ASan 和 UBSan 同样通过。
- **路由器离线验证通过：**真实的 3,071 行跟踪中，804/804 帧解码成功，共 771,840 个样本，RMS 3232.3，峰值 32767，没有解码错误。
- **剩余风险：**需要扩大 macOS/固件覆盖范围，并加强长时间运行时的时钟漂移处理。

`install.sh` 明确要求传入系统音频风险令牌；若已有程序包则拒绝覆盖；若旧的首选 UID 仍存在也会单独拒绝安装。没有新批准的有界测试和回滚监视器时，不要绕过这些保护。

## 配置项（`SiriRemoteMic.config.h`）

`kDevice_Name` 为 `"Siri Remote Mic"`；`kManufacturer_Name` 为 `"Holodata.au"`；`kPlugIn_BundleID` 为 `au.holodata.SiriRemoteMic`；`kDriver_Name` 为 `"SiriRemoteMic"`；`kHas_Driver_Name_Format` 为 0（不添加 “2ch” 后缀）。Info.plist 中的工厂 UUID 为 `75E269CD-FD8C-457E-B1A9-BDEED64F001F`，为避免和 BlackHole 安装冲突而重新生成，并映射至原样未改的 `BlackHole_Create` 工厂符号。

安全覆盖项还将 `kPlugIn_HasBox=false`、`kPlugIn_HasDevice2=false`、`kDevice_HasIcon=false`、`kCanBeDefaultDevice=false` 和 `kCanBeDefaultSystemDevice=false`。

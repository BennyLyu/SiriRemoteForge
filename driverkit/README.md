# Siri Remote 麦克风 DriverKit 概念验证

本目录包含一个本地 HIDDriverKit 替换方案实验，以及独立的 macOS 主机程序。它们与 HyperVibe 主应用保持隔离，因为主应用使用不同的签名/运行时方式，且必须保留现有辅助功能和输入监控授权。

## 当前结果

两个目标均可使用 Xcode 26.6 和 DriverKit 25.5 SDK 成功构建：

- `SiriRemoteMicDriver` 是 arm64 `IOUserHIDEventService` DEXT；
- `SiriRemoteMicHost` 会嵌入该 DEXT，只提交明确的 Activate 或 Deactivate 请求；
- 两个程序包都能使用本地 Apple Development 身份从内向外签名，开启强化运行时，并通过严格的嵌套 `codesign` 验证。

2026-07-20 首次签名启动时，AMFI 在主机程序进入 `main` 前就终止了它：

```text
AppleMobileFileIntegrityError Code=-413 "No matching profile found"
taskgated-helper: no eligible provisioning profiles found
```

因此没有提交 `OSSystemExtensionRequest`，没有注册系统扩展，`systemextensionsctl list` 也没有变化。这是明确的配置描述文件签发问题，并不代表麦克风测试结果，也无法通过管理员或 root 权限绕过。

## 驱动行为

Driver personality 只匹配已测试遥控器的音频 HID 接口：

- Apple 厂商 ID `76`，产品 ID `789`；
- 主用法页 `0x0C`、用法 `0x04`，对应 IORegistry 中此前枚举为蓝牙 HID 接口 5 的节点。BLE `IOHIDInterface` 提供方没有暴露可用于匹配的 `bInterfaceNumber`，因此 personality 不依赖它。

它使用默认匹配类别和实验性探测分数 `8000`，高于观测到的 `AppleEmbeddedBluetoothAudio` 分数 `7175`。因此，注册后它会成为候选替换驱动，而不是被动监听者。现有 provider 可能仍需重新创建或匹配，必须始终在 IORegistry 中确认接口归属。

`Start` 成功后，DEXT 会通过 `IOHIDInterface` provider 发送有证据支持的 Feature 报告 `0xFF [0xAF]`，并记录精确的 `IOReturn`。随后它会记录输入报告，每份报告最多 209 字节。只有结合启用返回码和确认后的接口归属，零帧测试结果才有意义。

## 仅构建和签名，不启用

```sh
cd driverkit
./build-host.sh

# 粘贴 `security find-identity -v -p codesigning` 显示的准确身份字符串。
./sign-host-development.sh 'Apple Development: Name (certificate suffix)'

# 两个目标均存在符合条件的配置描述文件后，在相同的由内向外签名流程中嵌入它们。
./sign-host-development.sh 'Apple Development: Name (certificate suffix)' \
  /path/to/SiriRemoteMicHost.provisionprofile \
  /path/to/SiriRemoteMicDriver.provisionprofile
```

`build-host.sh` 会先构建 DEXT，再编译主机程序，并将扩展嵌入 `Contents/Library/SystemExtensions/com.hypervibe.SiriRemoteMicDriver.dext`。它只会写入仓库中被忽略的 `.build/` 目录。这两个脚本都不会启动主机、安装系统扩展、修改 SIP、重启蓝牙、改变配对状态，也不会停止 HyperVibe。

签名脚本只有一个参数时，会在没有配置描述文件的情况下进行结构完整性用的开发签名。带三个参数时，会先验证并嵌入主机和 DEXT 的配置描述文件，再由内向外签名。在两种模式间切换时，先重新运行 `build-host.sh`，避免被忽略的旧构建产物继续包含旧配置描述文件。

不带参数启动时，主机程序不会执行操作。其 UI 提供明确的 Activate 和 Deactivate 按钮；可执行文件也只接受 `--activate` 或 `--deactivate` 其中之一，用于受控试验。

## 配置描述文件门槛

当前证书可用于开发签名，但此机器没有适用于以下受限 entitlement 的配置描述文件：

- DEXT：`com.apple.developer.driverkit`、`com.apple.developer.driverkit.transport.hid` 和 `com.apple.developer.driverkit.family.hid.eventservice`；
- 主机：`com.apple.developer.system-extension.install`。

证书授权的是签名身份；配置描述文件则分别授权身份、bundle ID 和 entitlement 集合。因此签名后的主机仍会在应用代码运行前被 AMFI 终止。证书显示的括号后缀也不是 Team Identifier；此处实际签名的 TeamIdentifier 是 `5S6YD5B7F4`。

2026-07-20，显式执行的 `xcodebuild -allowProvisioningUpdates` 请求已发送到 Apple 服务。Xcode 在生成配置描述文件前就拒绝了请求：当前登录的**个人开发团队**（`James Zhang`）不支持 DriverKit Transport HID、DriverKit Family HID EventService 或 DriverKit development 能力。由此可知当前免费/个人团队无法自动配置；这不是源码、证书或本机管理员权限的问题。

取得合格配置描述文件后，必须先将完整主机程序包复制到 `/Applications` 或其他系统认可的 Applications 目录，再进行启用。否则从 `.build/` 启动启用主机时会遇到另一个独立限制：[`unsupportedParentBundleLocation`](https://developer.apple.com/documentation/systemextensions/ossystemextensionerror/unsupportedparentbundlelocation)。

修改恢复模式/SIP 安全状态属于单独流程；仓库中的脚本都不会执行此操作。

## 有界启用验收测试

两个配置描述文件就绪后，首次测试应保持范围狭窄，并确保可恢复：

1. 重新构建，签名时嵌入两个合格配置描述文件，验证两个强化运行时签名，并将完整主机程序复制到 `/Applications`；
2. 暂时停止唯一的常规 HyperVibe 进程，并确认它的接口 5 HID 客户端已退出；
3. 从已安装主机提交启用请求；若系统要求，则在 **系统设置 → 通用 → 登录项与扩展 → 驱动程序** 中批准；
4. 检查 `systemextensionsctl` 状态和 IORegistry 归属。启用只会注册 DEXT；未完成这两项检查前，不要推断它已替换 `AppleEmbeddedBluetoothAudio`；
5. 确认 DEXT 自动发送的 Feature `0xFF [0xAF]` 返回成功，再按住 Siri 键进行一次有时限的采集；
6. 以接口 5 回调为采集标准：本地 `reportID=0xFF`、预期长度的变化负载，以及看起来合理的 Opus 字节（常见 TOC `0xB8`）。线上/GATT 报告 `0xFA` 是协议证据；macOS 会把该接口的本地 ID 改写为 `0xFF`，因此不可要求本地 ID 必须是字面上的 `0xFA`；
7. 请求停用；如系统返回延迟或需要重启，按其要求处理；验证 `AppleEmbeddedBluetoothAudio` 已恢复，并重新启动且只保留一个不带参数的 HyperVibe 实例。

在真正捕获到随时间变化的麦克风负载前，不要添加 Opus 解码或虚拟 CoreAudio 设备。

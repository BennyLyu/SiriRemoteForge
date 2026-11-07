# 参与贡献

感谢你关注这个项目。本项目规模不大，观点也比较鲜明；下面主要说明几件容易处理错的事，而不是繁琐的流程。

## 准备开发环境

```bash
git clone https://github.com/HOLODATA-COM/SiriRemoteForge.git
cd SiriRemoteForge
swift test --package-path SiriRemoteCore   # 引擎测试，无需硬件
cd app && ./build.sh && ./create_app_bundle.sh
```

`SiriRemoteCore/` 是无外部依赖、带单元测试的 SwiftPM 包，大部分逻辑应放在这里。
`app/` 由 `swiftc` 直接编译；核心源码会编入同一个二进制文件。因此，在 `app/` 新增源码时，也要把文件加入 `app/build.sh` 的 `SWIFT_FILES` 列表。

运行程序需要三代 Apple TV Siri Remote 和辅助功能权限；开发引擎逻辑则不需要这些条件。

## 修改行为前请了解

- **只能运行一个实例。** 程序会独占遥控器的 HID 接口；第二个进程收不到输入，第一个实例也会停止响应。启动前先关闭旧实例。
- **`--test-*` 参数不会独占遥控器。** 这些参数只用于渲染 HUD 截图，随后退出。操作后请恢复正常实例。
- **编辑后要验证 `config.jsonc`。** 文件格式错误时，程序会静默回退到默认值。仓库中的示例由 `ExampleConfigTests` 检查；个人配置需要自行验证。
- **解析优先级不能随意调整。** 图层和界面展示都依赖特定规则，这些规则源于已修复的具体问题。修改 `Controller.site(_:)` 或 `MappingEngine` 前，先阅读 README 中的“图层”和“标签与图标”章节。相关行为由测试固定。
- **界面几何尺寸应通过测量确认。** 截图并检查像素。项目历史上有几次看似合理、实际错误的对齐修复。

## Pull request

- 尽量把逻辑放进 `SiriRemoteCore`，并为它添加测试；这部分可以在没有硬件时验证。
- 说明你实际验证了什么、在哪种环境下验证。“可以编译”和“在我的遥控器上可用”是两种不同的结论；都很有用，但请明确写出是哪一种。
- 提交说明要讲清楚改了什么，以及旧行为为什么不正确。
- CI 必须通过。

## 报告问题

请提供 macOS 版本、遥控器通过 USB-C 还是蓝牙连接、`config.jsonc` 中相关配置，以及 `/tmp/hypervibe.log` 的对应内容。按键绑定“不工作”的原因通常能从日志中看出。

# HyperVibe 公开 API 探测工具

这两个最小命令行工具可复现 Feedback Assistant 报告中使用公开框架的探测步骤。它们不包含 HyperVibe 输入代码、私有框架、数据包采集、蓝牙地址、凭据或设备专用常量。

要求：macOS 13 或更高版本、Xcode 命令行工具，以及已在蓝牙设置中配对的第三代 USB-C Siri Remote。

请在 `PublicAPIProbe` 目录中运行以下命令。

## InputProbe

`InputProbe` 会请求 Game Controller 提供所有当前已连接的控制器，打印公开配置和元素别名，并为收到的每个值变化回调添加时间戳。

```sh
swift run InputProbe --seconds 30
```

若未返回已配对的遥控器，请再使用 Game Controller 显式发现功能运行一次：

```sh
swift run InputProbe --discover --seconds 30
```

在 30 秒窗口内按下每个实体按键，转动并轻点点击环，然后用一根手指扫过整个触控表面。将控制台输出复制到输入 API 的 Feedback 报告中。探测工具不会阻止系统手势，也不会独占输入路由。

## AudioInputProbe

`AudioInputProbe` 会枚举公开 CoreAudio API 提供的输入设备，并打印其名称、UID、制造商、传输代码和标称采样率。

```sh
swift run AudioInputProbe
```

请在安装 HyperVibe 虚拟麦克风前，在干净的测试系统上运行。如果已安装虚拟设备，其名称和 `SRMDevice_UID` 可证明该设备由项目创建，而不是 Siri Remote 的原生音频端点。

## 附件清理

附加前请检查控制台输出。移除 Shell 提示符中的 Mac 用户名，并省略蓝牙地址、序列号、无关音频设备 UID 或其他个人标识。请将完整源码目录与清理后的输出一并附上，以便 Apple 构建完全相同的公开框架探测工具。

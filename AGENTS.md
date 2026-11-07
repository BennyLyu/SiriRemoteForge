# 项目操作规则

以下规则适用于所有本地 macOS App 构建和测试部署，必须遵守。

1. 仅使用 `/Applications/HyperVibe.app` 进行实时测试。
2. 编译并打包用户测试版后，将已验证的程序包安装到 `/Applications/HyperVibe.app`。不要把 `app/HyperVibe.app` 当作实时测试应用启动。
3. 保留现有的稳定签名身份 `siriRemote Local Signing`。替换已安装应用前，先验证程序包签名。
4. 本地开发构建不得在未说明的情况下退回临时 ad-hoc 签名。如果稳定身份不可用或签名验证失败，应在安装或重启前停止并告知用户。
5. 仅在完成安装和签名验证后重启应用；随后确认只有一个 UI 进程正在运行，且其可执行文件路径为 `/Applications/HyperVibe.app/Contents/MacOS/HyperVibe`。
6. 将 HyperVibe 视为常驻应用。除非用户明确要求停止，否则每次完成本地构建、安装、需要重载的配置更改或交付用户测试版本后，都要确保 `/Applications/HyperVibe.app` 恰好以一个 UI 进程运行。应自动完成，不要等待用户提出。

公开发布程序包可以使用其明确指定的发布签名流程，但不得用它替代稳定签名的本地开发应用。

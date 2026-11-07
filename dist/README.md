# 发布程序包

发布构建脚本会从干净的代码检出生成三个供 Apple 芯片 Mac 使用的下载包：

- **`HyperVibe-Full-Setup-VERSION-arm64.pkg`** — macOS 原生安装器。用户批准一次管理员操作后，即可安装应用、虚拟麦克风、路由器、按需采集服务和卸载器。随后 HyperVibe 会打开实时系统检查页，供用户自行决定 macOS 要求由用户控制的隐私权限。
- **`HyperVibe-VERSION-macOS-arm64.zip`** — 仅包含菜单栏应用。解压后将 `HyperVibe.app` 移到“应用程序”，首次打开时右键并选择 **打开**。
- **`HyperVibe-Full-Setup-VERSION-arm64.zip`** — 包含应用、Siri Remote Mic HAL 插件、路由器、采集守护进程和卸载器。此高级选项会安装系统音频和蓝牙采集组件，需要管理员密码。

`SHA256SUMS.txt` 列出原生安装器及两个归档文件的校验和。GitHub 会另外提供源码归档。

发布前同时更新 `README.md` 与 `dist/release-notes-vVERSION.md` 中的用户说明；尤其确认新版本的 `SUFeedURL`、Appcast 下载链接和发布校验脚本全部指向 BennyLyu/SiriRemoteForge。旧渠道安装包不会自动切换更新源，发布说明需提醒用户手动安装一次 Fork 版本。

## 构建公开 Release 资源

本机构建只需要安装 Xcode 命令行工具。构建脚本会下载带固定校验和的官方 libopus 1.6.1 源码，将其编译为 macOS 13 目标版本，并静态链接进发布路由器。构建机和目标 Mac 都不需要 Homebrew。

```sh
dist/build-release.sh 0.2.0-beta.10
```

命令要求工作区干净，会重新构建所有发布二进制，写入数字形式的应用版本和单调递增的内部构建号，并在隔离的暂存程序包中执行可重复的临时签名；随后运行路由器和 HAL 离线测试，并生成：

```text
dist/build/0.2.0-beta.10/
├── HyperVibe-0.2.0-beta.10-macOS-arm64.zip
├── HyperVibe-Full-Setup-0.2.0-beta.10-arm64.zip
├── HyperVibe-Full-Setup-0.2.0-beta.10-arm64.pkg
└── SHA256SUMS.txt
```

生成的内容由 Git 忽略，因为个人程序包可能包含私有材料。只上传上面经过审核的四个文件，不要上传整个 `dist/build/` 目录。
`build-release.sh` 最后会运行 `audit-release.sh`；该脚本会独立解压两个归档，并检查签名/校验和、版本和架构是否正确，以及是否包含 Homebrew 运行时链接、缺少的许可证声明、私有路径、作者配置、PacketLogger 或视频文件等问题。libopus 源码归档和构建输出缓存在 Git 忽略的 `dist/build/` 路径下。构建脚本不会改写 `app/HyperVibe.app`，因此不会影响本地运行的稳定签名开发版和 macOS 隐私授权。

最后一步还会更新并签名仓库根目录的 `appcast.xml`。HyperVibe 默认每天检查 BennyLyu/SiriRemoteForge 的更新源，并可在后台下载仅含应用的 ZIP。每个归档在解压前都会由 Sparkle 使用 Ed25519 公钥验证；公开更新源还会额外进行整体签名。私钥保留在维护者的登录钥匙串中，绝不能提交到仓库。使用 Sparkle 的 `generate_keys -x` 将其备份到加密离线存储；在替换的发布机器上使用 `generate_keys -f` 导入。自动更新只替换 `HyperVibe.app`，不会重启系统音频，也不会要求管理员密码。若用户要安装或更新虚拟麦克风及其特权服务，仍需单独选择“完整安装”。此前从其他发布渠道安装的版本可能仍使用旧更新源，需手动安装一次本 Fork 的版本。

## 公开程序包的安全边界

默认使用公开模式。它始终使用 `examples/config.jsonc`，并拒绝自定义配置和 PacketLogger。负载文件有独立的 SHA-256 清单，特权安装器在修改系统前会先验证清单。

“完整安装”会：

1. 安装应用和 `HyperVibe Uninstall.app`；
2. 保留原有的蓝牙 `HCITraces` 首选项；
3. 安装 HAL 插件并重启 `coreaudiod`；
4. 监控 `coreaudiod` 25 秒；若 CPU 连续三秒保持在 85% 或以上，则恢复此前的插件；
5. 仅在检查通过后安装按需采集守护进程。

卸载器会移除应用、HAL 插件、守护进程和支持二进制，然后恢复原来的 `HCITraces` 值。它会有意保留用户的 `~/.config/siriremote` 目录，以及用户单独安装的 Apple PacketLogger。

如果没有 PacketLogger，已安装的守护进程不会改动蓝牙 HCI 调试跟踪，遥控器语音功能将保持关闭。之后安装 PacketLogger，会在下次需要麦克风时延迟启用跟踪采集；无需重启。

## PacketLogger 与遥控器语音

遥控器语音采集需要从 [*Xcode Additional Tools*](https://developer.apple.com/download/all/?q=Additional+Tools+for+Xcode) 获取 Apple PacketLogger，并放在 `/Applications/PacketLogger.app`。本公开项目不得重新分发它，公开 Release 资源也绝不包含它。缺少 PacketLogger 时，“完整安装”会提供 Apple 下载页面；应用和内置麦克风回退功能仍可使用。

在你控制的设备之间进行私下传输时，可以选择另一种独立模式来打包本地配置，以及（遵守 Apple 许可的前提下）已有的 PacketLogger 副本：

```sh
dist/package.sh --personal --version local --config /path/to/config.jsonc
dist/package.sh --personal --version local --config /path/to/config.jsonc --with-packetlogger
```

个人版输出会显示警告，绝不能上传到 GitHub。

## 签名与 Gatekeeper

这些 Beta 应用程序包使用临时 ad-hoc 签名，未经 Apple 公证。这里有意关闭强化运行时，因为它会终止遥控器触控板所用的私有 MultitouchSupport 回调。临时签名标识不能作为可信的钥匙串对端身份，因此公开 Beta 不会调用绑定证书的凭据助手。原生云端听写仍可用：密钥以仅当前用户可读的明文 JSON 文件保存在 Application Support 中；共享配置和发布资源不会包含这些密钥。绑定证书的构建仍优先使用登录钥匙串。若 `HYPERVIBE_INSTALLER_SIGN_IDENTITY` 未指定有效的 Developer ID Installer 身份，原生安装包不会签名；应用签名与安装器签名使用 Apple 的不同证书类型。在配置好 Developer ID 签名和公证前，首次启动请使用 **右键 → 打开**。不要建议用户全局关闭 Gatekeeper。

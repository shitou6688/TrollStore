# 巨魔商店

巨魔商店是一个永久签名的非越狱应用，可以永久安装任何你打开的 IPA 文件。

它之所以能工作是因为 AMFI/CoreTrust 的一个漏洞，iOS 无法正确验证具有多个签名者的二进制文件的代码签名。

支持版本：14.0 beta 2 - 16.6.1, 16.7 RC (20H18), 17.0

## 安装巨魔商店

关于安装巨魔商店，请参考 [ios.cfw.guide](https://ios.cfw.guide/installing-trollstore) 上的指南

16.7.x（不包括 16.7 RC）和 17.0.1+ 版本将永远不会被支持（除非发现第三个 CoreTrust 漏洞，这不太可能）。

## 更新巨魔商店

当有新的巨魔商店更新可用时，在巨魔商店设置顶部会出现一个安装按钮。点击按钮后，巨魔商店将自动下载更新、安装并重启。

或者（如果出现问题），你可以在 Releases 下下载 TrollStore.tar 文件并在巨魔商店中打开，巨魔商店将安装更新并重启。

## 卸载应用

从巨魔商店安装的应用只能从巨魔商店本身卸载，在"应用"标签页中点击应用或向左滑动即可删除。

## 持久化助手

巨魔商店使用的 CoreTrust 漏洞仅足以安装"系统"应用，这是因为 FrontBoard 在每次启动用户应用之前都会进行额外的安全检查（它调用 libmis）。不幸的是，无法安装新的"系统"应用来在图标缓存重新加载后保持。因此，当 iOS 重新加载图标缓存时，所有通过巨魔商店安装的应用（包括巨魔商店本身）将恢复为"用户"状态，并且无法再启动。

解决这个问题的唯一方法是安装一个持久化助手到系统应用中，这个助手可以用来重新将巨魔商店及其安装的应用注册为"系统"应用，使其可以再次启动，这个选项在巨魔商店设置中可用。

在越狱的 iOS 14 上，当使用 TrollHelper 进行安装时，它位于 /Applications 中，并且会在图标缓存重新加载后保持为"系统"应用，因此在 iOS 14 上使用 TrollHelper 作为持久化助手。

## URL 方案

从 1.3 版本开始，巨魔商店替换了系统 URL 方案 "apple-magnifier"（这样做是为了防止"越狱"检测能够像检测具有唯一 URL 方案的巨魔商店那样检测它）。这个 URL 方案可以用来直接从浏览器安装应用，或者从应用本身启用 JIT（仅限 2.0.12 及以上版本），格式如下：

- `apple-magnifier://install?url=<IPA_URL>`
- `apple-magnifier://enable-jit?bundle-id=<Bundle_ID>`

在没有安装巨魔商店（1.3+）的设备上，这只会打开放大镜应用。

## 功能

IPA 中的二进制文件可以具有任意权限，使用 ldid 和你想要的权限进行假签名（`ldid -S<path/to/entitlements.plist> <path/to/binary>`），巨魔商店在安装时使用假根证书重新签名时会保留这些权限。这给你提供了很多可能性，下面解释了一些。

### 被禁止的权限

iOS 15 在 A12+ 上禁止了以下三个与运行未签名代码相关的权限，没有 PPL 绕过就不可能获得这些权限，使用这些权限签名的应用在启动时会崩溃。

`com.apple.private.cs.debugger`

`dynamic-codesigning`

`com.apple.private.skip-library-validation`

### 解除沙盒限制

你的应用可以使用以下任一权限在非沙盒环境中运行：

```xml
<key>com.apple.private.security.container-required</key>
<false/>
```

```xml
<key>com.apple.private.security.no-container</key>
<true/>
```

```xml
<key>com.apple.private.security.no-sandbox</key>
<true/>
```

如果你仍然希望为你的应用保留沙盒容器，推荐使用第三个。

你可能还需要 platform-application 权限才能让这些正常工作：

```xml
<key>platform-application</key>
<true/>
```

请注意，platform-application 权限会导致一些副作用，比如沙盒的某些部分变得更严格，所以你可能需要额外的私有权限来绕过这些限制。（例如，之后你需要为每个想要访问的 IOKit 用户客户端类添加异常权限）。

为了让具有 `com.apple.private.security.no-sandbox` 和 `platform-application` 的应用能够访问其自己的数据容器，你可能需要额外的权限：

```xml
<key>com.apple.private.security.storage.AppDataContainers</key>
<true/>
```

### Root 助手

当你的应用不在沙盒中时，你可以使用 posix_spawn 生成其他二进制文件，你也可以使用以下权限以 root 身份生成二进制文件：

```xml
<key>com.apple.private.persona-mgmt</key>
<true/>
```

你也可以将自己的二进制文件添加到应用包中。

之后你可以使用 [TSUtil.m 中的 spawnRoot 函数](./Shared/TSUtil.m#L79) 以 root 身份生成二进制文件。

### 使用巨魔商店无法实现的功能

- 获得适当的平台化（`TF_PLATFORM` / `CS_PLATFORMIZED`）
- 生成启动守护进程（需要 `CS_PLATFORMIZED`）
- 向系统进程注入 tweak（需要 `TF_PLATFORM`、用户空间 PAC 绕过和 PMAP 信任级别绕过）

### 编译

要编译巨魔商店，请确保已安装 [theos](https://theos.dev/docs/installation)。另外确保已安装 [brew](https://brew.sh/)，并从 brew 安装 [libarchive](https://formulae.brew.sh/formula/libarchive)。

## 致谢和进一步阅读

[@alfiecg_dev](https://twitter.com/alfiecg_dev/) - 通过补丁差异分析找到了允许巨魔商店工作的 CoreTrust 漏洞，并致力于自动化绕过。

Google 威胁分析组 - 作为野外间谍软件链的一部分发现了 CoreTrust 漏洞并向 Apple 报告。

[@LinusHenze](https://twitter.com/LinusHenze) - 找到了用于通过 TrollHelperOTA 在 iOS 14-15.6.1 上安装巨魔商店的 installd 绕过，以及巨魔商店 1.0 中使用的原始 CoreTrust 漏洞。

[Fugu15 演示](https://youtu.be/rPTifU1lG7Q)

[关于第一个 CoreTrust 漏洞的详细说明](https://worthdoingbadly.com/coretrust/)

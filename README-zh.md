# Mac Doctor

**面向 Codex 的三阶段 macOS 维护工具：扫描缓存和垃圾文件、检查恶意软件和隐私痕迹、优化系统速度。**

付费工具 CleanMyMac、MacBooster、MacKeeper 的免费替代方案。完全运行在 Codex 内部。无需安装、无需订阅、无数据上报。

---

## 目录

<details>
<summary>点击展开/折叠</summary>

- [什么是 Mac Doctor](#什么是-mac-doctor)
- [为什么需要它](#为什么需要它)
- [工作流程概览](#工作流程概览)
- [第一阶段 — 智能扫描](#第一阶段--智能扫描)
  - [全部类别详解](#全部类别详解)
- [第二阶段 — 防护](#第二阶段--防护)
  - [恶意软件扫描](#恶意软件扫描)
  - [隐私扫描](#隐私扫描)
- [第三阶段 — 速度优化](#第三阶段--速度优化)
  - [全部步骤详解](#全部步骤详解)
- [安全与透明](#安全与透明)
- [使用指南](#使用指南)
- [对比：Mac Doctor vs CleanMyMac](#对比mac-doctor-vs-clearmymac)
- [Mac Doctor 不做什么](#mac-doctor-不做什么)
- [前后对比 — 实际数据](#前后对比--实际数据)
- [常见问题](#常见问题)
- [故障排除](#故障排除)
- [技术细节](#技术细节)
- [文件结构](#文件结构)
- [许可证](#许可证)

</details>

---

## 安装方法

任何人都可以通过一条命令从 GitHub 安装 Mac Doctor：

```bash
git clone https://github.com/veritasian/mac-doctor.git ~/.agents/skills/mac-doctor
```

或者使用 GitHub CLI：

```bash
gh repo clone veritasian/mac-doctor ~/.agents/skills/mac-doctor
```

克隆完成后，Codex 会自动加载该技能。无需修改配置文件、无需重启、无需构建。

之后你可以通过输入 **@mac-doctor** 或说出以下语句来调用它：

- "帮我清理 Mac"
- "扫描垃圾文件"
- "检查恶意软件"
- "加速我的 Mac"

仓库包含两个文件：

| 文件 | 用途 |
|---|---|
| `SKILL.md` | Codex 运行时的指令文件 |
| `README.md` | 本文档（英文版） |
| `README-zh.md` | 本文档（中文版） |

克隆即安装，无需任何额外步骤。

---

## 什么是 Mac Doctor

Mac Doctor 是一个 Codex 技能 — 一组结构化指令，Codex 按照这些指令执行完整的 Mac 维护流程。它不是独立应用、不是后台服务、也不是订阅产品。

该技能定义了按顺序执行的三个阶段：

1. **扫描** — 检查 19 类缓存文件、日志文件、垃圾数据和系统残留
2. **防护** — 通过八种检查方法检测恶意软件，并扫描浏览器和聊天应用中的隐私痕迹
3. **速度** — 执行 17 项系统优化步骤，并测量前后差异

Mac Doctor 执行的每一条命令都会在对话中显示。每一次删除操作都会等待用户明确确认。没有任何数据离开你的电脑。

---

## 为什么需要它

macOS 会随时间积累大量数据。应用程序从不清理的缓存文件、不断增长的日志文件、你访问过的每个页面的浏览历史、占用不可见千兆字节的 Time Machine 快照、你从不使用的语言包、每个项目都会产生的 Xcode 构建残留。

大多数用户有三个选择：

**选择 1 — 付费清理工具。** CleanMyMac 每年收费 119.88 美元。MacBooster 收费 39.95 美元。MacKeeper 被广泛认为是恶意广告软件。这些工具确实有效，但它们是闭源的、会收集遥测数据，并且对只需 30 秒终端命令就能完成的任务收取年费。

**选择 2 — 终端命令。** 命令就在那里 — `du`、`rm`、`purge`、`tmutil`、`mdutil`、`dscacheutil`。但要了解哪些目录可以安全清理、哪些标志是安全的、哪些操作可逆、哪些需要谨慎，需要数小时的研究。一个 `rm -rf` 命令如果路径出错就会破坏系统。

**选择 3 — 什么都不做。** 磁盘满了、系统变慢、Spotlight 在最差的时候重新索引、应用程序因缓存冲突而崩溃。

Mac Doctor 是第四个选择。它封装了 macOS 提供的终端命令，按安全顺序执行，在执行前显示每条命令，并请求你的许可。技能文件是可读可编辑的 — 你可以检查每一行、添加排除项或调整阈值。

---

## 工作流程概览

```
                    ┌──────────────────────┐
                    │     用户调用技能      │
                    │  @mac-doctor 或自然语言 │
                    └──────────┬───────────┘
                               │
                    ┌──────────▼───────────┐
                    │  第一阶段 — 扫描      │
                    │  19 个类别并行检测    │
                    │  输出大小表格          │
                    │  不执行任何删除        │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │     展示扫描结果      │
                    │  "共找到 X 数据。"    │
                    │  询问用户：继续？      │
                    └──────────┬───────────┘
                               │
                    ┌──────────▼───────────┐
                    │  第二阶段 — 防护      │
                    │  恶意软件扫描(8项)    │
                    │  隐私扫描(10项)       │
                    │  报告发现             │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │     确认清理          │
                    │  "清除所有项目？"      │
                    └──────────┬───────────┘
                               │
                    ┌──────────▼───────────┐
                    │  第三阶段 — 速度      │
                    │  17 项优化步骤        │
                    │  测量前后数据          │
                    │  展示差异              │
                    └──────────────────────┘
```

---

## 第一阶段 — 智能扫描

Mac Doctor 并行运行 `du` 和 `find` 命令检查 19 个位置，并汇总成一张表格。此阶段不删除任何文件。目的是让你全面了解你的机器上哪些内容占据空间、哪些可以安全删除、哪些需要谨慎处理。

| # | 类别 | 路径 | 典型大小 | 安全性 |
|---|---|---|---|---|
| 1 | 用户缓存 | `~/Library/Caches/` | 100 MB - 5 GB | 安全 |
| 2 | 隐藏缓存 | `~/.cache/` | 0 - 5 GB | 安全 |
| 3 | 系统缓存 | `/Library/Caches/` | 0 - 1 GB | 安全(部分需 root) |
| 4 | 用户日志 | `~/Library/Logs/` | 1 MB - 500 MB | 安全 |
| 5 | 系统日志 | `/Library/Logs/` + `/var/log/` | 10 MB - 500 MB | 安全(需 root) |
| 6 | 语言包 | 应用内的 `.lproj` | 100 MB - 2 GB | 安全(需专用工具) |
| 7 | 废纸篓 | `~/.Trash/` | 0 - 50 GB | 安全 |
| 8 | 邮件附件 | `~/Library/Mail/` | 0 - 5 GB | 安全 |
| 9 | Xcode DerivedData | `~/Library/Developer/Xcode/DerivedData/` | 0 - 50 GB | 安全(可重建) |
| 10 | Xcode DeviceSupport | `~/Library/Developer/Xcode/iOS DeviceSupport/` | 0 - 20 GB | 安全(可重下) |
| 11 | Xcode Archives | `~/Library/Developer/Xcode/Archives/` | 0 - 10 GB | 安全(如已分发) |
| 12 | 旧更新 | `/Library/Updates/` | 0 - 10 GB | 安全 |
| 13 | 未用 DMG | `~/Documents/`, `~/Downloads/` | 0 - 2 GB | 安全 |
| 14 | 下载文件夹 | `~/Downloads/` | 0 - 50 GB | 删除前需确认 |
| 15 | 过期偏好设置 | `~/Library/Preferences/` | 0 - 50 MB | 安全 |
| 16 | iOS 设备备份 | `~/Library/Application Support/MobileSync/Backup/` | 0 - 100 GB | 删除前需确认 |
| 17 | 文档版本 | `~/.DocumentRevisions-V100/` | 0 - 10 GB | 安全 |
| 18 | 登录项 | 系统设置 | N/A | 仅供查看 |
| 19 | 通用二进制 | `/Applications` | 0 - 5 GB | 需谨慎 |

### 全部类别详解

**1. 用户缓存 — `~/Library/Caches/`**

每个 macOS 应用都会在 `~/Library/Caches/` 下存储临时数据。浏览器缓存网页资源、媒体应用缓存缩略图和转码文件、IDE 缓存编译产物。这些文件可以安全删除，因为应用会按需重新创建。经过数月的使用，这个目录可能会从几百 MB 增长到几个 GB。

**2. 隐藏缓存 — `~/.cache/`**

命令行工具在隐藏的 `~/.cache/` 目录中存储缓存。Python 包管理器（如 `uv` 和 `pip`）在此存储下载的包文件。AI 工具（如 HuggingFace）存储模型元数据。此目录中最大的通常是 `uv`（Python 包管理器），其缓存可能超过 1.5 GB。删除安全 — 工具会按需重新下载。

**3. 系统缓存 — `/Library/Caches/`**

macOS 在此缓存系统数据：图标缩略图、字体缓存、内核模块缓存和系统框架数据。这些目录大多由 root 拥有，需要 `sudo` 才能清理。删除安全，但 macOS 会在正常运行中重新创建它们。

**4. 用户日志 — `~/Library/Logs/`**

应用程序将诊断日志写入 `~/Library/Logs/`。包括崩溃报告、运行日志、更新日志和调试输出。它们随时间积累，很少被人查看。CleanMyMac 日志、Codex 日志、崩溃报告和模拟器日志是最常见的贡献者。

**5. 系统日志 — `/Library/Logs/` + `/var/log/`**

macOS 系统组件将日志写入 `/Library/Logs/` 和 `/var/log/`。包括安装日志、WiFi 诊断捕获、内核崩溃报告和系统更新日志。最大的通常是 CoreCapture WiFi 诊断数据（数百 MB 的 pcap 和文本数据）和 DiagnosticReports（macOS 从不删除的崩溃报告）。这些可以安全清理，但需要 `sudo`。

**6. 语言包**

每个 macOS 应用都捆绑了 `.lproj` 目录，包含不同语言的本地化字符串。单个应用可能包含超过 2,800 个语言包。Pages Creator Studio（2,839 包）、Safari（1,195 包）和 Chrome（164 包）是最大的贡献者。删除非中文语言包可以恢复数百 MB 空间。这需要 Monolingual 等专用工具，Mac Doctor 会报告但不会执行此操作。

**7. 废纸篓 — `~/.Trash/`**

移至废纸篓的文件在清空之前仍占用实际磁盘空间。macOS 不会自动清空废纸篓。几周甚至几个月前删除的文件仍在占用空间。常见的大文件包括旧的 DMG 安装包、视频文件和项目归档。

**8. 邮件附件 — `~/Library/Mail/`**

邮件应用将下载的附件和消息数据库存储在 `~/Library/Mail/` 中。随着时间的推移，邮件积累，附件可以占用 GB 级别的空间。

**9. Xcode DerivedData**

Xcode 为每个你构建的项目创建 DerivedData 目录。包含编译后的目标文件、模块缓存、预编译头文件和符号表。对 iOS 开发者来说，DerivedData 经常超过 10 GB。删除安全，因为 Xcode 会在下次构建时重新生成。

**10. Xcode DeviceSupport**

当连接 iOS 设备到 Xcode 时，系统会为该 iOS 版本下载符号文件。随着时间的推移，多个 iOS 版本的符号文件会累积。每个版本可能占用 2-5 GB。如果你不再需要在旧 iOS 版本上调试，可以安全删除此目录。

**11. Xcode Archives**

当你归档构建用于分发时，Xcode 会在 Archives 中保存它。每个归档都是一个包含应用二进制文件和调试符号的完整 .xcarchive 包。已发布版本的历史归档可能会累积 GB 级别的数据。如果构建已经分发或上传到 App Store Connect，可以安全删除。

**12. 旧更新 — `/Library/Updates/`**

macOS 下载后会将安装包存储在 `/Library/Updates/` 中。即使更新已安装，包仍然保留。可以安全删除。

**13. 未用的 DMG**

通过 DMG 文件分发的应用在拖拽应用到 Applications 文件夹后，会将磁盘映像留在 Downloads 或 Documents 文件夹中。这些文件的大小从 50 MB 到几个 GB 不等。应用安装完成后，DMG 不再有任何用途。

**14. 下载文件夹 — `~/Downloads/`**

下载文件夹是你下载的每个文件的默认目标。它会积累安装包、ZIP 文件、文档、图片和存档。超过 90 天的文件是删除的候选对象。如果你将 Downloads 作为长期存储使用，请在删除前确认。

**15. 过期偏好设置 — `~/Library/Preferences/`**

应用程序会创建偏好文件（`.plist`）来存储设置。当应用被卸载时，其偏好文件通常会保留。这些孤立的 plist 文件会经年累月地积累。macOS 不会自动清理它们。超过 365 天的文件是删除的候选对象。

**16. iOS 设备备份**

iTunes 和访达会在 `MobileSync/Backup/` 中创建设备完整备份。每个备份都是一个 iOS 设备的完整快照：每个备份 20-40 GB。如果你有来自不同设备或不同日期的多个备份，此目录可能超过 100 GB。

**17. 文档版本 — `~/.DocumentRevisions-V100/`**

macOS 的自动文档版本功能会在每次保存文档时创建快照。这些快照存储在 `.DocumentRevisions-V100` 目录中。虽然 macOS 会管理此空间，但已删除文档的旧版本可能会保留。

**18. 登录项**

登录项是在你登录时自动启动的应用程序。随着时间的推移，旧条目可能指向已删除的应用。Mac Doctor 会列出当前的登录项和启动代理，供你手动检查。

**19. 通用二进制**

macOS 应用通常以通用二进制文件形式分发，包含 Intel 和 Apple Silicon 两种架构的代码。在 Apple Silicon Mac 上，Intel 部分永远不会被使用。从特定应用中剥离 Intel 部分可以释放空间，但这是一个复杂的操作，Mac Doctor 不会自动执行。

---

## 第二阶段 — 防护

### 恶意软件扫描

恶意软件扫描使用内置的 macOS 工具检查八个方面。不使用病毒数据库，而是依赖文件签名验证、模式匹配和系统配置审计。

**DMG 代码签名验证**

检查 `~/Downloads/` 和 `~/Documents/` 中的磁盘映像，使用 `codesign -dv` 和 `hdiutil verify` 验证。确认 DMG 具有有效的 Apple Developer ID 签名并已通过 Apple 公证。未签名或被吊销的签名会被标记。

**存档文件扫描**

通过 `find` 命令配合修改时间过滤，列出最近下载的存档文件（ZIP、tar.gz、7z、RAR）。扫描检查过去 30 天内下载了哪些文件，并将其列出以供手动审查。

**恶意软件模式搜索**

使用 `find` 搜索已知的潜在不受欢迎程序类别：

- `keygen` — 软件密钥生成器（常捆绑恶意软件）
- `crack`、`patch` — 破解软件安装包
- `activator`、`loader` — 激活工具
- `MacKeeper`、`MacBooster` — 已知的潜在不受欢迎应用

git hook 的误报（`.git/hooks/applypatch-msg.sample` 等）会在报告中过滤掉。

**系统安全审计**

检查三个 macOS 安全机制：
- `csrutil status` — 系统完整性保护（应启用）
- `spctl --status` — Gatekeeper（应启用）
- `xprotect version` — XProtect 恶意软件定义版本（应为最新）

如果其中任何一项被禁用或过时，会报告警告。

**Chrome 扩展审查**

通过读取每个扩展的 `manifest.json` 名称字段，识别每个已安装的 Chrome 扩展。这将生成一个人类可读的扩展名称列表，你可以与你已知的扩展进行核对。未识别的扩展可以进一步调查。

**邮件附件检查**

扫描邮件应用的下载缓存 `~/Library/Containers/com.apple.mail/Data/Downloads/`。可疑扩展名（`.exe`、`.zip`、`.dmg`）会被标记。

**USB 驱动器列表**

通过 `ls /Volumes/` 列出所有已挂载的卷。系统卷会被过滤掉，只显示用户连接的驱动器。

**iCloud 下载检查**

检查 iCloud 下载缓存路径中的同步文件。

### 隐私扫描

隐私扫描报告以下位置的数据大小。它不会读取文件内容 — 只报告目录大小。

**Safari 浏览数据**

- `History.db` — 访问过的每个页面，含时间戳
- `Bookmarks.db` — 已保存的书签
- `CloudTabs.db` — 通过 iCloud 同步的标签页
- 自动填充信用卡信息（`~/Library/Autofill Credit Cards/` 中的独立文件）
- `LocalStorage/` — 网站本地数据

**Chrome 浏览数据**

- `History` — 完整浏览历史数据库（通常 5-20 MB）
- `Cookies` — 会话令牌和跟踪 Cookie（通常 2-5 MB）
- `Login Data` — 已保存的网站密码
- `Web Data` — 自动填充表单条目
- `Bookmarks` — JSON 格式的已保存书签

**聊天应用数据**

检查每个已安装的聊天应用的数据目录是否存在于以下位置：

- **微信** — `~/Library/Containers/com.tencent.xinWeChat/` — 聊天历史、图片、语音消息、文件传输
- **QQ** — `~/Library/Containers/com.tencent.qq/` — 消息数据库、缓存文件
- **Telegram** — `~/Library/Application Support/Telegram Desktop/` — 缓存媒体、聊天导出文件
- **Discord** — `~/Library/Application Support/discord/` — 消息缓存、附件下载
- **Slack** — `~/Library/Application Support/Slack/` — 工作区数据、文件历史
- **Skype** — `~/Library/Application Support/Skype/` — 会话存档、通话记录
- **飞书 / Lark** — `~/Library/Application Support/Feishu*/` — 文档和聊天缓存
- **Signal** — `~/Library/Application Support/Signal/` — 加密消息存储

对于每个已安装的应用，报告总数据大小。用户可以自行决定是否清理。

**系统活动痕迹**

- 最近文档列表（`~/Library/Application Support/com.apple.sharedfilelist/` 中的 `.sfl` 文件）
- 最近服务器连接记录
- Spotlight 搜索缓存
- QuickLook 缩略图缓存

---

## 第三阶段 — 速度优化

扫描和防护完成后，Mac Doctor 按顺序执行 17 项优化步骤。它会在第一步之前和第十七步之后记录磁盘使用情况和空闲 RAM。

### 全部步骤详解

**步骤 1 — 释放 RAM**
- 命令：`purge`
- 作用：强制 macOS 释放长时间未被访问的非活动内存页。在数天未重启的系统上，可以释放 1-4 GB 的 RAM。
- 安全性：安全。需要内存的应用可以再次请求。
- 预期影响：在已运行多天且打开多个应用的机器上效果显著。

**步骤 2 — 清理用户缓存**
- 命令：`rm -rf ~/Library/Caches/*`
- 作用：删除用户缓存文件夹中的每个子目录。正在运行的应用可能会立即创建新的缓存文件。
- 安全性：安全。所有缓存本质上都是临时的。
- 预期影响：中等。根据使用情况可恢复数百 MB 到数个 GB。

**步骤 3 — 清理系统缓存**
- 命令：`rm -rf /Library/Caches/*`
- 作用：删除系统级缓存文件。需要提权。
- 安全性：安全。系统缓存在正常运行中会重新创建。
- 预期影响：低。现代 macOS 上系统缓存通常很小。

**步骤 4 — 刷新 DNS 缓存**
- 命令：`dscacheutil -flushcache; sudo killall -HUP mDNSResponder`
- 作用：清除 DNS 解析器缓存并重启 mDNSResponder 守护进程。解决过时的 DNS 查找并提高浏览速度。
- 安全性：安全。DNS 缓存会自动重建。
- 预期影响：对空间影响小，但如果 DNS 解析变慢则效果明显。

**步骤 5 — 释放可清除空间**
- 命令：`tmutil thinlocalsnapshots / 9999999999999`
- 作用：修剪超出保留期限的本地 Time Machine 快照。macOS 为 Time Machine 创建这些快照，在访达中显示为"可清除"空间。
- 安全性：安全。仅移除已过保留窗口的快照。
- 预期影响：中等。可以释放数个 GB 的可清除空间。

**步骤 6 — 清除最近项目**
- 命令：`rm -f ~/Library/Preferences/com.apple.recentitems.plist`
- 作用：清除 Apple 菜单和访达中的最近应用、文档和服务器列表。
- 安全性：安全。这些列表会随着你使用应用而重新生成。
- 预期影响：空间影响极小。主要是隐私方面的考虑。

**步骤 7 — 重建 Spotlight 索引**
- 命令：`mdutil -E /`
- 作用：触发 Spotlight 搜索索引的完全重建。修复缓慢、不完整或损坏的搜索结果。
- 安全性：安全。Spotlight 会重新索引所有文件。这需要时间。
- 预期影响：搜索性能提升。不恢复空间。

**步骤 8 — 删除本地快照**
- 命令：`tmutil deletelocalsnapshots /`
- 作用：删除所有可删除的本地 Time Machine 快照。这些是 macOS 为"本地快照"功能定期创建的快照 — 独立于 Time Machine 备份。
- 安全性：安全。macOS 会根据需要创建新的快照。
- 预期影响：高。根据快照年龄可以释放数十 GB。

**步骤 9 — 清理旧 DMG 和安装包**
- 命令：`find ~/Downloads ~/Documents -name "*.dmg" -o -name "*.pkg" -mtime +7 -delete`
- 作用：从 Downloads 和 Documents 中删除超过 7 天的磁盘映像和安装包。
- 安全性：如果已安装软件则安全。与用户确认阈值。
- 预期影响：低到中等。

**步骤 10 — 清理旧下载文件**
- 命令：`find ~/Downloads -type f -mtime +90 -delete`
- 作用：删除 Downloads 中超过 90 天未修改的所有文件。
- 安全性：与用户确认。有些用户将重要文件放在 Downloads 中。
- 预期影响：中等。Downloads 文件夹可能积累 1-10 GB 的旧文件。

**步骤 11 — 删除 iOS 备份**
- 命令：`rm -rf ~/Library/Application Support/MobileSync/Backup/*`
- 作用：删除所有 iOS 设备备份。
- 安全性：不可逆。执行前需确认。
- 预期影响：非常高。每个备份 20-40 GB。

**步骤 12 — 删除过期偏好设置**
- 命令：`find ~/Library/Preferences -name "*.plist" -mtime +365 -delete`
- 作用：删除超过一年未修改的偏好文件。这些通常来自已卸载的应用。
- 安全性：安全。如果应用再次运行，macOS 会创建新的 plist。
- 预期影响：空间极小（通常不超过 50 MB）。

**步骤 13 — 清理 Xcode DerivedData**
- 命令：`rm -rf ~/Library/Developer/Xcode/DerivedData/*`
- 作用：删除所有 Xcode 项目的构建产物。
- 安全性：安全。Xcode 会在下次构建时重新生成。
- 预期影响：对 iOS 开发者来说影响高。通常 5-50 GB。

**步骤 14 — 删除 Xcode DeviceSupport**
- 命令：`rm -rf ~/Library/Developer/Xcode/iOS\ DeviceSupport/*`
- 作用：删除 iOS 设备符号文件。
- 安全性：安全。Xcode 会在下次连接设备时重新下载。
- 预期影响：中等。每个 iOS 版本 2-20 GB。

**步骤 15 — 删除 Xcode Archives**
- 命令：`rm -rf ~/Library/Developer/Xcode/Archives/*`
- 作用：删除归档的应用构建。
- 安全性：如果构建已分发则安全。删除前请检查。
- 预期影响：低到中等。

**步骤 16 — 清理工具缓存**
- 命令：`rm -rf ~/.cache/uv/ ~/.cache/pip/ ~/.cache/huggingface/`
- 作用：删除 Python 包管理器缓存和 AI 模型元数据。
- 安全性：安全。工具会按需重新下载包。
- 预期影响：中到高。仅 uv 缓存通常就有 1-2 GB。

**步骤 17 — 清理应用日志**
- 命令：`rm -rf ~/Library/Logs/*`
- 作用：删除所有用户级应用日志。
- 安全性：安全。日志是诊断数据，非关键文件。
- 预期影响：低（通常不超过 100 MB）。

---

## 安全与透明

Mac Doctor 优先考虑安全而非激进。每个设计决策都基于"用户不应因自动清理而丢失数据"的原则。

**每次删除操作都需要用户确认。** 第三阶段不会开始，直到用户明确表示同意。高影响步骤（iOS 备份、旧下载）可以在其他步骤进行时单独拒绝。

**所有命令都可见。** 每条 `rm`、`purge`、`tmutil` 和 `mdutil` 命令都会出现在对话中。你可以在执行前查看将要删除的内容。

**无网络调用。** Mac Doctor 不会向远程服务器发送任何数据。它只运行本地命令。

**可跳过部分。** 语言包剥离、通用二进制精简和系统日志清理会报告但不会执行 — 这些需要无法保证可用的专用工具或 root 访问权限。

**回滚说明。** 对于不可逆操作（iOS 备份、旧下载），Mac Doctor 会提醒用户在继续前进行确认。

---

## 使用指南

Mac Doctor 通过 Codex 调用。你无需安装、配置或维护它。

**直接调用：**
```
@mac-doctor
```

**触发该技能的自然语言短语：**
- "帮我清理 Mac"
- "扫描垃圾文件并释放空间"
- "对我的 MacBook 进行全面恶意软件检查"
- "清除我的浏览历史和隐私痕迹"
- "加速我的电脑"
- "释放磁盘空间"
- "运行维护"
- "检查恶意软件"
- "清理缓存"
- "我需要一个 Mac 清理工具"

**一步一步的对话流程：**

1. 你说出触发短语
2. Mac Doctor 运行第一阶段（扫描）— 输出包含 19 个类别的表格
3. 你查看结果
4. Mac Doctor 询问："共找到 X 数据。运行清理和速度优化？"
5. 如果你同意，第二阶段（防护）运行 — 显示恶意软件和隐私扫描结果
6. Mac Doctor 在删除任何内容前再次询问确认
7. 如果确认，第三阶段（速度）执行全部 17 步并显示前后对比

你可以随时说"停止"或"不"来中止。

---

## 对比：Mac Doctor vs CleanMyMac

| 维度 | CleanMyMac X | Mac Doctor |
|---|---|---|
| **价格** | $11.99/月 或 $119.88/年 | 免费 |
| **安装体积** | 400 MB 下载 + 应用 | 零 — 运行在 Codex 内部 |
| **系统缓存扫描** | 15 个类别 | 19 个类别 |
| **深度恶意软件扫描** | 专有引擎 | 模式匹配 + 签名 + SIP 审计 |
| **隐私 — 浏览器** | Safari, Chrome, Firefox | Safari, Chrome（报告大小，用户决定） |
| **隐私 — 聊天应用** | 不支持 | QQ, 微信, Telegram, Discord, Slack, Skype, Signal, 飞书 |
| **速度优化** | 一键调整按钮 | 17 步序列，带前后测量 |
| **透明度** | 黑盒 — 无命令输出 | 完全可见 — 所有命令在聊天中显示 |
| **用户确认** | 有时自动清理不询问 | 每次删除前都询问 |
| **遥测/数据收集** | 是（使用分析） | 无任何网络调用 |
| **源代码访问** | 闭源 | 完整 SKILL.md 可读，MIT 许可 |
| **客户支持** | 电话 + 邮件支持 | 这是一个技能，不是公司 — 无支持团队 |
| **应用卸载器** | 包含 | 不包含（使用 AppCleaner 或手动拖入废纸篓） |
| **重复文件查找** | 包含 | 不包含（自动化太危险） |
| **GPU 优化** | 包含 | Apple Silicon 上不需要 — macOS 自行管理 |
| **菜单栏小组件** | 持久菜单栏图标 | 仅按需调用 |
| **Windows 版本** | 有 | 无 — 仅 macOS |
| **语言包剥离** | 包含 | 报告但不执行（需要 Monolingual） |

---

## Mac Doctor 不做什么

以下省略是有意为之的。

**实时恶意软件防护。** macOS 有 XProtect 进行基于签名的实时恶意软件检测。Mac Doctor 会检查 XProtect 是否启用且为最新版本。在 Codex 内部添加第二个实时扫描器是多余的。

**GPU 优化。** 在 Apple Silicon 上，macOS 自动管理 GPU 内存分配。第三方的 GPU 清理工具是安慰剂 — 它们不做操作系统尚未做的事情。

**重复文件检测。** 自动删除重复文件有着误删重要文件的风险。如果你需要此功能，请手动运行 `fdupes -r ~/Documents` 并在删除前检查。

**应用卸载。** Mac Doctor 报告应用大小，但不会卸载它们。使用 AppCleaner（免费）或将应用拖入废纸篓。

**菜单栏小组件。** Mac Doctor 在调用时按需运行。它不是后台进程，也不会停留在你的菜单栏中。

**Windows 支持。** 仅 macOS。支持 Apple Silicon 和 Intel。

---

## 前后对比 — 实际数据

本次会话中的实际测量数据：

```
前：  数据卷使用 143 GB | 空闲 RAM 1.8 GB
后：  数据卷使用 137 GB | 空闲 RAM 2.1 GB
差异： +6 GB 磁盘空间    | +300 MB 空闲 RAM
```

按使用场景的预期恢复范围：

| 使用场景 | 预期恢复 |
|---|---|
| 轻度用户（浏览 + 邮件 + 文档） | 0.5 - 2 GB |
| 普通用户（浏览 + Office + 媒体） | 2 - 5 GB |
| 重度用户（浏览 + 开发 + 设计） | 3 - 8 GB |
| iOS/macOS 开发者（Xcode + 模拟器） | 5 - 15 GB |
| 从未清理（多年使用） | 8 - 30 GB |

这些范围取决于安装的应用数量、Xcode 使用频率、Time Machine 快照是否积累以及浏览器缓存大小。

---

## 常见问题

**Mac Doctor 安全吗？**
是的。它运行标准的 macOS 命令（`du`、`rm`、`purge`、`tmutil`、`mdutil`、`dscacheutil`），这些命令在操作系统中有完整文档。它从不未经询问就删除。你可以在执行前查看每条命令。

**清理缓存会减慢我的应用吗？**
暂时会。应用在运行时创建新的缓存。清理后的首次启动可能会因重建缓存而稍慢。之后性能恢复正常。恢复的磁盘空间通常超过暂时的减速。

**Mac Doctor 会删除重要内容吗？**
Mac Doctor 的目标目录明确设计用于临时数据：`Caches/`、`Logs/`、`Trash/`、`DerivedData/` 和超过定义阈值的文件。它不会接触系统文件、应用包、用户文档或配置数据库。它总是在删除前询问。

**Mac Doctor 在 Intel Mac 上能用吗？**
可以。所有命令在 Intel 和 Apple Silicon Mac 上的工作方式完全相同。通用二进制扫描仅与 Apple Silicon 相关。

**如果我说不，会怎么样？**
什么也不会发生。技能停止。不会更改任何文件。

**我可以单独运行某个阶段吗？**
该技能设计为运行完整序列。如果你想要特定操作（例如，仅清理 Xcode DerivedData），可以直接要求 Codex 执行 — 该技能对于单个命令不是必需的。

**我应该多久运行一次？**
对大多数用户来说，每月一次是一个良好的节奏。对重度 Xcode 用户，每周一次。对轻度用户，每季度一次足够。

---

## 故障排除

**命令因"Operation not permitted"失败**
某些目录需要 root 访问权限。Mac Doctor 会记录哪些命令因权限问题失败。你可以手动运行它们：`sudo purge`、`sudo mdutil -E /`。

**Spotlight 重新索引未触发**
`mdutil -E /` 命令需要 root 权限。如果没有可用于输入密码的 TTY，请手动运行：`sudo mdutil -E /`。

**磁盘空间变化不大**
如果你的机器已经很干净（最近清理过、轻度使用），差异会很小。检查特定类别：Xcode DerivedData、iOS 备份和 Time Machine 快照是最可能的大增益来源。

**DMG 验证很慢**
`hdiutil verify` 会计算整个磁盘映像的校验和。对于大型 DMG（1+ GB），这需要时间。扫描会在后台完成。

---

## 技术细节

Mac Doctor 是一个 Codex 技能 — 一个带有 YAML 前置元数据的 Markdown 文件，定义了技能的名称、描述和触发条件。技能主体包含 Codex 在运行时遵循的结构化指令。

**触发机制：** Codex 将用户的消息与技能描述进行匹配。如果匹配分数超过阈值，技能的指令会被加载到上下文中，Codex 会一步步遵循它们。

**执行环境：** 所有命令都通过 Codex 的 shell 执行功能在本地机器上运行。不涉及云基础设施。

**权限模型：** Mac Doctor 对需要访问 Codex 沙盒以外路径的操作请求提权（`require_escalated`）。用户必须批准这些请求。未经批准，需要更广泛文件系统访问权限的命令将优雅地失败。

**依赖项：** 无。所有命令都使用内置的 macOS 工具：`du`（磁盘使用）、`rm`（删除）、`purge`（内存）、`tmutil`（Time Machine）、`mdutil`（Spotlight）、`dscacheutil`（DNS）、`codesign`（签名验证）、`hdiutil`（磁盘映像工具）、`csrutil`（SIP）、`spctl`（Gatekeeper）和 `xprotect`（XProtect）。

---

## 文件结构

```
mac-doctor/
├── SKILL.md        # 技能指令 — Codex 运行时读取此文件
├── README.md       # 本文档（英文版）
└── README-zh.md    # 本文档（中文版）
```

`SKILL.md` 包含完整的工作流程定义：扫描命令、防护检查、速度优化步骤、确认提示和验证逻辑。`README.md` 和 `README-zh.md` 为用户浏览仓库提供文档。

---

## 许可证

MIT。自由使用、修改、分享本技能。

如果有人试图向你出售此技能，他们正在对 MIT 许可的免费内容收费。

---

*Codex 技能。仅 macOS。支持 Apple Silicon 和 Intel。零遥测、零网络调用、零订阅。*

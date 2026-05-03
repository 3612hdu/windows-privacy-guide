# Windows 网络与共享：你的电脑在给陌生人传文件

## 系列文章

- [第一章：概览](https://github.com/3612hdu/windows-privacy-guide)
- [第二章：遥测系统](https://github.com/3612hdu/windows-privacy-guide/blob/master/02-telemetry-deep-dive.md)
- [第三章：广告追踪](https://github.com/3612hdu/windows-privacy-guide/blob/master/03-advertising-tracking.md)
- [第四章：位置服务](https://github.com/3612hdu/windows-privacy-guide/blob/master/04-location-services.md)
- 第五章（本文）：网络与共享

---

## 一、Wi-Fi Sense：你的 Wi-Fi 密码会自动分享

Wi-Fi Sense 是 Windows 10 引入的一个功能，旨在让连接 Wi-Fi 更"方便"：

**它做什么：**
1. 自动连接你的朋友分享的 Wi-Fi 网络
2. 把你的 Wi-Fi 密码分享给你的联系人（Outlook、Skype）
3. 加密存储你所有连接过的 Wi-Fi 密码到 Microsoft 服务器

**为什么默认开启时是个问题：**
- 如果你用 Microsoft 账户登录，你曾经连接过的每个 Wi-Fi 的密码都存储在微软云端
- 你的 Outlook 和 Skype 联系人（不是你选的，是微软自动关联的）可以自动连接你的 Wi-Fi
- Facebook 联系人曾经也在分享范围内（Facebook 在 2017 年退出了该功能）

注册表关闭：
```
HKLM\SOFTWARE\Microsoft\PolicyManager\default\WiFi\AllowWiFiHotSpotReporting
DWORD: value = 0
```

或者在设置 → 网络和 Internet → Wi-Fi → 管理已知网络中关闭"与联系人共享"。

---

## 二、P2P 更新分发（传递优化）：你的电脑是微软的 CDN

Windows 10/11 引入了一个叫**传递优化**（Delivery Optimization）的功能。它的工作原理：

1. 你下载了一个 Windows 更新
2. 你的电脑自动成为这个更新的"分发节点"
3. 局域网内其他电脑（或互联网上的陌生人）可以从你的电脑下载这个更新

**这意味着：**
- 你的网络带宽被用于给其他 Windows 用户推送更新
- 你的电脑在后台持续上传数据
- 默认情况下，不仅局域网内的设备可以获取，**互联网上的其他 Windows 设备也可以**

微软称这能"加快更新传播速度"。但本质上是把你设备的网络变成了微软的免费 CDN。

**关闭传递优化：**
设置 → Windows 更新 → 高级选项 → 传递优化 → 关闭"允许从其他电脑下载"

注册表：
```
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\DeliveryOptimization\Config
DWORD: DODownloadMode = 0
```

有效值：
- `0` = 关闭
- `1` = 仅局域网
- `2` = 局域网 + 互联网（默认）
- `3` = 仅互联网

---

## 三、时间同步与 NTP：微软知道你的电脑什么时候开机

Windows 默认连接到 `time.windows.com` 进行时间同步。这本身是正常的（精确时间对 HTTPS/TLS 安全连接至关重要），但有一个隐私细节：

每次时间同步请求中，你的设备会发送：
- 你的 IP 地址
- 设备名称（部分场景下）
- 请求的时间戳

虽然 NTP 协议本身不包含设备标识信息，但微软可以通过分析 NTP 请求的 IP 和时间模式，推断：
- 设备是否在线
- 设备的开机/关机时间规律
- 设备的地理位置（基于 IP）

这对大多数用户来说不是主要隐私威胁，但值得了解。

---

## 四、DNS 查询：你的网络服务商和微软知道你在访问哪些网站

默认情况下，Windows 使用你网络连接中配置的 DNS 服务器（通常是你的 ISP）。这意味着你的 ISP 可以看到你访问过的**每一个网站域名**。

Windows 11 默认启用了 DNS over HTTPS (DoH)，将查询加密后发送。但如果你用的 DNS 服务器仍然是 ISP 的默认设置，加密 DNS 只是让你的 ISP 从"看到明文 DNS"变成"看到加密 DNS 去自己的服务器"——并没有真正隐藏你的浏览行为。

**建议：** 如果你在意隐私，切换到不记录日志的第三方 DNS：
- Cloudflare：1.1.1.1
- Quad9：9.9.9.9
- 或者配置 VPN 级别的 DNS

Windows 本身不提供 VPN——这不在本系列范围内。

---

## 五、网络发现与文件共享：默认配置的泄漏风险

如果你连接到一个新的网络（比如公共 Wi-Fi），Windows 会询问你"是否要在此网络上发现其他设备"。如果你选了"是"，以下内容会暴露给同一网络上的所有设备：
- 你的设备名称
- 共享文件夹列表
- 网络打印机列表

建议：
- 公共 Wi-Fi → 选"否"，不要允许网络发现
- 家庭/工作网络 → 可以允许
- 定期检查"设置 → 网络和 Internet → 高级网络设置 → 高级共享设置"

---

## 六、蓝牙与附近设备：持续扫描的安全代价

Windows 11 的蓝牙功能**即使在你没有主动使用蓝牙时**，也会持续扫描附近的蓝牙设备。这是为了支持：
- 快速配对（Swift Pair）：检测附近的蓝牙耳机/鼠标并弹窗提示
- 查找我的设备：通过蓝牙信标定位
- 附近共享：检测附近的 Windows 设备

扫描行为本身不发送数据到微软，但**收集到的附近设备 MAC 地址列表**可以被任何有蓝牙权限的应用读取。

如果不需要频繁使用蓝牙外设，关闭蓝牙是一个简单有效的隐私保护措施。

---

## 七、汇总：你应该调整哪些网络设置？

按优先级排序：

| 优先级 | 设置 | 操作 |
|--------|------|------|
| **高** | 传递优化 | 关闭或限制为"仅局域网" |
| **高** | Wi-Fi Sense | 关闭密码共享 |
| **中** | 公共 Wi-Fi 网络发现 | 设为"否" |
| **低** | 蓝牙扫描 | 不用时关闭 |
| **低** | DNS | 切换到不记录日志的 DNS |
| **低** | 时间同步 | 风险极低，可保留 |

---

## 结语

这不是一个"微软很邪恶"的系列。微软收集的大部分数据都有合理的工程理由——调试崩溃、检测错误趋势、改善用户体验。Windows 11 在很多方面也比 10 更透明（诊断数据查看器就是一个好例子）。

**但这个系列的核心观点是：这些设置散落在十几个不同的页面和数百页文档里，大多数用户不知道它们存在，也不知道关闭后会发生什么。**

我们的目标是让你知道，然后你自己决定。

---

## 参考资料

- [Microsoft Privacy Statement](https://privacy.microsoft.com/zh-cn/privacystatement)
- [Windows 11 Diagnostic Data](https://learn.microsoft.com/en-us/windows/privacy/configure-windows-diagnostic-data-in-your-organization)
- [Delivery Optimization](https://learn.microsoft.com/en-us/windows/deployment/update/waas-delivery-optimization)
- [Wi-Fi Sense FAQ](https://web.archive.org/web/20160801031757/http://www.microsoft.com/en-us/windows/windows-10-wifi-sense-faq)

---

*本文基于 Windows 11 24H2 行为分析和微软官方文档编写。系列完结。*

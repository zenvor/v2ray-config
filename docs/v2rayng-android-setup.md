# v2rayNG 安卓新手机配置教程

适用于官方 v2rayNG 2.2.6、普通 VPN 模式和由客户端生成的节点配置。先添加并选择可用节点，再按下面的步骤配置。自行导入的完整 Xray JSON 配置不保证采用这些界面设置。

## 1. 开启本地 DNS 和虚拟 DNS

进入「设置」→「VPN 设置」：

- **启用本地 DNS：开启。** 让客户端处理应用的 DNS 请求，使用下面配置的远程、境内 DNS，而不是只依赖 VPN DNS。
- **启用虚拟 DNS（FakeDNS）：可开启，随后验证常用应用。** 在本地返回虚拟 IP，并在连接进入内核时恢复域名，减少部分连接的 DNS 等待，方便按域名分流；不是让真实 IP 的查询本身变快，也不保证所有应用更快。

再到「设置」→「核心设置」：

- **启用流量探测：保持默认开启。** 它从 HTTP、TLS 等流量中探测域名，帮助按域名分流。2.2.6 中文界面不叫“域名嗅探”。
- **启用 routeOnly：本教程保持默认关闭。** 普通流量探测可能将原目标 IP 改为域名，随后重新解析；开启 routeOnly 则让普通嗅探结果仅用于路由判断。若要改变它，应另行验证分流与应用兼容性。

开启虚拟 DNS 后，v2rayNG 会自动为内核配置 `fakedns` 域名恢复；这与普通流量探测不同，不需要额外寻找“域名嗅探”开关。这里的默认开启是客户端生成配置时的行为，不代表 Xray 在所有配置下都自动开启嗅探。

开启本地 DNS 后，「VPN DNS」变为不可编辑是正常现象，无需修改它。

FakeDNS 不会让所有连接都省去真实解析：直连域名仍需取得真实 IP，未命中域名规则、需要判断 IP 归属的连接也要解析。应用自行使用加密 DNS 的查询不一定经过这里的 DNS 模块。

若个别应用无法使用，或关闭 VPN 后暂时无法联网，可关闭虚拟 DNS、重新连接并重启受影响应用；仍不恢复时，可切换网络或重启手机以清理残留状态。本地 DNS 保持开启。普通流量探测与 FakeDNS 是不同机制，不应仅凭上传变慢就认定 FakeDNS 有问题。

## 2. 设置 DNS

在「设置」→「核心设置」中分别填写以下地址，每栏只填一个，保存时检查没有多余空格：

| 设置项 | 地址 |
| --- | --- |
| 远程 DNS | `https://cloudflare-dns.com/dns-query` |
| 境内 DNS | `https://dns.alidns.com/dns-query` |

远程 DNS 使用 Cloudflare DoH，境内 DNS 使用阿里 DoH。这里的「本地 DNS」是客户端的 DNS 处理功能，不代表把所有域名交给国内 DNS。

客户端根据启用的域名路由规则生成 DNS 匹配规则：匹配直连域名集合的真实查询优先使用境内 DNS；未匹配任何 DNS 域名规则、也未命中 DNS Hosts 的未知域名使用远程 DNS。境内 DNS 项设置了 `skipFallback`，不会作为未知域名的通用备用服务器。

这不代表国内域名绝不查询远程 DNS：境内查询失败或结果不符合筛选条件时，可能回退到远程 DNS；同一域名匹配多条规则也可能形成多个 DNS 条目。每栏只填一个，避免客户端因远程与境内地址总数大于两个而自动启用并行查询，但不能据此保证日志中永远只有一次查询。FakeDNS 开启时，应用可能先收到虚拟 IP，真正需要真实解析时再使用相应服务器。

在本教程规则下，阿里 DoH 查询走直连，Cloudflare DoH 查询走代理。需要可用节点；不要将远程地址改为 `https+local://`，该形式会绕过内核路由直接查询。

## 3. 更新 Geo 数据文件

1. 从主界面菜单进入「资源文件」。
2. 将「Geo 文件来源」选为 **`Loyalsoldier/v2ray-rules-dat`**。
3. 点击右上角下载按钮，检查 `geosite.dat`、`geoip.dat`、`geoip-only-cn-private.dat` 的下载结果和更新时间。

导入 JSON 规则不会更新这些文件。域名集合和中国 IP 判断依赖实际安装的数据；2.2.6 会将路由中的 `geoip:cn`、`geoip:private` 转为引用 `geoip-only-cn-private.dat`，因此不要只更新 `geosite.dat`。精简 IP 文件的下载地址由客户端单独指定，不完全取决于上面的 Geo 来源选择。

## 4. 导入路由规则

1. 在手机浏览器地址栏粘贴并打开以下链接：

   <https://raw.githubusercontent.com/zenvor/v2ray-config/main/rules/v2rayng.json>

2. 复制页面中的 **JSON 全文**。剪贴板里应该是以 `[` 开头的规则内容，而不是链接。
3. 打开 v2rayNG →「路由设置」。
4. 点击右上角「更多」按钮 →「从剪贴板导入规则集」。不要选择软件内置模板的「导入预定义规则集」。
5. 在「域名策略」中选择 **`IPIfNonMatch`**。
6. 检查规则顺序和开关，然后重新连接 VPN。

已有自定义规则的手机先用「导出至剪贴板」备份。成功导入会替换未锁定的旧规则；锁定规则保留，并排在新导入规则前面，可能优先影响分流或造成重复，需要逐条检查。规则导入不会自动修改 DNS、FakeDNS 或域名策略，前面步骤需要手动完成。

## 5. 检查导入结果

规则应按下面的顺序排列：

1. 阻断 UDP 443
2. AI 服务域名代理
3. Google 域名代理
4. 绕过局域网 IP
5. 绕过局域网域名
6. 绕过中国公共 DNS IP
7. 绕过中国公共 DNS 域名
8. 绕过中国 IP
9. 绕过中国域名

各条规则可独立开关。不要在最后添加覆盖所有连接的通用代理规则，否则可能阻止 `IPIfNonMatch` 对未知域名解析后再次匹配 IP 规则。

当前策略：先阻断 UDP 443；其余连接命中已收录的 AI 域名时优先代理，国内域名和局域网按规则直连。未命中前面规则、只有域名的请求，由 `IPIfNonMatch` 触发真实解析后再匹配 IP 规则：中国 IP 直连，其余默认代理；解析结果中任一 IP 命中中国 IP 规则即可触发直连。请求已有真实 IP 时按实际可见的目标和规则顺序判断，普通嗅探可能改变目标。

未被前置规则覆盖的新 AI 域名也遵循未知域名策略，不能保证一定代理。已知 AI 域名规则还包含共享的第三方服务，其他应用访问这些匹配域名时也会走代理。关闭某条直连规则后，仍可能命中其他直连规则。

## 6. 验证联网

连接后检查 ChatGPT／Claude、国内视频连续播放与切换，以及 QQ 图片、视频发送；再切换一次 Wi-Fi／移动网络，并检查关闭 VPN 后国内应用是否正常联网。播放正常不等于上传也正常。

若某个应用明显变慢，保持同一网络和文件，分别比较 VPN 关闭、VPN 开启且 FakeDNS 关闭、VPN 开启且 FakeDNS 开启。切换设置后重连 VPN、重启受影响应用，再对照日志；区分发送前等待和上传过程缓慢。

规则文件不会自动订阅更新。以后更新规则或换手机时，重新打开链接、复制 JSON 全文并导入。

## 核对依据

- [v2rayNG 2.2.6 设置项与默认值](https://github.com/2dust/v2rayNG/blob/2.2.6/V2rayNG/app/src/main/res/xml/pref_settings.xml)
- [2.2.6 中文界面名称](https://github.com/2dust/v2rayNG/blob/2.2.6/V2rayNG/app/src/main/res/values-zh-rCN/strings.xml)
- [2.2.6 DNS、FakeDNS 与路由配置生成](https://github.com/2dust/v2rayNG/blob/2.2.6/V2rayNG/app/src/main/java/com/v2ray/ang/core/CoreConfigManager.kt)
- [2.2.6 规则导入与锁定规则保留逻辑](https://github.com/2dust/v2rayNG/blob/2.2.6/V2rayNG/app/src/main/java/com/v2ray/ang/handler/SettingsManager.kt)
- [2.2.6 资源文件下载逻辑](https://github.com/2dust/v2rayNG/blob/2.2.6/V2rayNG/app/src/main/java/com/v2ray/ang/viewmodel/UserAssetViewModel.kt)
- [Xray 路由文档](https://xtls.github.io/config/routing.html)、[DNS 文档](https://xtls.github.io/config/dns.html)、[FakeDNS 文档](https://xtls.github.io/config/fakedns.html)

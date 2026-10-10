# v2ray-config

适用于 v2rayNG（Android）和 v2rayN（电脑）的 Xray 路由规则与 DNS 参考配置。

新手机配置请看：[v2rayNG 安卓配置教程](docs/v2rayng-android-setup.md)。

## 策略

以 v2rayNG 2.2.6 内置白名单为基础，保留原始八条规则及相对顺序，仅插入 AI 优先代理规则：

UDP443 阻断 → AI 域名优先代理 → Google 域名代理 → 局域网 IP 直连 → 局域网域名直连 → 中国公共 DNS IP 直连 → 中国公共 DNS 域名直连 → 中国 IP 直连 → 中国域名直连 → 未匹配默认代理。

各类直连规则独立开关，不将私网和中国规则合并。Loyalsoldier 的 `geosite:cn` 已覆盖 `.cn` 后缀，不额外添加重复的 `.cn` 直连规则。关闭局域网 IP 直连不等于关闭局域网域名直连，需按需求分别调整；关闭某条规则后仍会继续匹配其他启用规则。

`geosite:google`、`geosite:cn`、`geosite:private`、`geoip:cn`、`geoip:private` 必须存在于客户端实际数据文件。这里提供可移植的基础规则，未包含 Mihomo 的广告集合、Apple/iCloud、游戏和 Google Play 特殊规则；导入前将自己需要的旧规则合并进去。域名清单保留社区 datadog/sift 关键词，其他应用访问共享主机也会被代理。

## 导入

- v2rayNG：复制 `rules/v2rayng.json` 全文 → 路由设置 → 从剪贴板导入规则集。不是“导入预定义规则集”（那个入口选择软件内置模板）。先导出备份；导入可能替换未锁定旧规则，锁定规则可能合并后改变顺序，应逐条复核。
- v2rayN：路由设置中新建独立规则集，使用从文件/剪贴板导入 `rules/v2rayn.json`，然后选择它。两文件当前内容一致，但分别保留发布路径；不同客户端版本的导入模型需实际验证。
- 两端域名解析策略设置为 `IPIfNonMatch`。不要在末尾加覆盖所有端口/network 的通用代理规则，否则第一轮总能匹配，阻止未知域名进入解析后二次匹配。确认生成配置的第一个 outbound 是 `proxy`，未匹配连接才默认代理。
- 规则导入只导入规则，不自动设置 DNS、解析策略、嗅探、VPN或节点。

## DNS

v2rayNG 推荐启用本地 DNS 和虚拟 DNS（FakeDNS），并设置：

- 远程 DNS：`https://cloudflare-dns.com/dns-query`
- 境内 DNS：`https://dns.alidns.com/dns-query`
- 域名策略：`IPIfNonMatch`

具体操作、规则导入和检查方法见[安卓配置教程](docs/v2rayng-android-setup.md)。v2rayN 的 DNS 设置需单独配置。

`examples/xray-dns-routing.fragment.json` 为手动配置 Xray 的 DNS 与路由参考片段，需要与完整配置合并使用。

## 维护与验证

导入不是持续订阅；v2rayNG 2.2.6 该页面没有自定义规则 URL 导入。仓库已公开，可匿名下载 raw 文件；客户端仍需人工下载/复制导入更新。

前置域名覆盖根域与新增子域，不能自动覆盖新独立域名或只有 IP 的请求。未覆盖域名解析到 CN IP 仍会直连；本配置不是账号安全保证。

Google 代理及中国公共 DNS IP/域名直连列表来自 v2rayNG 2.2.6 的内置白名单 `custom_routing_white`，保留完整列表。DNS IP 直连规则只决定访问这些服务器时的出站，不代表启用这些服务器解析其他域名；DNS 服务器仍由客户端设置决定。

来源：
- https://github.com/2dust/v2rayNG/blob/2.2.6/V2rayNG/app/src/main/assets/custom_routing_white
- https://help.openai.com/zh-hans-cn/articles/9247338-network-recommendations-for-chatgpt-errors-on-web-and-apps
- https://code.claude.com/docs/en/desktop#network-access-requirements
- https://code.claude.com/docs/en/network-config#network-access-requirements
- https://x.com/wlzh/status/2108017417670860900 （补充社区域名，不采用帖子 direct/IP/ASN）
- https://github.com/2dust/v2rayNG/blob/2.2.6/V2rayNG/app/src/main/java/com/v2ray/ang/dto/entities/RulesetItem.kt
- https://github.com/2dust/v2rayN/wiki/Description-of-custom-routing-rules
- https://xtls.github.io/config/routing.html
- https://xtls.github.io/config/dns.html

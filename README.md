# Shadowrocket Smart Split

这是一份面向中国大陆网络环境的长期常驻分流配置：Shadowrocket 可以一直开启，国内常用 App/服务优先 `DIRECT`，国外常用服务优先走 `国外代理`，其余未知流量最终也走代理。

配置不包含节点、账号或订阅地址。导入后仍需使用自己的节点订阅。

## 使用前的关键设置

1. 在 Shadowrocket 中导入 `shadowrocket_smart_split.conf`。
2. 确认你的美国服务器已添加，并在 Shadowrocket 的节点列表中通过“连通性测试”。本配置不要求地区节点组；“国外代理”直接使用 `PROXY`，因此单节点也适用。
3. Shadowrocket 首页 → 全局路由 → 选择“配置”。不要选择“代理”；选择“代理”会绕过本文件的分流规则。
4. 开启 Shadowrocket 后，先打开一个国内 App 和一个国外服务，观察请求记录中的策略结果。

## 分流顺序

| 顺序 | 范围 | 策略 | 目的 |
| --- | --- | --- | --- |
| 0 | 局域网、本机地址 | `DIRECT` | 保证局域网设备和本机服务可用 |
| 1 | OpenAI、Google、YouTube、X/Twitter、Telegram、TikTok、Instagram、Facebook、GitHub、Reddit、Discord、Microsoft、Spotify、PayPal | `国外代理` | 先处理明确的国外服务，避免被国内综合规则误判 |
| 2 | 微信、支付宝、京东、拼多多、美团、大众点评、饿了么、高德、百度、抖音、小红书、微博、知乎、B站、网易云音乐、QQ、12306、主要银行/证券服务 | `DIRECT` | 高频国内业务优先直连 |
| 3 | `China.list` | `DIRECT` | 国内域名综合兜底 |
| 4 | `Global.list` | `国外代理` | 已知全球服务补充兜底 |
| 5 | `GEOIP,CN` | `DIRECT` | 中国 IP 兜底 |
| 6 | `FINAL` | `国外代理` | 新出现的国外服务不会因漏收录而默认直连 |

规则是按请求的域名/IP/用户代理匹配的，不是把整个 App 永久锁定为一种线路。因此，在微信里打开一个国外网页时，该网页仍可能走代理；这属于预期行为。

## DNS 设计

- `doh.pub`、阿里 DoH 和阿里/腾讯公共 DNS 用于中国大陆环境下的常规解析，优先保证国内 CDN 和国内直连服务的可用性。
- `direct-dns-server` 明确指定直连域名使用上述解析器，避免导入后因默认值变化而产生差异。
- `dns-direct-fallback-proxy = true` 允许直连域名解析失败时经代理重试，减少国外域名被国内 DNS 污染或拦截后打不开的问题。
- `hijack-dns` 接管常见硬编码的 `8.8.8.8/8.8.4.4` DNS 请求，避免 App 绕过 Shadowrocket 的 DNS 处理。
- 保留 IPv6，但不优先使用 IPv6；代理流量仅屏蔽 QUIC，国内直连不强制禁用 UDP/443。

这套设置的取舍是“国内解析优先、国外失败可回退”，而不是把所有 DNS 查询都强制交给某一个海外 DNS。若你的网络本身有稳定的 IPv6 或自建 DNS，可以再单独调整，不建议一开始同时改动多项。

## 如何验证

### 微信、小红书是否直连

1. 打开 Shadowrocket 的请求记录/日志。
2. 微信发送一条消息、打开公众号或小程序；关注 `weixin.qq.com`、`wechat.com`、`servicewechat.com`、`qpic.cn`、`wx.qq.com` 等请求，策略应显示 `DIRECT`。
3. 打开小红书首页并刷新；其规则命中的请求应显示 `DIRECT`。
4. 如果某个微信小程序内嵌了外部网站，那个外部网站可能显示为 `国外代理`，不要把它当成微信主链路误分流。

### ChatGPT、YouTube 是否代理

1. 打开 ChatGPT，观察 `openai.com`、`chatgpt.com`、`oaiusercontent.com` 等请求，策略应显示 `国外代理`。
2. 打开 YouTube 并播放视频，观察 `youtube.com`、`googlevideo.com`、`ytimg.com`、`ggpht.com` 等请求，策略应显示 `国外代理`。
3. 如果策略正确但页面仍打不开，先检查这条美国服务器是否可用；不要先把规则改成 `DIRECT`。

请求记录里的策略结果比只看 iPhone 顶部的 VPN 图标更可靠。VPN 图标只表示 Shadowrocket 的系统网络接口已开启，不代表所有连接都经过代理节点。

## 规则源与维护

本配置只引用 `blackmatrix7/ios_rule_script` 的 `Shadowrocket` 目录规则，避免把 `QuantumultX` 专用文本直接当成 Shadowrocket 规则使用。

- 规则仓库：[blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script)
- 国内综合规则：[Shadowrocket/China.list](https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/rule/Shadowrocket/China/China.list)
- 更激进的可选规则：[Shadowrocket/ChinaMaxNoIP.list](https://raw.githubusercontent.com/blackmatrix7/ios_rule_script/master/rule/Shadowrocket/ChinaMaxNoIP/ChinaMaxNoIP.list)

默认使用 `China.list`，因为它规模较小、误判面更容易控制。当前 `ChinaMaxNoIP` 约 11 万条规则，并包含更宽泛的 TLD、关键词和用户代理匹配；如果你把它替换进来，建议替换而不是和 `China.list` 同时启用，并保留本配置中 TikTok、Microsoft 等国外例外规则在它之前。

建议每月或遇到某个 App 分流异常时：

1. 在 Shadowrocket 中更新规则。
2. 检查规则地址是否仍返回文本而不是 404/HTML。
3. 查看请求记录，确认实际命中的策略。
4. 只对确认误判的域名增加一条窄范围规则，不要为了“规则数量更多”而加入大量关键词规则。

本版本没有启用 MITM 或 URL Rewrite。除非你明确需要 HTTPS 解密，否则不建议为分流而开启证书解密；这能减少 iOS 证书固定、登录和支付类 App 的兼容问题。

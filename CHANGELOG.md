# Changelog

## 2026-09-04

- 新建 `shadowrocket_smart_split.conf`，不覆盖原始 `lazy_group.conf`。
- 将原配置中的 `QuantumultX/WeChat`、`QuantumultX/China`、`QuantumultX/Global` 引用改为 `Shadowrocket` 规则源，避免格式混用。
- 增加微信、支付宝、京东、拼多多、美团/大众点评、饿了么、高德、百度、抖音、小红书、微博、知乎、B站、网易云音乐、12306、QQ/Tencent 以及主要银行/证券规则。
- 增加 Instagram、Reddit、Discord、Microsoft 等国外服务规则，并将 TikTok 置于国内综合规则之前，避免被国内宽泛用户代理规则抢先匹配。
- 增加淘宝、天猫、QQ、腾讯域名的少量明确直连规则；未引用已确认不存在的通用 `Bank`、`Securities`、`QQ`、`Taobao` 目录。
- 将国内综合兜底设为较保守的 `China.list`，保留 `GEOIP,CN,DIRECT` 和 `FINAL,国外代理`。
- 合并并简化代理组：所有国外规则统一使用 `国外代理`，避免原配置中大小写不一致的策略组名称带来的兼容性风险。
- DNS 增加明确的 `direct-dns-server`、代理回退和 DNS 劫持设置；保留 IPv4 优先与“仅阻断代理 QUIC”。
- 移除与分流目标无关的 Google URL Rewrite 和 MITM 配置，降低长期运行时的证书/登录兼容风险。

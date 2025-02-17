# [ShellCrash](https://github.com/juewuy/ShellCrash) 使用 [sing-box PuerNya 版内核](https://github.com/PuerNya/sing-box)进行 DNS 分流教程-ruleset方案
注：
- 1. 此方案适用于 [sing-box](https://github.com/SagerNet/sing-box)，采用 `rule_set` 规则
- 2. 搭配 [AdGuardHome](https://github.com/AdguardTeam/AdGuardHome) 并将 AdGuardHome 作为上游时不要使用该方法
- 3. DNS 分流简单来说就是**指定国内域名走国内 DNS 解析，其它域名包括国外域名走 fakeip**
- 5. 此方案自定义规则参考 [DustinWin/ruleset_geodata/ruleset](https://github.com/DustinWin/ruleset_geodata#%E4%BA%8C-ruleset-%E8%A7%84%E5%88%99%E9%9B%86%E6%96%87%E4%BB%B6%E8%AF%B4%E6%98%8E)
- 6. 所有步骤完成后，请连接 SSH 执行命令 `$CRASHDIR/start.sh restart` 后生效
---
# 一、 导入 mihomo 内核
可参考《[ShellCrash 配置-ruleset 方案/导入 sing-box PuerNya 版内核](https://github.com/DustinWin/clash_singbox-tutorials/blob/main/%E6%95%99%E7%A8%8B%E5%90%88%E9%9B%86/sing-box/%E5%9F%BA%E7%A1%80%E7%AF%87/ShellCrash%20%E9%85%8D%E7%BD%AE-ruleset%20%E6%96%B9%E6%A1%88.md#%E4%B8%80-%E5%AF%BC%E5%85%A5-sing-box-puernya-%E7%89%88%E5%86%85%E6%A0%B8)》里的步骤进行操作
# 二、 ShellCrash 设置
1. 进入主菜单 -> 2 内核功能设置 -> 2 切换 DNS 运行模式 -> 4 DNS 进阶设置，将“当前基础 DNS”和“PROXY-DNS”都设置为“null”  
<img src="https://github.com/DustinWin/clash_singbox-tutorials/assets/45238096/387ab5e9-b659-4469-9e74-aee264d53944" width="60%"/>

2. 其它设置可参考《[ShellCrash 配置-ruleset 方案](https://github.com/DustinWin/clash_singbox-tutorials/blob/main/%E6%95%99%E7%A8%8B%E5%90%88%E9%9B%86/sing-box/%E5%9F%BA%E7%A1%80%E7%AF%87/ShellCrash%20%E9%85%8D%E7%BD%AE-ruleset%20%E6%96%B9%E6%A1%88.md)》
# 三、 导入 dns.json 文件
注：
- 1. `dns.fakeip.exclude_rule` 中的 `"rule_set": [ "fakeip-filter" ]` 已添加 [ShellCrash/public/fake_ip_filter.list](https://github.com/juewuy/ShellCrash/blob/dev/public/fake_ip_filter.list)（已添加 AdGuardHome 相关域名，包括：`adguardteam.github.io`、`adrules.top`、`anti-ad.net` 和 `static.adtidy.org`，防止作为下游时检查更新和下载“DNS 黑名单”失败），提高兼容性
- 2. `dns.fakeip.exclude_rule` 中的 `"rule_set": [ "private" ]` 已添加 [TrackersList](https://github.com/XIU2/TrackersListCollection/blob/master/all.txt)，防止 [BT 下载](https://github.com/c0re100/qBittorrent-Enhanced-Edition)无法连接 TrackersList UDP 协议

连接 SSH 后执行 `vi $CRASHDIR/jsons/dns.json`，按一下 Ins 键（Insert 键），粘贴如下内容：
```
{
  "dns": {
    "servers": [
      { "tag": "dns_block", "address": "rcode://success" },
      { "tag": "dns_direct", "address": [ "https://1.12.12.12/dns-query", "https://223.5.5.5/dns-query" ], "detour": "DIRECT" },
      { "tag": "dns_proxy", "address": [ "https://8.8.8.8/dns-query", "https://1.1.1.1/dns-query" ] },
      { "tag": "dns_fakeip", "address": "fakeip" }
    ],
    "rules": [
      { "outbound": "any", "server": "dns_direct" },
      { "clash_mode": "Direct", "query_type": [ "A", "AAAA" ], "server": "dns_direct" },
      { "clash_mode": "Global", "query_type": [ "A", "AAAA" ], "server": "dns_proxy" },
      { "rule_set": [ "ads" ], "server": "dns_block" },
      { "rule_set": [ "cn" ], "query_type": [ "A", "AAAA" ], "server": "dns_direct" },
      { "query_type": [ "A", "AAAA" ], "server": "dns_fakeip" }
    ],
    "final": "dns_direct",
    "strategy": "prefer_ipv4",
    "independent_cache": true,
    "reverse_mapping": true,
    "mapping_override": true,
    "fakeip": { "enabled": true, "inet4_range": "198.18.0.0/15", "inet6_range": "fc00::/18", "exclude_rule": { "rule_set": [ "fakeip-filter", "private" ] } }
  }
}
```

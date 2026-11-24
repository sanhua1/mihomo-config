# Mihomo Config

供 Sub-Store「文件 → mihomo 配置」使用的通用模板。仓库只保存分流、策略组、DNS、嗅探和 TUN 配置，不保存节点或订阅凭据。

## 架构

| 组成 | 职责 |
| --- | --- |
| GitHub Mihomo Template | 保存稳定的 `proxy-groups`、`rule-providers` 和 `rules` |
| Sub-Store 动态节点 | 从单条订阅或组合订阅生成 `proxies` |
| 最终配置 | Sub-Store 把节点写入模板的 `proxies` 字段 |

```text
[GitHub Mihomo Template] ──┐
                           ├── Sub-Store mihomo 配置 ── 最终完整配置
[Sub-Store 动态节点] ──────┘
```

## Sub-Store 使用方法

1. 先创建单条订阅或组合订阅。
2. 进入「文件」，新建类型为「mihomo 配置」的文件。
3. 来源选择「远程」，链接填入下方 Raw URL，模式选择「作为 mihomo 配置」。
4. 在「操作」中添加「从订阅添加节点」。
5. 来源选择单条订阅或组合订阅，选中对应名称，节点添加方式选择「覆盖」。
6. 即时预览并保存。预览结果应同时包含 `proxies`、`proxy-groups`、`rule-providers` 和 `rules`。

```text
[订阅/组合订阅] ── 生成 mihomo 节点 ── 写入 proxies
[远程模板] ─────── 读取完整配置 ──────┴── 输出配置
```

模板 Raw URL：

```text
https://raw.githubusercontent.com/sanhua1/mihomo-config/main/templates/main.yaml
```

Sub-Store 当前实现会把所选订阅或组合订阅生成的节点写入 `proxies`。`✈️ 节点选择` 使用 `include-all: true` 收纳全部节点。`🎥 观影` 也收纳相同节点，并提供“🎯 全球直连”和“✈️ 节点选择”选项，默认直连。`英雄联盟登录` 独立列出相同的全部有效节点，并提供 `DIRECT` 选项。

## 模板约定

- `templates/main.yaml` 不含 `proxies` 和 `proxy-providers`，也不含任何真实节点。
- `✈️ 节点选择` 不按国家或地区分类，只排除流量、到期时间、官网等非节点条目。
- `🏠 固定直连` 只含 `DIRECT`，`🚀 固定代理` 只引用 `✈️ 节点选择`，用于覆盖后续规则。
- 固定规则位于 `rules` 最顶部；当前 `qidian.com`、`090227.xyz`、`jarimedical.com` 以及包含 `bili` 或 `gitee` 的域名固定直连，包含 `mineru` 或 `micuapi` 的域名固定走代理。关键词只匹配域名，不匹配网页内容。
- `🎥 观影` 匹配 `uhdgo.com`、`uhdnow.com`、`pro.emby.moe`、`cfroute.dingone.top` 及其子域名，其中 `cfroute.dingone.top` 规则须位于 `dingone.top` 规则之前；节点过滤条件与 `✈️ 节点选择` 保持一致。
- `英雄联盟登录` 不按地区过滤节点，使用与 `✈️ 节点选择` 相同的非节点排除条件。Riot 登录域名及客户端、游戏进程（含 macOS 的 `Riot Client` 及其 Helper）交给该组；`英雄联盟下载` 同样列出全部有效节点和 `DIRECT`，接管 `riotcdn.net`、`rgpub.io`、`featureassets.org`、`prodregistryv2.org` 等更新与资源下载域名，规则优先于登录规则；`151.106.0.0/16`、`13.214.0.0/16` 的 UDP 流量优先固定直连，TCP 不受这两条直连规则影响。
- `📈 网络测试` 可直接选择与 `✈️ 节点选择` 相同的全部有效节点，保留“🎯 全球直连”和“✈️ 节点选择”选项，默认直连。
- 其余服务策略组保持二元选择，并按原配置保留直连优先或节点优先的顺序。
- 公共规则来自 ACL4SSR。自定义规则位于 `rules` 顶部，优先于所有 `RULE-SET`。
- Mihomo Party 的控制器、面板和口令由客户端管理，不写入公共模板。

修改分流时，先确认 `rules` 目标组存在于 `proxy-groups`。新增 `RULE-SET` 时，还要在 `rule-providers` 中加入同名规则源。需要过滤订阅中的非节点条目时，同步调整 `✈️ 节点选择`、`🎥 观影`、`📈 网络测试`、`英雄联盟登录` 和 `英雄联盟下载` 的 `exclude-filter`。

## 英雄联盟登录

在客户端启用 TUN 和规则模式，再在 `英雄联盟登录` 中手动选择能登录台服的节点。该选择独立于 `✈️ 节点选择` 和 `🎮 游戏平台`。选择 `DIRECT` 时，Mihomo 会通过系统网络发起出站连接。macOS 上使用网易 UU 双开时，按下方的启动顺序操作。

客户端更新和资源下载走 `英雄联盟下载`，可单独选一个下载快的节点，不影响登录走 `DIRECT` 配合加速器。若在系统代理绕过中加入了 `riotcdn.net`，需将其移除，否则更新下载会绕过 Clash。

对局 UDP 直连网段来自参考配置，需以实际游戏连接为准；可在客户端的连接记录中确认对局命中 `DIRECT`、登录和客户端连接命中 `英雄联盟登录`。

### macOS 的 Clash TUN 与网易 UU 双开

主模板已包含 UU 双开路由。Sub-Store 按原地址生成的订阅会带上这组路由，Clash Party 更新订阅后即可加载。需要双开时，先让 UU 开始加速英雄联盟台服，再开启 Clash TUN。

这组路由让普通公网 IPv4 优先进入 Clash，并将 `163.163.0.0/16` 交给系统已有的路由。UU 运行时，其台服登录地址继续走 UU。UU 的 DNS 地址 `6.6.6.6/32` 通过更具体的系统路由走 UU。这组 IPv4 路由会随主模板下发到所有设备。已在 macOS 上验证台服登录和 STUN 出口，对局内加速需单独验证。

## DNS 防泄露

- `tun.strict-route: true`：阻止 Windows 多网卡并发 DNS、局域网网关 DNS 等绕过 TUN 的查询。若向日葵、局域网游戏等软件异常，可在客户端临时关闭对比。
- `nameserver` 使用 `1.1.1.1`、`8.8.8.8` 的 DoH，并通过 `#✈️ 节点选择` 经代理发出；`geosite:private,cn` 域名由 `nameserver-policy` 交给 `doh.pub`、`alidns` 直连解析。
- `direct-nameserver`、`default-nameserver`、`proxy-server-nameserver` 使用国内 DoH，直连域名和节点域名的解析不受影响；模板内不含明文 DNS。
- Clash Party 的 TUN 设置由客户端管理，会覆盖模板中的 `strict-route`。需在客户端「虚拟网卡」设置中手动开启「严格路由」，并在 `%APPDATA%\mihomo-party\work\config.yaml` 中确认 `strict-route: true` 已生效。
- `geosite:cn` 依赖 GeoSite 数据，mihomo 首次启动会自动下载。
- 浏览器的「使用安全 DNS」会绕过 mihomo 的 DNS，测试泄露前建议关闭。

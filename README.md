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

Sub-Store 当前实现会把所选订阅或组合订阅生成的节点写入 `proxies`。`✈️ 节点选择` 使用 `include-all: true` 收纳全部节点。`🎥 观影` 也收纳相同节点，并提供“🎯 全球直连”和“✈️ 节点选择”选项，默认直连。`英雄联盟台服` 独立列出相同的全部有效节点，并提供 `DIRECT` 选项。

## 模板约定

- `templates/main.yaml` 不含 `proxies` 和 `proxy-providers`，也不含任何真实节点。
- `✈️ 节点选择` 不按国家或地区分类，只排除流量、到期时间、官网等非节点条目。
- `🏠 固定直连` 只含 `DIRECT`，`🚀 固定代理` 只引用 `✈️ 节点选择`，用于覆盖后续规则。
- 固定规则位于 `rules` 最顶部；当前 `qidian.com`、`090227.xyz` 以及包含 `bili` 或 `gitee` 的域名固定直连，包含 `mineru` 或 `micuapi` 的域名固定走代理。关键词只匹配域名，不匹配网页内容。
- `🎥 观影` 匹配 `uhdgo.com`、`uhdnow.com` 及其子域名，节点过滤条件与 `✈️ 节点选择` 保持一致。
- `英雄联盟台服` 不按地区过滤节点，使用与 `✈️ 节点选择` 相同的非节点排除条件。Riot 相关域名及常见客户端、游戏进程交给该组；`151.106.0.0/16`、`13.214.0.0/16` 的 UDP 流量优先固定直连，TCP 不受这两条直连规则影响。
- 其余服务策略组保持二元选择，并按原配置保留直连优先或节点优先的顺序。
- 公共规则来自 ACL4SSR。自定义规则位于 `rules` 顶部，优先于所有 `RULE-SET`。
- Mihomo Party 的控制器、面板和口令由客户端管理，不写入公共模板。

修改分流时，先确认 `rules` 目标组存在于 `proxy-groups`。新增 `RULE-SET` 时，还要在 `rule-providers` 中加入同名规则源。需要过滤订阅中的非节点条目时，同步调整 `✈️ 节点选择`、`🎥 观影` 和 `英雄联盟台服` 的 `exclude-filter`。

## 英雄联盟台服

在客户端启用 TUN 和规则模式，再在 `英雄联盟台服` 中手动选择能登录台服的节点。使用游戏加速器时，可选 `DIRECT`，让登录和客户端流量通过系统网络出站，由加速器按其配置接管。该选择独立于 `✈️ 节点选择` 和 `🎮 游戏平台`。

对局 UDP 直连网段来自参考配置，需以实际游戏连接为准；可在客户端的连接记录中确认对局命中 `DIRECT`、登录和客户端连接命中 `英雄联盟台服`。

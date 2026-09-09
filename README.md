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

Sub-Store 当前实现会把所选订阅或组合订阅生成的节点写入 `proxies`。`✈️ 节点选择` 使用 `include-all: true` 收纳全部节点，其余服务组只保留“✈️ 节点选择”和“🎯 全球直连”两类选择。

## 模板约定

- `templates/main.yaml` 不含 `proxies` 和 `proxy-providers`，也不含任何真实节点。
- `✈️ 节点选择` 不按国家或地区分类，只排除流量、到期时间、官网等非节点条目。
- 服务策略组保持二元选择，并按原配置保留直连优先或节点优先的顺序。
- 公共规则来自 ACL4SSR。自定义规则位于 `rules` 顶部，优先于所有 `RULE-SET`。
- Mihomo Party 的控制器、面板和口令由客户端管理，不写入公共模板。

修改分流时，先确认 `rules` 目标组存在于 `proxy-groups`。新增 `RULE-SET` 时，还要在 `rule-providers` 中加入同名规则源。需要过滤订阅中的非节点条目时，只调整 `✈️ 节点选择` 的 `exclude-filter`。

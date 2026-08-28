# wallet-cli chain

查询链参数、资源单价与节点信息。

三个只读查询，用于费用估算、质押/投票决策和故障排查。注意与 [`networks`](../networks.md) 区分：后者只列出本地已知的网络，不访问任何节点；而 `chain` 查询的是 `--network` 选中的那个节点。

## 用法

```
wallet-cli chain COMMAND
```

## 子命令

| 命令 | 页面 | 说明 |
|---|---|---|
| `chain params` | [params.md](params.md) | 链上治理参数 |
| `chain prices` | [prices.md](prices.md) | 能量/带宽单价与备注费用 |
| `chain node` | [node.md](node.md) | 所连节点的状态（版本 / 同步 / 对等节点） |

## 另请参见

[`networks`](../networks.md) · [能量与带宽](../../concepts/energy-bandwidth.md) · [故障排查](../../troubleshooting.md)

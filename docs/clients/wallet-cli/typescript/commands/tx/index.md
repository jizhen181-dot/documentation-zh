# wallet-cli tx

构建、发送、广播、联署以及查看交易。

## 用法

```
wallet-cli tx COMMAND
```

## 子命令

| 命令 | 说明 |
|---|---|
| [`tx send`](send.md) | 用人类可读的 `--amount` 发送原生 TRX 或 TRC20/TRC10 token |
| [`tx broadcast`](broadcast.md) | 广播一笔预先签名的交易 |
| [`tx status`](status.md) | 查看交易的确认状态 |
| [`tx info`](info.md) | 查看交易的完整详情和交易回执 |
| [`tx sign`](sign.md) | 为交易 hex 添加你的签名（多签联署） |
| [`tx approvals`](approvals.md) | 查看交易 hex 的签名权重和已批准签名者列表 |
| [`tx multisig`](multisig.md) | 通过 TronLink 服务进行多签协作（列出 / 创建 / 联署 / 监听） |

## 交易生命周期

```
build ──sign──> submit ──solidify──> confirmed
  │                │                     │
  └ --dry-run      └ default return      └ tx status: confirmed/failed
    stops here       point ("submitted")   (pending/not_found while in flight)
```

`tx send` 一步完成构建+签名+提交（用 `--dry-run` / `--sign-only` 可以提前停下）；`tx broadcast` 提交在别处签好名的交易；`tx status` / `tx info` 用于观察结果。**提交不等于确认**——脚本必须遵循[机器接口 → 脚本安全](../../machine-interface.md#script-safety-never-mistake-submitted-for-confirmed)。

## 多签联署

对于需要多个签名的账户，有两条联署路径：

- **链上方式**——[`tx sign`](sign.md) / [`tx approvals`](approvals.md) / [`tx broadcast`](broadcast.md)：发起方用 `--sign-only` 生成一段部分签名的 hex，每位签名者用 `tx sign` 追加自己的签名（把 hex 依次传递下去），一旦达到阈值，任何人都可以广播它。自给自足，不需要任何服务。
- **服务方式**——[`tx multisig`](multisig.md)：由 TronLink 多签服务保管交易、累计签名并推送通知。开启一次签名收集就是发起方自己的第一个签名，达到阈值后由服务负责广播。这是可选的便利层，需要凭据。

无论走哪条路径，[`tx approvals`](approvals.md) 都能回答“现在够了吗？”，而 [`tx broadcast`](broadcast.md) 会拒绝提交尚未达到阈值的交易——没有收集齐的交易永远不会到达节点。

联署所依赖的账户权限结构，参见 [`permission`](../permission/index.md)。

## 另请参见

[`account history`](../account/index.md) · [网络与费用](../../concepts/networks.md) · [脚本编写指南](../../guide/scripting.md)

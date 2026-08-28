# wallet-cli account

查询链上账户状态，以及激活账户和为账户命名。

## 用法

```
wallet-cli account COMMAND
```

子命令默认作用于**当前账户**；可用 `--account <accountId|label>` 覆盖，或用 `wallet-cli use <account>` 更改默认值。前四条是只读查询；`activate` 和 `set` 会改变链上状态，需要 master password。

## 子命令

| 命令 | 说明 | 数据来源 |
|---|---|---|
| [`account balance`](balance.md) | 原生余额（TRX/SUN） | 节点 RPC |
| [`account info`](info.md) | 原始账户数据，含带宽/能量 | 节点 RPC |
| [`account history`](history.md) | 交易历史 | **需要 TronGrid** |
| [`account portfolio`](portfolio.md) | 原生 + token 余额，尽力而为的 USD 估值 | 节点 RPC + 价格源 |
| [`account activate`](activate.md) | 激活尚不存在的账户（不转账） | 广播 |
| [`account set`](set.md) | 设置链上名称 / 账户 id（一次性） | 广播 |

## 另请参见

[`list`](../list.md)——本地账户（不访问链） · [网络与资源](../../concepts/networks.md)

# 网络

wallet-cli 通过**规范 id** 来指定网络——`family:chain`：

```bash
wallet-cli networks -o json
```

```console
{"schema":"wallet-cli.result.v1","success":true,"command":"networks","data":[
  {"id":"tron:mainnet","family":"tron","chainId":"mainnet","feeModel":"tron-resource"},
  {"id":"tron:nile","family":"tron","chainId":"nile","feeModel":"tron-resource"},
  {"id":"tron:shasta","family":"tron","chainId":"shasta","feeModel":"tron-resource"}],
 "meta":{"durationMs":16,"warnings":[]}}
```

| Id | 说明 | TRX 价值 |
|---|---|---|
| `tron:mainnet` | TRON 主网 | **具有真实价值** |
| `tron:nile` | 主要测试网；水龙头在 nileex.io | 无——可自由使用 |
| `tron:shasta` | 备用测试网 | 无 |

## 命令如何选择网络 {#how-a-command-picks-its-network}

1. 命令上显式的 `--network <id>`；
2. 否则使用 `config.defaultNetwork`（`wallet-cli config defaultNetwork tron:nile`）；
3. 两者都没有的链上命令会提示你必须指定网络。

你的**地址在每个网络上都相同**，但余额、token 和交易按网络完全分离。来自 Nile 的 txid 在主网上并不
存在——在主网查询它会返回 `not_found`/`rpc_error`。

## 费用：`tron-resource` 模型 {#fees-the-tron-resource-model}

TRON 不像 EVM 链那样收取 gas。交易消耗**带宽**（字节数），智能合约调用还会消耗**能量**；不足的
部分通过燃烧 TRX 补足，而质押 TRX 可以持续获得配额。完整模型、质押命令以及解质押等待期见
[能量与带宽](energy-bandwidth.md)。

单位：**1 TRX = 1,000,000 SUN**。JSON 响应以字符串形式返回原始 SUN（`"balance": "1976489000"`
= 1976.489 TRX）；text 输出显示人类可读的 TRX。

## 另请参见

- [`account info`](../commands/account/info.md)——显示你当前的带宽/能量使用量和上限
- [快速上手](../guide/getting-started.md)——在 Nile 上给账户充值

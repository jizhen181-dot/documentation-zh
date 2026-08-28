# wallet-cli tx info

查看交易的完整详情和交易回执。

## 用法

```
wallet-cli tx info --txid <id> [options]
```

## 说明

获取完整的交易对象及其执行回执（资源消耗、合约执行结果）。它适合用于事后取证和费用分析；如果只是想简单确认“交易上链了吗？”，[`tx status`](status.md) 的开销更小——它的四个状态值是稳定的，可以直接针对它们编程。

注意两者失败方式的差异：`tx status` 对未知交易返回 `not_found`，退出码为 0；而 `tx info` 遇到未知 txid 时就是普通的**错误**（`rpc_error`，退出码 1）——因为没有任何详情可以展示（见下面的示例）。

## 选项

| 选项 | 说明 |
|---|---|
| `--txid <string>` | **必填。** TRON 交易 id/哈希 |

此外还有[全局选项](../index.md#global-options-every-command)。

## 示例

```bash
wallet-cli tx info --txid 52332505ab6b605aff626aaef2b07f3718d4bac8f45cdab1c0ea9465eb98e065 --network tron:nile
```

```console
TxID    52332505ab6b605aff626aaef2b07f3718d4bac8f45cdab1c0ea9465eb98e065
From    TMSgJxtPw29AFEHMXsjGo4kWV7UwbCToHJ
To      TGkbaCYB4kRBc3Q6wjqkACefUvRwf2KzkH
Amount  1 TRX
Status  SUCCESS
Block   #69,084,269
```

`-o json` 会返回完整详情（`transaction` 是原始交易，`info` 是交易回执；这里用 `{…}` 省略）：

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"tx.info","data":{"txid":"52332505ab6b605aff626aaef2b07f3718d4bac8f45cdab1c0ea9465eb98e065","from":"TMSgJxtPw29AFEHMXsjGo4kWV7UwbCToHJ","to":"TGkbaCYB4kRBc3Q6wjqkACefUvRwf2KzkH","amount":"1","symbol":"TRX","status":"SUCCESS","blockNumber":69084269,"transaction":{…},"info":{…}},"meta":{"durationMs":1396,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

未知的 txid 会直接报错（`rpc_error`，退出码 1）——这与 `tx status` 的 `not_found`（退出码 0）不同：

```json
{"schema":"wallet-cli.result.v1","success":false,"command":"tx.info","error":{"code":"rpc_error","message":"TRON getTransaction failed: Transaction not found"},"meta":{"durationMs":1033,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出

`data` 是结构化的交易详情：顶层是人类可读的摘要，`transaction` 存放原始交易对象，`info` 存放执行回执。它们的结构沿用节点的交易模型；只有[machine-interface](../../machine-interface.md)中列出的字段保证稳定，其余字段可能随节点模型而变化。

| 字段 | 类型 | 含义 |
|---|---|---|
| `txid` | string | 交易 id |
| `from` | string | 发送方地址 |
| `to` | string | 接收方地址 |
| `amount` | string | 转账金额（人类可读单位） |
| `symbol` | string | 资产符号（例如 `TRX`） |
| `status` | string | 执行结果（例如 `SUCCESS`） |
| `blockNumber` | number | 区块高度 |
| `transaction` | object | 原始 TRON 交易对象（`raw_data`、`signature`、`txID` 等） |
| `info` | object | 执行回执（`receipt` 资源用量、`contractResult`、`blockTimeStamp` 等） |

## 退出码

`0` 已找到 · `1` 执行失败——包括*未找到*（`rpc_error`） · `2` 用法错误。

## 另请参见

[`tx status`](status.md) · [`account history`](../account/history.md) · [费用与资源](../../concepts/networks.md#fees-the-tron-resource-model)

# wallet-cli account balance

显示原生 TRX 余额。

## 用法

```
wallet-cli account balance [options]
```

## 说明

从节点获取当前账户（或 `--account` 指定账户）的原生 TRX 余额。只读；无需解锁。

## 选项

仅[全局选项](../index.md#global-options-every-command)（`--account`、`--network` 等）。

## 示例

```bash
wallet-cli account balance --network tron:nile
```

```console
Label    main
Balance  1976.489 TRX
```

```bash
wallet-cli account balance --network tron:nile -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"account.balance","data":{"address":"TMSgJxtPw29AFEHMXsjGo4kWV7UwbCToHJ","balance":"1976489000","decimals":6,"symbol":"TRX"},"meta":{"durationMs":1114,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出

| 字段 | 类型 | 含义 |
|---|---|---|
| `address` | string | 被查询的 base58 地址 |
| `balance` | string | 以 SUN 计的原始余额（`"1976489000"` = 1976.489 TRX） |
| `decimals` | number | TRX 为 `6` |
| `symbol` | string | `TRX` |

## 退出码

`0` · `1` 执行失败（节点不可达、超时） · `2` 用法错误。

## 另请参见

[`account portfolio`](portfolio.md)——包含 token · [`account info`](info.md) · [单位：TRX 与 SUN](../../concepts/networks.md#fees-the-tron-resource-model)

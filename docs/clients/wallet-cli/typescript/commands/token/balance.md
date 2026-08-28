# wallet-cli token balance

查询单个 token 的余额（TRC20 或 TRC10）。

## 用法

```
wallet-cli token balance (--contract <address> | --asset-id <id>) [options]
```

## 说明

查询当前账户（或 `--account` 指定的账户）在某一个 token 上的余额。二选一，只能给一个：TRC20 token 用 `--contract`，TRC10 资产用 `--asset-id`。只读——不需要密码，也不会签名。

## 选项

| 选项 | 说明 |
|---|---|
| `--contract <string>` | TRC20 合约地址；`--contract` / `--asset-id` 二者必选其一 |
| `--asset-id <string>` | TRC10 数字资产 id；`--asset-id` / `--contract` 二者必选其一 |

此外还有[全局选项](../index.md#global-options-every-command)。

## 示例

```bash
wallet-cli token balance --contract TXYZopYRdj2D9XRtbG411XZZ3kM5VkAeBf --network tron:nile
```

```console
Label    main
Symbol   USDT
Balance  1,204.56
```

```bash
wallet-cli token balance --contract TXYZopYRdj2D9XRtbG411XZZ3kM5VkAeBf --network tron:nile -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"token.balance","data":{"address":"TMSgJxtPw29AFEHMXsjGo4kWV7UwbCToHJ","token":"TXYZopYRdj2D9XRtbG411XZZ3kM5VkAeBf","balance":"1204560000","symbol":"USDT","decimals":6},"meta":{"durationMs":15,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出

| 字段 | 类型 | 含义 |
|---|---|---|
| `address` | string | 被查询的账户（base58） |
| `token` | string | 合约地址（TRC20）或资产 id（TRC10） |
| `balance` | string | 以 token 最小单位表示的原始余额（`"1204560000"` ÷ 10^`decimals`） |
| `symbol` | string | Token 符号 |
| `decimals` | number | token 精度 |

## 退出码

`0` 成功 · `1` 执行失败（`rpc_error`、`timeout`） · `2` 用法错误（`invalid_value`——未给出选择器，或两个选择器同时给出）。

## 另请参见

[`token info`](info.md) · [`account portfolio`](../account/portfolio.md) · [`tx send`](../tx/send.md)

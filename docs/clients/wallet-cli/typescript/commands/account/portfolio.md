# wallet-cli account portfolio

显示原生 + token 余额，附带尽力而为的 USD 估值。

## 用法

```
wallet-cli account portfolio [options]
```

## 说明

把账户的原生 TRX 与地址簿中的 token 余额汇总到一个视图中，并**尽力而为**地附加来自外部价格源的 USD 价格：当价格不可用时（测试网上很常见），`priceUsd` / `valueUsd` 为 `null`，命令仍然成功。它是 `account` 组中最慢的查询——需要向价格源扇出请求。

## 选项

仅[全局选项](../index.md#global-options-every-command)。

## 示例

```bash
wallet-cli account portfolio --network tron:nile
```

```console
"main" Portfolio
| Token | Balance  | Price (USD) | Value (USD) |
| ----- | -------- | ----------- | ----------- |
| TRX   | 1969.421 | -           | -           |
Total ≈ -
```

```bash
wallet-cli account portfolio --network tron:nile -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"account.portfolio","data":{"network":"tron:nile","account":"wlt_4473p34m.0","address":"TMSgJxtPw29AFEHMXsjGo4kWV7UwbCToHJ","priceSource":"coingecko","holdings":[{"kind":"native","symbol":"TRX","decimals":6,"rawBalance":"1976489000","balance":"1976.489","priceUsd":null,"valueUsd":null}],"totalValueUsd":null},"meta":{"durationMs":11031,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出

| 字段 | 类型 | 含义 |
|---|---|---|
| `network` / `account` / `address` | string | 查询上下文 |
| `priceSource` | string | e.g. `coingecko` |
| `holdings[].kind` | string | `native` 或各类 token |
| `holdings[].symbol` / `decimals` | — | token 标识 |
| `holdings[].rawBalance` | string | 基础单位 |
| `holdings[].balance` | string | 人类可读单位 |
| `holdings[].priceUsd` / `valueUsd` | number\|null | **尽力而为的估算**；无价格时为 `null` |
| `totalValueUsd` | number\|null | 已定价持仓之和；全部无价格时为 `null` |

## 退出码

`0`（即使所有价格均为 `null`） · `1` 执行失败 · `2` 用法错误。

## 另请参见

[`account balance`](balance.md) · `token`——管理决定这里出现哪些 token 的地址簿

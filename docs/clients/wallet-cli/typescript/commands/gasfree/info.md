# wallet-cli gasfree info

显示你的 GasFree 地址、激活状态、`nonce` 与费率表。

## 用法

```
wallet-cli gasfree info [options]
```

## 说明

通过服务方的 API 只读查看：该账户的 GasFree 地址（确定性派生）、它的激活状态和当前 `nonce`，以及服务方支持的 token 及其激活费和每笔转账费（均以该 token 本身计费）。

**GasFree 地址**是资产收付所用的地址——要通过 GasFree 收取 USDT，把这个地址给付款方即可。首次向外转账时，服务方会在链上激活它并收取激活费。费率表和受支持的 token 取自服务方的实时配置，因此输出就是 API 返回的内容。

需要服务方的 API 凭据（`gasfreeApiKey` / `gasfreeApiSecret`，用 [`config`](../config.md) 设置）。

## 选项

没有命令级选项；仅[全局选项](../index.md#global-options-every-command)（`--network` 用于选择服务环境，以及 `--account`）。

## 示例

```bash
wallet-cli gasfree info --account main --network tron:nile
```

```console
Account          main (TQkXm4vN8pR2sD6fWbYc3LhJa9Ee5Zt7Uw)
GasFree address  TVjsyZ7fYF3qCcNaMxN5PMWmSgYcCyqZfw
Status           active
Nonce            4

Supported tokens (1)
  Token  Activation fee  Transfer fee
  USDT   1 USDT          0.5 USDT
```

```bash
wallet-cli gasfree info --account main --network tron:nile -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"gasfree.info","data":{"ownerAddress":"TQkXm4vN8pR2sD6fWbYc3LhJa9Ee5Zt7Uw","gasFreeAddress":"TVjsyZ7fYF3qCcNaMxN5PMWmSgYcCyqZfw","active":true,"nonce":4,"tokens":[{"symbol":"USDT","address":"TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t","decimals":6,"activateFee":"1000000","transferFee":"500000"}]},"meta":{"durationMs":380,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出

| 字段 | 类型 | 含义 |
|---|---|---|
| `ownerAddress` | string | 该账户自身的 TRON 地址 |
| `gasFreeAddress` | string | 派生出的 GasFree 地址（收付款都用它） |
| `active` | boolean | GasFree 地址是否已在链上激活 |
| `nonce` | number | 该地址当前的 `nonce` |
| `tokens[]` | array | 受支持的 token：`{symbol, address, decimals, activateFee, transferFee}`——费用以该 token 的最小单位计 |

## 退出码

`0` 成功 · `1` 执行失败（`gasfree_credentials_missing`；`gasfree_integrity`——服务方在 token 列表与地址响应中给出的费用元数据自相矛盾；`provider_error`——服务出错 / 触发限流；`unsupported_network`） · `2` 用法错误（`invalid_value`）。

## 另请参见

[`gasfree transfer`](transfer.md) · [`gasfree trace`](trace.md) · [`config`](../config.md)

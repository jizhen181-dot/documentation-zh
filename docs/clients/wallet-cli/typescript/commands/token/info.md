# wallet-cli token info

查看 token 元数据（name / symbol / decimals / totalSupply）。

## 用法

```
wallet-cli token info (--contract <address> | --asset-id <id>) [options]
```

## 说明

直接从链上读取某个 token 的元数据——纯 RPC 查询，完全不涉及你的账户。二选一，只能给一个：TRC20 用 `--contract`，TRC10 用 `--asset-id`。

## 选项

| 选项 | 说明 |
|---|---|
| `--contract <string>` | TRC20 合约地址；`--contract` / `--asset-id` 二者必选其一 |
| `--asset-id <string>` | TRC10 数字资产 id；`--asset-id` / `--contract` 二者必选其一 |

此外还有[全局选项](../index.md#global-options-every-command)。

## 示例

```bash
wallet-cli token info --contract TXYZopYRdj2D9XRtbG411XZZ3kM5VkAeBf --network tron:nile
```

```console
Name      Tether USD
Symbol    USDT
Decimals  6
```

```bash
wallet-cli token info --contract TXYZopYRdj2D9XRtbG411XZZ3kM5VkAeBf --network tron:nile -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"token.info","data":{"contract":"TXYZopYRdj2D9XRtbG411XZZ3kM5VkAeBf","name":"Tether USD","symbol":"USDT","decimals":6,"totalSupply":"17600000000030000000"},"meta":{"durationMs":15,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出

| 字段 | 类型 | 含义 |
|---|---|---|
| `contract` | string | TRC20 合约地址（TRC10 时为 `assetId`） |
| `name` | string | token 名称 |
| `symbol` | string | Token 符号 |
| `decimals` | number | token 精度 |
| `totalSupply` | string | 总供应量，以最小单位表示的原始整数 |

## 退出码

`0` 成功 · `1` 执行失败（`token_metadata_unavailable`——该合约没有暴露 ERC20 风格的元数据；`rpc_error`） · `2` 用法错误（`invalid_value`）。

## 另请参见

[`token add`](add.md) · [`token balance`](balance.md) · [`contract info`](../contract/info.md)

# wallet-cli contract call

只读合约调用（triggerConstantContract）。

## 用法

```
wallet-cli contract call --contract <address> --method <sig> [--params <json>] [options]
```

## 说明

以 `constant`（只读）方式调用合约方法：不签名、不广播、不消耗任何费用。调用方地址取自当前账户（或 `--account`）——有些方法（例如用到 `msg.sender` 的 `balanceOf` 这类 `view`）会关心是谁在调用。

参数是一个 JSON 数组，元素为与方法签名逐一对应的 `{type, value}` 对象。

## 选项

| 选项 | 说明 |
|---|---|
| `--contract <string>` | **必填。** 合约地址 |
| `--method <string>` | **必填。** 函数签名，例如 `balanceOf(address)` |
| `--params <string>` | ABI 参数的 JSON 数组，元素形如 `{type,value}`；省略表示不传参数 |

此外还有[全局选项](../index.md#global-options-every-command)。

## 示例

```bash
wallet-cli contract call --contract TXYZopYRdj2D9XRtbG411XZZ3kM5VkAeBf --method "balanceOf(address)" --params '[{"type":"address","value":"TMSgJxtPw29AFEHMXsjGo4kWV7UwbCToHJ"}]' --network tron:nile
```

```console
Method  balanceOf
Result  0000000000000000000000000000000000000000000000000000000000000000 (raw)
```

```bash
wallet-cli contract call --contract TXYZopYRdj2D9XRtbG411XZZ3kM5VkAeBf --method "balanceOf(address)" --params '[{"type":"address","value":"TMSgJxtPw29AFEHMXsjGo4kWV7UwbCToHJ"}]' --network tron:nile -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"contract.call","data":{"contract":"TXYZopYRdj2D9XRtbG411XZZ3kM5VkAeBf","method":"balanceOf(address)","result":["0000000000000000000000000000000000000000000000000000000000000000"]},"meta":{"durationMs":15,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出

| 字段 | 类型 | 含义 |
|---|---|---|
| `contract` | string | 被调用的合约地址 |
| `method` | string | 实际调用的方法签名 |
| `result` | string[] | ABI 编码的原始返回字（hex）；需按方法的返回类型自行解码 |

## 退出码

`0` 成功 · `1` 执行失败（`rpc_error`、合约 `revert`） · `2` 用法错误（`invalid_value`——方法签名或参数 JSON 有误）。

## 另请参见

[`contract send`](send.md) · [`contract info`](info.md) · [`token balance`](../token/balance.md)

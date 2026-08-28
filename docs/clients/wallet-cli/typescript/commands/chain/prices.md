# wallet-cli chain prices

显示能量/带宽单价与备注费用。

## 用法

```
wallet-cli chain prices [options]
```

## 说明

显示当前的能量单价、带宽单价和备注费用，可用于估算操作是否需要燃烧 TRX 以及预计费用。该命令只读，
不需要账户或密码。

节点返回的是一条价格*历史*时间线；文本输出只显示当前值（最后一段），`-o json` 则保留完整的 `history`。

**单位**：单价一律以 **SUN** 表示（1 TRX = 1,000,000 SUN）——这是业界惯例，`--fee-limit` 之类的选项也以 SUN 计价；备注费用属于普通金额，因此按 TRX 显示。json 输出统一使用 SUN。

## 选项

没有命令级选项；仅[全局选项](../index.md#global-options-every-command)（`--network`）。

## 示例

```bash
wallet-cli chain prices --network tron:nile
```

```console
Energy price      210 SUN / unit    (current)
Bandwidth price   1,000 SUN / unit  (current)
Memo fee          1 TRX
```

```bash
wallet-cli chain prices --network tron:nile -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"chain.prices","data":{"energy":{"currentSunPerUnit":210,"history":[{"since":1542607200000,"price":100},{"since":1670515200000,"price":210}]},"bandwidth":{"currentSunPerUnit":1000,"history":[{"since":1542607200000,"price":10},{"since":1614456000000,"price":1000}]},"memoFeeSun":"1000000"},"meta":{"durationMs":21,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出

| 字段 | 类型 | 含义 |
|---|---|---|
| `energy.currentSunPerUnit` | number | 当前能量单价，每单位多少 SUN |
| `energy.history[]` | array | `{since (epoch 毫秒), price}` 价格时间线 |
| `bandwidth.currentSunPerUnit` | number | 当前带宽单价，每单位多少 SUN |
| `bandwidth.history[]` | array | `{since, price}` 价格时间线 |
| `memoFeeSun` | string | 备注费用，单位 SUN |

## 退出码

`0` 成功 · `1` 执行失败（`rpc_error`、`timeout`） · `2` 用法错误。

## 另请参见

[`chain params`](params.md) · [`chain node`](node.md) · [能量与带宽](../../concepts/energy-bandwidth.md) · [`tx send`](../tx/send.md)

# wallet-cli tx status

查看交易的确认状态。

## 用法

```
wallet-cli tx status --txid <id> [options]
```

## 说明

通过**四种状态**报告交易进度。发送后，脚本和智能体可以轮询该命令，判断交易是否已经入块及执行成功。
这四个状态值属于稳定接口，在兼容版本中不会改名或删除，因此程序可以直接据此分支处理（参见
[machine-interface](../../machine-interface.md) 中的 `wallet-cli.result.v1` 输出规范）。

| `data.state` | 含义 | 能否停止轮询？ |
|---|---|---|
| `confirmed` | 已入块并取得执行回执；带有 `blockNumber` | 可以——视为成功 |
| `failed` | 已入块，但链上执行失败 | 可以——视为失败 |
| `pending` | 节点已经看到，但尚未入块 | 不能——继续轮询 |
| `not_found` | 被查询的节点不认识它（网络选错了？还没传播过来？） | 不能——超过截止时间后转为**状态未知**，而不是失败 |

!!! warning "`confirmed` 表示已入块，不表示已固化"

    本命令读取的是 FullNode 的**未确认视图**。`confirmed` 说明交易已被打包进区块、回执（手续费、
    能量消耗、执行结果）已经确定，但**不代表已固化、不可回滚**。需要最终性级别的确认时，请另行
    查询 SolidityNode，或对比 [`chain node`](../chain/node.md) 的 `solidBlock.number`。

!!! danger "`not_found` 超时不等于失败"

    `not_found` 只说明**当前被查询的这个节点**不知道这笔交易，它可能仍在其他节点的内存池中。
    不能据此判定失败，更不能直接重发——那会造成重复付款。超过你设定的截止时间后应当作**状态
    未知**，先通过 SolidityNode 或区块浏览器对账，再决定下一步。

## 选项

| 选项 | 说明 |
|---|---|
| `--txid <string>` | **必填。** TRON 交易 id/哈希 |

此外还有[全局选项](../index.md#global-options-every-command)。

## 示例

```bash
wallet-cli tx status --txid 7d9b6a08505537f7fd51ed4fb4223ce89098403d26e8d3fe07bdb3d625a46364 --network tron:nile
```

```console
TxID    7d9b6a08505537f7fd51ed4fb4223ce89098403d26e8d3fe07bdb3d625a46364
Status  confirmed ✅
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"tx.status","data":{"txid":"7d9b6a08505537f7fd51ed4fb4223ce89098403d26e8d3fe07bdb3d625a46364","state":"confirmed","confirmed":true,"failed":false,"blockNumber":68822193},"meta":{"durationMs":1006,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

未知的 txid 属于**成功**，返回 `state: "not_found"`（退出码 0）——查询本身是成功的，只是答案为“不在那里”：

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"tx.status","data":{"txid":"0000…0000","state":"not_found","confirmed":false,"failed":false},"meta":{"durationMs":1022,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出

| 字段 | 类型 | 含义 |
|---|---|---|
| `txid` | string | 回显所查询的 id |
| `state` | string | `confirmed` / `failed` / `pending` / `not_found` |
| `confirmed` / `failed` | boolean | 与 `state` 对应的布尔值，便于直接分支判断 |
| `blockNumber` | number | 状态为已确认时才有 |

## 退出码

`0` 查询已作答（包括 `not_found`） · `1` 执行失败（节点不可达、超时） · `2` 用法错误。

## 另请参见

[`tx info`](info.md)——完整详情和交易回执 · [`tx send`](send.md) · [脚本安全](../../machine-interface.md#script-safety-never-mistake-submitted-for-confirmed)

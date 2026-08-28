# wallet-cli chain node

所连节点的状态（版本 / 同步 / 对等节点）。

## 用法

```
wallet-cli chain node [options]
```

## 说明

显示所连接节点的版本、最新块与已固化块高度、同步状态以及对等节点数量。排查交易问题时，可以先用
该命令确认节点是否已经同步。

版本、块高度和对等节点数来自节点的 `getnodeinfo`。同步状态通过比较最新块时间戳与本地时间计算：
最新块距当前时间不超过 3 个出块间隔（TRON 每 3 秒出一个块，即 9 秒）时，显示为 `in sync`。
公共网关（如 TronGrid）可能不提供对等节点或机器信息，此时对应行显示 `—`，JSON 中为 `null`。

## 选项

没有命令级选项；仅[全局选项](../index.md#global-options-every-command)（`--network`）。

## 示例

```bash
wallet-cli chain node --network tron:nile
```

```console
Endpoint     https://nile.trongrid.io
Version      java-tron 4.7.7
Head block   69,093,315  2026-07-11 15:29:21 (~2s ago — in sync)
Solid block  69,093,296  (19 blocks behind head)
Peers        30 connected / 27 active
```

```bash
wallet-cli chain node --network tron:nile -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"chain.node","data":{"endpoint":"https://nile.trongrid.io","version":"java-tron 4.7.7","p2pVersion":"11111","headBlock":{"number":69093315,"timestamp":1783783761000},"solidBlock":{"number":69093296},"lagBlocks":19,"inSync":true,"peers":{"connected":30,"active":27}},"meta":{"durationMs":24,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出

| 字段 | 类型 | 含义 |
|---|---|---|
| `endpoint` | string | 所查询的节点 URL |
| `version` | string | 节点软件版本 |
| `p2pVersion` | string | P2P 协议版本 |
| `headBlock` | object | 最新块 `{number, timestamp}` |
| `solidBlock` | object | 已固化块 `{number}` |
| `lagBlocks` | number | 最新块与已固化块的高度差 |
| `inSync` | boolean | 最新块是否新鲜（块龄 ≤ 9 秒） |
| `peers` | object \| null | `{connected, active}`；端点隐藏该信息时为 `null` |

## 退出码

`0` 成功 · `1` 执行失败（`rpc_error`、`timeout`） · `2` 用法错误。

## 另请参见

[`chain params`](params.md) · [`networks`](../networks.md) · [故障排查](../../troubleshooting.md)

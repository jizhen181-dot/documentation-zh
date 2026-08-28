# wallet-cli block

获取区块（省略参数时取最新区块）。

## 用法

```
wallet-cli block [<number>] [options]
```

## 参数

- `number`——要获取的区块高度；省略则取最新区块

## 选项

仅[全局选项](index.md)。

## 注意事项

需要 `--network`（或 config.defaultNetwork）。

## 示例

```bash
wallet-cli block --network tron:nile
```

```console
Number        #69,093,315
Time          2026-07-11 15:29:21 UTC
Transactions  212
```

```bash
wallet-cli block 69093315 --network tron:nile -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"block","data":{"block":{"blockID":"0000000041e6a3c3…","block_header":{"raw_data":{"number":69093315,"txTrieRoot":"…","witness_address":"41…","parentHash":"…","version":31,"timestamp":1783783761000},"witness_signature":"…"},"transactions":[{…}]}},"meta":{"durationMs":126,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出

`data.block` 是节点返回的原始 TRON 区块，未经任何修改。其确切结构与节点的区块结构一致；关键字段见下表（上面示例中较长的哈希和完整交易列表以 `…` 省略）。

| 字段 | 类型 | 含义 |
|---|---|---|
| `block.blockID` | string | 区块哈希 |
| `block.block_header.raw_data.number` | number | 区块高度 |
| `block.block_header.raw_data.timestamp` | number | 出块时间（epoch 以来的毫秒数，UTC） |
| `block.block_header.raw_data.witness_address` | string | 出块的 SR，hex 格式（`41…`） |
| `block.transactions` | array | 区块内的交易（为空时省略该字段） |

## 退出码

`0` 成功 · `1` 执行失败 · `2` 用法错误。参见 [machine-interface](../machine-interface.md)。

## 另请参见

[网络](../concepts/networks.md)

# wallet-cli networks

列出已知网络。

## 用法

```
wallet-cli networks [options]
```

## 选项

仅[全局选项](index.md)。

## 示例

```bash
wallet-cli networks
```

```console
| Network      | Family | Chain   | Fee model     |
| ------------ | ------ | ------- | ------------- |
| tron:mainnet | tron   | mainnet | tron-resource |
| tron:nile    | tron   | nile    | tron-resource |
| tron:shasta  | tron   | shasta  | tron-resource |
```

```bash
wallet-cli networks -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"networks","data":[{"id":"tron:mainnet","family":"tron","chainId":"mainnet","feeModel":"tron-resource"},{"id":"tron:nile","family":"tron","chainId":"nile","feeModel":"tron-resource"},{"id":"tron:shasta","family":"tron","chainId":"shasta","feeModel":"tron-resource"}],"meta":{"durationMs":2,"warnings":[]}}
```

## 输出

`data` 是一个数组，每个已知网络对应一项。本地命令——没有 `chain` 块。

| 字段 | 类型 | 含义 |
|---|---|---|
| `id` | string | 规范网络 id（`family:chain`） |
| `family` | string | 链系，例如 `tron` |
| `chainId` | string | 该链系内的链标识，例如 `mainnet` |
| `feeModel` | string | 手续费模型，例如 `tron-resource` |

## 退出码

`0` 成功 · `1` 执行失败 · `2` 用法错误。参见 [machine-interface](../machine-interface.md)。

## 另请参见

[网络概念](../concepts/networks.md) · [`config`](config.md)

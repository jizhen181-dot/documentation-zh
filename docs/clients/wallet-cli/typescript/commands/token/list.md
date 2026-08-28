# wallet-cli token list

列出 token 地址簿（official + user）。

## 用法

```
wallet-cli token list [options]
```

## 说明

列出当前账户（或 `--account` 指定的账户）在所选网络上可见的全部 token：内置的 **official** 层，加上你自己添加的 **user** 条目。`source` 列用于区分两者。`tx send --token <symbol>` 解析符号时用的就是这张表。只读，只涉及本地数据和元数据——不需要密码。

## 选项

没有命令级选项；仅[全局选项](../index.md#global-options-every-command)（`--network` / `--account` 决定地址簿的作用范围）。

## 示例

```bash
wallet-cli token list --network tron:nile
```

```console
| Symbol | Name       | Source | Contract / ID                      |
| ------ | ---------- | ------ | ---------------------------------- |
| USDT   | Tether USD | user   | TXYZopYRdj2D9XRtbG411XZZ3kM5VkAeBf |
```

> `official` 层只在**主网**上内置（例如 USDT、USDC）；测试网没有 official 层，因此列出的每一条都是你用 `token add` 添加的 `user` 条目。

```bash
wallet-cli token list --network tron:nile -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"token.list","data":{"network":"tron:nile","account":"wlt_b2.0","tokens":[{"kind":"trc20","id":"TXYZopYRdj2D9XRtbG411XZZ3kM5VkAeBf","symbol":"USDT","decimals":6,"name":"Tether USD","source":"user"}]},"meta":{"durationMs":13,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出

`data` 包含 `network`、`account` 和 `tokens[]`——每个 token 一条：

| 字段 | 类型 | 含义 |
|---|---|---|
| `kind` | string | `trc20` / `trc10` |
| `id` | string | 合约地址或资产 id |
| `symbol` | string | Token 符号（`tx send --token` 用的就是它） |
| `decimals` | number | token 精度 |
| `name` | string | token 名称 |
| `source` | string | `official`（内置） / `user`（你自己添加的） |

## 退出码

`0` 成功 · `1` 执行失败 · `2` 用法错误。

## 另请参见

[`token add`](add.md) · [`token remove`](remove.md) · [发送 token](../../guide/send-tokens.md)

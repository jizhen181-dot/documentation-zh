# wallet-cli tx sign

为交易 hex 添加你的签名（多签联署）。

## 用法

```
wallet-cli tx sign (--hex <hex> | --file <path> | --transaction <json>) [--offline] [--out <path>] [options]
```

## 说明

为一段（未签名或部分签名的）交易 hex 添加当前账户的签名，输出新的 hex，并报告已累计的签名权重距离该权限组的阈值还差多少。

两种输入模式：用 `--hex` / `--file` 时，它只追加一个签名，同时保留此前已有的签名；用 `--transaction` 时，它接收未签名的交易 JSON，保留原来直接单签的流程。

这是链上的联署路径：发起方用 `tx send --sign-only`（或任何以 `--sign-only` 模式运行的广播类命令）生成一段部分签名的 hex，各位联署人依次运行 `tx sign`——把 hex 一个人一个人地传下去——一旦权重达到阈值，任何人都可以用 [`tx broadcast --hex`](broadcast.md) 广播最终的 hex。所有签名都必须在交易过期之前收集完毕（默认约 60 秒，通过 `--expiration` 最长可延至 24 小时）。

签名意味着用你的密钥为这笔交易背书：该命令不会展示预览，也不会要求确认，而是从 `--password-stdin` 读取 master password 后直接签名——若想在不签名的前提下查看交易，请用 [`tx approvals`](approvals.md)。它**不会**广播，也**没有 `--permission-id`**（权限组已经固定在交易体里，会显示在 `Permission` 行上）。仅观察账户会以 `watch_only_no_signer` 失败。

### 签名前会校验什么

使用 `--hex` / `--file` 时，命令会连接节点。只有当前账户属于交易指定权限组（否则返回
`not_authorized`），且尚未签署该交易（否则返回 `already_signed`）时，CLI 才会签名。这两项检查均在
解密密钥前完成，避免使用不符合条件的账户签名。

`--offline` 会跳过这两项检查，且完全不访问节点——适用于没有网络的签名机。此时与签名资格有关的错误只有在广播时才会暴露出来，所以请事先确认签名账户确实在权限组内。

所有模式都会校验交易数据完整性，离线模式也不例外。TRON 交易包含三种相关表示：`raw_data` 是可读的
交易内容，`raw_data_hex` 是节点实际执行的编码，`txID` 则是签名覆盖的哈希。格式本身不能保证三者
一致，因此攻击者可能让 `raw_data` 显示 1 TRX，却让实际签名内容对应 1000 TRX。为防止此类篡改，
`tx sign` 要求 `txID` 等于 `raw_data_hex` 的 SHA-256；对于可解码的合约类型，还要求 `raw_data`
重新编码后与 `raw_data_hex` 完全一致，否则返回 `tx_integrity` 并拒绝签名。

有四种合约类型无法被内置的解码器重新编码——`UnfreezeAssetContract`、`ShieldedTransferContract`、`MarketSellAssetContract`、`MarketCancelOrderContract`。`--hex` / `--file` 输入中若包含其中之一，会以 `invalid_transaction` 被拒绝；这类交易请改用 `--transaction` JSON 方式签名。

## 选项

| 选项 | 说明 |
|---|---|
| `--hex <hex>` | **必填**（三选一）。完整的 `protocol.Transaction` hex |
| `--file <path>` | **必填**（三选一）。包含交易 hex 的文件（hex 较长时建议用它） |
| `--transaction <json>` | **必填**（三选一）。未签名的 TRON 交易 JSON；为兼容直接单签流程而保留 |
| `--offline` | 在本地签名而不访问节点；跳过签名者权限检查和批准权重检查。仅可与 `--hex` / `--file` 同用 |
| `--out <path>` | 把结果 hex 写入文件（权限 0644，原子写入）而不是输出到 stdout |

此外还有[全局选项](../index.md#global-options-every-command)和 `--password-stdin`。

交易通过 argv 传入，而不是 stdin：它不是机密信息，这样也能把 fd 0 留给 `--password-stdin`。

## 示例

示例中的 `$PW` 是你的 master password，通过 `--password-stdin` 从 stdin 传入。

发起方先用 `tx send --sign-only` 生成一份部分签名的 `tx.hex`：

```bash
echo "$PW" | wallet-cli tx send --to TBy6mQ7Y3nJ8sD2fWpXk4LhVc9Ra1Zt5Ub --amount 1000 --sign-only --permission-id 2 --expiration 86400000 --network tron:nile --password-stdin > tx.hex
```

第二位签名者追加自己的签名——没有预览，也没有确认；回执中包含交易内容和进度两个区块：

```bash
echo "$PW" | wallet-cli tx sign --file tx.hex --account cosigner --out tx.signed.hex --network tron:nile --password-stdin
```

```console
✅ Signature added
  Signer   TXe4Kd8nP2rF9gH5jL3mV6cW1bN7yS0aQz  (weight 1)
  Hex      written to tx.signed.hex

Transaction
  TxID        9c1...
  Type        Transfer TRX — 1,000 TRX
  From        TQkXm4vN8pR2sD6fWbYc3LhJa9Ee5Zt7Uw
  To          TBy6mQ7Y3nJ8sD2fWpXk4LhVc9Ra1Zt5Ub
  Permission  active "finance" (id 2)  threshold 2
  Expires     2026-07-14 15:32 (~23h)

Progress  2 / 2 — threshold reached
| Approved signer                    | Weight |
| ---------------------------------- | ------ |
| TQkXm4vN8pR2sD6fWbYc3LhJa9Ee5Zt7Uw  |      1 |
| TXe4Kd8nP2rF9gH5jL3mV6cW1bN7yS0aQz  |      1 |

! Broadcast it: wallet-cli tx broadcast --file tx.signed.hex
```

使用 `--offline` 时拿不到权限组名称、阈值和各签名者的权重，因此回执会退化为仅包含本地可推导的字段，并会明确说明这一点。`Signatures` 是签名的**数量**，不是累计权重：

```bash
echo "$PW" | wallet-cli tx sign --file tx.hex --account cosigner --offline --network tron:nile --password-stdin
```

```console
✅ Signature added
  Signer  TXe4Kd8nP2rF9gH5jL3mV6cW1bN7yS0aQz
  Hex     0a02...9f31

Transaction (local inspection)
  TxID        9c1...
  Type        Transfer TRX — 1,000 TRX
  From        TQkXm4vN8pR2sD6fWbYc3LhJa9Ee5Zt7Uw
  To          TBy6mQ7Y3nJ8sD2fWpXk4LhVc9Ra1Zt5Ub
  Permission  active (id 2)
  Signatures  1
  Expires     2026-07-14 15:32 (~23h)

! Approval state was not checked online. Inspect it with: wallet-cli tx approvals --hex <hex-above>
```

```bash
echo "$PW" | wallet-cli tx sign --file tx.hex --account cosigner --out tx.signed.hex --network tron:nile --password-stdin -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"tx.sign","data":{"kind":"tx-sign","signer":"TXe4Kd8nP2rF9gH5jL3mV6cW1bN7yS0aQz","hex":"0a02...9f31","checked":true,"transaction":{"txId":"9c1...","contractType":"TransferContract","operation":"Transfer TRX","from":"TQkXm4vN8pR2sD6fWbYc3LhJa9Ee5Zt7Uw","to":"TBy6mQ7Y3nJ8sD2fWpXk4LhVc9Ra1Zt5Ub","rawAmount":"1000000000","permissionId":2,"expiration":1784388720000,"expired":false,"signatures":2},"signerWeight":1,"approval":{"txId":"9c1...","contractType":"TransferContract","operation":"Transfer TRX","from":"TQkXm4vN8pR2sD6fWbYc3LhJa9Ee5Zt7Uw","to":"TBy6mQ7Y3nJ8sD2fWpXk4LhVc9Ra1Zt5Ub","rawAmount":"1000000000","permission":{"id":2,"name":"finance","threshold":2},"currentWeight":2,"missingWeight":0,"thresholdReached":true,"approved":[{"address":"TQkXm4vN8pR2sD6fWbYc3LhJa9Ee5Zt7Uw","weight":1},{"address":"TXe4Kd8nP2rF9gH5jL3mV6cW1bN7yS0aQz","weight":1}],"expiration":1784388720000,"expired":false,"signatures":2}},"meta":{"durationMs":310,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出

两种输入模式返回的结构不同。

`--hex` / `--file`（对交易 hex 签名）：

| 字段 | 类型 | 含义 |
|---|---|---|
| `kind` | string | `"tx-sign"` |
| `signer` | string | 刚刚完成签名的地址 |
| `hex` | string | 追加了新签名之后的交易 hex |
| `checked` | boolean | 是否执行了联网的权限/批准校验——`--offline` 下为 `false` |
| `transaction` | object | 本地解码出的摘要：`txId`、`contractType`、`operation`、`from`、`to`、`rawAmount`、`permissionId`（只是一个标量——不含权限组名称和阈值）、`expiration`、`expired`、`signatures`（数量） |
| `signerWeight` | number | 该签名者在权限组中的权重。仅当 `checked` 为 `true` 时才有 |
| `approval` | object | 联网获取的权威批准状态，结构与 [`tx approvals`](approvals.md) 的 `data` 相同。仅当 `checked` 为 `true` 时才有 |

`transaction` 在两种模式下都存在且结构一致，因此使用方可以无条件读取；而在访问 `approval` 之前，请先检查 `checked`。

`--transaction`（直接对 JSON 签名）返回的结构与 `tx send --sign-only` 输出的完全一致，因此使用方无需分支处理：

| 字段 | 类型 | 含义 |
|---|---|---|
| `kind` | string | `"sign"` |
| `mode` | string | `"sign-only"` |
| `address` | string | 产生该签名的地址 |
| `txId` | string | 交易 id |
| `signed` | object | 已签名的交易——正是 [`tx broadcast`](broadcast.md) 所接受的形式 |

`--transaction` 模式不会报告 `fee`：交易不是在这里构建的，也就没有做过任何估算。

## 退出码

`0` 成功 · `1` 执行失败（`tx_integrity`——交易数据的三种表示不一致、`invalid_transaction`、`tx_expired`、`not_authorized`——该账户不在权限组的密钥列表中、`already_signed`、`watch_only_no_signer`、`auth_failed`、`signing_rejected`、`rpc_error`） · `2` 用法错误（`invalid_value`、`missing_option`）。

## 另请参见

[`tx approvals`](approvals.md) · [`tx broadcast`](broadcast.md) · [`tx multisig`](multisig.md) · [`permission show`](../permission/show.md)

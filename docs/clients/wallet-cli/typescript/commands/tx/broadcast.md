# wallet-cli tx broadcast

广播一笔预先签名的交易。

## 用法

```
wallet-cli tx broadcast (--hex <hex> | --file <path> | --transaction <json> | --tx-stdin)
                        [--dry-run] [--network <id>] [options]
```

## 说明

提交一笔在别处签好名的交易。无需解锁钱包，因为交易已经签好名了。签名后的输入可以是 **hex**——用 `--hex` 内联传入，或用 `--file` 从文件读取（即 `--sign-only` 和 `tx sign` 输出的格式）——也可以是 **JSON**——用 `--transaction` 内联传入，或用 `--tx-stdin` 从 stdin 读取。四者必须且只能选其一；hex 较长时建议用 `--file`。

预先签名的交易本身不携带网络信息，因此需要用 `--network` 指明广播到哪个网络（省略时回退到配置中的默认网络）。

### 提交前的校验

广播不是盲目执行的。无论交易以哪种形式传入，都会先被解码并校验，注定无法成功的交易会在本地被拒绝，而不会发送出去：

- **已过期** → `tx_expired`
- **签名权重不足** → `not_authorized`，并指出还差多少权重

正是这项校验让它可以安全地作为多签流程的最后一步：尚未达到权限阈值的交易永远不会到达节点。

> 携带一个以上签名的交易在广播时会额外产生链上的 **1 TRX 多签手续费**；无论试运行还是真实广播，都会以 `multiSignFeeSun` 字段报告出来。

## 选项

| 选项 | 说明 |
|---|---|
| `--hex <hex>` | 内联传入已签名的交易 hex |
| `--file <path>` | 包含已签名交易 hex 的文件（大小上限略大于 1 MiB） |
| `--transaction <string>` | 内联传入已签名的 TRON 交易 JSON |
| `--tx-stdin` | 从 stdin（fd 0）读取已签名的交易 JSON |
| `--dry-run` | 校验签名、阈值、过期时间以及动态的多签手续费，但**不广播**；不能与 `--wait` 同用 |
| `--wait` / `--wait-timeout <ms>` | 广播后轮询直到已确认/失败（上限默认 60000） |

此外还有[全局选项](../index.md#global-options-every-command)。

## 示例

从文件广播一段已签名的 hex：

```bash
wallet-cli tx broadcast --file tx.signed.hex --network tron:nile
```

```console
⏳ Broadcast
  TxID    72a315303323125708f426c77b94c5215afd8964ed27d67e49c29b56e29078f5
  Status  pending — not yet on-chain
! Track it: wallet-cli tx info --network tron:nile --txid 72a315303323125708f426c77b94c5215afd8964ed27d67e49c29b56e29078f5
```

或者内联传入 hex，并查看 JSON 交易回执：

```bash
wallet-cli tx broadcast --hex 0a02...9f31 --network tron:nile -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"tx.broadcast","data":{"kind":"broadcast","stage":"submitted","txId":"72a315303323125708f426c77b94c5215afd8964ed27d67e49c29b56e29078f5"},"meta":{"durationMs":926,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出

`data` 随阶段而变：

| 阶段 | 字段 |
|---|---|
| 默认（提交） | `kind`、`stage: "submitted"`、`txId` |
| `--wait`（已确认/失败） | 同上，另加 `confirmed`、`blockNumber`、`failed` 以及结果字段 |

与 `tx send` 一样，默认的返回点是**提交**——请通过 `--wait` 或 [`tx status`](status.md) 来确认。

## 退出码

`0` 已提交 · `1` 执行失败（节点拒绝了该交易、超时） · `2` 用法错误（输入源传了多个，或一个都没传）。

## 另请参见

[`tx send --sign-only`](send.md) · [`tx status`](status.md) · [脚本编写指南](../../guide/scripting.md#sign-here-broadcast-there)

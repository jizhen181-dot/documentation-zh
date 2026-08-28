# wallet-cli contract deploy

部署智能合约。

## 用法

```
wallet-cli contract deploy --abi <json> --bytecode <hex> --fee-limit <sun>
                           [--params <json>]
                           [--dry-run | (--sign-only | --build-only) [--expiration <ms>] | --wait [--wait-timeout <ms>]] [--permission-id <n>] [options]
```

## 说明

以当前账户（或 `--account`）部署编译好的合约 `bytecode`，并返回新合约的地址。这里 `--fee-limit` 是**必填**的（部署极耗能量，不存在安全的默认值）。构造函数参数只能通过 `--params` 传入——参数类型取自你所传 `--abi` 中的 `constructor` 条目。

在开始构建之前会先检查两处格式，二者都会以退出码 `2` 的 `invalid_value` 报出：

- **这里的 `--params` 接受的是原始位置参数值**，例如 `[100, "T..."]`。[`contract call`](call.md) 和 [`contract send`](send.md) 所用的 `{"type","value"}` 形式在这里会被拒绝——deploy 改为从 ABI 的 `constructor` 中读取类型。
- **ABI 的 `constructor` 条目必须带有字符串类型的 `stateMutability`**（`"nonpayable"` 或 `"payable"`）。`solc` 会生成它；手工裁剪过的 ABI，或由 0.5 之前版本 `solc` 产出的 ABI，可能没有这一项。

执行模型与其他广播类命令一致：`--dry-run` 预览，`--sign-only` 输出已签名的交易供 [`tx broadcast`](../tx/broadcast.md) 使用，默认在提交时返回，`--wait` 则阻塞直到已确认/失败。

需要一个账户。只有会签名的模式才需要 master password（通过 `--password-stdin`）——`--dry-run` 和 `--build-only` 不会解锁钱包，无需密码即可运行。在签名模式下，仅观察账户会以 `watch_only_no_signer` 失败。

## 选项

| 选项 | 说明 |
|---|---|
| `--abi <string>` | **必填。** 合约 ABI，以 JSON 数组字符串形式给出 |
| `--bytecode <string>` | **必填。** 编译产出的 `bytecode`，hex 格式（带不带 `0x` 前缀均可） |
| `--fee-limit <number>` | **必填。** 允许燃烧的最高能量费用，单位 SUN |
| `--params <string>` | 构造函数参数，以原始位置参数值的 JSON 数组给出，例如 `[100, "T..."]`；类型取自 ABI 的 `constructor`。省略表示不传构造函数参数 |
| `--dry-run` | 只估算，不签名/不广播；与 `--sign-only` / `--build-only` 互斥 |
| `--sign-only` | 只签名不广播，输出已签名的 hex；与 `--dry-run` / `--build-only` 互斥；配合 `--expiration` 使用 |
| `--build-only` | 只构建，输出**未签名**的 hex；与 `--dry-run` / `--sign-only` 互斥；配合 `--expiration` 使用 |
| `--expiration <ms>` | 交易过期时间（毫秒），最大 `86400000`（24 小时）；仅可与 `--sign-only` 或 `--build-only` 同用；省略时使用节点默认值（约 60 秒） |
| `--permission-id <n>` | 用于签名的权限组（0=owner，1=witness，2-9=active）；默认 `0` |
| `--wait` / `--wait-timeout <ms>` | 广播后轮询直到已确认/失败（上限默认取配置 `waitTimeoutMs`，内置 60000） |
| `--password-stdin` | 从 stdin 读取 master password |

此外还有[全局选项](../index.md#global-options-every-command)。

## 示例

示例中的 `$PW` 是你的 master password（来自环境变量、密码管理器等），通过 `--password-stdin` 从 stdin 传入。

```bash
echo "$PW" | wallet-cli contract deploy --abi "$(cat MyToken.abi.json)" --bytecode "$(cat MyToken.bin)" --fee-limit 1000000000 --network tron:nile --password-stdin
```

```console
⏳ Contract deployed
  Address  TXg3jWThoa5AxuwRA4aRyFAhmRN9hjhQFU
  TxID     b7c...
  Status   pending — not yet on-chain
! Track it: wallet-cli tx info --network tron:nile --txid b7c...
```

```bash
echo "$PW" | wallet-cli contract deploy --abi "$(cat MyToken.abi.json)" --bytecode "$(cat MyToken.bin)" --fee-limit 1000000000 --network tron:nile --password-stdin -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"contract.deploy","data":{"kind":"contract-deploy","contractAddress":"TXg3jWThoa5AxuwRA4aRyFAhmRN9hjhQFU","stage":"submitted","txId":"b7c..."},"meta":{"durationMs":15,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出

`data` 随阶段而变：

| 阶段 | 字段 |
|---|---|
| 默认（提交） | `kind: "contract-deploy"`、`contractAddress`（确定性推导出的新地址）、`stage: "submitted"`、`txId` |
| `--wait`（已确认） | 同上，另加 `confirmed`、`blockNumber`、`feeSun`、`failed` |

## 退出码

`0` 已提交（早退模式下为已构建/已签名） · `1` 执行失败（`watch_only_no_signer`、`auth_failed`、`rpc_error`、`timeout`） · `2` 用法错误（`missing_option`——没给 `--fee-limit`；`invalid_value`——ABI 或 `bytecode` 有误、`--params` 写成了 `{"type","value"}` 形式，或 ABI 的 `constructor` 缺少字符串类型的 `stateMutability`）。

## 另请参见

[`contract info`](info.md) · [`contract send`](send.md) · [`tx status`](../tx/status.md)

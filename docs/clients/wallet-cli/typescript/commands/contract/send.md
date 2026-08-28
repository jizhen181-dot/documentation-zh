# wallet-cli contract send

改变链上状态的合约调用（triggerSmartContract）。

## 用法

```
wallet-cli contract send --contract <address> --method <sig> [--params <json>]
                         [--call-value-sun <n>] [--fee-limit <sun>]
                         [--dry-run | (--sign-only | --build-only) [--expiration <ms>] | --wait [--wait-timeout <ms>]] [--permission-id <n>] [options]
```

## 说明

以当前账户（或 `--account`）构建、签名并广播一次会改变链上状态的合约调用。参数沿用与 [`contract call`](call.md) 相同的 `{type,value}` JSON 数组约定；`--call-value-sun` 用于在调用时附带原生 TRX。

两种提前退出方式：`--dry-run` 预览能量开销（`estimateEnergy`），既不签名也不广播；`--sign-only` 完成签名并打印交易，供之后 [`tx broadcast`](../tx/broadcast.md) 使用。

**该命令默认在提交时返回**（`stage: "submitted"`）——加 `--wait` 可阻塞直到已确认/失败。使用 `--wait` 时，链上执行失败（`revert` / `OUT_OF_ENERGY`）会以 `stage: "failed"` 返回，并在 `result` 中给出原因。

需要一个账户。只有会签名的模式才需要 master password（通过 `--password-stdin`）——`--dry-run` 和 `--build-only` 不会解锁钱包，无需密码即可运行。在签名模式下，仅观察账户会以 `watch_only_no_signer` 失败。

## 选项

| 选项 | 说明 |
|---|---|
| `--contract <string>` | **必填。** 合约地址 |
| `--method <string>` | **必填。** 函数签名，例如 `transfer(address,uint256)` |
| `--params <string>` | ABI 参数的 JSON 数组，元素形如 `{type,value}` |
| `--call-value-sun <number>` | 随调用附带的原生 TRX，单位 SUN（默认 0） |
| `--fee-limit <number>` | 允许燃烧的最高能量费用，单位 SUN（默认 100000000） |
| `--dry-run` | 只估算能量，不签名/不广播；与 `--sign-only` / `--build-only` 互斥 |
| `--sign-only` | 只签名不广播，输出已签名的 hex；与 `--dry-run` / `--build-only` 互斥；配合 `--expiration` 使用 |
| `--build-only` | 只构建，输出**未签名**的 hex；与 `--dry-run` / `--sign-only` 互斥；配合 `--expiration` 使用 |
| `--expiration <ms>` | 交易过期时间（毫秒），最大 `86400000`（24 小时）；仅可与 `--sign-only` 或 `--build-only` 同用；省略时使用节点默认值（约 60 秒） |
| `--permission-id <n>` | 用于签名的权限组（0=owner，1=witness，2-9=active）；默认 `0` |
| `--wait` / `--wait-timeout <ms>` | 广播后轮询直到已确认/失败（上限默认取配置 `waitTimeoutMs`，内置 60000） |
| `--password-stdin` | 从 stdin 读取 master password |

此外还有[全局选项](../index.md#global-options-every-command)。

## 示例

示例中的 `$PW` 是你的 master password（来自环境变量、密码管理器等），通过 `--password-stdin` 从 stdin 传入。

默认——广播并返回**已提交**的回执：

```bash
echo "$PW" | wallet-cli contract send --contract TXYZopYRdj2D9XRtbG411XZZ3kM5VkAeBf --method "transfer(address,uint256)" --params '[{"type":"address","value":"TSx72ViULFepRGCS4PM5dP4FqD1d8qggCc"},{"type":"uint256","value":"1000000"}]' --network tron:nile --password-stdin
```

```console
⏳ Called transfer
  TxID    c8d...
  Status  pending — not yet on-chain
! Track it: wallet-cli tx info --network tron:nile --txid c8d...
```

```bash
echo "$PW" | wallet-cli contract send --contract TXYZopYRdj2D9XRtbG411XZZ3kM5VkAeBf --method "transfer(address,uint256)" --params '[...]' --network tron:nile --password-stdin -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"contract.send","data":{"kind":"contract-send","stage":"submitted","txId":"c8d...","method":"transfer(address,uint256)","contract":"TXYZopYRdj2D9XRtbG411XZZ3kM5VkAeBf"},"meta":{"durationMs":15,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

加上 `--wait` 会阻塞直到确认——成功时：

```bash
echo "$PW" | wallet-cli contract send --contract TXYZopYRdj2D9XRtbG411XZZ3kM5VkAeBf --method "transfer(address,uint256)" --params '[...]' --network tron:nile --wait --password-stdin
```

```console
✅ Called transfer
  Contract  TXYZopYRdj2D9XRtbG411XZZ3kM5VkAeBf
  TxID      0adc5737b724d35c486a05a169b64a01ad311ed27f79d308f245b00c69b3bc42
  Block     #69,095,391
  Energy    14,584
  Fee       0.345 TRX
  Status    success
```

链上执行失败（例如能量不足）会返回 `stage: "failed"`：

```bash
echo "$PW" | wallet-cli contract send --contract TXYZopYRdj2D9XRtbG411XZZ3kM5VkAeBf --method "transfer(address,uint256)" --params '[...]' --network tron:nile --wait --password-stdin
```

```console
❌ Called transfer
  TxID    c8d...
  Block   #66,000,123
  Energy  31,200
  Status  failed
  Reason  OUT_OF_ENERGY
```

## 输出

`data` 随阶段而变：

| 模式 | 字段 |
|---|---|
| 默认（提交） | `kind: "contract-send"`、`stage: "submitted"`、`txId`、`method`、`contract` |
| `--wait`（已确认/失败） | 同上，但 `stage` 为 `"confirmed"` 或 `"failed"`，另加 `confirmed`、`blockNumber`、`feeSun`、`energyUsed`、`result`（`SUCCESS` / `OUT_OF_ENERGY` 等）、`failed` |
| `--dry-run` | `kind`、`mode: "dry-run"`、`fee`（含 `feeModel`、估算的 `energy`、`availableEnergy`）、未签名的 `tx` |
| `--sign-only` | `kind`、`mode: "sign-only"`、`hex`（已签名交易的 hex）、`signed`（同一笔交易的 TRON tx 对象形式，含 `signature[]`）、`address`（签名者）、`txId`、`fee`、`method`、`contract` |
| `--build-only` | `kind`、`mode: "build-only"`、`hex`（**未签名**交易的 hex）、未签名的 `tx`（TRON tx 对象）、`fee`、`method`、`contract` |

## 退出码

`0` 已提交（早退模式下为已构建/已签名） · `1` 执行失败（`watch_only_no_signer`、`auth_failed`、`rpc_error`、`timeout`——超时后交易可能仍在途中，请用 [`tx status`](../tx/status.md) 查询） · `2` 用法错误（`invalid_value`、模式冲突）。

## 另请参见

[`contract call`](call.md) · [`contract deploy`](deploy.md) · [`tx broadcast`](../tx/broadcast.md) · [能量与带宽](../../concepts/energy-bandwidth.md)

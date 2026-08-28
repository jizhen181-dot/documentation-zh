# wallet-cli asset update

修改你所发行 TRC10 中的可变字段。

## 用法

```
wallet-cli asset update [--description <s>] [--url <url>]
                        [--free-net-per-account <n>] [--public-free-net <n>]
                        [--dry-run | (--sign-only | --build-only) [--expiration <ms>] | --wait [--wait-timeout <ms>]]
                        [--permission-id <n>] [options]
```

## 说明

本命令不接受 token 参数：它始终作用于签名账户所发行的那个 TRC10。没有发行过的账户会以 `not_an_issuer` 失败。

**永远只有四个字段可以修改**——描述、URL、每位持有人的免费带宽，以及共享的免费带宽池。供应量、精度、ICO 比率、ICO 窗口和冻结批次在发行时即固定，链上没有任何办法更改它们。

只传你要修改的字段。其余字段会从链上读出后原样写回，因此不会被悄悄清空；但至少要给出一个字段。回执会展示这四个字段当前的值。

**该命令默认在提交时返回**（`stage: "submitted"`），而不是确认时——加 `--wait` 可阻塞直到已确认/失败。需要一个账户。只有会签名的模式才需要 master password（通过 `--password-stdin`）——`--dry-run` 和 `--build-only` 不会解锁钱包，无需密码即可运行。在签名模式下，仅观察账户会以 `watch_only_no_signer` 失败。

## 选项

| 选项 | 说明 |
|---|---|
| `--description <s>` | 新的描述，≤ 200 字节（省略则保持不变） |
| `--url <url>` | 新的项目页面，非空，≤ 256 字节（省略则保持不变） |
| `--free-net-per-account <n>` | 每位持有人可用的免费带宽（省略则保持不变） |
| `--public-free-net <n>` | 持有人共享的免费带宽池（省略则保持不变） |
| `--dry-run` | 只构建和估算，不签名/不广播；与 `--sign-only` / `--build-only` 互斥 |
| `--sign-only` | 只签名不广播，输出已签名的 hex；与 `--dry-run` / `--build-only` 互斥；配合 `--expiration` 使用 |
| `--build-only` | 只构建，输出**未签名**的 hex；与 `--dry-run` / `--sign-only` 互斥；配合 `--expiration` 使用 |
| `--expiration <ms>` | 交易过期时间（毫秒），最大 `86400000`（24 小时）；仅可与 `--sign-only` 或 `--build-only` 同用；省略时使用节点默认值（约 60 秒） |
| `--permission-id <n>` | 用于签名的权限组（0=owner，1=witness，2-9=active）；默认 `0` |
| `--wait` / `--wait-timeout <ms>` | 广播后轮询直到已确认/失败（上限默认取配置 `waitTimeoutMs`，内置 60000） |
| `--password-stdin` | 从 stdin（fd 0）读取 master password |

此外还有[全局选项](../index.md#global-options-every-command)。

## 示例

示例中的 `$PW` 是你的 master password（来自环境变量、密码管理器等），通过 `--password-stdin` 从 stdin 传入。

```bash
echo "$PW" | wallet-cli asset update --url https://mytoken.io/v2 --network tron:nile --wait --password-stdin
```

```console
✅ Asset updated
  Asset             MyToken  (id 1000123)
  Issuer            TQkXm4vN...5Zt7Uw (main)
  Url               https://mytoken.io/v2
  Description       Demo TRC10
  Free net/account  0
  Public free net   0
  TxID              9e3...
  Block             57,883,190
  Fee               0 TRX  (295 bandwidth)
  Status            success
```

```bash
echo "$PW" | wallet-cli asset update --url https://mytoken.io/v2 --network tron:nile --wait --password-stdin -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"asset.update","data":{"kind":"asset-update","stage":"confirmed","txId":"9e3...","confirmed":true,"blockNumber":57883190,"failed":false,"assetId":"1000123","name":"MyToken","issuerAddress":"TQkXm4vN...","url":"https://mytoken.io/v2","description":"Demo TRC10","freeAssetNetLimit":0,"publicFreeAssetNetLimit":0,"feeSun":0,"resource":{"netUsage":295,"netFeeSun":0,"energyUsage":0,"energyFeeSun":0}},"meta":{"durationMs":6480,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出

`data` 随阶段而变：

| 阶段 | 字段 |
|---|---|
| 默认（提交） | `kind: "asset-update"`、`stage: "submitted"`、`txId`、`assetId`、`name`、`issuerAddress`，以及提交时的那四个字段 |
| `--wait`（已确认） | 同上，另加 `stage: "confirmed"`、`confirmed`（boolean）、`blockNumber`、`feeSun`、`resource`、`failed` |

这四个字段是 `url`、`description`、`freeAssetNetLimit` 和 `publicFreeAssetNetLimit`——始终四个都在，包括那些原样读回、未做修改的字段。

## 退出码

`0` 已提交（早退模式下为已构建/已签名） · `1` 执行失败（`not_an_issuer`——该账户没有发行过 TRC10；`watch_only_no_signer`；`auth_failed`） · `2` 用法错误（`missing_option`——一个字段都没给；`invalid_value`——URL 或描述过长，带宽限额超出范围）。

## 另请参见

[`asset issue`](issue.md) · [`asset info`](info.md) · [脚本安全](../../machine-interface.md#script-safety-never-mistake-submitted-for-confirmed)

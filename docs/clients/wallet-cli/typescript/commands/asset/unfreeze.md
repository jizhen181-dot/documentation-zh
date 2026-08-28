# wallet-cli asset unfreeze

释放你所发行 TRC10 中已到期的冻结供应量。

## 用法

```
wallet-cli asset unfreeze
                          [--dry-run | (--sign-only | --build-only) [--expiration <ms>] | --wait [--wait-timeout <ms>]]
                          [--permission-id <n>] [options]
```

## 说明

把发行时冻结、且锁定期已结束的那部分已发行供应量，退回到发行方的余额中。

本命令不接受任何参数：它始终作用于签名账户所发行的 token，链上既不接受"哪一个批次"，也不接受"多少数量"——**所有已到期的批次会在一笔交易中全部释放**。尚未到期的批次不受影响；等它们到期后再次运行本命令即可。

批次的到期时间是发行时的 `--start` 加上该批次的 `days`，而不是 token 实际发行的那一刻：创建 token 时，链会把每个批次的 `expire_time` 写为 `start_time + days × 86400000`。由此得到的日期可在 [`asset info`](info.md) 的 `Frozen` 小节中看到。

它与释放已质押 TRX 的 [`stake unfreeze`](../stake/unfreeze.md) 无关；两者唯一的共同点只是名字。

**该命令默认在提交时返回**（`stage: "submitted"`），而不是确认时——加 `--wait` 可阻塞直到已确认/失败。需要一个账户。只有会签名的模式才需要 master password（通过 `--password-stdin`）——`--dry-run` 和 `--build-only` 不会解锁钱包，无需密码即可运行。在签名模式下，仅观察账户会以 `watch_only_no_signer` 失败。

## 选项

本命令没有自己的选项。

| 选项 | 说明 |
|---|---|
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
echo "$PW" | wallet-cli asset unfreeze --network tron:nile --wait --password-stdin
```

```console
✅ Frozen supply released
  Asset         MyToken  (id 1000123)
  Issuer        TQkXm4vN...5Zt7Uw (main)
  Released      100,000,000 MyToken
  Still frozen  50,000,000 MyToken
  TxID          6a5...
  Block         57,883,560
  Fee           0 TRX  (288 bandwidth)
  Status        success
```

```bash
echo "$PW" | wallet-cli asset unfreeze --network tron:nile --wait --password-stdin -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"asset.unfreeze","data":{"kind":"asset-unfreeze","stage":"confirmed","txId":"6a5...","confirmed":true,"blockNumber":57883560,"failed":false,"assetId":"1000123","name":"MyToken","issuerAddress":"TQkXm4vN...","releasedAmount":100000000000000,"stillFrozenAmount":50000000000000,"feeSun":0,"resource":{"netUsage":288,"netFeeSun":0,"energyUsage":0,"energyFeeSun":0}},"meta":{"durationMs":6410,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出

`data` 随阶段而变：

| 阶段 | 字段 |
|---|---|
| 默认（提交） | `kind: "asset-unfreeze"`、`stage: "submitted"`、`txId`、`assetId`、`name`、`issuerAddress` |
| `--wait`（已确认） | 以上内容，外加 `stage: "confirmed"`、`confirmed`（boolean）、`blockNumber`、`feeSun`、`resource`、`failed`、`releasedAmount`、`stillFrozenAmount` |

`releasedAmount` 和 `stillFrozenAmount` 是原始金额（最小单位），反映已确认交易实际完成的结果。

## 退出码

`0` 已提交（早退模式下为已构建/已签名） · `1` 执行失败（`not_an_issuer`——该账户没有发行过 TRC10；`no_frozen_supply`；`not_yet_unfreezable`——还没有任何批次到期；`watch_only_no_signer`；`auth_failed`） · `2` 用法错误。

## 另请参见

[`asset info`](info.md) · [`asset issue`](issue.md) · [`stake unfreeze`](../stake/unfreeze.md) · [脚本安全](../../machine-interface.md#script-safety-never-mistake-submitted-for-confirmed)

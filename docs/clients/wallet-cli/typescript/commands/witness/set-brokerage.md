# wallet-cli witness set-brokerage

设置 SR 保留的出块奖励份额。

## 用法

```
wallet-cli witness set-brokerage <percent>
                                 [--dry-run | (--sign-only | --build-only) [--expiration <ms>] | --wait [--wait-timeout <ms>]]
                                 [--permission-id <n>] [options]
```

## 说明

`<percent>` 就是**佣金比例**（brokerage）——SR 为自己保留的出块奖励百分比；剩下的 `100 − percent` 按票数比例分配给它的投票人。默认值为 20，广播之前会在本地校验它是 0–100 之间的整数。

它就是 [`vote list`](../vote/list.md) 中 `brokeragePct` 报告的那个数值；该页面的 `Reward ratio` 列是它的补数——即投票人的份额。设为 `100` 意味着投票人从你出的块中一无所获。

任何已注册的见证人都可以设置它，无论是否当选。执行操作的账户必须是候选人，否则命令会以 `not_a_witness` 失败。

**该命令默认在提交时返回**（`stage: "submitted"`），而不是确认时——加 `--wait` 可阻塞直到已确认/失败。需要一个账户。只有会签名的模式才需要 master password（通过 `--password-stdin`）——`--dry-run` 和 `--build-only` 不会解锁钱包，无需密码即可运行。在签名模式下，仅观察账户会以 `watch_only_no_signer` 失败。

## 选项

| 选项 | 说明 |
|---|---|
| `<percent>` | **必填。** SR 保留的份额，0–100 的整数 |
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

自己保留 20 %，把 80 % 分给投票人：

```bash
echo "$PW" | wallet-cli witness set-brokerage 20 --network tron:nile --wait --password-stdin
```

```console
✅ Brokerage set
  Witness    TSRmq8kP...9dEf (main)
  Brokerage  20%
  TxID       f8c...
  Block      57,881,402
  Fee        0 TRX  (269 bandwidth)
  Status     success
```

```bash
echo "$PW" | wallet-cli witness set-brokerage 20 --network tron:nile --wait --password-stdin -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"witness.set-brokerage","data":{"kind":"witness-set-brokerage","stage":"confirmed","txId":"f8c...","confirmed":true,"blockNumber":57881402,"failed":false,"witnessAddress":"TSRmq8kP...","brokerage":20,"feeSun":0,"resource":{"netUsage":269,"netFeeSun":0,"energyUsage":0,"energyFeeSun":0}},"meta":{"durationMs":6470,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出

`data` 随阶段而变：

| 阶段 | 字段 |
|---|---|
| 默认（提交） | `kind: "witness-set-brokerage"`、`stage: "submitted"`、`txId`、`witnessAddress`、`brokerage` |
| `--wait`（已确认） | 同上，另加 `stage: "confirmed"`、`confirmed`（boolean）、`blockNumber`、`feeSun`、`resource`、`failed` |

`brokerage` 是当前生效的值，类型为 number。

## 退出码

`0` 已提交（早退模式下为已构建/已签名） · `1` 执行失败（`not_a_witness`、`watch_only_no_signer`、`auth_failed`） · `2` 用法错误（`invalid_value`——percent 缺失、不是整数，或超出 0–100 范围）。

## 另请参见

[`witness create`](create.md) · [`vote list`](../vote/list.md) · [`reward balance`](../reward/balance.md) · [脚本安全](../../machine-interface.md#script-safety-never-mistake-submitted-for-confirmed)

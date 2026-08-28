# wallet-cli asset participate

用 TRX 参与某个 TRC10 的 ICO。

## 用法

```
wallet-cli asset participate <asset> --pay <trx>
                             [--dry-run | (--sign-only | --build-only) [--expiration <ms>] | --wait [--wait-timeout <ms>]]
                             [--permission-id <n>] [options]
```

## 说明

在募集窗口内，按 token 发行时设定的固定比率从其发行中买入。这是参与 ICO，而不是市场交易——token 出自发行方的剩余供应量，价格没有商量余地。发行方地址由 token 自动解析得到，因此无需另行传入。

**`--pay` 是你花掉的 TRX，不是你收到的 token 数量。** 你得到的是 `floor(pay × tokens ÷ trx)`，其中 `trx:tokens` 是该 token 发行时的比率——即支付金额乘以单价后向下取整，因为链上按整数先乘后除。TRX 会被全额转出，因此被截断的余数不会退还；损失小于 1 sun，而当比率的 `trxNum` 为 1 时根本不会发生。如果 `--pay` 小到连一个最小单位都买不到，命令会在本地失败，而不会广播。

执行操作的账户不能是该 token 自己的发行方。

**该命令默认在提交时返回**（`stage: "submitted"`），而不是确认时——加 `--wait` 可阻塞直到已确认/失败。需要一个账户。只有会签名的模式才需要 master password（通过 `--password-stdin`）——`--dry-run` 和 `--build-only` 不会解锁钱包，无需密码即可运行。在签名模式下，仅观察账户会以 `watch_only_no_signer` 失败。

## 选项

| 选项 | 说明 |
|---|---|
| `<asset>` | **必填。** token id 或名称；全数字的值按 id 解析 |
| `--pay <trx>` | **必填。** 要花费的 TRX（不是 token 数量），> 0 |
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

在一个按 `1:100` 发行的 token 上花费 100 TRX：

```bash
echo "$PW" | wallet-cli asset participate 1000124 --pay 100 --network tron:nile --wait --password-stdin
```

```console
✅ Participated in ICO
  Asset        BetaToken  (id 1000124)
  Issuer       TBeta9mR...8pLx
  Participant  TQkXm4vN...5Zt7Uw (main)
  Paid         100 TRX
  Received     10,000 BetaToken
  TxID         4c8...
  Block        57,883,402
  Fee          0 TRX  (301 bandwidth)
  Status       success
```

```bash
echo "$PW" | wallet-cli asset participate 1000124 --pay 100 --network tron:nile --wait --password-stdin -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"asset.participate","data":{"kind":"asset-participate","stage":"confirmed","txId":"4c8...","confirmed":true,"blockNumber":57883402,"failed":false,"assetId":"1000124","name":"BetaToken","issuerAddress":"TBeta9mR...","participantAddress":"TQkXm4vN...","paidSun":100000000,"receivedAmount":10000000000,"feeSun":0,"resource":{"netUsage":301,"netFeeSun":0,"energyUsage":0,"energyFeeSun":0}},"meta":{"durationMs":6450,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出

`data` 随阶段而变：

| 阶段 | 字段 |
|---|---|
| 默认（提交） | `kind: "asset-participate"`、`stage: "submitted"`、`txId`、`assetId`、`name`、`issuerAddress`、`participantAddress`、`paidSun`、`receivedAmount` |
| `--wait`（已确认） | 同上，另加 `stage: "confirmed"`、`confirmed`（boolean）、`blockNumber`、`feeSun`、`resource`、`failed` |

`paidSun` 是花费的 TRX，以 sun 计；`receivedAmount` 是 token 数量，以其最小单位计（text 模式下两者都按人类可读单位显示）。

## 退出码

`0` 已提交（早退模式下为已构建/已签名） · `1` 执行失败（`asset_not_found`——没有这个 token；`not_in_ico_window`——不在募集窗口内；`self_participation`——这个 token 是你自己发行的；`insufficient_balance`；`watch_only_no_signer`；`auth_failed`） · `2` 用法错误（`missing_option`——没有给 `--pay`；`invalid_amount`——`--pay` 不是十进制数，或小数位超过 6 位；`invalid_value`——`--pay` ≤ 0，或小到买不了一个最小单位）。

## 另请参见

[`asset info`](info.md) · [`tx send`](../tx/send.md) · [`exchange trade`](../exchange/trade.md) · [脚本安全](../../machine-interface.md#script-safety-never-mistake-submitted-for-confirmed)

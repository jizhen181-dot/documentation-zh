# wallet-cli asset issue

发行一个 TRC10 token，并锁定其 ICO 条款。

## 用法

```
wallet-cli asset issue --name <name> --supply <n> --price <trx>:<tokens>
                       --start <datetime> --end <datetime> --url <url>
                       [--abbr <s>] [--precision <0-6>] [--description <s>]
                       [--free-net-per-account <n>] [--public-free-net <n>]
                       [--freeze <amount>:<days> ...]
                       [--dry-run | (--sign-only | --build-only) [--expiration <ms>] | --wait [--wait-timeout <ms>]]
                       [--permission-id <n>] [options]
```

## 说明

创建一个 TRC10 token，并在同一笔交易中固定其 ICO 条款：总供应量、精度、TRX 兑 token 的比率、募集窗口，以及全部冻结批次。

**此操作不可撤销。** 发行费用会被燃烧——即链参数 `getAssetIssueFee`，目前约 1,024 TRX，可用 [`chain params`](../chain/params.md) 读取——而且一个账户**终身只能发行一个 TRC10**。此后只有描述、URL 和两项免费带宽限额还能修改（[`asset update`](update.md)）；其余一切都是永久的。因此回执会完整回显 token 定义，因为这就是最终的定义。

**`--price` 会按 `--precision` 换算。** 链上把比率存为一对整数 `trxNum` / `num`，满足 `num ÷ trxNum = tokens × 10^precision ÷ (trx × 10^6)`，并约分至最简。于是 `--price 1:100` 在 `--precision 6` 下存为 `trxNum=1, num=100`，而在 `--precision 0` 下存为 `trxNum=10000, num=1`——同一个参数，链上比率却不同。约分后两个值都必须落在正 int32 范围内；否则命令以 `invalid_value` 失败，什么都不会广播。

金额（`--supply`、`--freeze`）以**完整 token** 计——`--supply 1000000000 --precision 6` 在链上会成为 `total_supply` 为 `1000000000000000`。

日期按 **UTC** 解析，格式为 `YYYY-MM-DD` 或 `YYYY-MM-DD HH:mm:ss`；只写日期表示 `00:00:00`。`--start` 必须晚于链上的当前时间，因此只写日期最早也要到明天——想当天开售，就把时间一并写上。

限制条件在广播前于本地校验：`--name` 和 `--abbr` 为 1–32 个可见 ASCII 字符（`0x21`–`0x7E`，因此不能有空格，也不能有非 ASCII 字符）；`--url` 必填且不超过 256 字节；`--description` 不超过 200 字节；`--precision` 为 0–6；`--end` 必须晚于 `--start`；每个 `--freeze` 批次的天数须落在 `getMinFrozenSupplyTime`…`getMaxFrozenSupplyTime` 之间，批次数量不超过 `getMaxFrozenSupplyNumber`，各批次金额之和不超过总供应量；两项免费带宽限额都必须低于 `getOneDayNetLimit`。

**该命令默认在提交时返回**（`stage: "submitted"`），而不是确认时——加 `--wait` 可阻塞直到已确认/失败。需要一个账户。只有会签名的模式才需要 master password（通过 `--password-stdin`）——`--dry-run` 和 `--build-only` 不会解锁钱包，无需密码即可运行。在签名模式下，仅观察账户会以 `watch_only_no_signer` 失败。

## 选项

| 选项 | 说明 |
|---|---|
| `--name <name>` | **必填。** token 名称，1–32 个可见 ASCII 字符 |
| `--supply <n>` | **必填。** 总供应量，以完整 token 计，> 0 |
| `--price <trx>:<tokens>` | **必填。** ICO 比率，完整 TRX 兑完整 token（如 `1:100`）；两侧都必须 > 0，按 `--precision` 换算 |
| `--start <datetime>` | **必填。** ICO 开始时间，UTC；必须在将来 |
| `--end <datetime>` | **必填。** ICO 结束时间，UTC；必须晚于 `--start` |
| `--url <url>` | **必填。** 项目页面，非空，≤ 256 字节 |
| `--abbr <s>` | token 缩写；字符规则同 `--name`（默认：空） |
| `--precision <0-6>` | 小数位数（默认 `0`） |
| `--description <s>` | 简短描述，≤ 200 字节（默认：空） |
| `--free-net-per-account <n>` | 每位持有人可用的免费带宽（默认 `0`） |
| `--public-free-net <n>` | 持有人共享的免费带宽池（默认 `0`） |
| `--freeze <amount>:<days>` | **可重复。** 冻结批次；金额以完整 token 计，例如 `100000000:30` |
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
echo "$PW" | wallet-cli asset issue --name MyToken --abbr MTK --supply 1000000000 --price 1:100 --precision 6 \
  --start 2026-08-01 --end 2026-08-31 --url https://mytoken.io --description "Demo TRC10" \
  --freeze 100000000:30 --freeze 50000000:90 --network tron:nile --wait --password-stdin
```

```console
✅ Asset issued
  Asset             MyToken  (id 1000123)
  Issuer            TQkXm4vN...5Zt7Uw (main)
  Total supply      1,000,000,000
  Precision         6
  Price             1 TRX = 100 MyToken
  ICO start time    2026-08-01 00:00 UTC
  ICO end time      2026-08-31 00:00 UTC
  Url               https://mytoken.io
  Description       Demo TRC10
  Free net/account  0
  Public free net   0
  Frozen (2)
    100,000,000  for 30 days
     50,000,000  for 90 days
  TxID              7d1...
  Block             57,883,010
  Fee               1,024 TRX  (312 bandwidth)
  Status            success
```

```bash
echo "$PW" | wallet-cli asset issue --name MyToken --abbr MTK --supply 1000000000 --price 1:100 --precision 6 \
  --start 2026-08-01 --end 2026-08-31 --url https://mytoken.io --network tron:nile --wait --password-stdin -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"asset.issue","data":{"kind":"asset-issue","stage":"confirmed","txId":"7d1...","confirmed":true,"blockNumber":57883010,"failed":false,"assetId":"1000123","name":"MyToken","abbr":"MTK","totalSupply":1000000000000000,"precision":6,"price":"1:100","trxNum":1,"num":100,"startTime":1785542400000,"endTime":1788134400000,"url":"https://mytoken.io","description":"Demo TRC10","freeAssetNetLimit":0,"publicFreeAssetNetLimit":0,"frozenSupply":[{"amount":100000000000000,"days":30},{"amount":50000000000000,"days":90}],"feeSun":1024000000,"resource":{"netUsage":312,"netFeeSun":0,"energyUsage":0,"energyFeeSun":0}},"meta":{"durationMs":6720,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出

`data` 随阶段而变：

| 阶段 | 字段 |
|---|---|
| 默认（提交） | `kind: "asset-issue"`、`stage: "submitted"`、`txId`，以及下面列出的 token 定义字段（不含 `assetId`） |
| `--wait`（已确认） | 以上内容，外加 `stage: "confirmed"`、`confirmed`（boolean）、`blockNumber`、`feeSun`、`resource`、`failed`，以及 `assetId`——由链分配，因此只有确认后才可知 |

定义字段：`name`、`abbr`、`totalSupply`（原始值）、`precision`、`price`（原样保留的 `trx:tokens` 字符串）连同存储的 `trxNum` / `num` 对、`startTime` / `endTime`（epoch 以来的毫秒数）、`url`、`description`、`freeAssetNetLimit`、`publicFreeAssetNetLimit`，以及 `frozenSupply[]`（`amount` 为原始值，`days`）。

## 退出码

`0` 已提交（早退模式下为已构建/已签名） · `1` 执行失败（`already_issued_asset`——该账户已经发行过一个；`insufficient_balance`——余额低于发行费用；`watch_only_no_signer`；`auth_failed`） · `2` 用法错误（`missing_option`——缺少必填参数；`invalid_asset_name`——名称或缩写不在 1–32 个可见 ASCII 字符范围内；`invalid_value`——比率、精度、日期、带宽限额或冻结批次超出范围，或换算后比率超出 int32）。

## 另请参见

[`asset update`](update.md) · [`asset info`](info.md) · [`asset participate`](participate.md) · [`chain params`](../chain/params.md) · [脚本安全](../../machine-interface.md#script-safety-never-mistake-submitted-for-confirmed)

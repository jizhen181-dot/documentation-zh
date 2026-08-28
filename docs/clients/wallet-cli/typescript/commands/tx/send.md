# wallet-cli tx send

用人类可读的 `--amount` 发送原生 TRX 或 TRC20/TRC10 token。

## 用法

```
wallet-cli tx send --to <address|contact> (--amount <n> | --raw-amount <n>)
                   [--token <symbol> | --contract <address> | --asset-id <id>] [--fee-limit <sun>]
                   [--dry-run | (--sign-only | --build-only) [--expiration <ms>] | --wait [--wait-timeout <ms>]]
                   [--permission-id <n>] [options]
```

## 说明

从当前账户（或 `--account` 指定的账户）构建、签名并提交一笔转账。发送什么取决于你传入的选择器：

- **不传** → 原生 TRX；
- `--token <symbol>` → 从本地 token 地址簿解析出的 token；
- `--contract <address>` → 按合约地址指定的 TRC20；
- `--asset-id <id>` → 按数字资产 id 指定的 TRC10。

金额：`--amount` 是人类可读单位（TRX，或按该 token 的小数位换算的 token 单位）；`--raw-amount` 是原始整数（SUN 或 token 最小单位）。两者必须且只能选其一。

TRX 固定使用 6 位小数；token 的小数位数由节点提供，TRC20 读取合约定义，TRC10 读取资产记录。
`--amount` 会根据该值换算为最小单位，因此节点返回错误的小数位数会导致签名金额发生偏差。CLI 会校验
协议允许的范围（TRC10 精度为 0..6），并拒绝 id 不匹配的资产记录，但无法识别范围内的错误值。需要
精确控制最小单位数量时，请使用 `--raw-amount`，该值不会再次换算。

提前退出的模式：`--dry-run` 只构建并估算——不签名、不广播，**但构建和估算本身仍会调用 RPC**，因此该命令依然会访问网络；`--sign-only` 会签名并打印已签名的交易 **hex**（它同样要先构建和估算，因此也需要联网）；`--build-only` 只构建、**不**签名，打印**未签名**的 hex，可交给离线机器上的 [`tx sign`](sign.md)。在多签场景下，`--permission-id` 用于选择签名所用的权限组，`--expiration` 则延长交易的有效期，好让联署人有时间补上各自的签名。

**该命令默认在提交时返回**（`stage: "submitted"`），而不是确认时——加 `--wait` 可阻塞直到已确认/失败，也可以轮询 [`tx status`](status.md)。

需要一个账户。只有**软件账户真正签名**时才需要通过 `--password-stdin` 传入 master password——链上签名命令不会弹出交互式提示，缺少它时命令会以 `auth_required` 失败。以下情况不需要密码：`--dry-run`（不签名）、`--build-only`（不签名）、以及 Ledger 账户（在设备上签名）。仅观察账户会以 `watch_only_no_signer` 失败。

## 选项

| 选项 | 说明 |
|---|---|
| `--to <address\|contact>` | **必填。** 接收方的 TRON base58 地址，或[联系人簿](../contact/index.md)中的一个名称 |
| `--amount <string>` | 人类可读金额；与 `--raw-amount` 互斥 |
| `--raw-amount <string>` | 以 SUN / token 最小单位表示的原始整数金额 |
| `--token <string>` | token 地址簿中的 token 符号；与 `--contract`、`--asset-id` 互斥 |
| `--contract <string>` | TRC20 合约地址 |
| `--asset-id <string>` | TRC10 数字资产 id |
| `--fee-limit <string>` | TRC20 转账最多可燃烧的 TRX 能量费用，单位 SUN（默认 100000000） |
| `--dry-run` | 只构建和估算；与 `--sign-only` / `--build-only` 互斥 |
| `--sign-only` | 只签名不广播，输出已签名的 hex；与 `--dry-run` / `--build-only` 互斥；配合 `--expiration` 使用 |
| `--build-only` | 只构建，输出**未签名**的 hex；与 `--dry-run` / `--sign-only` 互斥；配合 `--expiration` 使用 |
| `--expiration <ms>` | 交易过期时间（毫秒），最大 `86400000`（24 小时）；仅可与 `--sign-only` 或 `--build-only` 同用；省略时使用节点默认值（约 60 秒） |
| `--permission-id <n>` | 用于签名的权限组（0=owner，1=witness，2-9=active）；默认 `0` |
| `--wait` / `--wait-timeout <ms>` | 广播后轮询直到已确认/失败（上限默认 60000；到达上限时返回已提交的回执） |
| `--password-stdin` | 从 stdin 读取 master password |

此外还有[全局选项](../index.md#global-options-every-command)。

## 示例

> **关于密码**：下面的示例都省略了密码参数，以便把重点放在选择器参数上。用**软件账户实际签名**时，需要从 stdin 传入 master password——在命令前加上 `printf '%s' "$PW" |`，并在末尾追加 `--password-stdin`（参见上面的说明）；`--dry-run`、`--build-only` 和 Ledger 账户则不需要传入密码。

```bash
# 在 Nile 上发送 1 TRX
wallet-cli tx send --to TSx72ViULFepRGCS4PM5dP4FqD1d8qggCc --amount 1 --network tron:nile

# TRC20 用地址簿中的符号；TRC10 用资产 id
wallet-cli tx send --to T... --token USDT --amount 5 --network tron:nile
wallet-cli tx send --to T... --asset-id 1002000 --raw-amount 1000000 --network tron:nile

# 只演练，不签名
wallet-cli tx send --to TSx72ViULFepRGCS4PM5dP4FqD1d8qggCc --amount 1 --network tron:nile --dry-run -o json
```

提交回执（默认模式，文本与 json）：

```bash
printf '%s' "$PW" | wallet-cli tx send --to TGkbaCYB4kRBc3Q6wjqkACefUvRwf2KzkH --amount 1 --network tron:nile --password-stdin
```

```console
⏳ Sent 1 TRX
  To      TGkbaCYB4kRBc3Q6wjqkACefUvRwf2KzkH
  TxID    4574b646adc694e99a1f64e548b2bdf9da62621c2d833f77354f67b751fbd0c4
  Status  pending — not yet on-chain
! Track it: wallet-cli tx info --network tron:nile --txid 4574b646adc694e99a1f64e548b2bdf9da62621c2d833f77354f67b751fbd0c4
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"tx.send","data":{"kind":"send","stage":"submitted","txId":"4574b646adc694e99a1f64e548b2bdf9da62621c2d833f77354f67b751fbd0c4","rawAmount":"1000000","to":"TGkbaCYB4kRBc3Q6wjqkACefUvRwf2KzkH"},"meta":{"durationMs":2172,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出

`data` 随模式而变：

| 模式 | 字段 |
|---|---|
| 默认（提交） | `kind: "send"`、`stage: "submitted"`、`txId`、`rawAmount`（字符串）、`to`；当 `--to` 传的是联系人名称时还有 `toContact` |
| `--wait`（已确认） | 同上，但 `stage: "confirmed"`，另加 `confirmed`、`blockNumber`、`netUsed`（消耗的带宽）或 `feeSun`（燃烧的费用）、`failed` |
| `--wait`（已回滚） | 字段相同，但 `stage: "failed"` 且 `failed: true`——交易被打包后又发生了回滚 |
| `--dry-run` | `kind`、`mode: "dry-run"`、`fee`（`feeModel`，例如 `bandwidthBurnSunIfNoFreeze`）、未签名的 `tx`（TRON 交易对象，含 `txID`、`raw_data`）、`rawAmount`、`to` |
| `--sign-only` | `kind`、`mode: "sign-only"`、`hex`（已签名的交易 hex）、`signed`（同一笔交易的 TRON 交易对象形式，含 `signature[]`）、`address`（签名者）、`txId`、`fee`、`rawAmount`、`to` |
| `--build-only` | `kind`、`mode: "build-only"`、`hex`（**未签名**的交易 hex）、未签名的 `tx`（TRON 交易对象）、`fee`、`rawAmount`、`to` |

交易在链上执行失败时，响应中的 `success` 仍可能为 `true`、退出码为 `0`，因为 CLI 已完成提交和查询。
脚本必须根据 `data.stage` 判断交易结果，不能只检查退出码。

## 退出码

`0` 已提交（提前退出模式下为已构建/已签名） · `1` 执行失败（`rpc_error`、`timeout`——**超时时交易可能仍在途中；重发之前请先用 `tx status` 检查**） · `2` 用法错误（选择器/金额/模式冲突）。

`--wait` 报告 `stage: "failed"` 时退出码同样是 `0`：退出码反映的是命令本身，而不是链上的结果。参见[脚本安全](../../machine-interface.md#script-safety-never-mistake-submitted-for-confirmed)。

## 另请参见

[`tx status`](status.md) · [`tx broadcast`](broadcast.md) · [费用与资源](../../concepts/networks.md#fees-the-tron-resource-model) · [脚本安全](../../machine-interface.md#script-safety-never-mistake-submitted-for-confirmed)

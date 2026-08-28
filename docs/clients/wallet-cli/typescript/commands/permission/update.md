# wallet-cli permission update

替换账户的权限结构。

## 用法

```
wallet-cli permission update (--file <path> | --json <str>)
                             [--dry-run | (--sign-only | --build-only) [--expiration <ms>] | --wait [--wait-timeout <ms>]]
                             [--permission-id <n>] [options]
```

## 说明

用 `--file`（一个 JSON 文件）或 `--json`（内联 JSON 字符串）给出的新结构，替换账户**整个**权限结构——TRON 的 `UpdateAccountPermission` 采用替换语义，因此你提供的 JSON 就是全部结构。链上会为这次变更燃烧 **100 TRX**。

本命令不会弹出确认提示。它需要一个账户，以及通过 `--password-stdin` 传入的 master password；仅观察账户会以 `watch_only_no_signer` 失败。

**输入格式。** 权限 JSON 与 [`permission show -o json`](show.md) 的 `data` 结构相同（`owner` / `witness` / `actives`；密钥中的 `local` 字段可以省略）。每个 active 组的 `operations` 要写**合约类型名**，而不是原始位图——位图由 CLI 编码生成。要得到一份合法输入，最方便的办法是导出当前结构、编辑后再把文件提交回去。

CLI 会在构建交易前严格校验权限结构，不符合规则时返回用法错误（退出码 `2`），不会自动修正输入。
权限组的 `threshold` **不得超过各密钥权重之和**；无法达到的阈值会导致账户无法授权，因此会以
`invalid_permission` 被拒绝（`owner.threshold exceeds the total key weight`）。阈值和权重采用无损解析，
超出安全整数范围的值会被拒绝，不会进行舍入。

**编辑导出的结构。** `permission show -o json` 会为每个 active 组同时输出 `operations`（合约类型名）和 `operationsHex`（原始位图）。两者可以都提供，但必须**一致**——同一个组出现两种互相矛盾的描述，就意味着你审阅过的结构并不是最终上链的结构，因此这种不一致会被拒绝。改完 `operations` 之后，请删掉该组的 `operationsHex`，CLI 会重新生成它：

```bash
wallet-cli permission show -o json --network tron:nile | jq '.data' > perms.json
# 编辑 operations，然后从同一个 active 组中删掉已失效的 operationsHex
```

只改 `keys`、`threshold` 或 `name` 则无需删除它。

⚠️ **链上不做任何安全检查。** 即使新结构里没有任何你能签名的密钥，交易照样会成功，账户从此被永久锁死，链上无法补救。本 CLI 会给出两类**本地**警告，但**不会**阻止提交（在 JSON 中它们进入 `meta.warnings`，`success` 仍为 `true`）：

- **锁死风险**——当你本地可签名的 owner 密钥（软件账户 / Ledger）权重之和低于新的 owner 阈值时，会输出一行 `!`，说明你已无法独自满足 owner 阈值（完全没有权重时为 `owner_lockout`，从此需要联署人时为 `owner_lockout_partial`）。多方托管本来就意味着"我一个人达不到阈值"，所以这只是提示，不是拦截。
- **危险操作**——当某个 active 组包含 `Update Account Permissions` 时（该组因此能够修改权限本身，实际等同于 owner 级别），会输出一行 `!` 予以标记（`active_can_update_permission`）。

## 选项

| 选项 | 说明 |
|---|---|
| `--file <path>` | **必填**（二选一）。含新结构的 JSON 文件（结构同 `permission show -o json` 的 data）；替换全部内容 |
| `--json <string>` | **必填**（二选一）。含新结构的内联 JSON 字符串（结构相同） |
| `--dry-run` | 模拟回执——费用、变更后的结构卡片和各项警告——与真实提交一致；不签名、不广播、不需要密码。与 `--sign-only` / `--build-only` 互斥 |
| `--sign-only` | 构建并签名，输出已签名的 hex 而不广播（交给 [`tx broadcast`](../tx/broadcast.md) 做链上联署）。与 `--dry-run` / `--build-only` 互斥；配合 `--expiration` 使用 |
| `--build-only` | 只构建，输出**未签名**的 hex（交给 [`tx multisig --create`](../tx/multisig.md) 走服务中转的多签流程）。与 `--dry-run` / `--sign-only` 互斥；配合 `--expiration` 使用 |
| `--expiration <ms>` | 交易过期时间（毫秒），最大 `86400000`（24 小时）；仅可与 `--sign-only` 或 `--build-only` 同用；省略时使用节点默认值（约 60 秒） |
| `--permission-id <n>` | 用于签名的权限组（0=owner，1=witness，2-9=active）——修改权限属于 owner 级操作，因此通常为 `0`（默认 `0`） |
| `--wait` / `--wait-timeout <ms>` | 广播后轮询直到已确认/失败（上限默认取配置 `waitTimeoutMs`，内置 60000） |
| `--password-stdin` | 从 stdin 读取 master password |

此外还有[全局选项](../index.md#global-options-every-command)。

## 示例

示例中的 `$PW` 是你的 master password，通过 `--password-stdin` 从 stdin 传入。

先导出再编辑，以此准备新结构（无需手写操作位图）：

```bash
wallet-cli permission show --network tron:nile -o json | jq '.data' > perms.json
```

```bash
# 编辑 perms.json——例如把 owner 组改成 2-of-3
$EDITOR perms.json
```

用 `--wait` 提交。回执由交易记录，加上变更后的链上结构（确认后重新读回，卡片样式与 `permission show` 相同）组成，末尾附上所有 `!` 警告：

```bash
echo "$PW" | wallet-cli permission update --file perms.json --network tron:nile --wait --password-stdin
```

```console
✅ Permissions updated
  Account  main (TQkXm4vN8pR2sD6fWbYc3LhJa9Ee5Zt7Uw)
  TxID     b3c...
  Block    #84,335,102
  Fee      100.268 TRX
  Status   success

Permission Name   owner  (id 0)
Threshold         2
Authorized To     Address                             Weight
                  TQkXm4vN8pR2sD6fWbYc3LhJa9Ee5Zt7Uw  1      (this wallet: main)
                  TBy6mQ7Y3nJ8sD2fWpXk4LhVc9Ra1Zt5Ub  1
                  TXe4Kd8nP2rF9gH5jL3mV6cW1bN7yS0aQz  1

Permission Name   finance  (id 2, active)
Operation(s)      Transfer TRX · Transfer TRC10 · Trigger Smart Contract
Threshold         2
Authorized To     Address                             Weight
                  TQkXm4vN8pR2sD6fWbYc3LhJa9Ee5Zt7Uw  1      (this wallet: main)
                  TBy6mQ7Y3nJ8sD2fWpXk4LhVc9Ra1Zt5Ub  1
                  TXe4Kd8nP2rF9gH5jL3mV6cW1bN7yS0aQz  1

! Your local keys now hold 1 of 2 owner weight — co-signers are required
  for owner-level operations from now on.
```

JSON 回执中的 `data.permissions` 与 `permission show` 的 `data` **结构完全相同**，因此可以与变更前的
导出结果进行 diff；账户锁定警告位于 `meta.warnings`，而 `success` 仍为 `true`：

```bash
echo "$PW" | wallet-cli permission update --file perms.json --network tron:nile --wait --password-stdin -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"permission.update","data":{"kind":"permission-update","stage":"confirmed","txId":"b3c...","confirmed":true,"blockNumber":84335102,"feeSun":100268000,"failed":false,"permissions":{"address":"TQkXm4vN8pR2sD6fWbYc3LhJa9Ee5Zt7Uw","owner":{"id":0,"threshold":2,"keys":[{"address":"TQkXm4vN8pR2sD6fWbYc3LhJa9Ee5Zt7Uw","weight":1,"local":"main"},{"address":"TBy6mQ7Y3nJ8sD2fWpXk4LhVc9Ra1Zt5Ub","weight":1,"local":null},{"address":"TXe4Kd8nP2rF9gH5jL3mV6cW1bN7yS0aQz","weight":1,"local":null}]},"witness":null,"actives":[{"id":2,"name":"finance","threshold":2,"operations":["TransferContract","TransferAssetContract","TriggerSmartContract"],"operationsHex":"0600008000000000000000000000000000000000000000000000000000000000","keys":[{"address":"TQkXm4vN8pR2sD6fWbYc3LhJa9Ee5Zt7Uw","weight":1,"local":"main"},{"address":"TBy6mQ7Y3nJ8sD2fWpXk4LhVc9Ra1Zt5Ub","weight":1,"local":null},{"address":"TXe4Kd8nP2rF9gH5jL3mV6cW1bN7yS0aQz","weight":1,"local":null}]}]}},"meta":{"durationMs":6810,"warnings":[{"code":"owner_lockout_partial","message":"local keys hold 1 of 2 owner weight; co-signers are required for owner-level operations"}]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出

`data` 随模式而变：

| 模式 | 字段 |
|---|---|
| 默认（提交） | `kind: "permission-update"`、`stage: "submitted"`、`txId` |
| `--wait`（已确认） | 同上，但 `stage: "confirmed"`，并另加 `confirmed`、`blockNumber`、`feeSun`、`failed`，以及 `permissions`（结构同 `permission show` 的 data，从链上读回） |
| `--dry-run` | `kind`、`mode: "dry-run"`、`fee`（100 TRX 的变更费用），以及 `permissions`（变更后的结构）；没有 `txId` |
| `--sign-only` | `kind`、`mode: "sign-only"`、`hex`（已签名的交易 hex——交给 `tx broadcast --hex`）、`fee` |
| `--build-only` | `kind`、`mode: "build-only"`、`hex`（未签名的交易 hex——交给 `tx multisig --create`）、`fee` |

本地警告（`owner_lockout`、`owner_lockout_partial`、`active_can_update_permission`）在交易构建之前产生，以 `{code, message}` 对象的形式出现在 `meta.warnings` 中，且不影响 `success`——参见[读取 `meta.warnings`](../../machine-interface.md#reading-metawarnings)。

## 退出码

`0` 已提交（早退模式下为已构建/已签名/试运行） · `1` 执行失败（`invalid_permission`、`not_authorized`、`watch_only_no_signer`、`auth_failed`、`insufficient_balance`、`rpc_error`、`timeout`） · `2` 用法错误（`invalid_value`）。

在多签账户上，若累计签名权重低于权限阈值，提交会在**签名之后、广播之前**被拒绝，报 `not_authorized`（`signature threshold is not reached; missing N weight`）——不会发出任何内容，也不会燃烧费用。此时请改用 `--sign-only` + [`tx sign`](../tx/sign.md) 收齐剩余签名，再用 [`tx broadcast`](../tx/broadcast.md) 提交。`--sign-only` 和 `--build-only` 仍会返回部分签名的交易，联署流程正是由此开始的。

## 另请参见

[`permission show`](show.md) · [`tx sign`](../tx/sign.md) · [`tx broadcast`](../tx/broadcast.md) · [`tx multisig`](../tx/multisig.md) · [安全](../../concepts/security.md)

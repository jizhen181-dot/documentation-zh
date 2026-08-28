# wallet-cli contract set-origin-energy-limit

设置部署者为每次调用承担的能量上限。

## 用法

```
wallet-cli contract set-origin-energy-limit <address> <energy>
                                            [--dry-run | (--sign-only | --build-only) [--expiration <ms>] | --wait [--wait-timeout <ms>]]
                                            [--permission-id <n>] [options]
```

## 说明

设置 `origin_energy_limit`——即**部署者**愿意为对该合约的单次调用支付的能量上限。

它既不是对合约的限制，也不是对调用方的限制。部署者实际承担的量同时受三个因素约束：这个上限、部署者自己质押得到的能量，以及 [`contract set-user-resource-percent`](set-user-resource-percent.md) 设定的调用方/部署者分摊比例。部署者一侧覆盖不了的部分，都会落回调用方。有两种情况会让这个设置形同虚设：部署者没有质押能量（无论上限设成多少，补贴都是 0），或者用户承担比例是 100 %（部署者那一份为 0，这个上限根本用不上）。

`<energy>` 必须是**大于零**的整数——链会拒绝 0，因此本地就会拦下来，不会广播出去。

只有合约的部署者才能执行此操作；当前值见 [`contract info`](info.md)。设置在交易确认后立即生效。

**该命令默认在提交时返回**（`stage: "submitted"`），而不是确认时——加 `--wait` 可阻塞直到已确认/失败。需要一个账户。只有会签名的模式才需要 master password（通过 `--password-stdin`）——`--dry-run` 和 `--build-only` 不会解锁钱包，无需密码即可运行。在签名模式下，仅观察账户会以 `watch_only_no_signer` 失败。

## 选项

| 选项 | 说明 |
|---|---|
| `<address>` | **必填。** 要配置的合约；你必须是它的部署者 |
| `<energy>` | **必填。** 部署者为每次调用承担的能量，整数且 > 0 |
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
echo "$PW" | wallet-cli contract set-origin-energy-limit TQ5nJ8mV...4wRe 50000000 --network tron:nile --wait --password-stdin
```

```console
✅ Origin energy limit set
  Contract      TQ5nJ8mV...4wRe
  Deployer      TQkXm4vN...5Zt7Uw (main)
  Energy limit  50,000,000
  TxID          3a9...
  Block         57,882,265
  Fee           0 TRX  (290 bandwidth)
  Status        success
```

```bash
echo "$PW" | wallet-cli contract set-origin-energy-limit TQ5nJ8mV...4wRe 50000000 --network tron:nile --wait --password-stdin -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"contract.set-origin-energy-limit","data":{"kind":"contract-set-origin-energy-limit","stage":"confirmed","txId":"3a9...","confirmed":true,"blockNumber":57882265,"failed":false,"contractAddress":"TQ5nJ8mV...","deployerAddress":"TQkXm4vN...","originEnergyLimit":50000000,"feeSun":0,"resource":{"netUsage":290,"netFeeSun":0,"energyUsage":0,"energyFeeSun":0}},"meta":{"durationMs":6530,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出

`data` 随阶段而变：

| 阶段 | 字段 |
|---|---|
| 默认（提交） | `kind: "contract-set-origin-energy-limit"`、`stage: "submitted"`、`txId`、`contractAddress`、`deployerAddress`、`originEnergyLimit` |
| `--wait`（已确认） | 同上，另加 `stage: "confirmed"`、`confirmed`（boolean）、`blockNumber`、`feeSun`、`resource`、`failed` |

`originEnergyLimit` 即当前生效的值。

## 退出码

`0` 已提交（早退模式下为已构建/已签名） · `1` 执行失败（`contract_not_found`——没有这个合约、`not_contract_deployer`、`watch_only_no_signer`、`auth_failed`） · `2` 用法错误（`invalid_value`——地址格式非法，或能量不是大于 0 的整数）。

## 另请参见

[`contract set-user-resource-percent`](set-user-resource-percent.md) · [`contract info`](info.md) · [能量与带宽](../../concepts/energy-bandwidth.md) · [脚本安全](../../machine-interface.md#script-safety-never-mistake-submitted-for-confirmed)

# wallet-cli exchange inject

按交易对的储备比例向其注资。

## 用法

```
wallet-cli exchange inject <id> --token <TRX|asset-id>
                           (--amount <n> | --raw-amount <n>)
                           [--dry-run | (--sign-only | --build-only) [--expiration <ms>] | --wait [--wait-timeout <ms>]]
                           [--permission-id <n>] [options]
```

## 说明

**注资是双边的。** 你只指定其中一侧及其金额；链会按当前储备比例算出另一侧，并把它一并扣除。因此在一个持有 10,000 TRX 和 500,000 个 token 的交易对上执行 `--token TRX --amount 1000`，还会同时扣走 50,000 个 token——你需要**两侧都有足够余额**，而不只是你指定的那一侧。

只有交易对的创建者才能注资；其他任何账户都会以 `not_exchange_creator` 失败。

如果金额小到算出的另一侧取整后为零，链会拒绝这笔交易。这种情况会在本地对照当前储备提前拦下，不会广播出去。

**只用 id 指代 token**——`TRX`（或它的链上 id `_`）以及数字形式的 TRC10 id；TRC10 名称可能含有 `:`。`--amount` 以你所指定那一侧的完整 token 计，并按该侧的精度换算；`--raw-amount` 则以最小单位给出同样的数字。两者必须且只能给其中一个。

**该命令默认在提交时返回**（`stage: "submitted"`），而不是确认时——加 `--wait` 可阻塞直到已确认/失败。需要一个账户。只有会签名的模式才需要 master password（通过 `--password-stdin`）——`--dry-run` 和 `--build-only` 不会解锁钱包，无需密码即可运行。在签名模式下，仅观察账户会以 `watch_only_no_signer` 失败。

## 选项

| 选项 | 说明 |
|---|---|
| `<id>` | **必填。** 交易对 id |
| `--token <TRX\|asset-id>` | **必填。** 你要指定的那一侧 |
| `--amount <n>` | 该侧的金额，以完整 token 计；另一侧按储备比例跟随。`--amount` / `--raw-amount` 二选一 |
| `--raw-amount <n>` | 同一金额，以最小单位计。`--amount` / `--raw-amount` 二选一 |
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
echo "$PW" | wallet-cli exchange inject 12 --token TRX --amount 1000 --network tron:nile --wait --password-stdin
```

```console
✅ Liquidity injected
  Exchange id  12
  Creator      TQkXm4vN...5Zt7Uw
  Injected     1,000 TRX / 50,000 MyToken
  Reserves     11,000 TRX / 550,000 MyToken
  TxID         5c3...
  Block        #57,884,180
  Fee          0 TRX
  Status       success
```

```bash
echo "$PW" | wallet-cli exchange inject 12 --token TRX --amount 1000 --network tron:nile --wait --password-stdin -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"exchange.inject","data":{"kind":"exchange-inject","stage":"confirmed","txId":"5c3...","confirmed":true,"blockNumber":57884180,"failed":false,"exchangeId":12,"pair":"TRX:1000123","creatorAddress":"TQkXm4vN...","tokenId":"_","tokenQuant":"1000000000","tokenLabel":"TRX","tokenDecimals":6,"otherTokenId":"1000123","otherTokenQuant":"50000000000","otherTokenLabel":"MyToken","otherTokenDecimals":6,"reserveAfter":"11000000000","otherReserveAfter":"550000000000","feeSun":0},"meta":{"durationMs":6440,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出 {#output}

`data` 是扁平结构——你指定的那一侧、随之跟随的另一侧，以及此后两侧的储备：

| 字段 | 类型 | 含义 |
|---|---|---|
| `exchangeId` / `pair` / `creatorAddress` | number / string / string | 交易对及其创建者 |
| `tokenId` / `tokenQuant` | string | 你指定的那一侧，以及从中扣除的金额，以最小单位计 |
| `tokenLabel` / `tokenDecimals` | string / number | text 输出把该侧按完整 token 显示时所用的信息 |
| `otherTokenId` / `otherTokenQuant` / `otherTokenLabel` / `otherTokenDecimals` | — | 按比例算出的另一侧对应的同样四个字段 |
| `reserveAfter` / `otherReserveAfter` | string | 本次注资之后交易对两侧的余额，顺序与上面一致 |

TRX 以 `"_"` 标识；所有数量都是最小单位下的**字符串**。确认之前，另一侧金额和两侧储备来自本命令自己的精确计算；一旦确认，就由交易回执中的数字取而代之。`--wait` 会另加 `stage: "confirmed"`、`confirmed`、`blockNumber`、`feeSun`、`failed`。

## 退出码

`0` 已提交（早退模式下为已构建/已签名） · `1` 执行失败（`exchange_not_found`——没有这个交易对、`not_exchange_creator`、`token_not_in_exchange`、`exchange_closed`——某一侧储备为零、`transaction_rejected`——节点拒绝了它，例如余额不足、`watch_only_no_signer`、`auth_failed`） · `2` 用法错误（`missing_option`——没给 `--token`；`invalid_option`——`--amount` / `--raw-amount` 两个都给了或都没给；`invalid_amount`——金额不是十进制数字，或小数位数超过该 token 允许的位数；`invalid_value`——金额 ≤ 0，或小到算出的另一侧结果为零）。

## 另请参见

[`exchange withdraw`](withdraw.md) · [`exchange show`](show.md) · [`exchange create`](create.md) · [脚本安全](../../machine-interface.md#script-safety-never-mistake-submitted-for-confirmed)

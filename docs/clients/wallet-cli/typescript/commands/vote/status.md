# wallet-cli vote status

你当前的投票、投票权与奖励概览。

## 用法

```
wallet-cli vote status [options]
```

## 说明

把「质押 → 投票 → 奖励」这条链路汇总到一屏只读输出中：你当前的票数分配（含各 SR 的 APR 和分成比例）、你的投票权（TP 总量 / 已用 / 可用），以及当前可领取的奖励。

- **投票权（TP）**——total = 已质押的 TRX；used = 已投出的票数；available = total − used。
- **APR / 分成比例**——语义和数据来源都与 [`vote list`](list.md) 相同。值得时常复查：SR 随时可以修改自己的比例（链上的 UpdateBrokerage）——当初按 80% 投出的票，一旦比例降到 0%，就会悄无声息地不再产生收益。
- **0% 警告**——如果有票投在分成比例为 0% 的 SR 上，文本输出会追加一行 `!` 提示，json 则在 `meta.warnings` 中加入一条纯字符串，每个受影响的 SR 各一条。
- **可领取奖励**——数据来源与 [`reward balance`](../reward/balance.md) 相同；用 [`reward withdraw`](../reward/withdraw.md) 领取。

## 选项

没有命令级选项；仅[全局选项](../index.md#global-options-every-command)（`--network` / `--account`）。

## 示例

```bash
wallet-cli vote status --account main --network tron:nile
```

```console
Label           main
Voting power    1,500 TP  (used 1,000 / available 500)
Claimable       12.345678 TRX

Current votes (2)
| Name            | Votes | APR  | Reward ratio | Address                            |
| --------------- | ----- | ---- | ------------ | ---------------------------------- |
| TRONSCAN        | 600   | 4.8% | 80%          | TZ4UXDV5ZhNW7fb2AMSbgfAEZ7hWsnYS2g |
| Binance Staking | 400   | 0%   | 0%           | TT5W8MPbYJih9R586kTszb4LoybzUvCYm2 |
! 400 votes on Binance Staking earn nothing — 0% reward ratio
```

```bash
wallet-cli vote status --account main --network tron:nile -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"vote.status","data":{"address":"TQk...","votingPower":{"total":1500,"used":1000,"available":500},"claimableRewardSun":"12345678","votes":[{"witness":"TZ4...","name":"TRONSCAN","count":600,"rewardRatioPct":80,"brokeragePct":20,"aprPct":4.8},{"witness":"TT5...","name":"Binance Staking","count":400,"rewardRatioPct":0,"brokeragePct":100,"aprPct":0}]},"meta":{"durationMs":16,"warnings":["400 votes on TT5... (Binance Staking) earn nothing: reward ratio is 0%"]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出

| 字段 | 类型 | 含义 |
|---|---|---|
| `address` | string | 被查询的账户 |
| `votingPower.total` / `.used` / `.available` | number | TP 总量 / 已占用 / 可用 |
| `claimableRewardSun` | string | 当前可领取的奖励，单位 SUN |
| `votes[]` | array | 当前的票数分配：`witness`、`name`、`count`、`rewardRatioPct`、`brokeragePct`、`aprPct` |

分成比例为零的警告以纯字符串形式出现在 `meta.warnings` 中——参见[读取 `meta.warnings`](../../machine-interface.md#reading-metawarnings)。

## 退出码

`0` 成功 · `1` 执行失败（`rpc_error`） · `2` 用法错误（`invalid_value`）。

## 另请参见

[`vote cast`](cast.md) · [`vote list`](list.md) · [`reward balance`](../reward/balance.md) · [`stake info`](../stake/info.md)

# wallet-cli vote list

列出超级代表和候选人。

## 用法

```
wallet-cli vote list [--limit <n>] [--candidates] [options]
```

## 说明

列出 SR（默认是当选的 27 位）及其得票数、预估 APR 和分成比例——这些正是执行 [`vote cast`](cast.md) 之前需要看的数字。只读，不需要账户。

各列的含义：

- **APR**——投票者的预估年化收益，已按该 SR 的分成比例折算过。**尽力而为**：它不来自链上 RPC，而是取自区块浏览器/TronGrid 的数据；取不到时该列显示 `—`（json 中为 `null`）。
- **Reward ratio（奖励分配比例）**——SR 分配给投票者的奖励占比，数据来自链上。80% 表示投票者获得
  奖励的 80%；**0% 表示投票者不会获得奖励**。JSON 中还提供链上原始的 `brokeragePct`
 （= 100 − rewardRatioPct）。
- **排名与获奖资格**——第 1–27 名是当选的 SR（出块奖励 + 投票奖励）；第 28–127 名是合伙人（只有投票奖励）；127 名之外的候选人没有任何奖励，因此 `--limit` 的上限就是 127。

## 选项

| 选项 | 说明 |
|---|---|
| `--limit <number>` | 最多返回多少个名次（默认 27，最大 127）。默认只列出当选的 27 位 SR，所以除非同时加上 `--candidates`，调大这个值不会有任何效果 |
| `--candidates` | 同时列出未当选的候选人（第 28 名及之后），此时 `--limit` 才能超过 27、最多到 127 |

此外还有[全局选项](../index.md#global-options-every-command)。

## 示例

```bash
wallet-cli vote list --limit 3 --network tron:nile
```

```console
| Rank | Name            | Votes         | APR  | Reward ratio | Address                            |
| ---- | --------------- | ------------- | ---- | ------------ | ---------------------------------- |
| 1    | TRONSCAN        | 1,203,456,789 | 4.8% | 80%          | TZ4UXDV5ZhNW7fb2AMSbgfAEZ7hWsnYS2g |
| 2    | Binance Staking | 998,765,432   | 0%   | 0%           | TT5W8MPbYJih9R586kTszb4LoybzUvCYm2 |
| 3    | JustLend        | 876,543,210   | 4.9% | 80%          | TWxkzUeAiKcFvzXvJEcaTQCQqCuMednAtN |
```

```bash
wallet-cli vote list --limit 3 --network tron:nile -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"vote.list","data":{"witnesses":[{"rank":1,"name":"TRONSCAN","address":"TZ4UXDV5ZhNW7fb2AMSbgfAEZ7hWsnYS2g","voteCount":"1203456789","rewardRatioPct":80,"brokeragePct":20,"aprPct":4.8}]},"meta":{"durationMs":40,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出

`data.witnesses[]`——每个名次一条：

| 字段 | 类型 | 含义 |
|---|---|---|
| `rank` | number | 按得票数排的名次（1 = 得票最多） |
| `name` | string | SR 名称 |
| `address` | string | SR base58 address |
| `voteCount` | string | 总得票数，原始整数 |
| `rewardRatioPct` | number | 分给投票者的奖励百分比（来自链上） |
| `brokeragePct` | number | SR 自留的佣金比例（= 100 − `rewardRatioPct`） |
| `aprPct` | number \| null | 投票者的预估 APR；估算数据源不可用时为 `null` |

## 退出码

`0` 成功 · `1` 执行失败（`rpc_error`） · `2` 用法错误（`invalid_value`——limit 超出取值范围）。

## 另请参见

[`vote cast`](cast.md) · [`vote status`](status.md)

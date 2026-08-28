# wallet-cli proposal show

显示单个提案、它要设置的参数，以及它的批准进度。

## 用法

```
wallet-cli proposal show <id> [options]
```

## 说明

报告单个提案：它设置的每个参数都以 `name  value` 的形式连同单位一起列出，批准数与阈值的对比，以及创建时间和到期时间。只读，无需账户。

**这里显示的值是提案将要设置的值，不是当前生效的值。** 链上不记录提案创建时该参数原本是多少。对已结算的提案来说，当前值与那个基线毫无关系；而对已通过的提案，当前值*就是*该提案装上去的值。当前生效的值请用 [`chain params`](../chain/params.md) 查看。

文本输出只显示批准数。其背后的地址在 `json` 输出的 `approvedBy[]` 里，且是完整长度。

`State` 取自链上自身的取值，所以被创建者删除的提案在这里显示为 `canceled`。

## 选项

| 选项 | 说明 |
|---|---|
| `<id>` | **必填。** 提案 id |

此外还有[全局选项](../index.md#global-options-every-command)。

## 示例

一个仍在投票窗口内的提案：

```bash
wallet-cli proposal show 47 --network tron:nile
```

```console
Proposal #47
  State         voting
  Proposer      TSRmq8kP...9dEf
  Created time  2026-07-21 08:00 UTC
  Expiry time   2026-07-22 08:00 UTC
  Approvals     12 / 18
  Parameters    (1)
    getTransactionFee   15   sun/byte
```

一个在到期时达到阈值的提案——该值从那次统计起就已生效：

```bash
wallet-cli proposal show 45 --network tron:nile
```

```console
Proposal #45
  State         approved
  Proposer      TSRwd3nL...8vC
  Created time  2026-07-20 08:00 UTC
  Expiry time   2026-07-21 08:00 UTC
  Approvals     18 / 18
  Parameters    (1)
    getEnergyFee   140   sun
```

一个到期时未达阈值、且携带两个参数的提案：

```bash
wallet-cli proposal show 44 --network tron:nile
```

```console
Proposal #44
  State         disapproved
  Proposer      TSRee5...2xB
  Created time  2026-07-19 08:00 UTC
  Expiry time   2026-07-20 08:00 UTC
  Approvals     8 / 18
  Parameters    (2)
    getMaintenanceTimeInterval   10800000   ms
    getMaxCpuTimeOfOneTx               80   ms
```

```bash
wallet-cli proposal show 47 --network tron:nile -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"proposal.show","data":{"id":47,"proposerAddress":"TSRmq8kP...","state":"voting","createTime":1784620800000,"expirationTime":1784707200000,"approvals":12,"approvalThreshold":18,"reachedThreshold":false,"parameters":[{"id":3,"name":"getTransactionFee","value":15,"unit":"sun/byte"}],"approvedBy":["TSRaa1...","TSRbb2..."]},"meta":{"durationMs":22,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出

| 字段 | 类型 | 含义 |
|---|---|---|
| `id` | number | 提案 id |
| `proposerAddress` | string | 创建者，base58 |
| `state` | string | `voting` / `approved` / `disapproved` / `canceled` |
| `createTime` / `expirationTime` | number | 创建时间与投票窗口结束时间，epoch 以来的毫秒数 |
| `approvals` / `approvalThreshold` | number | 已投出的批准数，以及通过所需的数量 |
| `reachedThreshold` | boolean | `approvals` 是否已经达到 `approvalThreshold` |
| `parameters[]` | array | `id`、`name`、`value`（提案要设置的值）、`unit`；按 `id` 排序 |
| `approvedBy[]` | array | 已投出批准的地址，base58 （仅 `json`） |

没有取消时间戳：链上的提案记录只包含上述字段，因此 `canceled` 本身不带任何时间。

## 退出码

`0` 成功 · `1` 执行失败（`proposal_not_found`——没有该提案、`rpc_error`） · `2` 用法错误（`invalid_value`——id 不是数字）。

## 另请参见

[`proposal list`](list.md) · [`proposal approve`](approve.md) · [`chain params`](../chain/params.md)

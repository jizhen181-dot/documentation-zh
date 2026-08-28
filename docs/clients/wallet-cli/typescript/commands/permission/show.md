# wallet-cli permission show

显示账户的权限结构。

## 用法

```
wallet-cli permission show [options]
```

## 说明

以只读方式查看账户的各个权限组，包括 owner、witness（仅超级代表拥有）以及最多 8 个 active 组。
每组都会显示阈值和密钥（地址及权重），active 组还会显示解码后的可执行操作列表。建议在执行
`permission update` 前查看当前结构，也可以在签名前检查联署人的权限配置。

默认读取当前账户；`--account` 可覆盖它，并且也接受直接传入地址，因此你可以查看链上任意账户。

读懂输出——text 版式对应 TronScan 的权限页面，每个权限组一张"标签 / 取值"卡片：

- **Permission Name**——链上的 `permission_name` 及其 id（active 组会标注 `active`）。名称是在 [`permission update`](update.md) 创建该组时指定的；它只是一个助记名，在链上没有任何含义。从未修改过的账户会显示链上默认结构：一个 `owner` 组和一个 `active` 组，各自以账户自身地址作为唯一密钥，阈值为 `1`。
- **Operation(s)**——仅 active 组有。在链上这是一张 32 字节的位图（每个合约类型占一位）；text 会把它解码成人类可读的操作名（`Transfer TRX`、`Vote`……）并给出总数。这套标签与 TronScan 权限页面所显示的一致。JSON 则在 `operations` 中保留机器可读的合约类型名，并附带原始的 `operationsHex`。
- **Threshold**——一笔交易要对该权限组有效所需的签名权重之和。
- **Authorized To**——该组的密钥，以 `Address / Weight` 形式列出。本地钱包持有的软件账户或 Ledger
  密钥会标注 `(this wallet: <label>)`，便于确认本地可用的签名权重；这也是
  [`permission update`](update.md) 判断账户锁定风险的依据。

## 选项

没有本命令特有的选项；只有[全局选项](../index.md#global-options-every-command)（`--network`，以及 `--account`——它同样接受直接传入地址）。

## 示例

**从未修改过的账户**显示的是链上默认结构——active 组涵盖了所有普通操作类型：

```bash
wallet-cli permission show --account solo --network tron:nile
```

```console
Account  solo (TWfd2K9nP4rH7gL3jM6cV1bN8yS5aQ0eXt)

Permission Name   owner  (id 0)
Threshold         1
Authorized To     Address                             Weight
                  TWfd2K9nP4rH7gL3jM6cV1bN8yS5aQ0eXt  1      (this wallet: solo)

Permission Name   active  (id 2, active)
Operation(s)      Activate Account · Transfer TRX · Transfer TRC10
                  Vote · Issue TRC10 · Update Account Name
                  TRX Stake (1.0) · TRX Unstake (1.0)
                  Claim Voting Rewards · Create Smart Contract
                  Trigger Smart Contract · TRX Stake (2.0)
                  TRX Unstake (2.0) · Withdraw Unstaked TRX
                  Delegate Resources · Reclaim Resources
                  Cancel Unstake · …  (40 total)
Threshold         1
Authorized To     Address                             Weight
                  TWfd2K9nP4rH7gL3jM6cV1bN8yS5aQ0eXt  1      (this wallet: solo)
```

**多签账户**——这里 owner 组是 2-of-3，另有一个限定范围的 `finance` active 组负责日常转账。本钱包只持有其中一个密钥（`main`）；另外两个由外部联署人持有，因此没有标注：

```bash
wallet-cli permission show --account main --network tron:nile
```

```console
Account  main (TQkXm4vN8pR2sD6fWbYc3LhJa9Ee5Zt7Uw)

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
```

```bash
wallet-cli permission show --account main --network tron:nile -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"permission.show","data":{"address":"TQkXm4vN8pR2sD6fWbYc3LhJa9Ee5Zt7Uw","owner":{"id":0,"threshold":2,"keys":[{"address":"TQkXm4vN8pR2sD6fWbYc3LhJa9Ee5Zt7Uw","weight":1,"local":"main"},{"address":"TBy6mQ7Y3nJ8sD2fWpXk4LhVc9Ra1Zt5Ub","weight":1,"local":null},{"address":"TXe4Kd8nP2rF9gH5jL3mV6cW1bN7yS0aQz","weight":1,"local":null}]},"witness":null,"actives":[{"id":2,"name":"finance","threshold":2,"operations":["TransferContract","TransferAssetContract","TriggerSmartContract"],"operationsHex":"0600008000000000000000000000000000000000000000000000000000000000","keys":[{"address":"TQkXm4vN8pR2sD6fWbYc3LhJa9Ee5Zt7Uw","weight":1,"local":"main"},{"address":"TBy6mQ7Y3nJ8sD2fWpXk4LhVc9Ra1Zt5Ub","weight":1,"local":null},{"address":"TXe4Kd8nP2rF9gH5jL3mV6cW1bN7yS0aQz","weight":1,"local":null}]}]},"meta":{"durationMs":21,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出

| 字段 | 类型 | 含义 |
|---|---|---|
| `address` | string | 被查询的账户 |
| `owner` | object | owner 组 `{id, threshold, keys[]}` |
| `witness` | object \| null | witness 组（仅超级代表有），否则为 `null` |
| `actives[]` | array | active 组，每项为 `{id, name, threshold, operations[], operationsHex, keys[]}` |
| `…operations[]` | string[] | 该 active 组可执行的合约类型名 |
| `…operationsHex` | string | 原始的 32 字节操作位图，hex |
| `…keys[]` | array | 组内的密钥：`{address, weight, local}`——若由本地持有，`local` 为钱包标签，否则为 `null` |

## 退出码

`0` 成功 · `1` 执行失败（`rpc_error`） · `2` 用法错误（`invalid_value`；地址未激活 / 链上不存在时为 `not_found`）。

## 另请参见

[`permission update`](update.md) · [`tx sign`](../tx/sign.md) · [`tx approvals`](../tx/approvals.md) · [安全](../../concepts/security.md)

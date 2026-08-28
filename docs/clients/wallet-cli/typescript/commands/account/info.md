# wallet-cli account info

显示账户的链上原始数据，以及规范化后的带宽/能量资源摘要。

## 用法

```
wallet-cli account info [options]
```

## 说明

返回节点提供的完整账户对象，包括余额、权限和质押信息，并附带规范化的带宽与能量 `resources` 摘要。
可以通过该命令检查账户是否有足够资源，从而预估交易是否需要燃烧 TRX。

## 选项

仅[全局选项](../index.md#global-options-every-command)。

## 示例

```bash
wallet-cli account info --network tron:nile
```

```console
Label        main
Address      TMSgJxtPw29AFEHMXsjGo4kWV7UwbCToHJ
Balance      1969.421 TRX
Staked       12 TRX (energy 12 + bandwidth 0)
Energy       used 0 / 888
Bandwidth    used 374 / 600
Created      2026-06-30
Permissions  owner 1-of-1, 1 active group
```

```bash
wallet-cli account info --network tron:nile -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"account.info","data":{"address":"TMSgJxtPw29AFEHMXsjGo4kWV7UwbCToHJ","account":{"balance":"1976489000","create_time":1782787719000,"owner_permission":{…},"active_permission":[…],"frozenV2":[{},{"type":"ENERGY","amount":"12000000"},{"type":"TRON_POWER"}],…},"resources":{"bandwidth":{"used":0,"limit":600},"energy":{"used":0,"limit":888}}},"meta":{"durationMs":1914,"warnings":[]},"chain":{"family":"tron","network":"tron:nile","chainId":"nile"}}
```

## 输出

| 字段 | 类型 | 含义 |
|---|---|---|
| `address` | string | 被查询的 base58 地址 |
| `account` | object | 由 TRON 节点原样返回的账户对象：`balance`（SUN 字符串）、各类时间戳、`owner_permission` / `active_permission`（多签密钥与阈值）、`frozenV2`（按类型的质押金额）等；字段由节点决定，wallet-cli 不做重塑 |
| `resources.bandwidth` | object | `used` / `limit`（字节） |
| `resources.energy` | object | `used` / `limit` |

`resources` 块由 wallet-cli 规范化——稳定，可安全地编程依赖；`account` 由节点原样返回，其字段随节点/协议而变，不保证稳定。

## 退出码

`0` · `1` 执行失败 · `2` 用法错误。

## 另请参见

[`account balance`](balance.md) · `stake freeze`——获取资源 · [资源模型](../../concepts/networks.md#fees-the-tron-resource-model)

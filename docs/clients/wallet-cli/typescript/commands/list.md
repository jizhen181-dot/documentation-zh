# wallet-cli list

列出钱包/账户（无需解锁）。

## 用法

```
wallet-cli list [options]
```

## 说明

枚举本地存储的全部账户，涵盖所有种子钱包和导入的账户：HD 账户按 seed 分组，其余按类型分组（私钥 / 仅观察 / Ledger），并标出当前账户。只读取元数据——不需要 master password。

## 选项

仅[全局选项](index.md#global-options-every-command)。

## 示例

```bash
wallet-cli list
```

```console
HD  wlt_jj2vgz7m
├─ [0] main    TJxvjVUpQ2sVqW4WYN7iX96qWDLFoUU9NN  (active)
└─ [1] main-1  TJ4Pa3iF6ppS13RkeL8GHxNyaUfuyncqgS

private key
└─ hot         TMVQGm1qAQYVdetCeGRRkTWYYrLXuHK2HC

watch-only
└─ cold        TUEZSdKsoDHQMeZwihtdoBiN46zxhGWYdH
```

HD 账户按 seed 分组并带有 `[index]`；非 HD 条目（私钥 / 仅观察 / Ledger）按类型分组，没有 `[index]`。

```bash
wallet-cli list -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"list","data":[{"accountId":"wlt_jj2vgz7m.0","label":"main","type":"seed","index":0,"active":true,"addresses":{"tron":"TJxvjVUpQ2sVqW4WYN7iX96qWDLFoUU9NN"},"seedId":"wlt_jj2vgz7m"},{"accountId":"wlt_jj2vgz7m.1","label":"main-1","type":"seed","index":1,"active":false,"addresses":{"tron":"TJ4Pa3iF6ppS13RkeL8GHxNyaUfuyncqgS"},"seedId":"wlt_jj2vgz7m"},{"accountId":"wlt_w64e61jy","label":"hot","type":"privateKey","index":null,"active":false,"addresses":{"tron":"TMVQGm1qAQYVdetCeGRRkTWYYrLXuHK2HC"}},{"accountId":"wlt_bnd7sz5e","label":"cold","type":"watch","index":null,"active":false,"addresses":{"tron":"TUEZSdKsoDHQMeZwihtdoBiN46zxhGWYdH"},"family":"tron"}],"meta":{"durationMs":13,"warnings":[]}}
```

## 输出

`data` 是一个数组，每个账户对应一项：

| 字段 | 类型 | 含义 |
|---|---|---|
| `accountId` | string | 稳定 id；HD 账户为 `<seedId>.<index>`，非 HD 账户为独立的 `wlt_…` |
| `label` | string | 可读标签（用 `rename` 修改） |
| `type` | string | `seed`（HD）、`privateKey`、`watch`、`ledger` |
| `index` | number \| null | 在该 seed 内的 HD 派生索引；非 HD 账户为 `null` |
| `active` | boolean | 是否为各命令默认使用的账户 |
| `addresses.tron` | string | Base58 TRON 地址 |
| `seedId` | string | 所属种子钱包 id（仅 `seed` 账户） |
| `family` | string | 地址所属链系，例如 `tron`（仅 `watch` 账户） |

本地命令——没有 `chain` 块。

## 退出码

`0` 成功 · `2` 用法错误。参见 [machine-interface](../machine-interface.md#exit-codes)。

## 另请参见

`use` · `current` · [`create`](create.md) · [`account balance`](account/balance.md)

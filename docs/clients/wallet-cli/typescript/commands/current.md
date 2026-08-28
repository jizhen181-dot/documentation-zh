# wallet-cli current

显示当前（活跃）账户。

## 用法

```
wallet-cli current [options]
```

## 选项

| 选项 | 说明 |
|---|---|
| `--qr` | 额外在终端中把当前账户的地址渲染成可扫描的 QR 码，下方打印完整地址以便人工核对；仅对文本输出有效 |

此外还有[全局选项](index.md)（`--account` 可覆盖显示哪个账户）。

## 示例

```bash
wallet-cli current
```

```console
Active account: main-1
  TRON address  TRs9HgTuY3dT3yDasdFdP9WQHqL37891Ax
```

加上 `--qr` 会在地址下方用块状字符额外画出一个可扫描的收款 QR 码。这完全是本地操作——地址来自本地 keystore 元数据，不访问节点：

```bash
wallet-cli current --qr
```

```console
Active account: main-1
  TRON address  TRs9HgTuY3dT3yDasdFdP9WQHqL37891Ax

  [ scannable QR code of the address, drawn in the terminal ]
```

QR 码只是终端渲染的产物，在真实终端里（块状字符能正确对齐时）可以扫描；`--qr` 不会改变 `-o json` 的输出（程序消费方应取地址自行生成 QR 码）。如果终端宽度不足以容纳它，会降级为只打印地址并给出一条 `!` 提示。

```bash
wallet-cli current -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"current","data":{"accountId":"wlt_758891fa.1","label":"main-1","type":"seed","index":1,"active":true,"addresses":{"tron":"TRs9HgTuY3dT3yDasdFdP9WQHqL37891Ax"},"seedId":"wlt_758891fa"},"meta":{"durationMs":13,"warnings":[]}}
```

尚无当前账户时，命令会以 `missing_wallet_address` 失败（退出码 1）：

```bash
wallet-cli current
```

```console
error [missing_wallet_address]: no active account; import one first
```

## 输出

`data` 是当前活跃账户。本地命令——没有 `chain` 块。

| 字段 | 类型 | 含义 |
|---|---|---|
| `accountId` | string | 当前账户 id |
| `label` | string | 账户标签 |
| `type` | string | `seed` / `privateKey` / `watch` / `ledger` |
| `index` | number \| null | HD 派生索引；非 HD 账户为 `null` |
| `active` | boolean | 始终为 `true` |
| `addresses.tron` | string | Base58 TRON 地址 |
| `seedId` | string | 所属种子钱包 id（仅 `seed` 账户） |
| `family` | string | 链系，例如 `tron`（仅 `watch` 账户） |

## 退出码

`0` 成功 · `1` 执行失败 · `2` 用法错误。参见 [machine-interface](../machine-interface.md)。

## 另请参见

[`use`](use.md) · [`list`](list.md)

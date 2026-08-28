# wallet-cli import ledger

注册一个 Ledger 账户（仅观察；在设备上签名）。

## 用法

```
wallet-cli import ledger --app tron (--index <n> | --path <bip32> | --address <T…>) [--label <l>] [options]
```

## 选项

| 选项 | 说明 |
|---|---|
| `--app <tron>` | 要打开的 Ledger app，决定使用哪套派生方案  [必填] |
| `--index <number>` | 要导入的 HD 账户索引（与 --path/--address 互斥） |
| `--path <string>` | 显式指定的 BIP32 路径，例如 m/44'/195'/0'/0/0 |
| `--address <string>` | 已知地址，通过有限范围扫描定位 |
| `--scan-limit <number>` | 配合 --address 时扫描的索引个数（默认 20） |
| `--label <string>` | 唯一的账户标签，1-64 个字符 |

此外还有[全局选项](../index.md)。

## 注意事项

创建一条仅观察记录；本地不保存任何密钥。要求设备已解锁并打开 TRON app。参见 [Ledger 指南](../../guide/ledger.md)。

## 示例

```bash
wallet-cli import ledger --app tron --index 0 --label cold
```

```console
✅ Registered Ledger account "cold"
  Account ID  wlt_7h2k9d3m
  App         tron
  Path        m/44'/195'/0'/0/0
  Address     TMSgJxtPw29AFEHMXsjGo4kWV7UwbCToHJ

⚠️ No private key is stored locally. Signing requires device confirmation.
```

```bash
wallet-cli import ledger --app tron --index 0 --label cold -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"import.ledger","data":{"status":"created","accountId":"wlt_7h2k9d3m","label":"cold","type":"ledger","index":null,"active":true,"addresses":{"tron":"TMSgJxtPw29AFEHMXsjGo4kWV7UwbCToHJ"},"family":"tron","path":"m/44'/195'/0'/0/0"},"meta":{"durationMs":812,"warnings":[]}}
```

## 输出

`data` 携带新注册的 Ledger 账户——只有地址和派生路径，不含任何密钥。本地命令——没有 `chain` 块。

| 字段 | 类型 | 含义 |
|---|---|---|
| `status` | string | `"created"`；若该账户此前已注册，则为 `"existing"` |
| `accountId` | string | 稳定的账户 id |
| `label` | string | 账户标签 |
| `type` | string | `"ledger"`（在设备上签名） |
| `index` | number \| null | 非 HD 账户，恒为 `null`（设备上的索引体现在 `path` 中） |
| `active` | boolean | 是否成为当前账户 |
| `addresses.tron` | string | Base58 TRON 地址 |
| `family` | string | 该地址所属的链系（例如 `tron`） |
| `path` | string | 设备上的 BIP32 派生路径 |

## 退出码

`0` 成功 · `1` 执行失败 · `2` 用法错误。参见 [machine-interface](../../machine-interface.md)。

## 另请参见

[Ledger 指南](../../guide/ledger.md) · [`import watch`](watch.md)

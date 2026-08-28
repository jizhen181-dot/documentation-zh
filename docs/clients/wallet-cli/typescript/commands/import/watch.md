# wallet-cli import watch

注册一个仅观察地址（不含任何密钥）。

## 用法

```
wallet-cli import watch --address <T…> [--label <l>] [options]
```

## 选项

| 选项 | 说明 |
|---|---|
| `--address <string>` | 要跟踪的仅观察地址；TRON base58 T… 格式；链系自动识别  [必填] |
| `--label <string>` | 唯一的账户标签，1-64 个字符 |

此外还有[全局选项](../index.md)。

## 注意事项

无法签名——只能查询。适合监控冷存储余额。

## 示例

```bash
wallet-cli import watch --address TMSgJxtPw29AFEHMXsjGo4kWV7UwbCToHJ --label cold
```

```console
✅ Added watch-only account "cold"
  Address  TMSgJxtPw29AFEHMXsjGo4kWV7UwbCToHJ
  Note     read-only; signing operations will be rejected
```

```bash
wallet-cli import watch --address TMSgJxtPw29AFEHMXsjGo4kWV7UwbCToHJ --label cold -o json
```

```json
{"schema":"wallet-cli.result.v1","success":true,"command":"import.watch","data":{"status":"created","accountId":"wlt_jsyq8fxe","label":"cold","type":"watch","index":null,"active":true,"addresses":{"tron":"TMSgJxtPw29AFEHMXsjGo4kWV7UwbCToHJ"},"family":"tron"},"meta":{"durationMs":36,"warnings":[]}}
```

## 输出

`data` 携带新注册的仅观察账户——只有地址，不含任何密钥。本地命令——没有 `chain` 块。

| 字段 | 类型 | 含义 |
|---|---|---|
| `status` | string | `"created"` |
| `accountId` | string | 稳定的账户 id |
| `label` | string | 账户标签 |
| `type` | string | `"watch"`（只读，无法签名） |
| `index` | number \| null | 非 HD 账户，恒为 `null` |
| `active` | boolean | 是否成为当前账户 |
| `addresses.tron` | string | Base58 TRON 地址 |
| `family` | string | 该地址所属的链系（例如 `tron`） |

## 退出码

`0` 成功 · `1` 执行失败 · `2` 用法错误。参见 [machine-interface](../../machine-interface.md)。

## 另请参见

[`import ledger`](ledger.md) · [`account balance`](../account/balance.md)

# wallet-cli import mnemonic

导入一段 BIP39 助记词。**仅交互式。**

> **注意**：这里没有 `--mnemonic-stdin` / `--password-stdin` 选项。助记词和 master password **只能**通过隐藏的 TTY 提示输入——一段助记词足以取走全部资金，而走 stdin 太容易泄漏到管道、shell 历史和进程列表里。导入本身并不频繁，强制人工输入的代价很小。

## 用法

```
wallet-cli import mnemonic [--label <name>]
```

## 说明

从已有的 BIP39 助记词恢复一个 HD 钱包：派生出 #0 号账户，并用你的 master password 加密保存 seed。导入的钱包会成为当前账户。

交互流程（所有密钥均隐藏输入，绝不回显，也绝不出现在 argv 中）：

1. **master password**——首次使用时设置（需确认输入），或输入以解锁。
2. **标签**——可选的显示名称；留空则自动生成一个（例如 `wallet_ad8f21`）。
3. **助记词**——隐藏粘贴输入；驱动本 CLI 的 AI 或脚本永远看不到它。
4. **校验并保存**——词数或校验和不对会报 `invalid_mnemonic` 并重新提示；成功后派生出各地址，seed 以加密形式写入，绝不以明文存储。

没有 TTY 时，命令会以 `tty_required` 失败——本命令没有非交互式的路径。

## 选项

| 选项 | 说明 |
|---|---|
| `--label <string>` | 便于识别的唯一账户标签，1–64 个字符；省略则自动生成 |

此外还有[全局选项](../index.md#global-options-every-command)。

## 示例

```bash
wallet-cli import mnemonic --label restored
```

```console
? Set master password (hidden):
? Confirm master password:
? Paste recovery phrase (hidden):
✅ Imported wallet "restored"
  Account ID    wlt_d66fvems.0
  Type          HD
  TRON address  TUEZSdKsoDHQMeZwihtdoBiN46zxhGWYdH
  Active        yes

⚠️ Recovery phrase was read from hidden input and was not printed.
```

```bash
wallet-cli import mnemonic --label restored -o json
```

```console
? Set master password (hidden):
? Confirm master password:
? Paste recovery phrase (hidden):
{"schema":"wallet-cli.result.v1","success":true,"command":"import.mnemonic","data":{"status":"created","accountId":"wlt_d66fvems.0","label":"restored","type":"seed","index":0,"active":true,"addresses":{"tron":"TUEZSdKsoDHQMeZwihtdoBiN46zxhGWYdH"},"seedId":"wlt_d66fvems"},"meta":{"durationMs":38,"warnings":[]}}
```

## 输出

`data` 携带导入的账户——只有地址，绝不含任何密钥。本地命令——没有 `chain` 块。

| 字段 | 类型 | 含义 |
|---|---|---|
| `status` | string | `"created"` |
| `accountId` | string | 稳定 id `<seedId>.<index>` |
| `label` | string | 账户标签 |
| `type` | string | `"seed"`（HD 派生） |
| `index` | number | HD 派生索引（首个账户为 0） |
| `active` | boolean | 是否成为当前账户 |
| `addresses.tron` | string | Base58 TRON 地址 |
| `seedId` | string | 所属种子钱包 id |

## 退出码

`0` 导入成功 · `1` 执行失败（`tty_required`——没有可用于交互输入的 TTY；`auth_failed`；`password_mismatch`；`io_error`） · `2` 用法错误（助记词非法、标签重复）。

## 另请参见

[`import private-key`](private-key.md) · [`create`](../create.md) · [`change-password`](../change-password.md) · [故障排查](../../troubleshooting.md)

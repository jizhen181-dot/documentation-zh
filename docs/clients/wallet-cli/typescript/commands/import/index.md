# wallet-cli import

从已有的密钥或硬件设备导入钱包。

## 用法

```
wallet-cli import COMMAND
```

## 子命令

| 命令 | 说明 |
|---|---|
| [`import mnemonic`](mnemonic.md) | 导入一段 BIP39 助记词 |
| [`import private-key`](private-key.md) | 导入一个原始私钥 |
| [`import keystore`](keystore.md) | 从 Web3 keystore 文件导入一个账户 |
| `import ledger` | 注册一个 Ledger 账户（本地仅观察；在设备上签名）——`wallet-cli import ledger --help` |
| `import watch` | 注册一个仅观察地址（不含任何密钥）——`wallet-cli import watch --help` |

所有涉及密钥的子命令都只通过 stdin 选项或 TTY 提示接收密钥；CLI 不从 argv 或专用环境变量读取
敏感信息。shell 变量可以作为 stdin 的数据来源，但不建议长期 `export`。参见
[machine-interface → 敏感信息处理](../../machine-interface.md#secret-handling)。

## 另请参见

[`create`](../create.md) · [`list`](../list.md) · [快速上手](../../guide/getting-started.md)

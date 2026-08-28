# 使用 Ledger 硬件钱包

密钥始终留在设备上；wallet-cli 负责构建交易，由 Ledger 在屏幕上确认并签名。私钥从不接触你的电脑。

## 前置条件

- Ledger 已连接、已**解锁**，并且设备上已**打开 TRON app**；
- 事先已通过 Ledger Live 安装 TRON app。

## 1. 注册 Ledger 账户

```bash
wallet-cli import ledger --app tron --index 0 --label cold
```

该操作会在本地创建一条**仅观察**记录，不保存任何密钥材料；签名在设备上完成。有三种互斥的账户指定方式：

| 参数 | 适用场景 |
|---|---|
| `--index <n>` | 你知道 HD 账户索引（全部省略即为索引 0） |
| `--path <bip32>` | 你需要一个明确的派生路径，例如 `m/44'/195'/0'/0/0` |
| `--address <T…>` | 你知道地址；wallet-cli 会扫描索引来找到它（`--scan-limit`，默认 20） |

用 `wallet-cli list` 确认——该账户会与你的软件账户并列显示，并且可以配合 `use`、`--account` 以及所有查询命令使用。

## 2. 签名并发送

命令本身没有任何变化：

```bash
wallet-cli tx send --to T... --amount 1 --network tron:nile --account cold
```

CLI 不会提示输入密码，而是将交易详情显示在 **Ledger 屏幕上**。请在设备上核对收款方和金额后批准。
交易随后会正常广播，可通过 [`tx status`](../commands/tx/status.md) 查询状态。

这是你对抗地址替换类恶意软件的最佳防线：设备屏幕上显示的内容就是被签名的内容，与主机显示什么无关。

## 3. 设备没有响应时

设备调用与 RPC 受同一个 `--timeout` 限制（默认 60000 毫秒），失败时返回 `error.code: "timeout"`。请依次检查：

1. Ledger 是否已解锁，TRON app 是否已打开（而不是停在主界面）？
2. 重新插拔数据线；不要使用 USB hub。
3. 用更长的 `--timeout` 重试——设备上的确认时间也计入其中，所以要给自己留出阅读和按键的时间。

更多处理办法：[故障排查](../troubleshooting.md#timeout-exit-1)。

## 离线模式

Ledger 已将密钥与主机隔离，也可以配合签名与广播分离流程使用：在连接设备的机器上执行 `--sign-only`，
再在联网机器上执行 [`tx broadcast`](../commands/tx/broadcast.md)。参见
[脚本编写 → 分离签名与广播](scripting.md#sign-here-broadcast-there)。

## 另请参见

[`import ledger` 帮助](../commands/import/index.md) · [安全模型](../concepts/security.md) · [快速上手](getting-started.md)

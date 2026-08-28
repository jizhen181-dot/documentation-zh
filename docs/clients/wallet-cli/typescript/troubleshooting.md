# 故障排查

本页按照机器接口中定义的[错误码](machine-interface.md#error-codes)提供排查方法。错误码的正式定义以
机器接口为准，本页重点说明对应的处理步骤。

## `usage_error` / `invalid_value`（退出码 2） {#usage_error--invalid_value-exit-2}

命令构造有误——某个参数未知、缺失、冲突，或取值非法。

- 用准确的子命令重跑一次 `--help`：`wallet-cli tx send --help`。
- 常见冲突：`--amount` 与 `--raw-amount`；`--token` 与 `--contract` 与 `--asset-id`；`--dry-run`
  与 `--sign-only`；一次调用中出现两个 `*-stdin` 标志。
- `config` 上的 `invalid_value`：检查允许的键（`defaultNetwork`、`defaultOutput`、`timeoutMs`、
  `waitTimeoutMs`、`networks`）和取值（`defaultOutput` 为 `text` 或 `json`）。

## `weak_password`（退出码 2） {#weak_password-exit-2}

`create`（以及其他设置密码的命令）拒绝了这个 master password。它必须**至少 8 个字符**，并包含
**大写字母、小写字母、数字和特殊字符**（`!@#$%^&*()-_=+[]{};:,.?`）。错误消息会指明你没满足的
具体规则。

## `tty_required` / `auth_required`（退出码 2 / 退出码 1） {#tty_required--auth_required-exit-2--exit-1}

命令需要敏感信息，但未能读取。

- `tty_required`——没有挂载终端（CI、管道）。对于有 stdin 路径的命令，请提供对应的 `*-stdin` 标志
  （`--password-stdin`、`--tx-stdin`）。`import mnemonic`、`import private-key` 和
  `change-password` 只能交互执行——它们必须在真实 TTY 中运行；不存在非交互的替代方案。
- `auth_required`——该命令需要 master password；请传 `--password-stdin`，或以交互方式运行。
- `auth_failed`——密码错误（解密失败）；请重新输入。

## `timeout`（退出码 1） {#timeout-exit-1}

节点或 Ledger 设备在 `--timeout`（默认 60000 毫秒）内没有响应。

- 检查到该网络的基本连通性；如果处在代理之后，请确认 CLI 的流量确实经过了代理。
- 放宽上限：`--timeout 120000`。
- Ledger：确认设备已解锁且 TRON app 已打开，然后重试。
- **如果这发生在 `tx send` 上**：交易可能仍然已经提交。如果你有 txid，请先找回它并查 `tx status`，
  再决定是否重发。

## `rpc_error`（退出码 1） {#rpc_error-exit-1}

TRON 节点接受了连接，但拒绝了请求。消息中带有节点给出的原因，例如
`TRON getTransaction failed: Transaction not found`。

- *Transaction not found*：`--txid` 写错、`--network` 选错（拿 Nile 的 txid 去主网查），或者交易
  尚未传播开——过几秒后重试。
- *Insufficient balance / bandwidth / energy*：给账户充值，或质押以获得资源（`stake freeze`）
  ——资源的工作方式参见[网络](concepts/networks.md)；在 Nile 上请使用水龙头。
- TRC20 发送回滚：只有在确认收款方/合约无误之后，才考虑调高 `--fee-limit`（默认 100000000 SUN）。

## `internal_error`（退出码 1） {#internal_error-exit-1}

未预期的失败。为避免泄露敏感信息，返回消息会保持概括。添加 `--verbose` 重试可在 stderr 获取更多
诊断信息；如果问题能够稳定复现，请在 issue 中提供命令结构，但**不要**包含任何密码、助记词或私钥。

## 不属于错误码，但经常被问到 {#not-an-error-code-but-frequently-asked}

- **`tx status` 长时间显示 `pending`**——节点已看到交易但尚未入块；请继续轮询。如果超过你设定的
  截止时间仍停在 `pending`/`not_found`，这属于**状态未知**而不是失败：`not_found` 只说明当前被
  查询的节点不知道它，交易可能仍在其他节点的内存池中。请先通过 SolidityNode 或区块浏览器对账，
  确认没有落链之后再考虑重发——盲目重发会造成重复付款。
- **`tx status` 返回 `confirmed`，如何判断是否不可逆**——`confirmed` 读的是 FullNode 的未确认
  视图，只表示已入块并取得回执。需要最终性时请另行查询 SolidityNode。
- **"only one *-stdin flag can consume stdin per run"**——每次调用只能通过管道传入一项敏感信息；带密码发送
  时请用 `--password-stdin`，把助记词/私钥留在加密存储中。
- **忘记 master password**——没有恢复途径；请用你的 BIP39 助记词（`import mnemonic`）恢复到一个新
  钱包，并设置新密码。
- **`account history` 失败但其他查询正常**——历史查询需要 TronGrid 端点；普通的节点 RPC 不够。

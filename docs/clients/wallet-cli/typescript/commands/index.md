# 命令参考

每个命令——包括每个子命令——都有独立的页面，并遵循固定的结构（用法 · 说明 · 选项 · 示例 · 输出 · 退出码 · 另请参见）。命令组页面会列出并链接其下的子命令。

## 钱包与账户

| 命令 | 页面 |
|---|---|
| `create` | [create.md](create.md) |
| `import`（命令组） | [import/index.md](import/index.md) |
| `import mnemonic` | [import/mnemonic.md](import/mnemonic.md) *（仅支持交互式）* |
| `import private-key` | [import/private-key.md](import/private-key.md) *（仅支持交互式）* |
| `import keystore` | [import/keystore.md](import/keystore.md) *（仅支持交互式）* |
| `import ledger` | [import/ledger.md](import/ledger.md) |
| `import watch` | [import/watch.md](import/watch.md) |
| `list` | [list.md](list.md) |
| `use` | [use.md](use.md) |
| `current` | [current.md](current.md) |
| `derive` | [derive.md](derive.md) |
| `rename` | [rename.md](rename.md) |
| `backup` | [backup.md](backup.md) |
| `delete` | [delete.md](delete.md) |
| `change-password` | [change-password.md](change-password.md) |

## 交易

| 命令 | 页面 |
|---|---|
| `tx`（命令组） | [tx/index.md](tx/index.md) |
| `tx send` | [tx/send.md](tx/send.md) |
| `tx broadcast` | [tx/broadcast.md](tx/broadcast.md) |
| `tx status` | [tx/status.md](tx/status.md) |
| `tx info` | [tx/info.md](tx/info.md) |
| `tx sign` | [tx/sign.md](tx/sign.md) |
| `tx approvals` | [tx/approvals.md](tx/approvals.md) |
| `tx multisig` | [tx/multisig.md](tx/multisig.md) |

## 链上查询

| 命令 | 页面 |
|---|---|
| `account`（命令组） | [account/index.md](account/index.md) |
| `account balance` | [account/balance.md](account/balance.md) |
| `account info` | [account/info.md](account/info.md) |
| `account history` | [account/history.md](account/history.md) |
| `account portfolio` | [account/portfolio.md](account/portfolio.md) |
| `account activate` | [account/activate.md](account/activate.md) |
| `account set` | [account/set.md](account/set.md) |
| `block` | [block.md](block.md) |
| `chain`（命令组） | [chain/index.md](chain/index.md) |
| `chain params` | [chain/params.md](chain/params.md) |
| `chain prices` | [chain/prices.md](chain/prices.md) |
| `chain node` | [chain/node.md](chain/node.md) |

## 账户权限

| 命令 | 页面 |
|---|---|
| `permission`（命令组） | [permission/index.md](permission/index.md) |
| `permission show` | [permission/show.md](permission/show.md) |
| `permission update` | [permission/update.md](permission/update.md) |

## token 与合约

| 命令 | 页面 |
|---|---|
| `token`（命令组） | [token/index.md](token/index.md) |
| `token balance` | [token/balance.md](token/balance.md) |
| `token info` | [token/info.md](token/info.md) |
| `token add` | [token/add.md](token/add.md) |
| `token list` | [token/list.md](token/list.md) |
| `token remove` | [token/remove.md](token/remove.md) |
| `contact`（命令组） | [contact/index.md](contact/index.md) |
| `contact add` | [contact/add.md](contact/add.md) |
| `contact list` | [contact/list.md](contact/list.md) |
| `contact remove` | [contact/remove.md](contact/remove.md) |
| `contract`（命令组） | [contract/index.md](contract/index.md) |
| `contract call` | [contract/call.md](contract/call.md) |
| `contract send` | [contract/send.md](contract/send.md) |
| `contract deploy` | [contract/deploy.md](contract/deploy.md) |
| `contract info` | [contract/info.md](contract/info.md) |
| `contract clear-abi` | [contract/clear-abi.md](contract/clear-abi.md) |
| `contract set-origin-energy-limit` | [contract/set-origin-energy-limit.md](contract/set-origin-energy-limit.md) |
| `contract set-user-resource-percent` | [contract/set-user-resource-percent.md](contract/set-user-resource-percent.md) |
| `contract create2` | [contract/create2.md](contract/create2.md) |
| `gasfree`（命令组） | [gasfree/index.md](gasfree/index.md) |
| `gasfree info` | [gasfree/info.md](gasfree/info.md) |
| `gasfree transfer` | [gasfree/transfer.md](gasfree/transfer.md) |
| `gasfree trace` | [gasfree/trace.md](gasfree/trace.md) |

## 质押、投票与奖励

| 命令 | 页面 |
|---|---|
| `stake`（命令组） | [stake/index.md](stake/index.md) |
| `stake freeze` | [stake/freeze.md](stake/freeze.md) |
| `stake unfreeze` | [stake/unfreeze.md](stake/unfreeze.md) |
| `stake withdraw` | [stake/withdraw.md](stake/withdraw.md) |
| `stake cancel-unfreeze` | [stake/cancel-unfreeze.md](stake/cancel-unfreeze.md) |
| `stake delegate` | [stake/delegate.md](stake/delegate.md) |
| `stake undelegate` | [stake/undelegate.md](stake/undelegate.md) |
| `stake info` | [stake/info.md](stake/info.md) |
| `stake delegated` | [stake/delegated.md](stake/delegated.md) |
| `vote`（命令组） | [vote/index.md](vote/index.md) |
| `vote cast` | [vote/cast.md](vote/cast.md) |
| `vote list` | [vote/list.md](vote/list.md) |
| `vote status` | [vote/status.md](vote/status.md) |
| `reward`（命令组） | [reward/index.md](reward/index.md) |
| `reward balance` | [reward/balance.md](reward/balance.md) |
| `reward withdraw` | [reward/withdraw.md](reward/withdraw.md) |

## 治理

| 命令 | 页面 |
|---|---|
| `proposal`（命令组） | [proposal/index.md](proposal/index.md) |
| `proposal list` | [proposal/list.md](proposal/list.md) |
| `proposal show` | [proposal/show.md](proposal/show.md) |
| `proposal create` | [proposal/create.md](proposal/create.md) |
| `proposal approve` | [proposal/approve.md](proposal/approve.md) |
| `proposal delete` | [proposal/delete.md](proposal/delete.md) |
| `witness`（命令组） | [witness/index.md](witness/index.md) |
| `witness create` | [witness/create.md](witness/create.md) |
| `witness update` | [witness/update.md](witness/update.md) |
| `witness set-brokerage` | [witness/set-brokerage.md](witness/set-brokerage.md) |

## TRC10 资产与链上交易所

| 命令 | 页面 |
|---|---|
| `asset`（命令组） | [asset/index.md](asset/index.md) |
| `asset issue` | [asset/issue.md](asset/issue.md) |
| `asset update` | [asset/update.md](asset/update.md) |
| `asset participate` | [asset/participate.md](asset/participate.md) |
| `asset unfreeze` | [asset/unfreeze.md](asset/unfreeze.md) |
| `asset info` | [asset/info.md](asset/info.md) |
| `asset list` | [asset/list.md](asset/list.md) |
| `exchange`（命令组） | [exchange/index.md](exchange/index.md) |
| `exchange create` | [exchange/create.md](exchange/create.md) |
| `exchange inject` | [exchange/inject.md](exchange/inject.md) |
| `exchange withdraw` | [exchange/withdraw.md](exchange/withdraw.md) |
| `exchange trade` | [exchange/trade.md](exchange/trade.md) |
| `exchange show` | [exchange/show.md](exchange/show.md) |
| `exchange list` | [exchange/list.md](exchange/list.md) |

## 签名

| 命令 | 页面 |
|---|---|
| `message`（命令组） | [message/index.md](message/index.md) |
| `message sign` | [message/sign.md](message/sign.md) |
| `typed-data`（命令组） | [typed-data/index.md](typed-data/index.md) |
| `typed-data sign` | [typed-data/sign.md](typed-data/sign.md) |

## 本地命令

| 命令 | 页面 |
|---|---|
| `encoding`（命令组） | [encoding/index.md](encoding/index.md) |
| `encoding convert` | [encoding/convert.md](encoding/convert.md) |
| `address`（命令组） | [address/index.md](address/index.md) |
| `address generate` | [address/generate.md](address/generate.md) |
| `config` | [config.md](config.md) |
| `networks` | [networks.md](networks.md) |

## 全局选项（所有命令通用） {#global-options-every-command}

```
-o, --output <text|json>   result format (default: config.defaultOutput, built-in text)
--network <string>         canonical network id (chain commands; falls back to config.defaultNetwork)
--account <string>         accountId, label, or address (wallet-bound commands; falls back to active)
--timeout <number>         per RPC/device call timeout, ms (default: config.timeoutMs, built-in 60000)
-v, --verbose              extra diagnostic output
-h, --help / -V, --version
```

广播类（✍️）命令还额外支持 `--wait` / `--wait-timeout <ms>`（时间上限默认取配置 `waitTimeoutMs`，内置 60000）、提前退出模式 `--dry-run` / `--sign-only` / `--build-only`，以及多签选项 `--permission-id <n>` / `--expiration <ms>`。

三种提前退出模式互斥，且 `--expiration` 只有与 `--sign-only` 或 `--build-only` 同时使用时才被接受。违反其中任一条都属于用法错误，退出码为 `2`。具体错误码取决于检查发生的位置：在治理类以及 asset/exchange 的写操作中是 `invalid_value`，且错误信息给出的字段名是 `--input`，而不是你实际传入的那些选项——例如 `invalid --input: choose at most one of --dry-run, --sign-only, --build-only`。其他地方同样的冲突则报告 `invalid_option`。请按退出码分支，而不要按错误码字符串分支；参见 [machine-interface](../machine-interface.md#error-codes)。

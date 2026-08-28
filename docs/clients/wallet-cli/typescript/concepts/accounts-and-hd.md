# 账户与 HD 钱包

wallet-cli 如何组织你在 `list` 中看到的内容。

## 种子与账户 {#seeds-and-accounts}

一个**种子钱包**对应一条 BIP39 助记词；它可以派生出多个**账户**。id 正体现了这一点：

```
wlt_4473p34m        ← seedId (one mnemonic)
wlt_4473p34m.0      ← accountId = seedId.index (one address)
wlt_4473p34m.1
```

`create` 创建一个新种子以及 0 号账户；`derive --seed-id wlt_…` 从同一条助记词追加下一个账户（或
显式指定 `--index`）。在其他环境中恢复同一助记词会重新派生出相同的地址，因此助记词才是钱包备份，而
master password 只是本地保护的原因。注意 `create` 不会打印助记词；执行
[`backup`](../commands/backup.md) 可以把它导出到离线文件。

## 账户类型 {#account-types}

| 类型 | 创建方式 | 本地是否保存密钥材料？ | 能否签名？ |
|---|---|---|---|
| `seed` | `create`、`import mnemonic`、`derive` | 加密的种子 | 能 |
| private-key | `import private-key` | 加密的私钥；**无法派生** | 能 |
| ledger | `import ledger` | 无（仅观察条目） | 在设备上签名 |
| watch | `import watch` | 无 | 不能——只能查询 |

## 当前账户 {#the-active-account}

大多数与钱包绑定的命令都需要一个账户。解析顺序：

1. 命令上的 `--account <accountId|label|address>`；
2. 否则使用**当前**账户——用 `use <account>` 设置，由 `current` 显示，并在 `list` 中以 `(active)`
   标记。

标签唯一、1–64 个字符、可重命名（`rename`）——稳定的句柄始终是 `accountId`。

## 生命周期 {#lifecycle}

- `backup <account>` 把密钥材料和元数据导出到一个以 **0600** 权限创建、且不会覆盖的文件（默认位于
  当前工作目录）。请按照密钥文件的安全等级保护该文件，并注意命令的执行目录，因为 CLI 不会
  检查该目录是否是共享目录或纳入了版本控制。
- `delete` 删除账户；**删除 HD 钱包会从种子根开始级联**——该种子派生出的全部账户都会一并删除。
  链上资产不受影响：重新导入助记词即可恢复访问。
- 丢失 master password 在本地无法恢复；退路始终是助记词 → `import mnemonic`。

## 另请参见

[`list`](../commands/list.md) · [`create`](../commands/create.md) · [`import`](../commands/import/index.md) · [安全模型](security.md)

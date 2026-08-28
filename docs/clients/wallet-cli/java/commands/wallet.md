# 钱包管理

创建、导入、导出和管理本地钱包，以及会话状态（登录 / 锁定 / 切换）。

keystore 和密钥材料的相关概念，参见[环境准备 / 配置参考](../reference/config.md)。完整的首次运行
流程，参见[快速上手指南](../guide/getting-started.md)。

## RegisterWallet

注册你的钱包。你需要设置钱包密码；这会生成地址和私钥。

## ImportWallet

从 hex-string 格式的私钥导入钱包。你需要设置密码。

## ImportWalletByBase64

从 base64 格式的私钥导入钱包。你需要设置密码。

## ImportWalletByMnemonic

从助记词导入钱包。你需要设置密码并输入助记词。

```console
wallet> ImportWalletByMnemonic
Please input password.
password:
Please input password again.
password:
Please enter 12 words (separated by spaces) [Attempt 1/3]:
```

## ExportWalletMnemonic

导出钱包中该地址的助记词。

```console
wallet> ExportWalletMnemonic
Please input your password.
password:
exportWalletMnemonic successful !!
a*ert tw*st co*rect mat*er pa*s g*ther p*t p*sition s*op em*ty coc*nut aband*n
```

## ExportWalletKeystore

以 TronLink 钱包格式导出钱包 keystore。

```console
wallet> ExportWalletKeystore tronlink /tmp
Please input your password.
password:
exported keystore file : /tmp/TYdhEg8b7tXm92UDbRDXPtJNU6T9xVGbbo.json
exportWalletKeystore successful !!
```

## ImportWalletByKeystore

把 TronLink 格式的 keystore 文件导入 wallet-cli。

```console
wallet> ImportWalletByKeystore tronlink /tmp/tronlink.json
Please input password.
password:
Please input password again.
password:
fileName = TYQq6zp51unQDNELmT4xKMWh5WLcwpCDZJ.json
importWalletByKeystore successful !!
```

## ImportWalletByLedger

把 Ledger 设备上派生的账户导入 wallet-cli。

```console
wallet> ImportWalletByLedger
((Note:This will pair Ledger to user your hardward wallet)
Only one Ledger device is supported. If you have multiple devices, please ensure only one is connected.
Ledger device found: Nano X
Please input password.
password:
Please input password again.
password:
-------------------------------------------------
Default Account Address: TAT1dA8F9HXGqmhvMCjxCKAD29YxDRw81y
Default Path: m/44'/195'/0'/0/0
-------------------------------------------------
1. Import Default Account
2. Change Path
3. Custom Path
Select an option: 1
Import a wallet by Ledger successful, keystore file : ./Wallet/Ledger-TAT1dA8F9HXGqmhvMCjxCKAD29YxDRw81y.json
You are now logged in, and you can perform operations using this account.
```

## BackupWallet

备份你的钱包。你需要输入钱包密码；它会以 hex-string 格式导出私钥，例如：
`1234567890123456789012345678901234567890123456789012345678901234`

## BackupWallet2Base64

备份你的钱包。你需要输入钱包密码；它会以 base64 格式导出私钥，例如：
`ch1jsHTxjUHBR+BMlS7JNGd3ejC28WdFvEeo6uUHZUU=`

## ChangePassword

修改账户的密码。

## GenerateSubAccount

使用钱包中的助记词生成子账户。

```console
wallet> GenerateSubAccount
Please input your password.
password:

=== Sub Account Generator ===
-----------------------------
Default Address: TYEhEg7b7tXm92UDbRDXPtJNU6T9xVGbbo
Default Path: m/44'/195'/0'/0/1
-----------------------------

1. Generate Default Path
2. Change Account
3. Custom Path

Enter your choice (1-3): 1
mnemonic file : ./Mnemonic/TYEhEg7b7tXm92UDbRDXPtJNU6T9xVGbbo.json
Generate a sub account successful, keystore file name is TYEhEg7b7tXm92UDbRDXPtJNU6T9xVGbbo.json
generateSubAccount successful.
```

## ClearWalletKeystore

清除登录账户的钱包 keystore。

```console
wallet> ClearWalletKeystore

Warning: Dangerous operation!
This operation will permanently delete the Wallet&Mnemonic files of the Address: TABWx7yFhWrvZHbwKcCmFLyPLWjd2dZ2Rq
Warning: The private key and mnemonic words will be permanently lost and cannot be recovered!
Continue? (y/Y to proceed):y

Final confirmation:
Please enter: 'DELETE' to confirm the delete operation:
Confirm: (DELETE): DELETE

File deleted successfully:
- /wallet-cli/Wallet/TABWx8yFhWrvZHbwKcCmFLyPLWjd2dZ2Rq.json
- /wallet-cli/Mnemonic/TABWx8yFhWrvZHbwKcCmFLyPLWjd2dZ2Rq.json
ClearWalletKeystore successful !!!
```

## ResetWallet

删除所有本地钱包 keystore 文件和助记词文件，并按提示重新注册或导入钱包。

```console
wallet> resetwallet
User defined config file doesn't exists, use default config file in jar

Warning: Dangerous operation!
This operation will permanently delete the Wallet&Mnemonic files
Warning: The private key and mnemonic words will be permanently lost and cannot be recovered!
Continue? (y/Y to proceed, c/C to cancel):
y

Final confirmation:
Please enter: 'DELETE' to confirm the delete operation:
Confirm: (DELETE): DELETE
resetWallet  successful !!!
Now, you can RegisterWallet or ImportWallet again. Or import the wallet through other means.
```

## LoginAll

用统一的密码登录多个 keystore 账户。

```console
wallet> loginall
Please input your password.
password:
Use user defined config file in current dir
[========================================] 100%
The 1th keystore file name is TJEEKTmaVTYSpJAxahtyuofnDSpe2seajB.json
The 2th keystore file name is TX1L9xonuUo1AHsjUZ3QzH8wCRmKm56Xew.json
The 3th keystore file name is TVuVqnJFuuDxN36bhEbgDQS7rNGA5dSJB7.json
The 4th keystore file name is Ledger-TRvVXgqddDGYRMx3FWf2tpVxXQQXDZxJQe.json
The 5th keystore file name is TYXFDtn86VPFKg4mkwMs45DKDcpAyqsada.json
Please choose between 1 and 5
5
LoginAll  successful !!!
```

## Logout

登出当前钱包账户。

```console
wallet> Logout
Logout  successful !!!
```

## Lock

锁定登录账户，使其不允许签名和交易。需要在 [`config.conf`](../reference/config.md) 中设置
`lockAccount = true`。

```console
wallet> lock
lock  successful !!!
```

## Unlock

解锁已锁定的登录账户。默认在 300 秒后重新锁定；你可以传入以秒为单位的解锁时长。需要在
[`config.conf`](../reference/config.md) 中设置 `lockAccount = true`。

```console
wallet> unlock 60
Please input your password.
password:
unlock  successful !!!
```

## SwitchWallet

使用 `LoginAll` 登录之后，在多个钱包之间切换。

```console
wallet> switchwallet
The 1th keystore file name is TJEEKTmaVTYSpJAxahtyuofnDSpe2seajB.json
The 2th keystore file name is TX1L9xonuUo1AHsjUZ3QzH8wCRmKm56Xew.json
The 3th keystore file name is TVuVqnJFuuDxN36bhEbgDQS7rNGA5dSJB7.json
The 4th keystore file name is Ledger-TRvVXgqddDGYRMx3FWf2tpVxXQQXDZxJQe.json
The 5th keystore file name is TYXFDtn86VPFKg4mkwMs45DKDcpAyqsada.json
Please choose between 1 and 5
5
SwitchWallet  successful !!!
```

## ModifyWalletName

修改钱包的名称。

```console
wallet> ModifyWalletName new-name
Modify Wallet Name  successful !!
```

## 另请参见

- [account](account.md)——账户查询与元数据
- [network](network.md)——在网络之间切换
- [multisig](multisig.md) · [concepts/multisig](../concepts/multisig.md)

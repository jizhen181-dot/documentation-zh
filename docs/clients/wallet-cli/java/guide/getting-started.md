# 快速上手

首次运行的流程：构建、创建账户、解锁、查看，并发送第一笔 TRX——全部在交互式提示符中完成。

## 快速上手

构建、创建账户并发出第一笔转账——全部在交互式提示符中完成：

```console
# 1. 构建
$ git clone https://github.com/tronprotocol/wallet-cli.git
$ cd wallet-cli && ./gradlew build && cd build/libs

# 2. 启动交互式钱包
$ java -jar wallet-cli.jar

# 3. 在钱包提示符下：创建账户（或使用 ImportWallet）、解锁并查看
> RegisterWallet 123456      # 创建密码为 123456 的 keystore
> Login                      # 解锁账户
> GetAddress                 # 显示你的地址
> GetBalance                 # TRX 余额

# 4. 发送 1 TRX（金额以 SUN 计；1 TRX = 1,000,000 SUN）
> SendCoin <toAddress> 1000000
```

> 在主网上执行这些命令会使用**真实资产**。学习阶段请用 `SwitchNetwork` 切换到测试网（Nile 或
> Shasta），并从该网络的水龙头领取测试币。

## 如何创建账户

你可以通过向不存在的账户转账来创建账户，也可以用 **CreateAccount** 命令发起一笔交易来创建账户。
向不存在的账户转账有 **1 TRX** 的最低金额限制。通过 `CreateAccount` 命令创建账户同样会燃烧
**1 TRX**。

完整的 `CreateAccount` 示例见 [commands/account](../commands/account.md)。

## 下一步

- [command-flow](command-flow.md)——一次完整的端到端会话
- [commands/wallet](../commands/wallet.md)——创建 / 导入 / 备份钱包
- [commands/network](../commands/network.md)——切换到测试网
- [concepts/resources](../concepts/resources.md)——带宽与能量

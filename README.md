

# ⚠️请务必阅读完以下内容，再去使用

# 👛Chance Bitcoin Wallet

这是一个iOS终端的比特币钱包应用，此应用非完全节点钱包，是一个适合移动设备使用的瘦钱包，它只关心安全地保存好用户重要的私钥信息，并支持比特币账户的创建，发送比特币，接收比特币，查询钱包子账户，备份钱包，恢复钱包等功能。它不具有存储完整区块链数据，验证P2P网络广播的交易，挖矿等功能。

## ✨应用特点

- Swift3.0编写的iOS钱包应用，支持iOS9+
- 基于HDM模型建立的账户体系，只需一个“恢复密语和密码”创建的钱包最多可拥有2^31（约21亿）的子账户。
- 支持建立多重签名账户功能。
- 支持账户发送比特币，接收比特币，查询交易记录。
- 支持使用“恢复密语和密码”备份和恢复钱包，也可单独备份子账户的私钥。
- 使用Apple推荐的底层安全框架Keychain，只保存私钥到唯一的当前设备中。
- 支持Touch ID验证密码
- 钱包的“账户体系数据”可开启iCloud自动同步，使用“恢复密语和密码”进行恢复会自动同步对应的“账户体系数据”，即可找回所丢失的HDM账户和多重签名账户。既保证私钥的安全，也提供了极大的方便。
- 支持自由切换可信任的区块链云节点服务（blockchain.info | insight.bitpay.com等云节点服务），节省设备存储空间。
- 核心库由CoreBitcoin的Objective-C版提供。

## 💾数据结构

本应用对数据资料划分为三部分：“恢复密语和密码”，“账户体系”，“交易数据”。

- 恢复密语和密码：本应用使用Apple的keychain框架，加密钱包私钥保存到设备中，私钥不会使用icloud的自动备份功能，所以用户必须牢记钱包的“恢复密语和密码”，一旦设备损坏，可以在新设备中通过导入“恢复密语和密码”对钱包进行恢复，保证资产不会丢失。
- 账户体系：此数据资料也是非常重要的，虽然你可以凭借“恢复密语和密码”恢复HDM模型的钱包，但HDM模型的账户体系没有一个硬性的标准，不同的钱包有不同的体系，你只能使用一样的HDM钱包应用来恢复数据。本钱包的账户体系与根私钥信息分离保存，使用realm数据库保存账户体系，包含HDM账户的树高度位，索引位，多重签名账户的赎回脚本。数据库文件使用私钥种子sha256的hash值来命名，这样应用可以保存多份账户体系数据，再通过icloud进行自动同步，方便以后恢复钱包数据。
- 交易数据：为了节省设备资源，应用不使用P2P网络同步完整的区块链数据，只通过连接可信任的云节点，获取与账户相关的数据，及借助节点发送交易。同步下来的数据，会保存到本地的交易数据库，并且不会使用icloud自动同步，因为icloud的空间是有限的，扩展也需要支付Apple年费。

## 🔐安全方案

- “恢复密语和密码”这些私钥信息，使用Apple的keychain框架保存于当前设备中。更换新设备，或设备损坏就会丢失。如果你非常信任Apple公司，你可以放心使用此应用。应用中可以查看“恢复密语”，但“密码”不能查看，“密码”务必牢记！就算“恢复密语”泄露，但没有“密码”，也无法恢复原来的钱包。
- 在生成交易单时，私钥签名类型使用“SIGHASH_ALL”，该签名类型为默认类型，也是目前绝大部分交易采用的，顾名思义即签名整单交易。首先，组织所有输出、输入，就像上文分解Hex过程一样，每个输入都对应一个签名，暂时留空，其他包括sequence等字段均须填写，这样就形成了一个完整的交易Hex（只缺签名字段）。然后，每一个输入均需使用私钥对该段数据进行签名，签名完成后各自填入相应的位置，N个输入N个签名。简单理解就是：对于该笔单子，认可且只认可的这些输入、输出，并同意花费我的那笔输入。
- 为了节省设备资源，应用不使用P2P网络同步完整的区块链数据，只通过连接可信任的云节点，获取与账户相关的数据，及借助节点发送交易。如果你了解过比特币的发送交易知识，发送交易并不需要提供私钥，只需要提供私钥对“未花交易”的签名，而且私钥签名类型使用“SIGHASH_ALL”，所以云节点无法在广播交易前篡改交易单的输出部分。总之，使用功能丰富的可信任云节点，可以大大减少设备的存储空间，及简化连接比特币网络的过程，并安全保护好最重要的私钥。
- 普通的HDM账户，只需要其账户私钥就可以动用它的资产，如果该账户的私钥泄露了，也只会导致该账户的资产被黑客转移，不会影响其它账户。应用也只提供子账户的“私钥”和“公钥”查看。不提供根账户的“私钥”和“公钥”查看。
- 多重签名账户，这类账户由多个普通账户提供公钥而创建的，被称为“赎回脚本（redeemscript）”。其资产交易时必须由多个普通账户对交易进行（达到必要数目的）联合签名才能验证生效，是比特币里安全级别最高的账户，但使用起来较为复杂。

## 👨‍👩‍👧‍👦多重签名交易协议
当我们通过分享公钥创建了多重签名账户后，账户并不会收集其它公钥提供者的私钥，但有这些公钥生产出的“赎回脚本”，并保存在账户数据库中。这个“赎回脚本”的地址所收的资产就是大家的资产，当账户要动用资产发送交易时，需要大家提供必要数目的签名才能被比特币网络验证确认。

> 在联合签名这一步，我们有以下注意事项：

> - 我们不可能把所有的私钥提供给交易发起者去做签名，这样会违背了多重签名账户创造的意义。
> - 比特币的脚本系统是堆栈结构的，所以它验证签名时，需要的签名排序要与赎回脚本里的key排序一致，这样脚本才能验证正确。
> - 动用账户的未花交易记录的条数可能是多条的，每条记录都要进行签名。
> - 可能只有账户的维护者才持有赎回脚本，其他人没有必要长期持有赎回脚本，当有交易发起时，他们只要提供签名就是。

为了解决上述问题，我定义了一种多重签名交易协议。发起交易者生成一张待签名的交易单数据，通过网络发送给其它待签名者（不论签名顺序），对方接收后进行签名并验证已得到的签名是否足够，验证通过就广播给比特币网络，不通过就把签名信息合并到待签名的交易单中，转发给其它待签名者（不论签名顺序），如此类推，直到最后一个收集完所有签名的人广播交易到比特币网络。此协议数据文本使用“multisig://”作为协议标识开头。

> 文本格式：multisig:{"rawTx":"01000000021138...","redeemScriptHex":"532103324c4"...,"keySignatures":{"0":["3045022100...","304402..."],"2":["30450221...","3044022059bd..."]}}

“multisig:”后的内容其实是一个json结构体
```swift
{
    //rawTx，空白签名交易单的hex
    "rawTx": "01000000021138c1e2f7735a8589fa2a3df2426fbaf24a047330c5a0dc14ffadf7dd8767b10000000000ffffffff6d3c772c389561f20b4afec072cbb61cbe15b42aa37fb50e98ad78f3021241790000000000ffffffff02409c0000000000001976a914d1f9af334c34de5558c0d6785d2f22e25a9eff6d88ac80131c000000000017a914dde71db06038ed3a3bead881b20176dd6de8ec068700000000",

    //redeemScriptHex，多重签名赎回脚本的hex
    "redeemScriptHex": "532103324c40d0042fb1c21be5d0688e1e0c7090e16d0fc6abbb0563ba368bc2d7f20b2102c6287fd02ec3f5c3689d9152688553670d9f84fdc124c621e2ffe7be8ccc60bc21028c42e572d465753610915266f5f7855ebcb74b7f7db434a2732685cb290c20a953ae",

    //keySignatures，各私钥提供的交易签名hex
    "keySignatures": {
        //第1条钥匙所在赎回脚本的位置
        "0": [
            //对第1条未花交易输入的签名
            "3045022100ddccdb3a60054cb3fd8973ecf24ca0f09f27533a42a7d1b8f30768b03350513002201b034827ad6a3a23850d485bf09f4b6c1e4a15d081f22a376992e61aa37ca81501",
            //对第2条未花交易输入的签名
            "304402205f11cf0824a435f6445b20f941d74d4f1e879bd5b3b6c2cb72e58e5e335796b9022042cb6d3b28c70316e71b6ae6b4ca4103cb25bc3f57dab7f5fe9a5534f2d5e30f01"
        ],
        //第2条钥匙的交易签名信息
        "2": [
            "30450221008dd0d34169ecc73022f3ab7adb190068e8a85c09f5b578c73746d7118aaf3e3e02200830884592a60313a1432c494af62fe658361c23919401d59d6c850feac5216f01",
            "3044022059bdd1ff6a07bc3bcd4fc58f8a171ee2a77d7ac22c9f69a812d665032da48259022076fe2dcd69b2e0bf05e87c09a419e3f1202529f8a39858ae0170a84be34276e201"
        ]
    }
}

```

> 协议解析：

> - rawTx：不包含签名的交易单，用于给待签名者查看交易单详情——转账多少比特币到哪个地址上。
> - redeemScriptHex：每个待签名者都有可能是最后一个完成验证通过的，广播交易时需要，交易单的脚本需要包含赎回脚本信息。而且赎回脚本包公钥哈希，能用来确定待签名者的签名顺序位置。
> - keySignatures：交易单签名数据结构，这个结构以签名顺序号作为key值，对应签名提供者的多个交易输入的签名数据

最后，解析协议和封装协议的工作由我的钱包程序处理，在MultiSigTransaction类中实现。注意，这个协议并非比特币促进协议（BIP）里头的，仅仅适合这个钱包应用。

## 📦产品功能

![产品功能](https://github.com/zhiquan911/chance_btc_wallet/blob/master/Chance%20Wallet.png?raw=true)

## 🚀发展计划

- 开发CoreBitcoin for swift（独立的一个项目），上面已经提到现在使用的一个objective-c版的，因为作者还没有计划编写swift版的，为了未来对钱包的更好支持，我会编写一个swift语言版的bitcoin核心库。这样做有很多好处：

 - - swift现在已经开源，支持iOS/macOS/Linux，我对swift的未来非常有信心，也很喜欢用swift的特性重构项目。
 - - 隔离验证正在投票等待激活，可以在新的库做兼容隔离验证，一旦激活成功，这个库可以延伸更多能力。

- 开发支持比特币全球行情
- 开发以钱包账户为基础的IM系统，用于应用内收发信息，收发比特币，多重签名账户创建，多重签名交易

## 👾风险提醒

- 本应用受限Apple设备，最重要的私钥信息保存在keychain中，Apple是否留有后门我们不得已知
- 钱包“恢复密语和密码”一旦丢失且没有备份，将永远无法找回，等于所有账户的资产丢失。
- 多重签名账户，需要保存好多个账户的私钥，如果私钥丢失到低于必要的签名数目，将永远无法使用多重签名账户的资产。“赎回脚本”也十分重要，一旦丢失需要重新生成。应用的“账户体系”中以包含保存多重签名账户的赎回脚本，为了更安全，应用也提供导出赎回脚本，让用户自行备份。
- 可信任的云节点一旦关闭，比特币资产不会丢失，但钱包将无法显示出资产余额，也无法发出比特币。这时候，要等开发者升级软件，跟换云节点才能继续使用。或者导出子账户的私钥，导入到其它支持导入私钥的客户端中恢复，继续使用。

## 🤖公开版本

###[iOS 0.6.0](https://www.pgyer.com/C4qt)

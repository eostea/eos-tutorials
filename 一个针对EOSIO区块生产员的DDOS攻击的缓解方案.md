
> 本文内容由EOS42提供, 原文链接: https://blog.eos42.io/ddos-mitigation/

本文的目的是为了指导区块生产员如何能保护EOS网络免受DDOS攻击。

并也提供了一些有关云供应商提供的DDOS保护服务之测试结果。

未来我们将会继续发布更多能缓解DDOS攻击的测试和结果。

## 以下的服务被应用到测试中：

* 一个区块生产员节点，入站端口为8888,9876。
* 使用针对端口1234的SSPD攻击来执行DDOS攻击。
* 一个DDOS僵尸网络 - 没有提供额外信息

向已封闭端口进行攻击的原因是为了表明在端口在未打开的情况下之脆弱性。 SSDP攻击发生在网络层，这意味着你的防火墙必须拥有着能够处理所有数据包的能力。

## 关闭端口（ports）未必能够保护你能免受DDOS攻击。

当你从互联网上关闭一个端口时，所有定向于该端口的流量必须被防火墙清除，再此同时将需要一点CPU时间来删除每个请求。那就想像一下，如果你的防火墙正在被多个不同IP的数据包攻击，并且它们是以50 Gbps的速度来访问端口1234的。即使此端口是已关闭的，防火墙也几乎不可能可以处理这么大流量。


## 什么是SSDP DDOS攻击？
SSDP全称为Simple Service Discovery Protocol（普通服务发现协议），这是一种用于探索网络服务的网络协议。它主要用于连接互联网的Wifi路由器，网络摄像头以及许多其他的IOT设备。SSDP允许电子设备能够利用从UDP端口1900上通过的【通用即插即用（UPNP）】消息来轻松地将连接到其他设备。大多数现代电子设备会提供有关初始设置的简要说明，但是一旦设置好，这些UPNP端口就会向全网公开。

一个SSDP攻击将会从多个设备向这些有漏洞的物联网设备请求一个【UPNP发现数据包】，这是为了欺骗IP数据包来获取受害者的IP，然后所有的UPNP回应数据包都将会被发送到受害者的IP。

。SSPD是一种加大功率的攻击如果您向一个这样的设备请求UPNP，它将会回应他全部现有的服务。这一个UPNP数据包请求将会被分成多个回应数据包，并且都将会被发送回受害设备的IP。

![DDOS攻击场景](https://eosfans-static.strahe.com/photo/2018/2db6ceb2-9bde-4eae-ad5b-6897bf957c35.png?x-oss-process=image/resize,w_1920)
<DDOS攻击场景> 图片由https://www.corero.com/提供

### 1. DDOS攻击测试 - 针对于一个应用【Azure DDOS服务】的节点

> 说明：此测试是针对于一个在Azure的标准规格DDOS防卫服务上运行的节点，这是他们 提供的最高保护级别。

DDOS防卫方式：Azure规格（https://docs.microsoft.com/en-us/azure/virtual-network/ddos-protection-overview）

DDOS类型：速率为50Gbps的 SSPD

云提供商：微软Azure

BP节点类型：云

下图：从下面的结果可以看出，IP地址在被攻击期间没有太多反应。

![](https://eosfans-static.strahe.com/photo/2018/bf04df57-f685-4b6e-b501-7e29ab434a97.png?x-oss-process=image/resize,w_1920)

Azure DDOS显示节点在攻击的高峰期收到了大约6.25 GB / s（50 Gbp / s）的流量，节点只能抵挡住5.18 GB / s（41.44 Gbp / s）的流量。

![](https://eosfans-static.strahe.com/photo/2018/76f5c4a0-34e9-4dd3-8ccc-494978e2e6b1.png?x-oss-process=image/resize,w_1920)

总结：Azure能够做到一些保护，但它不能阻止一套完整的攻击，并且这套保护服务漏出了一个足够能将你IP地址以及在此IP上运行的服务丢失的数据包的数量。当受到攻击时，此节点无法正常地同步。

总体结果：抵抗失败

### 2. DDOS攻击测试 - 针对于一个应用【Google 云】并采用Google提供的【标准DDOS防卫】的节点

> 说明：此测试是针对于一个在Google的标准规格DDOS防卫服务上运行的节点，这是他们提供的最高保护级别。

DDOS防卫方式：谷歌云标准

DDOS类型：速率为50Gbps的SSPD

云提供商：Google

BP节点的类型：云

*此不是一个针对于Google Shield的测试。

![](https://eosfans-static.strahe.com/photo/2018/e49f0e8e-254f-4bd8-9b7c-b53272f56510.png?x-oss-process=image/resize,w_1920)

总结：Google Cloud做到了一些保护，但总的来说，节点无法正确地与区块链同步。在这其中实在是存在着太多的延迟以及丢失的数据包了，这也导致了节点无法正确同步。

总体结果：抵抗失败

### 3. DDOS攻击测试 - 针对于一个应用【AWS 标准DDOS防卫盾】的节点

> 说明：此测试是针对于一个在AWS的标准规格DDOS防卫服务上运行的节点（免费的服务），这是他们提供的保护级别中最低的一级。

DDOS防卫方式：AWS 标准DDOS防卫盾（https://aws.amazon.com/shield/tiers/）

DDOS类型：速率为50Gbps的SSPD

云提供商：AWS（亚马孙）

 BP节点的类型：云

![](https://eosfans-static.strahe.com/photo/2018/bd183625-994e-40ba-83f5-f32a8f44998a.png?x-oss-process=image/resize,w_1920)

结论：AWS提供了一些保护，但整体来说节点仍然会崩溃并且无法与区块链同步。同样先前一样，因为存在着太多的延迟以及丢失的数据包，导致了节点无法正确同步。

总体结果：抵抗失败

### 4. DDOS攻击测试 - 针对于一个应用【Google Load Balancing 服务】 (网络负载平衡器) 的节点

> 说明：在这测试其中，使用了一个向外部公开IP地址的Google网络负载均衡器，它将 TCP端口8888和9876上的流量负载平衡到两个运行在内部IP上的EOS节点。

DDOS防卫方式：Google网络负载均衡器

DDOS类型：速率为50Gbps的 SSPD

云提供商：Google

BP节点的类型：云

![](https://eosfans-static.strahe.com/photo/2018/eb3d6f55-377f-49a0-b74a-9d392cdc38af.png?x-oss-process=image/resize,w_1920)

![](https://eosfans-static.strahe.com/photo/2018/d5ac9c1d-205a-455e-8bf3-c8e7156b25ec.png?x-oss-process=image/resize,w_1920)

结论：Google负载均衡器承受了所有DDoS数据包，并没有影响任何传入流量。所有节点都能正确地与网络同步。

总结：抵抗成功

### 5. Incapsula DDOS防卫 –（ 正在测试中，即将推出。）

## 缓解DDOS攻击之策略

以下是一些关于如何保护自己免受DDoS攻击的指南。

• 将节点进行监视，并计划一个【故障转移方案】
最可靠与便利的方法是主动地对节点和网络进行监视，以便在受到攻击时你能随时将生产员密钥迁移到运行在其他IP或网络的节点上。

• 不要创建DNS记录
不要为你的区块生产员创建DNS记录。利用蛮力来扫描连接着一个域名地址记录是一个非常基本的手段。因为攻击者可能会提前收集相关的数据，请更改你之前在测试网中使用过的所有IP。

• 使用【Google负载平衡服务】
目前，根据EOS42的对各种云服务的研究，Google负载平衡服务可提供足够的DDoS保护。

• 不要在互联网上允许任何指向到你区块生成器的【传入连接】
以禁止【传入连接】的手段，亦指你区块生成器的IP是隐藏的。除非攻击者了解你的IP地址，否则他就不能执行攻击。

• 对于那些采用裸金属设备的用户们 – 请购买一个【TCP / UDP DDOS代理保护服务】或使用【Google负载平衡器】
通过这种类的服务，你可以将所有流量转发给区块生成器，并将防火墙配置为仅接受来自DDOS代理IP的流量。你的外部IP仍将暴露，不过因为所以的传入连接都将是通过DDOS代理服务而流入的，所以这个方案提供了一个额外的安全层，

请注意，上述的方法都不是完美的解决方案，因为对DDOS攻击的预防是一个非常困难的任务。

## 一些额外关于防火墙的设置

• 连接限制：设定有限的新连接请求，首选现有连接。
• 年龄过滤：空闲的连接将从IP列表上删除掉。
• 源码率过滤：当与DDoS攻击关联的IP地址数量是有限的时候，不符合标准的外部IP地址将会可以轻易地被识别。
• 动态过滤：创建一个短期过滤规则，并在一时间段后将其删除。

## 终极DDOS防卫 - BGP重路由
如果你有幸拥有一个C类子网和可以处理BGP路由的设备，你可以和一个DDOS擦洗服务合作。

在这种情况下，如果发生DDoS攻击，你将会把你的BGP路径重新发布并通过第三方机构把流量转发到你的IP地址，这将把所有的DDOS数据包擦洗掉（scrubbing），并且只有干净的数据包会传回你的网络。

不幸的是，由于C类子网的高成本以及相关需求规格，目前来说这个方案对我们大多数人来说都是遥不可及的。

## 总体结论

我希望这篇文章能够在社区对于如何防范DDOS攻击的方面做到一点启发。

从现在起到6月2日，EOS42将对社区提供免费的DDOS防御测试。如果你想测试节点，请通过电子邮件`charles@eos42.io`或电报Telegram `@ankh2054`联系我。

所有测试的结果都将会是保密的。


作者：Charles Holtzkampf

（此文章原意以英文为准。- https://blog.eos42.io/ddos-mitigation/）

EOS42电报群: https://t.me/EOS42

EOS42微博: https://weibo.com/eos42

## 本文地址：https://eosfans.io/topics/433


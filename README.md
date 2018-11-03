区块链中引入分片主要是为了解决可扩展性问题，所谓可扩展性是指系统整体的能力随着更多资源的加入会呈现出更高的吞吐，分片属于水平扩展的范畴。

目前主流的分片技术主要有网络分片、交易分片和状态分片。但在实际场景中分片间都不得不进行相互通信，频繁的交易或状态互换会让分片的效果大打折扣；而且很难保证数据可用性。

区块链场景可扩展性问题是否一定要按传统互联网服务的水平扩展思路去解决，答案还需要在实践中不断探索。但已经出现了将区块链场景下的可扩展性和分片提升到新的层次和范畴的技术，就是互操作性。

互操作性的思路是，不应该把所有的区块链应用设计为都由一条链来支撑，这本身就是不可扩展的；应该让应用按需构建自己的链，同时定义一套链和链之间的通信协议，大家都遵循这套协议来做必要性的跨链交易和价值流转。cosmos 定义的 IBC (InterBlockchain Communication protocol) 就是跨链协议的典型代表。

IBC 的核心理念是使链和链之间彼此为对方的轻客户端。在类 BFT 共识协议场景下，由于分叉的概率极低，可以认为当前最新区块的状态是确定的。轻客户端的验证只需要检查最新区块的验证者签名和 Merkle Root。

## IBC 协议关键流程

IBC协议定义的两个主要的交易类型数据包是 IBCBlockCommitTx 和 IBCPacketTx。

IBCBlockCommitTx 负责把交易发起链的当前最新区块的头部信息发送到目标链，目标链可以基于此数据获取到发起链当前最新的 Merkle Root hash 值。

IBCPacketTx 负责传递跨链转移代币的交易信息，具体的交易信息中同时也包含了相应的 Merkle proof。

以下图为例，将 Zone1 当前最新的区块的头传递给 Hub，Hub 就可以获取 Zone1 最新区块的 Merkle Root hash 值。

当 Hub 再接着收到来自 Zone1 的 IBCPacket 数据时，Hub 就可以利用之前的 Merkle Root 来验证当前 Merkle Proof 是否正确。

这里 Hub 需要 知道 Zone1 当前所有有效 validators 的公钥，因为每一个区块头都包含由超过 2/3 的 validators 的私钥签名。

![images](https://github.com/nil-zhang/sharding-using-ibc/blob/master/images/IBC-tx.png)

## 跨链消息传递
不同的 Zone 之间虽然设计为通过 Hub 来完成跨链交易，但消息在 Zone 和 Hub 之间传递还需要引入 Relay 程序，Relay 程序负责从原链生成 Merkle Proof 并组装成 Packet，然后将 Packet 发送到目标链。具体流程如下图所示。

1、客户端构造一个从 Zone A 到 Zone B 的跨链交易，并将交易发送到 Zone A；

2、Zone A 对交易进行验证和处理，如果是有效交易就将交易放到 面向Hub 的消息队列中；

3、Relay 程序监听消息队列，对入队的消息生成对应的 Merkle Proof，然后将消息和 Proof 作为 IBCPacketTx 的 Payload 发送给 Hub；

4、Hub 对消息进行验证，如果 Merkle Proof 有效就给 Zone B 的消息队列中放入消息；

5、Hub 和 Zone B 的 Relay 程序同样在监控 面向 Zone B 的消息队列，对入队的消息构造对应的 Merkle Proof 并发送给 Zone B。

处理结果会以收据的形式沿发送交易相反的路径发送到 Zone A 里面。

![images](https://github.com/nil-zhang/sharding-using-ibc/blob/master/images/IBC-relay.png)


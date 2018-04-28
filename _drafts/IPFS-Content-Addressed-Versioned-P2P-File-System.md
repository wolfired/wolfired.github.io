---
title: IPFS - 内容寻址、版本化、点对点文件系统
layout: post
---

# IPFS - 内容寻址、版本化、点对点文件系统（草案3）
#### Juan Benet &lt;juan@benet.ai&gt;

## **摘要**

星际文件系统（IPFS）是一种点对点分布式文件系统，它试图用相同的文件系统连接所有计算设备。某种程度上来说，IPFS与WEB很类似，但IPFS可以被看作一个BitTorrent群，它使用Git仓库交换对象。<span title="In other words, IPFS provides a high throughput content-addressed block storage model, with contentaddressed hyper links.">换句话说，IPFS提供了一个使用内容寻址超链接进行高吞吐量内容寻址块存储的模型。</span>这形成了广义的Merkle DAG，一种可以在其基础上构建版本化文件系统、块链甚至永久网站的数据结构。<span title="IPFS combines a distributed hashtable, an incentivized block exchange, and a self-certifying namespace.">IPFS结合了分布式哈希表、块交换激励与自我认证命名空间。</span>IPFS没有单点故障问题，也没节点间的信任问题。

## 1. 引言

过去有过多次建立全球分布式文件系统的尝试。其中有的取得重大进展，有的则完全失败。在众多学术尝试中，AFS[6]取得广泛成功并持续使用至今。除此之外[7, ?]则并没有获得更多关注。在学术研究之外，大多数成功的点对点文件分享应用主要面向大型（音频或视频）媒体文件。最值得注意的是，Napster、KaZaA和BitTorrent[2]部署了支持超过1亿用户并发的大型文件分发系统。即使在今天，BitTorrent也保持着大规模部署，每天都有着上千万的活跃节点[16]。这些应用程序服务的用户与分发的文件数量远多于相类似的学术文件系统。然而这些应用程序不是作为构建基础而设计的。虽然已经得到<span title="例如：各种Linux发行版使用BitTorrent传输安装镜像，而暴雪公司则用来分发游戏内容">_广泛应用_</span>，但依然没没有出现一个全球化、低延迟与分布式的文件系统。

也许这是因为对大多数用户来说已经有了一个足够用的系统：HTTP。迄今为止，HTTP是已部署最成功的文件分发系统。透过浏览器，HTTP对技术与社会有着深刻且广泛的影响力。它已经成为通过互联网传输文件事实上的方式。但它并没有利用过去15年发明的优秀文件分发技术。一方面，考虑到数不清的向后兼容和当前既得利益者的阻碍，更新Web基础设施近乎不可能。另一方面，自HTTP诞生以来新的协议已经出现并得到广泛使用。这些新协议欠缺的只是：增强现有的HTTP Web技术，引入新的功能而不会降低用户的体验。

业界已经不再使用HTTP，因为移动小文件相对便宜，即使对于拥有大量流量的小型组织也是如此。但是我们正在进入一个新的数据分发时代，面临新的挑战：（a）托管和分发PB量级的数据；（b）跨组织的大数据计算；（c）大容量的高清点播或实时媒体流；（d）海量数据的版本控制和链接；（e）防止重要文件的意外损坏；等等。总的来说就一句话：“海量数据，随存随取”。因为关键特性与带宽问题，我们已经放弃在各种的新数据分发协议中继续使用HTTP。未来，这些新协议将成为WEB的一部分。

<span title="Orthogonal to efficient data distribution, version control systems have managed to develop important data collaboration workflows.">为了要高效分发数据，版本控制系统已经设法开发重要的数据协作工作流程。</span>分布式源码版本控制系统Git开发了许多有用的方法来建模和实现分布式数据操作。Git工具链提供了大型文件分发系统严重缺乏的多种版本控制功能。受Git启发的新解决方案正在兴起，如[Camlistore](https://perkeep.org/)[?]：一个个人文件存储系统、[Dat](https://datproject.org/)[?]：一个数据协作工具链和数据管理器。Git的出现对文件分发系统的设计产生了深远影响，其内容寻址Merkle DAG数据模型支持强大的文件分发策略。还有待研究的是这种数据结构如何影响面向高吞吐量文件系统的设计，它如何升级Web自身。

本文介绍的IPFS，作为一种新颖的点对点版本控制文件系统，其试图解决前文提到的问题。IPFS从过去许多成功系统中总结经验。以接口为着重点，把各子系统整合成一个完整系统，以达到1+1>2的效果。IPFS的核心原则是把所有数据建模为同一个Merkle DAG。

## 2. 背景

本节回顾了IPFS整合的那些成功的点对点系统的重要属性。

### 2.1 分布式哈希表

分布式哈希表（DHT）广泛应用于协调和维护点对点系统的元数据。如BitTorrent MainlineDHT跟踪了torrent群组的部分节点。

#### 2.1.1 Kademlia DHT

Kademlia[10]是一种流行的DHT，其提供：

1. 在大型网络中也能实现高效查找：平均查询节点数为 $log_2 (n)$ 。（一个1000W节点的网络只要查询20个）。
2. 低协调开销：优化发往其它节点的控制消息数量。
3. 优先选择运行时间最长的节点来抵御各种攻击。
4. 广泛应用于包括Gnutella和BitTorrent等多种点对点网络，形成了超过2000W节点的网络[16]。

#### 2.1.2 [Coral DSHT](http://www.coralcdn.org/)

由于一些点对点文件系统直接在DHT中存储数据块，而这会浪费存储空间和带宽，因为不是所有的节点都需要这些数据块[5]。Coral DSHT从三个重要方面扩展了Kademlia：

1. Kademlia把值存储在那些节点ID离键“最近”（异或运算）的节点中。这就没有考虑到应用程序数据的区域性，忽略了可能已经存储数据的远节点，强制那些并不需要这些数据的近节点存储数据。这浪费了大量的存储空间带带宽。而Coral DSHT则只存储了能提供这些数据块的节点地址。

2. Coral DSHT把DHT API _get_value(key)_ 放宽成 _get_any_values(key)_ (the "sloppy" in DSHT)。这仍然能正常工作，因为Coral DSHT的用户只需要一个（参正常工作的）节点而不是全部。作为回报，Coral DSHT可以只分发数据的一部分到最近的节点从而避免过载（因为一个热门键而导致所有近节点超载）。

3. 另外，Coral DSHT管理着根据区域与范围划分的独立层次结构：集群。这使用节点首先查询同区域的节点，不查询远节点只查找附近数据，大大降低查找延迟。

#### 2.1.3 S/Kademlia DHT

S/Kademlia[1]在两个重要方面扩展了Kademlia以防止恶意攻击：

1. S/Kademlia提供了生成节点ID的安全方案，防止<span title="Sybill attacks">女巫攻击</span>。它要求节点生成<span title="PKI key pair">公共密钥对</span>用于派生出它们的ID、签名节点间的消息。其中一个方案使用<span title="proof-of-work">工作量证明</span>加密谜题来增加生成女巫的成本。

2. S/Kademlia节点通过不相交路径查找值，确保当网络中充斥着大量不诚实节点时，诚实节点也能正常连接。即使有一半的不诚实节点S/Kademlia也能保证85%的成功率。

### 2.2 块交换-BitTorrent



## 密码学原理

#### hash 相关
* collision resistance，H(x) 与 x 的唯一映射关系
    * 保证文件不被篡改
    * 是不可证明的，只有实例函数
    * MD5 是不 collision resistance 的
* hiding:保证由 H(x) 无法推出 x
* puzzle friendly:由 x 无法推出 H(x)
    * 用于作为挖矿的 proof of work

#### hash 封装是不可避免的
> 因为输入空间总比输出空间大

#### digital equivalent of a sealed envelope
* 预测结果不能提前公开，但又要保证不被篡改
```
H(xllnonce)
```
#### 挖矿
* 找到 nonce,与 header 合在一起，hash结果小于目标预值
```
H(block header)<=target
```

#### diffcult to solve, easy to verify
nonce 被发布后，验证很容易。

#### 比特币的 hash 函数
> SHA-256

#### 比特币的账户管理
* 去中心化
* public key,private key
* 对称加密:前提秘钥的分发是安全的
* 非对称加密:加密用的是接收方的公钥，解密用的是接收方的私钥
    * 公钥:账户，私钥：密码

#### 签名
* 对于加密，加密用的是接收方的公钥，解密用的是接收方的私钥
* 对于签名，私钥签名，公钥验证
* 在支付比特币时，比特币的持有者只需要在交易中提交公钥和签名即可。比特币中的所有人，都可以通过所提交的公钥和签名进行验证，并确认该交易是否有效。
* 私钥是一个数字，通常是随机选出来的。公钥从私钥通过椭圆曲线乘法计算得到。

---
## 数据结构

#### hahs pointer
* 普通指针存储的是节点在内存中的位置
* hash pointer
> 不止是位置，还能检测出该节点是否被篡改(Hash)
```
// 区块链
genesis block<-....<- most recent block
```
> hash 累计的做法，使得我们只需保留最后一个节点的hash值，就可以判断是否有人篡改了节点

#### merkle tree
* 叶节点为交易区块（data blocks）。其余节点为含 hash pointer 的节点。
* 叶节点的父节点 hash 值=叶节点 hash 值
* 所有节点都有 hash 值，且子节点 hash 累计值=父节点 hash 值。

#### merkle proof(proof of membership 或 proof of inclusion)
> 是一条路径，起点为待证明区块，终点为 merkle tree 根节点。

#### block header
merkle tree 根 hash 值


#### 轻节点
* 只保存 block header

#### 全节点
* 保存 merkle proof，block header，block body（该区块交易列表）
* merkle proof 由 block body 生成?

#### 轻节点如何判断一个待证明区块在 merkle tree 中?
> 轻节点存储有 block header,它会向全节点请求待证明区块对应的 merkle proof（准确说是一组构成路径的 hash 数组，据此配合待证明区块的 hash 值，可算出一个 根节点 hash 值，与轻节点存储有 block header 对比，即可知待证明区块是否在该在 merkle tree 中）。


#### 是否可证明某个轻节点不包含在merkle tree中？即proof of non-membership。
> 验证复杂度是线性的，难以证明，没有更高效的方法

#### 有环数据结构不适合hash指针

---
## 为什么要允许争夺记账权?

#### double spend attack
数字货币不可修改，但可以复制!!!

#### double spend attack 预防措施
> 维护一个交易列表

#### 去中心化的货币
* 谁来发行货币（铸币权）
> 由挖矿决定
* 如何预防 double spend attack
> hash pointer
* 去中心化后，数据如何同步？
> 取得分布式共识

#### block header
> 包含以下信息
* version
* hash of previous block header
* merkle root hash
* target
* nonce

#### 分布式共识
* FLP
在一个异步的系统里，只要有一个成员有问题，就无法达成共识
* CAP Theorem

#### 记账权
在本地根据一定的算力/运气获得符合要求的 nonce,则 可以发布该 nonce

#### 最长合法链
* 我们总是只承认当前最长链是合法的
* forking attack：不允许向中间某个节点插入分支节点
* 可能暂时出现相同长度的两条合法链，那么维持现状一段时间，直到某一分支找到下一个节点（算力。运气）

#### 为什么要允许争夺记账权
* block reward
> 可以获得发行一定量货币的权利（唯一产生新币的权利）。在比特币中，每个21万个区块，block reward 减半（50->25->12.5）
* coinBase
> 每笔 block reward 对应一个 coinBase。用 coinBase 可以修改 merkle tree，从而达到扩展 nonce 搜索空间的目的

---
## 比特币的实现

#### UTXO
* 全称为 Unspent Transaction
* 记录如下信息：产生此未花费输出的交易hash指针，此未花费输出属于此交易的第几个输出
* 由全节点维护

#### progress free
* 整个区块产生下一个区块的时间概率，服从指数分布。
* 这次过了10分钟没挖到矿，下次可能还要10分钟。
* 保证算力强的矿工，挖到矿的概率虽然相对高，但不会高到离谱

#### 比特币发行总量总量是固定的
* 产生出来比特币的数量2100w（210000*50+210000*12.5+210000*6.25......=2100万）BTC, 呈几何序列 geometric series
* 比特币的稀缺性是人为造成的

#### BitCoin is secured by mining
* 只要大部分算力掌握在诚实的节点手里，整个系统的安全性是有保障的
* 挖矿对于矿工有利可图，而对于整个交易系统而言，意义在于维护交易的安全性。

#### irrevocable ledger（区块链账本是不可篡改的）
* 当某笔交易刚加入区块链时，不被认为是不可篡改的。直到多个后续交易被确认(three confirmation,six confirmation)，形成较长后续链后，才认为该交易是不可被篡改的。

#### selfish money
* 非法交易挖到区块（找到 nonce）后先不发布，直到形成较长后续链后，覆盖合法交易链。
* 但是该做法需要较强算力，不计较 block reward，这样有强算力的非法矿工还是比较少数的。
---
## 挖矿难度

#### 调整挖矿难度
调整目标空间于输出空间中所占比例

#### 难度低是好事吗?
出块时间越短，诚实节点算力被分散。

#### 调整频率
2016个区块调整一次挖矿难度

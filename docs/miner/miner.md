<h1 align="center">挖矿</h1>

为提升公链网络的活跃度，annchain 支持启用激励模式，用以奖励成功提块的节点。节点可以获取一定份额的share，并抵押share参与竞选共识委员会。成功参选共识委员会后，参与提块并获得相应的奖励。

### share是什么

share是annchain用于参选共识委员会的份额。annchain 在初始搭建链网路时，通过配置文件决定share的初始发行规模。此后share不会再次增发，总数量恒定。

查询share示例

	./anntool query share --account_pubkey 6F7FF67AC795CF39D7B1E14BB97998F3420F877D5934F91C63DAEBA711A8EC4D

### 如何获取share

share份额总数恒定，但是可以在账户之间转移。share份额由一对公私钥确定归属权，每个参与者都可以生成一对公私钥，通过各种途径获取share份额，参与竞选共识委员会。

transfer 示例

	./anntool share send --nodeprivkey 2AB8D84BE5DA3BE9F5B26403BD3CF864AD58B1B447984C42EF79E6CB9D7747AA --to 7A5F697438CB432F512BEEA9301D602F71EF4019 --value 100 --nonce 0

### 参选共识委员会

> 抵押

持有share份额的参与者，通过发送签名并发送指定交易到链上，抵押自己的部分或全部share份额。在到达竞选时间之前，参与者可以选择撤回。如果参选成功，则share份额将被冻结。

抵押示例

	./anntool share guarantee  --evmprivkey 2AB8D84BE5DA3BE9F5B26403BD3CF864AD58B1B447984C42EF79E6CB9D7747AA --nodeprivkey  95E28888C1FC96B97A4B6959F562B89E4E11BD8569BCDF8AC46A79D27A7336416F7FF67AC795CF39D7B1E14BB97998F3420F877D5934F91C63DAEBA711A8EC4D  --value 100 --nonce 0

> 赎回

成功参选的参与者，如果在下一轮竞选阶段时未成功参选，则可以发送指定交易赎回抵押的share份额。成功参选的参与者也可以通过发送指定交易声明自己不参与下一轮竞选，在当前轮结束之后，即可发送交易赎回抵押的share份额。

赎回与声明不参选示例

	./anntool share redeem  --evmprivkey 2AB8D84BE5DA3BE9F5B26403BD3CF864AD58B1B447984C42EF79E6CB9D7747AA --nodeprivkey  95E28888C1FC96B97A4B6959F562B89E4E11BD8569BCDF8AC46A79D27A7336416F7FF67AC795CF39D7B1E14BB97998F3420F877D5934F91C63DAEBA711A8EC4D  --value 100 --nonce 0

### 竞选规则

每一轮竞选，分为抵押阶段和VRF阶段。抵押阶段允许share持有者发送交易进行share抵押；VRF阶段不允许抵押，现有的共识委员会在此阶段对当前阶段的首个区块Hash进行随机投票，节点收集投票，在当前阶段结束时，使用VRF计算得到一个随机数，用以产生下一阶段的共识委员会。

### 奖励规则

annchain 可以自由制定相应的奖励规则。共识产生的区块中，记录了当前区块的提块节点，开发者可以根据这些信息以及相应的经济模型对节点进行奖励。

### 关于节点

节点拥有唯一的根据可选的椭圆曲线算法生成的公私钥账户。参与挖矿的节点需要通过抵押share来参与出块。

### 关于共识

安链使用PBFT作为其共识算法。
##### PBFT的阶段提交
- PROPOSAL:根据block高度和权重选择某个指定的节点，将交易打包成block并广播， 发起Proposal
- PREVOTE:所有节点验证proposal，广播自己的签名，并收集其他节点的签名
- PRECOMMIT: 如果收到其他节点+2/3权重的pre-vote签名，则签名并广播pre-commit消息
- COMMIT:超过2/3的pre-commit即表示block成功生成

##### 一致性与可用性

- 超过2/3的pre-commit即表示block成功生成
- 当超过1/3权重节点宕机或出现任意故障时，整个系统处于不可用状态，无法处理新的交易请求
- 当恢复到大于2/3权重节点正常时，系统恢复可用状态，可处理新交易请求，系统历史状态不丢失

##### 区块大小
安链的block默认可打包10000笔交易，使用者可根据实际情况修改block中可包含的最大交易量。

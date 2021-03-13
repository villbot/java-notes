# 分布式理论:一致性算法 Paxos

> Paxos算法是Lamport提出的一种基于消息传递的分布式一致性算法，使其获得2013年图灵奖。   
>   ![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Paxos/page25image27422912.png)   
>   
> Paxos由Lamport于1998年在《The Part-Time Parliament》论文中首次公开，最初的描述使用希腊的一个小岛 Paxos作为比喻，描述了Paxos小岛中通过决议的流程，并以此命名这个算法，但是这个描述理解起来比较有挑战 性。后来在2001年，Lamport觉得同行不能理解他的幽默感，于是重新发表了朴实的算法描述版本《Paxos Made Simple》   
> 自Paxos问世以来就持续垄断了分布式一致性算法，Paxos这个名词几乎等同于分布式一致性。Google的很多大型 分布式系统都采用了Paxos算法来解决分布式一致性问题，如Chubby、Megastore以及Spanner等。::开源的 ZooKeeper，以及MySQL 5.7推出的用来取代传统的主从复制的MySQL Group Replication等纷纷采用Paxos算法:: 解决分布式一致性问题。   
然而，Paxos的最大特点就是难，不仅难以理解，更难以实现。 

## Paxos 解决了什么问题 
分布式系统才用多副本进行存储数据 , 如果对多个副本执行序列不控制, 那多个副本执行更新操作,由于网络延迟 超时 等故障到值各个副本的数据不一致.
我们希望每个**副本的执行序列是 [ op1 op2 op3 …. opn ]** 不变的, 相同的.
Paxos 一次来确定不可变变量 opi的取值 , 每次确定完Opi之后,各个副本执行opi操作,一次类推。 
结论: **Paxos算法需要解决的问题就是如何在一个可能发生上述异常的分布式系统中，快速且正确地在集群内部对某个 数据的值达成一致.** 
注:**这里某个数据的值并不只是狭义上的某个数，它可以是一条日志，也可以是一条命令(command)。。。根据应用场 景不同，某个数据的值有不同的含义** 

## 背景
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Paxos/page26image27775040.png) 
我们假设一种情况，在一个集群环境中，要求所有机器上的状态是一致的，其中有2台机器想修改某个状态，机器 A 想把状态改为 A，机器 B 想把状态改为 B，那么到底听谁的呢? 
有的同学会想到，可以像 2PC，3PC 一样引入一个协调者，谁先到，听谁的 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Paxos/page27image27535104.png) 
但是如果，协调者宕机了呢?
所以需要对协调者也做备份，也要做集群。这时候，问题来了，这么多协调者，听谁的呢?
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Paxos/page27image27535520.png) 
Paxos 算法就是为了解决这个问题而生的 

## Paxos相关概念 
首先一个很重要的概念叫提案(Proposal)。最终要达成一致的value就在提案里。
**提案 (Proposal):Proposal信息包括提案编号 (Proposal ID) 和提议的值 (Value)** 
在Paxos算法中，有如下角色: 
	* **Client**:客户端
	客户端向分布式系统发出 ，并等待 。例如，对分布式文件服务器中文件的写请求。 
	* **Proposer**:提案发起者 
	提案者提倡客户请求，试图说服Acceptor对此达成一致，并在发生冲突时充当协调者以推动协议向前发展 
	* **Acceptor**:决策者，可以批准提案 
	Acceptor可以接受(accept)提案;如果某个提案被选定(chosen)，那么该提案里的value就被选定 了 
	* **Learners**:最终决策的学习者 
	学习者充当该协议的复制因素 

![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Paxos/page28image27293296.png) 

### 问题描述
假设有一组可以提出提案的进程集合，那么对于一个一致性算法需要保证以下几点:
	* 在这些被提出的提案中，只有一个会被选定 
	* 如果没有提案被提出，就不应该有被选定的提案。 
	* 当一个提案被选定后，那么所有进程都应该能学习(learn)到这个被选定的value 

### 推导过程
#### 最简单的方案——只有一个Acceptor 
假设只有一个Acceptor(可以有多个Proposer)，只要Acceptor接受它收到的第一个提案，则该提案被选定，该 提案里的value就是被选定的value。这样就保证只有一个value会被选定。 
但是，如果这个唯一的Acceptor宕机了，那么整个系统就无法工作了! 因此，必须要有多个Acceptor! 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Paxos/page29image27526160.png) 

#### 多个Acceptor 
多个Acceptor的情况如下图。那么，如何保证在多个Proposer和多个Acceptor的情况下选定一个value呢? 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Paxos/page29image27525328.png) 
下面开始寻找解决方案。 
首先我们希望即使只有一个Proposer提出了一个value，该value也最终被选定。 
那么，就得到下面的约束: 

::**P1:一个Acceptor必须接受它收到的第一个提案。**:: 

但是，这又会引出另一个问题:如果每个Proposer分别提出不同的value，发给不同的Acceptor。根据P1， Acceptor分别接受自己收到的第一个提案，就导致不同的value被选定。出现了不一致。如下图: 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Paxos/page30image27292256.png) 
刚刚是因为『一个提案只要被一个Acceptor接受，则该提案的value就被选定了』才导致了出现上面不一致的问 题。因此，我们需要加一个规定: 

::**规定:一个提案被选定需要被半数以上的Acceptor接受**:: 

这个规定又暗示了:『一个Acceptor必须能够接受不止一个提案!』不然可能导致最终没有value被选定。比如上图的情况。v1、v2、v3都没有被选定，因为它们都只被一个Acceptor的接受。 
所以在这种情况下，我们使用一个全局的编号来标识每一个Acceptor批准的提案，当一个具有某value值的提案被半数以上的Acceptor批准后，我们就认为该value被选定了. 
根据上面的内容，我们现在虽然允许多个提案被选定，但必须保证所有被选定的提案都具有相同的value值。否则又会出现不一致。
于是有了下面的约束: 

::**P2:如果某个value为v的提案被选定了，那么每个编号更高的被选定提案的value必须也是v。**:: 

一个提案只有被Acceptor接受才可能被选定，因此我们可以把P2约束改写成对Acceptor接受的提案的约束P2a。 

::**P2a:如果某个value为v的提案被选定了，那么每个编号更高的被Acceptor接受的提案的value必须也是v。**:: 

只要满足了P2a，就能满足P2。 

![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Paxos/page31image27289968.png) 

但是，考虑如下的情况:假设总的有5个Acceptor。Proposer2提出[M1,V1]的提案，Acceptor2-5(半数以上)均 接受了该提案，于是对于Acceptor2-5和Proposer2来讲，它们都认为V1被选定。Acceptor1刚刚从宕机状态恢复 过来(之前Acceptor1没有收到过任何提案)，此时Proposer1向Acceptor1发送了[M2,V2]的提案(V2≠V1且 M2>M1)，对于Acceptor1来讲，这是它收到的第一个提案。根据P1(一个Acceptor必须接受它收到的第一个提 案。),Acceptor1必须接受该提案!同时Acceptor1认为V2被选定。这就出现了两个问题: 
	1. Acceptor1认为V2被选定，Acceptor2~5和Proposer2认为V1被选定。出现了不一致。
	2. V1被选定了，但是编号更高的被Acceptor1接受的提案[M2,V2]的value为V2，且V2≠V1。这就跟P2a(如果某个value为v的提案被选定了，那么每个编号更高的被Acceptor接受的提案的value必须也是v)矛盾了。 
	 
所以我们要对P2a约束进行强化! 
P2a是对Acceptor接受的提案约束，但其实提案是Proposer提出来的，所有我们可以对Proposer提出的提案进行约束。得到P2b: 

::**P2b:如果某个value为v的提案被选定了，那么之后任何Proposer提出的编号更高的提案的value必须也是v。**:: 

由P2b可以推出P2a进而推出P2。 那么，如何确保在某个value为v的提案被选定后，Proposer提出的编号更高的提案的value都是v呢? 

只要满足P2c即可: 

::**P2c:对于任意的Mn和Vn,如果提案[Mn,Vn]被提出，那么肯定存在一个由半数以上的Acceptor组成的集合S，满足以下 两个条件中的任意一个:**:: 
::**	1.要么S中每个Acceptor都没有接受过编号小于Mn的提案。**::
::**	2.要么S中所有Acceptor批准的所有编号小于Mn的提案中，编号最大的那个提案的value值为Vn**:: 

从上面的内容，可以看出，从P1到P2c的过程其实是对一系列条件的逐步增强，如果需要证明这些条件可以保证一 致性，那么就可以**进行反向推导:P2c =>P2b=>P2a=>P2**,然后通过P2和P1来保证一致性 

小结：目的在多个proposer和多个Acceptor中选定一个唯一方案
规则：
		1. 如果Acceptor没有接受过提案。有提案就接受最先达到的
		2. Proposer 提交填的时候，如果Acceptor已经接受过提案，提交的提案的value必须和上次一样不变
		3. 如果 Acceptor接受天时，要求提案的编号不能比上次的提案编号更小，如果编号小会忽略，提交提案的值，Acceptor接收时，value必须和上次接收的value值一样。

### 接受提案 
#### Proposer生成提案 
接下来来学习，在P2c的基础上如何进行提案的生成 
这里有个比较重要的思想:
Proposer生成提案之前，应该先去『学习』已经被选定或者可能被选定的value，然后 以该value作为自己提出的提案的value。如果没有value被选定，Proposer才可以自己决定value的值。这样才能达 成一致。这个学习的阶段是通过一个『Prepare请求』实现的。 
于是我们得到了如下的提案生成算法:
1. Proposer选择一个新的提案编号N，然后向某个Acceptor集合(半数以上)发送请求，要求该集合中的每个Acceptor做出如下响应(response)
	* Acceptor向Proposer承诺保证不再接受任何编号小于N的提案。 
	* 如果Acceptor已经接受过提案，那么就向Proposer反馈已经接受过的编号小于N的，但为最大编号的提案的值。 
我们将该请求称为编号为N的Prepare请求。 

2. 如果Proposer收到了半数以上的Acceptor的响应，那么它就可以生成编号为N，Value为V的提案[N,V]。
这里的V是所有的响应中编号最大的提案的Value。
如果所有的响应中都没有提案，那么此时V就可以由Proposer自己选择。 
生成提案后，Proposer将该提案发送给半数以上的Acceptor集合，并期望这些Acceptor能接受该提案。我们称该请求为Accept请求。 


#### Acceptor接受提案 
刚刚讲解了Paxos算法中Proposer的处理逻辑，怎么去生成的提案，下面来看看Acceptor是如何批准提案的 

根据刚刚的介绍，一个Acceptor可能会受到来自Proposer的两种请求，分别是Prepare请求和Accept请求，对这两 类请求作出响应的条件分别如下 
	* Prepare请求:Acceptor可以在任何时候响应一个Prepare请求 
	* Accept请求:在不违背Accept现有承诺的前提下，可以任意响应Accept请求 
因此，对Acceptor接受提案给出如下约束: 
::P1a:一个Acceptor只要尚未响应过任何编号大于N的Prepare请求，那么他就可以接收受这个编号为N的提案。::

### 算法优化
上面的内容中，分别从Proposer和Acceptor对提案的生成和批准两方面来讲解了Paxos算法在提案选定过程中的算法细节，同时也在提案的编号全局唯一的前提下，获得了一个提案选定算法，接下来我们再对这个初步算法做一个小优化，尽可能的忽略Prepare请求 

#### Acceptor 算法优化

![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Paxos/page33image55202064.png) 

> ::如果Acceptor收到一个编号为N的Prepare请求，在此之前它已经响应过编号大于N的Prepare请求。根据P1a，该 Acceptor不可能接受编号为N的提案。因此，该Acceptor可以忽略编号为N的Prepare请求。::   

通过这个优化，每个Acceptor只需要记住它已经批准的提案的最大编号以及它已经做出Prepare请求响应的提案的 最大编号，以便出现故障或节点重启的情况下，也能保证P2c的不变性，而对于Proposer来说，只要它可以保证不 会产生具有相同编号的提案，那么就可以丢弃任意的提案以及它所有的运行时状态信息 

### paxos算法描述
综合前面的讲解，我们来对Paxos算法的提案选定过程进行下总结，那结合Proposer和Acceptor对提案的处理逻辑，就可以得到类似于两阶段提交的算法执行过程 

Paxos算法分为两个阶段。具体如下: 

![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Paxos/page34image55277584.png) 
1. 阶段一:
	* Proposer选择一个提案编号N，然后向半数以上的Acceptor发送编号为N的Prepare请求。 
	* 如果一个Acceptor收到一个编号为N的Prepare请求，且N大于该Acceptor已经响应过的所有Prepare请求 的编号，那么它就会将它已经接受过的编号最大的提案(如果有的话)作为响应反馈给Proposer，同时该 Acceptor承诺不再接受任何编号小于N的提案。 
2. 阶段二: 
	* 如果Proposer收到半数以上Acceptor对其发出的编号为N的Prepare请求的响应，那么它就会发送一个针 对[N,V]提案的Accept请求给半数以上的Acceptor。注意:V就是收到的响应中编号最大的提案的value，如果 响应中不包含任何提案，那么V就由Proposer自己决定。 
	* 如果Acceptor收到一个针对编号为N的提案的Accept请求，只要该Acceptor没有对编号大于N的Prepare 请求做出过响应，它就接受该提案。 
当然，实际运行过程中，每一个Proposer都有可能产生多个提案，但只要每个Proposer都遵循如上所述的算法运行，就一定能够保证算法执行的正确性 

### Learner学习被选定的value 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Paxos/page35image55055648.png) 
1. 方案一: Learner获取一个已经被选定的提案的前提是，该提案已经被半数以上的Acceptor批准，因此，最简单的 做法就是一旦Acceptor批准了一个提案，就将该提案发送给所有的Learner 
很显然，这种做法虽然可以让Learner尽快地获取被选定的提案，但是却需要让每个Acceptor与所有的Learner逐 个进行一次通信，通信的次数至少为二者个数的乘积 
2. 方案二: 
另一种可行的方案是，我们可以让所有的Acceptor将它们对提案的批准情况，统一发送给一个特定的Learner(称 为主Learner), 各个Learner之间可以通过消息通信来互相感知提案的选定情况，基于这样的前提，当主Learner 被通知一个提案已经被选定时，它会负责通知其他的learner 
在这种方案中，Acceptor首先会将得到批准的提案发送给主Learner,再由其同步给其他Learner.因此较方案一而 言，方案二虽然需要多一个步骤才能将提案通知到所有的learner，但其通信次数却大大减少了，通常只是 Acceptor和Learner的个数总和，但同时，该方案引入了一个新的不稳定因素:主Learner随时可能出现故障 
3. 方案三: 
在讲解方案二的时候，我们提到，方案二最大的问题在于主Learner存在单点问题，即主Learner随时可能出现故 障，因此，对方案二进行改进，可以将主Learner的范围扩大，即Acceptor可以将批准的提案发送给一个特定的 Learner集合，该集合中每个Learner都可以在一个提案被选定后通知其他的Learner。这个Learner集合中的 Learner个数越多，可靠性就越好，但同时网络通信的复杂度也就越高 


### 如何保证Paxos算法的活性 
根据前面的内容讲解，我们已经基本上了解了Paxos算法的核心逻辑，那接下来再来看看Paxos算法在实际过程中的一些细节 

活性:最终一定会发生的事情:最终一定要选定value 

假设存在这样一种极端情况，有两个Proposer依次提出了一系列编号递增的提案，导致最终陷入死循环，没有value被选定,具体流程如下: 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Paxos/page36image55307856.png) 

解决:通过选取主Proposer，并规定只有主Proposer才能提出议案。这样一来只要主Proposer和过半的Acceptor 能够正常进行网络通信，那么但凡主Proposer提出一个编号更高的提案，该提案终将会被批准，这样通过选择一个 主Proposer，整套Paxos算法就能够保持活性 

# 分布式理论:一致性算法 Raft 

Raft算法: Raft是一种为了管理复制日志的一致性算法。 Raft提供了和Paxos算法相同的功能和性能，但是它的算法结构和Paxos不同。Raft算法更加容易理解并且更容易构建实际的系统。

Raft将一致性算法分解成了3模块 
	1. 领导人选举 
	2. 日志复制 
	3. 安全性 
Raft算法分为两个阶段，首先是选举过程，然后在选举出来的领导人带领进行正常操作，比如日志复制等。 

## Leader选举 
Raft 通过选举一个领导人，然后给予他全部的管理复制日志的责任来实现一致性。 在Raft中，任何时候一个服务器都可以扮演下面的角色之一: 

	* 领导者(leader):处理客户端交互，日志复制等动作，一般一次只有一个领导者 
	* 候选者(candidate):候选者就是在选举过程中提名自己的实体，一旦选举成功，则成为领导者 
	* 跟随者(follower):类似选民，完全被动的角色，这样的服务器等待被通知投票 

而影响他们身份变化的则是选举。 

![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Raft/page37image55278416.png) 

Raft使用心跳机制来触发选举。当server启动时，初始状态都是follower。每一个server都有一个定时器，超时时 间为election timeout(一般为150-300ms)，如果某server没有超时的情况下收到来自领导者或者候选者的任何 消息，定时器重启，如果超时，它就开始一次选举。 
http://thesecretlivesofdata.com/raft/ 动画演示 

下面用图示展示这个过程: 

1. 初始状态下集群中的所有节点都处于follower 状态。 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Raft/page37image55263856.png) 
2. 某一时刻，其中的一个 follower 由于没有收到 leader 的 heartbeat 率先发生 election timeout 进而发起选举 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Raft/page37image55277792.png) 
3. 只要集群中超过半数的节点接受投票，candidate 节点将成为即切换 leader 状态。 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Raft/page38image55265728.png) 
4. 成为 leader 节点之后，leader 将定时向 follower 节点同步日志并发送 heartbeat。 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Raft/page38image55266768.png) 


## 节点异常
集群中各个节点的状态随时都有可能发生变化。从实际的变化上来分类的话，节点的异常大致可以分为四种类型: 
* leader 不可用;
* follower 不可用;
* 多个 candidate 或多个 leader; 
* 新节点加入集群。 

### leader不可用
下面将说明当集群中的 leader 节点不可用时，raft 集群是如何应对的。 
* 一般情况下，leader 节点定时发送 heartbeat 到 follower 节点。 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Raft/page39image55290848.png) 
* 由于某些异常导致 leader 不再发送 heartbeat ，或 follower 无法收到 heartbeat 。 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Raft/page39image55297872.png)

* 当某一 follower 发生 election timeout 时，其状态变更为 candidate，并向其他 follower 发起投票。 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Raft/page39image55305984.png)

* 当超过半数的 follower 接受投票后，这一节点将成为新的 leader，leader 的步进数加 1 并开始向follower同步日志。 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Raft/page40image55281072.png)

* 当一段时间之后，如果之前的 leader 再次加入集群，则两个 leader 比较彼此的步进数，步进数低的 leader 将 切换自己的状态为 follower。 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Raft/page40image55286480.png)

* 较早前 leader 中不一致的日志将被清除，并与现有 leader 中的日志保持一致。 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Raft/page40image55286688.png)

### follower 节点不可用
follower 节点不可用的情况相对容易解决。因为集群中的日志内容始终是从 leader 节点同步的，只要这一节点再次加入集群时重新从 leader 节点处复制日志即可。

* 集群中的某个 follower 节点发生异常，不再同步日志以及接收 heartbeat。 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Raft/page41image55287936.png) 

* 经过一段时间之后，原来的 follower 节点重新加入集群。 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Raft/page41image55292928.png)  

* 这一节点的日志将从当时的 leader 处同步。 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Raft/page41image55292096.png) 

### 多个 candidate 或多个 leader
在集群中出现多个 candidate 或多个 leader 通常是由于数据传输不畅造成的。出现多个 leader 的情况相对少见， 
但多个 candidate 比较容易出现在集群节点启动初期尚未选出 leader 的“混沌”时期。 

* 初始状态下集群中的所有节点都处于 follower 状态。 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Raft/page42image55283776.png) 

* 两个节点同时成为 candidate 发起选举。 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Raft/page42image55243776.png)

* 两个 candidate 都只得到了少部分 follower 的接受投票。 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Raft/page42image55244816.png)

*  candidate 继续向其他的 follower 询问。 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Raft/page43image55262032.png)

* 由于一些 follower 已经投过票了，所以均返回拒绝接受。 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Raft/page43image55258912.png)

* candidate 也可能向一个 candidate 询问投票。 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Raft/page43image55259536.png)
 
* 在步进数相同的情况下，candidate 将拒绝接受另一个 candidate 的请求。 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Raft/page44image55265312.png)
* 由于第一次未选出 leader，candidate 将随机选择一个等待间隔(150ms ~ 300ms)再次发起投票。 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Raft/page44image55265936.png)

* 如果得到集群中半数以上的 follower 的接受，这一 candidate 将成为 leader。 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Raft/page44image55270512.png)

* 稍后另一个 candidate 也将再次发起投票。 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Raft/page45image55266976.png)

* 由于集群中已经选出 leader，candidate 将收到拒绝接受的投票。 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Raft/page45image55280864.png)

* 在被多数节点拒绝之后，并已知集群中已存在 leader 后，这一 candidate 节点将终止投票请求、切换为 follower，从 leader 节点同步日志。 
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Raft/page45image55282944.png)

### 日志复制(保证数据一致性)
日志复制的过程 
	Leader选出后，就开始接收客户端的请求。Leader把请求作为日志条目(Log entries)加入到它的日志中， 然后并行的向其他服务器发起 AppendEntries RPC复制日志条目。当这条日志被复制到大多数服务器上，Leader 将这条日志应用到它的状态机并向客户端返回执行结果。 

下图表示了当一个客户端发送一个请求给领导者，随后领导者复制给跟随者的整个过程。
![](%E5%88%86%E5%B8%83%E5%BC%8F%E7%90%86%E8%AE%BA%E4%B8%80%E8%87%B4%E6%80%A7%E7%AE%97%E6%B3%95%20Raft/page46image55292304.png) 
 
1. 客户端的每一个请求都包含被复制状态机执行的指令。 
2. leader把这个指令作为一条新的日志条目添加到日志中，然后并行发起 RPC 给其他的服务器，让他们复制这条信息。
3. 跟随者响应ACK,如果 follower 宕机或者运行缓慢或者丢包，leader会不断的重试，直到所有的 follower 最终 都复制了所有的日志条目。 
4. 通知所有的Follower提交日志，同时领导人提交这条日志到自己的状态机中，并返回给客户端。 

**可以看到，直到第四步骤，整个事务才会达成。中间任何一个步骤发生故障，都不会影响日志一致性。**

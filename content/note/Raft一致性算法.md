# 节点状态标识

## 持久化的状态

1. 任期 【term】
2. 在当前任期给谁投了票 【votedFor】
3. 日志条目

## 内存状态

1. 已提交的日志索引最大值
2. 已应用到状态机的日志索引最大值

## Leader节点特殊状态

1. nextIndex[],  每一个从节点的下一个应该发送的日志记录的索引值
2. matchIndex[] , 每一个节点的当前已复制的日志记录的索引最大值。

# RPC请求文档

## AppendEntriesRPC【追加日志请求】

### 请求参数

1. 自身的任期 【term】
2. leader的标识id , 用于follower重定向客户端的请求 【leaderId】
3. 最新日志条目的上一条日志条目的索引     【prevLogIndex】
4. 上一条日志条目的索引的任期    【prevLogTerm】
5. Follower 需要存储的日志条目数组  【entries】
6. Leader 自身的 已提交的日志条目索引值 【leaderCommit】

### 返回值

1. 是否成功 【success】
2. Follower的当前任期. 【currentTerm】

### Follower 响应请求的逻辑

1. 如果leader 任期 小于 follower任期时，返回false 
2. 如果 follower 中 的日志条目中没有找到 与 prevLogIndex ,prevLogTerm 匹配的条目时，返回false
3. Follower 中 日志条目的同样的index 上的位置  的条目的任期 与 leader节点的冲突时，使用主节点的日志条目覆盖
4. 追加所有的新条目到当前日志条目中
5. 如果 leaderCommit 大于 自身的 commitIndex , 将 自身的 commitIndex 设置为 leaderCommit 和 已复制的条目的最大的索引值 中的较小值。

用途：leader发起的请求，用于复制日志到从节点 或者 心跳 。



## RequestVote RPC 【候选者收集选票的请求】

## 请求参数

1. 候选者当前的任期 【term】
2. 发起收集选票的候选者的id 【candidatedId】
3. 候选者最新的一条日志条目的索引值
4. 候选者最新的一条日志条目的任期

## 返回值

1. 候选者是否成功收集选票 【voteGranted】
2. 回应节点自身的 任期 【currentItem】

## 响应逻辑

1. 如果候选者任期 小于 当前节点任期时， 返回false
2. 自身的voteFor 为 空时，并且 候选者的记录 至少与自己 一样新时，返回true。 否则返回false。 

# 日志复制的安全性

1. Leader 发送追加日志的RPC到Follower, 大多数Follower 复制成功后，同意请求。Leader 收到大多数同意后，提交该条目。
2. 候选者 成为 leader 后，写入一个条目指令为空的条目【no-op】，然后向各点发起AppendEntries RPC， 促使各个节点的日志在当前任期之前的节点都提交。   

## leader条目，均为已提交的日志条目，leader崩溃

已提交的条目 意味着 半数以上节点 已经成功复制。当leader 崩溃，新的leader 一定是从已经成功复制的节点产生，不会丢失日志。 

## leader 已写入，大多数子节点已经复制的，leader崩溃

当leader 崩溃，新的leader 一定是从已经成功复制的节点产生，不会丢失日志。

## leader 已写入，子节点未复制的，leader 发起了追加RPC后崩溃

### 在大多数节点复制前，节点接收到了候选者的投票RPC  赢得选票

leader S1 在 index 上 写入 term 后 崩溃 。

S2 赢得选举，成为leader 。

当新的leader S2接受了新的写请求，写入日志条目 index ,term +1 。

假设之后的某段时间S1 节点恢复，成为新的Follwer。

当S2 复制日志到 S1时，index 位置 记录冲突。S1 从 S2 的匹配点开始，复制条目 追加到自身的日志条目中 。【保证每一个index上的条目仅提交一次】

### 在大多数节点复制后，节点接收到候选者的投票RPC , 不可能赢得选票


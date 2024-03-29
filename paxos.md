# paxos 算法

一篇 paxos 入门教程，从基本的分布式中的**复制**的问题出发，通过逐步解决和完善几个相关问题，最后推导出 paxos 的算法。

## 1、分布式系统要解决的问题

![](./images/paxos_00.png)

把一堆运行的机器协同起来，让多个机器成为一个整体系统。在这个系统中，每个机器都必须让系统中的状态达成一致。例如三副本集群如果一个机器上上传了一张图片，那么另外 2 台机器上也必须复制这张图片过来，整个系统才处于一个一致的状态。

分布式系统的一致性问题最终都归结为**分布式存储**的一致性。

几乎所有的分布式存储都必须用某种冗余的方式在廉价硬件的基础上搭建高可靠的存储。而冗余的基础就是**多副本策略**，一份数据存多份。多副本保证了可靠性，而副本之间的一致，就需要 paxos 这类分布式一致性算法来保证。

paxos 是什么？

- 在分布式系统中保证多副本数据**强一致**的算法。


## 2、主从异步复制

![](./images/paxos_01.png)

问题：

1. 数据复制到 slave 节点之前，如果 master 节点宕机，有可能造成数据丢失。不满足可靠性。


## 3、主从同步复制

![](./images/paxos_02.png)

问题：

1. master，slave1，slave2 三个节点有任何一个节点宕机，整个系统就都不可用。不满足可用性。

ps：复制状态机(Replicated State Machine)

![](./images/state_machine.png)


## 4、主从半同步复制

![](./images/paxos_03.png)

既保证了可靠性，也保证了可用性。

问题：

1. 各节点数据不一致，导致选主困难。

![](./images/04_01.png)

2. 当 slave 节点数据可读时，更新数据之后，连续两次读取数据可能不一致。

![](./images/04_02.png)

3. 并发写

![](./images/paxos_04.png)

4. 并发读写

![](./images/paxos_05.png)

5. 节点连续宕机，选主，写入

![](./images/04_04.png)

6. 节点的加入和退出，client 获取节点信息

![](./images/04_05.png)

7. 删除

通过 RSM

![](./images/04_03.png)


## 5、多数派读写

![](./images/paxos_06.png)

节点没有主从之分，不需要进行选举

多数派写：client 向多个节点（大多数节点或者全部节点）发送写请求，只有大多数节点写入成功，该写请求才算成功。

多数派读：client 向多个节点（大多数节点或者全部节点）发送读请求，只有大多数节点返回数据，**并且返回的数据一致**，该读请求才算成功。

问题：
1. 多数派写完之后可能无法进行多数派读

![](./images/05_01.png)

2. 更新数据之后，连续两次读取数据可能不一致

![](./images/05_02.png)

3. 并发写

4. 并发读写

5. 节点的加入和退出，client 获取节点信息

6. 删除


## 6、poxos

主要角色：

![](./images/paxos_timeline_new_proof_1.jpg)

主要过程：

![](./images/paxos_timeline_new_proof_2.jpg)

1. phase-1：prepare(P 操作)

    * 写入和更新之前先多数派读

2. phase-2：accept(A 操作)

    * 修复数据

    * 写入和更新数据（多数派写）

数据写入和读取过程：

![](./images/06_01.png)

并发写：

![](./images/paxos_07.png)

删除，以删除 b(rnd=2) 为例：

1. 通过一定的办法使 3 个节点的状态达成一致，也就是 3 个节点都是 b(rnd=2)
2. 确保所有 Proposer 正在提交的 rnd 都超过 2
3. 确保索引的 Acceptor 拒绝所有小于等于 2 的请求
4. 进行删除操作

删除的同时有可能有 Proposer 提交 rnd = 1，rnd = 2，rnd = 3 的数据

1. 不能直接删除
2. 删除 rnd = 2 的数据之后，Proposer 不能再提交 rnd = 2 的数据
3. 不能反复进行删除操作

![](./images/06_02.png)

优点：没有停机时间，只剩下一台机器也可以正常工作

缺点：

1. 数据不一致对业务侧造成影响，有可能读到旧数据
2. 数据恢复需要从头开始进行修正，比较困难
3. 加入新节点之后也从头开始进行修正，比较困难

改进：

独立进程进行数据修正

![](./images/paxos_08.png)


参考

* https://zhuanlan.zhihu.com/p/145044486


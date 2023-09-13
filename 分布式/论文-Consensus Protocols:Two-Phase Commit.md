# Consensus Protocols: Two-Phase Commit

For the next few articles here, I’m going to write about one of the most fundamental concepts in distributed computing - of equal importance to the theory and practice communities. The *consensus problem* is the problem of getting a set of nodes in a distributed system to agree on something - it might be a value, a course of action or a decision. Achieving consensus allows a distributed system to act as a single entity, with every individual node aware of and in agreement with the actions of the whole of the network.

>the theory and practice communities 理论界和实践界
>
>The *consensus problem* is the problem of getting a set of nodes in a distributed system to agree on something 
>
>共识问题是让分布式系统中的一组节点就某件事达成一致的问题
>
>a course of action 一个行动过程
>
>每个节点知道并同意整个网络的行为

For example, some possible uses of consensus are:

- deciding whether or not to commit a transaction to a database
- synchronising clocks by agreeing on the current time
- agreeing to move to the next stage of a distributed algorithm (this is the famous *replicated state machine* approach)
- electing a leader node to coordinate some higher-level protocol

> 协商一致的一些可能用途：
>
> 决定是否将事务提交到数据库
>
> 通过商定当前时间来同步时钟
>
> 同意进入分布式算法的下一阶段（这是著名的复制状态机方法）
>
> 选择一个领导节点来协调一些更高级别的协议

Such a simple-sounding problem has surprisingly been at the core particularly of theoretical distributed systems research for over twenty years. How come? As I see it, the answers are threefold.

> simple-sounding 听起来很简单

First, it turns out that consensus is a sufficiently difficult problem that it can be used to characterize the differences between different strengths of system model. As [highlighted in my post on the FLP result](http://hnr.dnsalias.net/wordpress/?p=49), consensus is solvable in synchronous systems, but *not* in asynchronous ones if there is the possibility of any failures. Therefore the problem is at the phase boundary between the two models and turns out to be heavily dependent on timing assumptions.

> 首先，一致性是明显很困难的问题，在被用于表征系统模型不同强度之间的差异。正如在我关于FLP结果的帖子中强调的，一致性在同步系统中可以被解决，但是在异步系统中如果存在故障的可能性，则不好解决。因此，问题处于两个模型之间的相位边界，并在很大程度上取决于时间假设。

Second, getting a consensus protocol right is hard. Simple solutions don’t work very well, exhibiting undesirable behaviours and occasionally acting incorrectly. Mike Burrows, inventor of the Chubby service at Google, says that “there is only one consensus protocol, and that’s Paxos” - all other approaches are just broken versions of Paxos. The Paxos protocol, conceived by Leslie Lamport, is famously subtle and a bit difficult to understand (although Lamport didn’t necessarily help this problem by presenting the protocol intially by an allegory to Greek parliamentary proceedings). Whether or not you believe this point of view, it’s clear that a good consensus protocol is surprisingly hard to find.

> 其次，保证共识协议正确性也是很困难的
>
> subtle    adj.微妙的;  巧妙的;  狡猾的;  敏锐的;  不易察觉的;  不明显的;  机智的;  机巧的;  
>
> allegory n.寓言;  讽喻;  寓言体;  讽喻法;  
>
> （尽管兰波特并不一定有助于解决这个问题，他最初以寓言的方式向希腊议会程序介绍了该议定书）

Thirdly, consensus is an important problem. Distributed databases rely on it. In fact, most distributed systems are built upon it. Group membership systems, fault-tolerant replicated state machines, data stores - all the typical distributed systems are at some level predicated upon being able to solve consensus. Partly, this is because consensus is isomorphic to another important problem - *atomic broadcast*, which is the ability to deliver messages in a network reliably and in a total order to all nodes. We will see later in these articles why atomic broadcast and consensus are two instances of the same problem.

> 第三，一致性是很重要的问题
>
> Partly 部分原因是
>
> [isomorphic](javascript:) 同构的 ; 同形的       broadcast 广播

## Formal Definition 形式化定义

So consensus is very important. The first thing a theoretician does with apparently important problems is to formalise them, the better for reasoning about them. I’m going to keep the theory light here, but we need to get straight what constitutes a solution to the consensus problem. Thankfully, it involves only three properties.

> theoretician 理论家
>
> the better for reasoning about them 更有利于对他们进行推理
>
> keep the theory light 保持理论的简洁

We say that a node *decides* once it has reached it’s decision about the value it thinks everyone else agrees on. More formal definitions say that each node has an output register that, once the protocol has terminated, contains the value to agree upon. Writing to that register is the act of *deciding*.

> 一个节点在达到它认为其他人都同意的值后就决定了他的决定。
>
> an output register 输出寄存器
>
> 更正式的定义是，每个节点都有一个输出寄存器，一旦协议终止，它就包含要同意的值。写信到登记册是决定的行为。

Nodes *propose* values by suggesting them as the value to agree upon. The value that a node proposes is considered pre-determined - the protocol should not mandate the node to propose any particular value (we’ll see exactly why shortly).

> 节点通过建议值作为要商定的值来提出值。节点提出的值被认为是预先确定的——协议不应该强制节点提出任何特定的值（我们很快就会明白原因）。

With those two definitions in mind, and given a set N of nodes in a distributed system, we’ll consider a consensus protocol correct if and only if:

1. *Agreement* - all nodes in *N* decide on the same value
2. *Validity* - the value that is decided upon must have been proposed by some node in *N*
3. *Termination* - all nodes eventually decide

> 考虑到这两个定义，给出一个集合，在分布式系统中的N个节点中，我们考虑一个正确的共识协议，当且仅当：
>
> 1. 协议     - 在 N 中所有 node 有相同的值
> 2. 有效性 - 决定的值必须由N中的某个节点提出
> 3. 终止     - 所有节点最终决定

Agreement is an easy one to understand: we can’t really call something consensus if no consensus has been achieved. We might consider weakening this requirement to say that only a majority of nodes have to be in agreement, but that doesn’t actually change the strength of the problem or its required solutions much.

> Agreement 协议

Validity is a bit less intuitive - and seems kind of obvious at the same time. The problem with the agreement property is that it trivially allows protocols that just instruct every node to decide a default value, no matter what the actual state of the network. For example a database commit protocol that always voted “don’t commit”, no matter what the transaction, satisfies the agreement condition. Validity ensures that these protocols don’t pass muster.

> intuitive  adj. 直觉的;  易懂的;  有直觉力的;  使用简便的;  凭直觉得到的;  
>
> property  n. 财产;  性质;  房地产;  不动产;  财物;  庄园;  所有物;  房屋及院落;  
>
> trivially     adv. 平凡地;  平凡;  琐细地;  
>
> 协议属性的问题在于，它只允许协议指示每个节点决定默认值，而不管网络的实际状态如何。例如，无论事务是什么，始终投票“不提交”的数据库提交协议都满足协议条件。有效性确保这些协议不会通过审查。

Termination is also a reasonable requirement, but it bears thinking about why it’s needed - agreement only constrains *how* nodes decide, not *if* they decide. We need termination to ensure that the protocol actually does something useful, otherwise again we’d be open to trivial solutions that just don’t do anything - satisfying agreement and validity.

> reasonable requirement 合理的要求
>
> bears thinking about  值得思考
>
> 终止也是一个合理的要求，但值得思考为什么需要终止——协议只限制节点如何决定，而不是他们是否决定。我们需要终止协议，以确保协议确实做了一些有用的事情，否则我们将再次对琐碎的解决方案持开放态度，这些解决方案什么都不做——满足协议和有效性。

Note that we’ve not said anything - yet - about tolerance to failures. Designing a protocol that works even when participants may crash and fail will turn out to be crucially difficult, so we’ll be revisiting the effect of failures later.

> 请注意，我们还没有说过任何关于容忍失败的话。设计一个即使参与者可能崩溃和失败也能工作的协议将是极其困难的，因此我们稍后将重新审视失败的影响。

## Two-Phase Commit

Simple solutions to the consensus problem seem obvious. Think about how you would solve a real world consensus problem - perhaps trying to arrange a game of bridge for four people with three friends. You’d call all your friends in turn and suggest a game and a time. If that time is good for everybody you have to ring round and confirm, but if someone can’t make it you need to call everybody again and tell them that the game is off. You might at the same time suggest a new day to play, which then kicks off another round of consensus.

> a game of bridge 一场桥牌游戏
>
> the game is off    比赛结束
>
> kicks off 开启

This is a very natural way - although slightly simplified, omitting several human-specific optimisations - to achieve consensus. We can identify two steps in the process:

1. Contact every participant, suggest a value and gather their responses
2. If everyone agrees, contact every participant again to let them know. Otherwise, contact every participant to *abort* the consensus.

> although slightly simplified 尽管略有简化
>
> [omitting](javascript:) 忽略 ; 遗漏 ; 删除 ; 漏掉 ; 不做 ; 未能做 ; omit的现在分词
>
> human-specific optimisations 针对人类的优化

This skeleton describes possibly the simplest, most often used consensus algorithm called *two-phase commit*, or 2PC. As its name suggests, 2PC operates in two distinct phases. The first *proposal* phase involves proposing a value to every participant in the system and gathering responses. The second *commit-or-abort* phase communicates the result of the vote to the participants and tells them either to go ahead and decide or abort the protocol.

> skeleton   n.(建筑物等的)骨架，框架;  骨架;  骨骼;  梗概;  骨骼标本;  最少人员，基干人员;  骨瘦如柴的人(或动物);    
>
> ​                  adj.骨瘦如柴的;  (计划,组织或结构等)基本的;  
>
> 第一个提议阶段涉及向系统中的每个参与者提议一个值并收集响应。第二个提交或中止阶段将投票结果传达给参与者，并告诉他们继续决定或中止协议。

The process that proposes values is called the *coordinator*, and does not have to be specially elected - any node can act as the coordinator if they want to and therefore initiate a round of 2PC.

> 提出值的过程称为协调器，不必特别选择——任何节点都可以充当协调器（如果他们愿意），因此可以发起一轮2PC。

Does 2PC, as described, without worrying about failures, solve the consensus problem? Agreement is easily shown: every node decides the value proposed by the coordinator in phase one if and only if they are told do so by the coordinator in phase two. The coordinator sends the same decision to every node, so if one is told to decide a value they all are.

> 2PC是否如前所述，在不担心失败的情况下解决了共识问题？协议很容易显示：每个节点决定第一阶段协调器提出的值，当且仅当第二阶段协调器告诉它们这样做时。协调器向每个节点发送相同的决定，因此，如果一个节点被告知要决定一个值，那么它们都是。

Validity is also satisfied. 2PC does not trivially commit or abort - it aborts in every case except the one where every node agrees to commit. In all cases, the final value decided upon has been voted on by at least one node.

> 有效性也令人满意。2PC不会简单地提交或中止—它在任何情况下都会中止，除了每个节点都同意提交的情况。在所有情况下，所决定的最终值都已由至少一个节点投票决定。

Finally termination is guaranteed if every node is guaranteed to make progress and eventually return its vote to the coordinator, which then eventually communicates them to every node. Note that even asynchronous models guarantee this property in the absence of failures: eventually every message is processed and all responses are sent. There are no loops in the specification of 2PC, so no way for it to continue executing forever.

> 最后，如果保证每个节点都取得进展，并最终将其投票返回给协调器，协调器最终将它们传递给每个节点，则可以保证终止。请注意，即使异步模型在没有故障的情况下也能保证此属性：最终处理每个消息并发送所有响应。2PC规范中没有循环，因此它无法永远继续执行。

Therefore, 2PC does offer a solution to consensus. It has the benefit of being quite efficient - the number of messages exchanged is 3*n* for n nodes; it’s hard to see how that could be improved upon greatly.

However, as I have strongly hinted earlier, there is a fly in 2PC’s ointment. If nodes are allowed to fail - if even a single node can fail - then things get a good deal more complicated.

> 因此，2PC确实提供了达成共识的解决方案。它的优点是相当有效——n个节点交换的消息数为3n；很难看出如何能大大改进。
> 然而，正如我先前强烈暗示的，2PC的软膏里有一只苍蝇。如果允许节点发生故障——即使单个节点也可能发生故障——那么事情就会变得复杂得多。

## Crashes and Failures

As discussed elsewhere on this blog, nodes can crash in several ways. Most simply, they can crash and never recover. This is the ‘fail-stop’ model of distributed system failure. Instead, nodes could crash and may at some later date recover from the failure and continue executing. This is the ‘fail-recover’ model. Most generally, a node could deviate from the protocol specification in completely arbitrary ways. This is called ‘Byzantine failure’, and designing protocols to cope with it is an active area of research, and rather difficult. We consider the most commonly used model of failure, fail-recover (it is also arguably true that this is the most commonly observed failure mode as well).

>[deviate from](javascript:) 偏离 ; 不同于…                  specification n.规范;  规格;  说明书;  明细单;  
>
>cope with it  应对它
>
>正如本博客其他地方讨论的，节点会以几种方式崩溃。最简单的是，他们会崩溃而且永远无法恢复。这是分布式系统故障的“故障停止”模型。相反，节点可能会崩溃，并可能在稍后的某个日期从故障中恢复并继续执行。这是“故障恢复”模型。通常，节点可以完全任意地偏离协议规范。这被称为“拜占庭失败”，设计协议来应对它是一个活跃的研究领域，也是相当困难的。我们考虑最常用的故障模型，故障恢复（这也是可以证明的事实，这也是最常见的观察到的故障模式）。

How could a node failure throw a spanner in the works for 2PC? To answer this, we have to look at every state the protocol can take, and consider what happens if either the coordinator or one of the participants crashes in that state.

> 一个节点的故障怎么可能让2PC的工作陷入困境？要回答这个问题，我们必须查看协议可以采取的每个状态，并考虑如果协调器或参与者之一在该状态下崩溃会发生什么。

In phase one, before any messages are sent out, the coordinator could crash. This doesn’t cause us too much worry, as it simply means that 2PC never gets started (and therefore, vacuously, works correctly). If a participant node crashes before the protocol is initiated then nothing bad happens until the proposal message fails to reach the crashed node, so we can deal with that failure later.

> 在第一阶段，在发送任何消息之前，协调器可能会崩溃。这并没有引起我们太多的担心，因为它只是意味着2PC永远不会启动（因此，在真空中，工作正常）。如果一个参与者节点在协议启动之前崩溃，那么在建议消息未能到达崩溃的节点之前，不会发生任何不好的事情，因此我们可以稍后处理该故障。

Now consider the protocol after some of the proposal messages have been sent, but not all of them. If the coordinator crashes at this point we’ll have some nodes that have received a proposal and are starting a 2PC round, and some nodes that are unaware that anything is going on. If the coordinator doesn’t recover for a long time, the nodes that received the proposal are going to be blocked waiting for the outcome of a protocol that might never be finished. This can mean that no other instances of the protocol can be succesfully executed, as participants might have to take locks on resources when voting ‘commit’. These nodes will have sent back their votes to the coordinator - unaware that it has failed - and therefore can’t simply timeout and abort the protocol since there’s a possibility the coordinator might reawaken, see their ‘commit’ votes and start phase two of the protocol with a commit message.

> protocol	n.协议;  议定书;  (协议或条约的)附件;  (数据传递的)规程;  规约;  礼仪;  外交礼节;  条约草案;  科学实验计划;  
>
> ​                   v.拟(草案)；(把…)记入议定书;  
>
> proposal   n.建议;  提议;  动议;  求婚;  
>
> 现在，在发送了一些建议消息（但不是全部消息）之后考虑协议。如果协调器在此点崩溃，则会有一些节点已收到建议并正在开始2PC轮，还有一些节点不知道发生了什么。如果协调器长时间未恢复，则会阻止接收建议的节点，等待可能永远不会完成的协议结果。这意味着不能成功执行协议的其他实例，因为参与者在投票“提交”时可能必须锁定资源。这些节点会将它们的投票发回协调器—不知道它失败了—因此不能简单地超时并中止协议，因为协调器可能会重新唤醒、查看它们的“提交”投票并使用提交消息启动协议的第二阶段。

The protocol is therefore blocked on the coordinator, and can’t make any progress. We can add some mechanisms to deal with this - and we’ll describe these below - but this problem of being stuck waiting for some participant to complete their part of the protocol is something that 2PC will never quite shrug off.

> mechanisms 机制
>
> 因此，协议在协调器上被阻止，无法取得任何进展。我们可以添加一些机制来处理这个问题——我们将在下面描述这些机制——但是，等待参与者完成他们的协议部分的问题是2PC永远不会忽视的。

What we can do to paper over this crack, however, is to get another participant to take over the job of the co-ordinator once we realise that it has crashed. When a timeout occurs at some node, we can force that node to finish off the protocol that the co-ordinator started. This node can contact all the other participants, as in a phase one message, and find out which way they voted. This requires all nodes to keep in persistent storage the results of all 2PC executions, until they know that every other node has committed or aborted - otherwise if all nodes forget what they did for a given execution then the recovery node that takes over from the co-ordinator will have no way of recovering the state of the transaction. It’s also possible that only one node might know the result of the transaction if the co-ordinator fails in phase two before all nodes are told of they abort / commit decision.

> 然而，我们可以做的是，一旦我们意识到协调者的崩溃，就让另一个协调者去接管它的工作。当某个节点发生超时时，我们可以强迫节点完成协调器启动的协议。这个节点可以联系到其他参与者，例如在第一阶段的参与者，然后找出他们投票的方式。这要求所有节点将所有2PC执行的结果保存在持久性存储中，直到它们知道其他每个节点都已提交或中止—否则，如果所有节点忘记了它们对给定执行所做的操作，则从协调器接管的恢复节点将无法恢复事务的状态。也有可能只有一个节点知道事务的结果，如果协调器在第二阶段失败，那么所有节点都会被告知中止/提交决策。

However, if another participant node crashes before the recovery node can finish the protocol, the state of the protocol cannot be recovered. The recovery node can’t distinguish between all nodes having already voted to commit and the failed participant having committed (in which case it’s invalid to default to abort) and all nodes but the failed node having voted to commit and the failed node having voted to abort (in which case it’s invalid to default to commit).

> 但是，如果另一个参与者节点在恢复节点完成协议之前崩溃，则无法恢复协议的状态。恢复节点无法区分已投票提交的所有节点和已提交的失败参与者（在这种情况下，默认为中止无效）以及除投票提交的失败节点和投票中止的失败节点之外的所有节点（在这种情况下，默认为提交无效）。

The same kind of arguments apply - as broadly already covered - in phase two. If the co-ordinator crashes before the commit message has got to all participants, we need a recovery node to take over and safely guide the protocol to its conclusion.

>在第二阶段，同样的论据也适用——正如已经广泛涵盖的那样。如果协调器在提交消息到达所有参与者之前崩溃，我们需要一个恢复节点来接管并安全地指导协议得出结论。

The worst case scenario is when the co-ordinator is itself a participant, and grants itself a vote on the outcome of the protocol. Then a crash to the co-ordinator takes out both it and a participant, guaranteeing that the protocol will remain blocked, and as a result of only one failure.

> 最坏的情况是，协调人本身就是参与者，并授予自己对议定书结果的投票权。然后，对协调器的崩溃会使它和参与者都失效，从而保证协议将保持阻塞状态，并且只会导致一次失败。

The co-ordinator typically will log the result of any succesful protocol in persistent storage, so that when it recovers it can answer inquiries about whether a transaction committed. This allows periodic garbage collection of the logs at the participant nodes to take place: the coordinator can tell nodes that no-one will try to recover a mutually committed transaction and that they can erase its existence from their log (that said, the log might be kept around to be able to recover a node’s state after a crash).

> 协调器通常会将任何成功协议的结果记录在持久性存储中，以便在恢复时可以回答有关是否提交了事务的查询。这允许在参与者节点上对日志进行定期垃圾收集：协调器可以告诉节点没有人会尝试恢复相互提交的事务，并且可以从其日志中删除它的存在（也就是说，日志可能会保留，以便在崩溃后恢复节点的状态）。

## Conclusions

2PC is still a very popular consensus protocol, because it has a low message complexity (although in the failure case, if every node decides to be the recovery node the complexity can go to �(�2)*O*(*n*2) ). A client that talks to the co-ordinator can have a reply in 3 message delays’ time. This low latency is very appealing for some applications.

> 2PC仍然是一种非常流行的一致性协议，因为它具有较低的消息复杂度（尽管在故障情况下，如果每个节点都决定作为恢复节点，那么复杂度可以达到（2）O（n2））。与协调员交谈的客户端可以在3个消息延迟时间内得到答复。这种低延迟对于某些应用程序非常有吸引力。

However, the fact the 2PC can block on co-ordinator failure is a significant problem that dramatically hurts availability. If transactions can be rolled back at any time, then the protocol can recover as nodes time out, but if the protocol has to respect any commit decisions as permanent, the wrong failure can bring the whole thing to a juddering halt.

> 然而，2PC可能阻塞协调器故障这一事实是一个严重影响可用性的问题。如果事务可以在任何时候回滚，那么协议可以在节点超时时恢复，但是如果协议必须将任何提交决策视为永久性的，那么错误的失败可能会使整个过程陷入停顿。

That said, there are still cutting-edge research projects that embrace 2PC. The most significant recently is [Sinfonia](http://research.microsoft.com/users/aguilera/papers/sinfonia-aguilera-sosp2007.pdf) (PDF), which won a best paper award at SOSP 2007. They build a kind of distributed memory on top of 2PC but make co-ordinator crashes recoverable from (via a time out and the intervention of a recovery node). Memory node crashes are considered ‘game over’ scenarios (but the memory nodes themselves may be highly replicated). It’s a good paper and I recommend a read.

> 也就是说，仍然有尖端的研究项目采用2PC。最近最重要的是Sinfonia（PDF），它在SOSP 2007中获得了最佳论文奖。它们在2PC上构建了一种分布式内存，但使协调器崩溃可以从中恢复（通过超时和恢复节点的干预）。内存节点崩溃被认为是“游戏结束”场景（但内存节点本身可能被高度复制）。这是一篇好论文，我建议大家读一读。

Next time I’ll describe 3PC, which removes the blocking problems from 2PC at the cost of an extra message delay. Leave comments or questions in the blog, or [mail me directly](mailto:henry.robinson@gmail.com).

> 下次我将描述3PC，它以额外的消息延迟为代价从2PC中消除阻塞问题。在博客中留下评论或问题，或者直接给我发邮件。

## 一致性问题

在分布式系统中，一致性(Consistency，早期也叫 Agreement)是指对于系统中的多个服务节点，给定一系列操作，在协议（往往通过某种共识算法）保障下，试图使得它们对处理结果达成某种程度的一致。

如果分布式系统能实现“一致”，对外就可以呈现为一个功能正常的，且性能和稳定性都要好很多的“虚处理节点”。

举个例子，某影视公司旗下有西单和中关村的两个电影院，都出售某电影票，票一共就一万张。那么，顾客到达某个电影院买票的时候，售票员该怎么决策是否该卖这张票，才能避免超售呢？当电影院个数更多的时候呢？

这个问题在人类世界中，看起来似乎没那么难，你看，英国人不是刚靠 [投票](http://www.bbc.com/news/politics/eu_referendum/results) 达成了“某种一致”吗？

*注意：一致性并不代表结果正确与否，而是系统对外呈现的状态一致与否，例如，所有节点都达成失败状态也是一种一致。*

### 挑战

在实际的计算机集群系统（看似强大的计算机系统，很多地方都比人类世界要脆弱的多）中，存在如下的问题：

* 节点之间的网络通讯是不可靠的，包括任意延迟和内容故障；
* 节点的处理可能是错误的，甚至节点自身随时可能宕机；
* 同步调用会让系统变得不具备可扩展性。

要解决这些挑战，愿意动脑筋的读者可能会很快想出一些不错的思路。

为了简化理解，仍然以两个电影院一起卖票的例子。可能有如下的解决思路：

* 每次要卖一张票前打电话给另外一家电影院，确认下当前票数并没超售；
* 两家电影院提前约好，奇数小时内一家可以卖票，偶数小时内另外一家可以卖；
* 成立一个第三方的存票机构，票都放到他那里，每次卖票找他询问；
* 更多……

这些思路大致都是可行的。实际上，这些方法背后的思想，**将可能引发不一致的并行操作进行串行化**，就是现在计算机系统里处理分布式一致性问题的基础思路和唯一秘诀。只是因为计算机系统比较傻，需要考虑得更全面一些；而人们又希望计算机系统能工作的更快更稳定，所以算法需要设计得再精巧一些。

### 要求

规范的说，理想的分布式系统一致性应该满足：

* 可终止性（Termination）：一致的结果在有限时间内能完成；
* 共识性（Consensus）：不同节点最终完成决策的结果应该相同；
* 合法性（Validity）：决策的结果必须是其它进程提出的提案。

第一点很容易理解，这是计算机系统可以被使用的前提。需要注意，在现实生活中这点并不是总能得到保障的，例如取款机有时候会是“服务中断”状态，电话有时候是“无法连通”的。

第二点看似容易，但是隐藏了一些潜在信息。算法考虑的是任意的情形，凡事一旦推广到任意情形，就往往有一些惊人的结果。例如现在就剩一张票了，中关村和西单的电影院也分别刚确认过这张票的存在，然后两个电影院同时来了一个顾客要买票，从各自“观察”看来，自己的顾客都是第一个到的……怎么能达成结果的共识呢？记住我们的唯一秘诀：核心在于需要**把两件事情进行排序，而且这个顺序还得是大家都认可的**。

第三点看似绕口，但是其实比较容易理解，即达成的结果必须是节点执行操作的结果。仍以卖票为例，初始共有一万张票，如果两个影院各自卖出去一千张，那么达成的结果就是还剩八千张，决不能认为票售光了。

### 带约束的一致性

做过分布式系统的读者应该能意识到，绝对理想的强一致性（Strong Consistency）代价很大。除非不发生任何故障，所有节点之间的通信无需任何时间，这个时候其实就等价于一台机器了。实际上，越强的一致性要求往往意味着越弱的性能。

一般的，强一致性（Strong Consistency）主要包括下面两类：

* 顺序一致性（[Sequential Consistency](https://en.wikipedia.org/wiki/Sequential_consistency)）：Leslie Lamport 1979 年经典论文《How to Make a Multiprocessor Computer That Correctly Executes Multiprocess Programs》中提出，是一种比较强的约束，保证所有进程看到的 全局执行顺序（total order）一致，并且每个进程看自身的执行（local order）跟实际发生顺序一致。例如，某进程先执行 A，后执行 B，则实际得到的全局结果中就应该为 A 在 B 前面，而不能反过来。同时所有其它进程在全局上也应该看到这个顺序。顺序一致性实际上限制了各进程内指令的偏序关系，但不在进程间按照物理时间进行全局排序。
* 线性一致性（[Linearizability Consistency](https://en.wikipedia.org/wiki/Linearizability)）：Maurice P. Herlihy 与 Jeannette M. Wing 在 1990 年经典论文《Linearizability: A Correctness Condition for
Concurrent Objects》中共同提出，在顺序一致性前提下加强了进程间的操作排序，形成唯一的全局顺序（系统等价于是顺序执行，所有进程看到的所有操作的序列顺序都一致，并且跟实际发生顺序一致），是很强的原子性保证。但是比较难实现，目前基本上要么依赖于全局的时钟或锁，要么通过一些复杂算法实现，性能往往不高。

目前，高精度的石英钟的漂移率为 $$10^{-7}$$，人类目前最准确的原子震荡时钟的漂移率为 $$10^{-13}$$。Google 曾在其分布式数据库 Spanner 中采用基于原子时钟和 GPS 的“TrueTime”方案，能够将不同数据中心的时间偏差控制在 10ms 以内。方案简单粗暴而有效，但存在成本较高的问题。

强一致的系统往往比较难实现。很多时候，人们发现实际需求并没有那么强，可以适当放宽一致性要求，降低系统实现的难度。例如在一定约束下实现所谓最终一致性（Eventual Consistency），即总会存在一个时刻（而不是立刻），系统达到一致的状态，这对于大部分的 Web 系统来说已经足够了。这一类弱化的一致性，被笼统称为弱一致性（Weak Consistency）。

*莫非分布式领域也有一个测不准原理？这个世界为何会有这么多的约束呢？*

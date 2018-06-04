## 3-技术人员如何准备面试之理论篇

最近面试了几名同学，面试结果都不是很理想，他们自己也能感觉出来，所以和我说“因为最近在准备论文，所以没怎么准备面试”，“最近一直在上课，没有时间准备面试”。像这种情况我觉得真的非常可惜，虽说面试有时是看运气的，但面试前还是一定要准备好再投简历，并格外珍惜每次面试的机会，否则有时候你以为你错过的只是一次面试机会，但是过了几年之后你可能会发现你错过了一辈子。

如果我们把面试上理想的公司比喻为在通关一场游戏的话，那么准备面试阶段其实就是在找攻略并加练习的阶段。

那么我们要从哪几个方面来准备面试呢？我觉得一共要准备以下六个方面：温习基础知识、多刷题、准备自我介绍、总结技术亮点、梳理职业规划、了解目标公司和职位。

#### 第一，温习基础知识
面试不同的公司，各面试官提的问题肯定不同，但是基础技术是每个面试官通常都会问的，所以面试前务必要好好复习基础知识。基础技术问题有常用算法、SQL、数据库事务、事务隔离级别、TCP 和 HTTP 协议、线程和进程、操作系统等。比如，如果你应聘 Java 程序员，则对应的基础技术知识就有垃圾回收机制、JDK 集合类的实现原理、内存模型、并发编程、运行时数据区和类加载机制等，这些 Java 的基础知识点就是 Java 的特点。

#### 第二，多刷题
面试官会从海量的技术题库中抽几道技术题面试你，网上有各大公司的技术面试题，你自己可以全部过一遍，做到心里有数。如果你都会做了，面试的时候其实会增加很多自信。

当然好的面试官会考察你擅长的技术，看看你对技术深度的掌握程度。但是也有面试官会问他自己擅长的技术，这样就需要你有一定的技术宽度才能应对各种问题，所以你需要尽力去准备，准备得越充分，面试时就会越从容，最后面试成功的概率也就会越大。

除此之外，建议你还要刷一些测试思维能力的题目，比如谷歌面试时的一道题目，有一栋 100 层高的大楼，给你两个完全相同的玻璃球，假设从某一层开始，丢下玻璃球会摔碎，那么怎么利用手中的两个球用最少次数找到这个临界层的层数呢？最简单的做法是拿着球一层一层地尝试。稍微好一点的做法是用二分法，先去 50 层测试，这样可以直接排除一半的楼层，如果没有摔碎，再去 75 层测试。更优的做法是，先拿一个球去 10 层试试，然后每次增加 10 层，用一个球缩小范围，再用另外一个球一层层地试，这样最多 20 次就可以测试出目标楼层。

#### 第三，准备自我介绍
大部分面试官一般都会让应聘者首先做个自我介绍以描述自己的基本情况，其次是描述自己的技术亮点，做过的亮点项目或产品。如果没有技术亮点或做过亮点项目，可以讲下在某个项目中解决的难点，这里有个技巧就是回忆下在解决哪个事情上花费的时间最长就是最有挑战的点，另外学习能力强也是亮点，比如快速搭建出 Redis。自我介绍回合是应聘者最主动的一个回合，因为在这个回合主要是应聘者说，面试官听，所以我认为这个回合非常重要，如果介绍得非常好，就可以让面试官对你有好感，我建议在这部分准备一个五分钟时长的自我介绍。但是在面试中很多同学的自我介绍一分钟都不到，其实工作经验较少的，也可以准备学习经验，比如如何快速学习新技术或学习英语。

举个例子，如果让我去做自我介绍我会这么说：

>“我叫方腾飞，方向的方，经济腾飞的腾飞，目前在某大型互联网公司负责带团队建设数据和风险域，以及团队架构工作。我有 10 年以上的 Java 开发经验，超过 3 年的系统架构，擅长业务架构，负责过十几个系统架构设计工作。有很强的推动能力，组织过数十人的团队完成过大团队双 11 稳定性工作，目前也在推动团队进行单元化建设工作。有较强的技术影响力，12 年利用业余时间创办了并发编程网 ifeve.com，目前已经是国内知名的技术网站，世界排名最高 2W 左右，日访问量数万。擅长和爱好写作，从 05 年到现在已经写了数百篇文章，文章曾多次在 InfoQ 和程序员杂志发表，是畅销书《Java 并发编程的艺术》的作者之一。”

如果面试官还有兴趣，接下来我会再介绍下我做过的最有技术含量的项目和最能体现我架构能力的项目。

你按照这个思路讲下去才算是一个比较完善的自我介绍。

#### 第四，总结技术亮点
技术人员平时做的项目很多，到面试之前必须静下心来总结一下，自己做过的项目和技术亮点。举个例子，如果让我说亮点技术产品，我会说“我开发过一个 Java 动态模块框架 JarsLink，是阿里巴巴的开源项目，用于提高部门后台开发效率，目前部门的后台开发、数据采集和指标计算都会使用这个框架。”如果面试官对这个框架感兴趣，我会继续展开，为什么要做这个框架？它解决了什么问题？这个框架有哪些挑战？我是如何解决这些挑战的？如果让我说最能体现我架构能力的项目，我会讲小微融资架构，会讲一下业务面临的痛点以及针对这些痛点我所想到的解决方案和架构演进方向。

整理好亮点之后，就可以把最优秀的几个亮点放在自我介绍里。

#### 第五，梳理职业规划
每个人都应该好好梳理下自己的技术规划，如果不知道怎么梳理职业规划，可以参考下别人的职业规划。比如，我的职业规划是逐步形成由点到线，线到面，面到体的技术架构能力。从能架构一个系统，到某个业务线的架构，再到部门架构，最后能够驾驭某个生态体系的架构。一个系统就是一个点，几个系统组成线，几十个系统组成一个矩阵就是一个面，数不清的系统组成一个体。如果从职位上来看规划的话，是从开发工程师、到高级开发工程师、再到技术专家、然后到架构师、技术主管、技术总监和 CTO。

#### 第六，了解目标公司和职位
你可以通过目标公司的官方网址、宣传资料和公司高管讲话中了解你应聘的公司和职位信息，每个公司的价值观不一样，对人的要求也不一样，知道了这家公司的价值观，你就可以有针对性地准备面试，并且还可以让 HR 感觉你来面试这家公司是做过充分准备的。熟悉应聘职位的描述和职位要求，能够让你知道准备哪些技术点，缩小准备面试的范围。

#### 总结
准备面试并不是临时抱佛脚就能全部完成的，而是要系统化地学习、在平时的工作中做积累，量变才能引起质变，面试进更好的公司只是一个水到渠成的过程，能力到了自然就能进更好的公司。多准备一点，胜算就多一分！
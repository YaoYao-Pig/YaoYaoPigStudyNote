# ICS-GNN: Lightweight Interactive Community Search via Graph Neural Network

| 项目 |                                                              |
| ---- | ------------------------------------------------------------ |
| 综述 | 否                                                           |
| 代码 |
| 地址 | https://vldb.org/pvldb/vol14/p1006-gao.pdf                   |
| 亮点 | 文章指出，现有方法存在两个主要问题：一是它们通常需要分两步进行，先爬取大部分网络，再寻找社区，但整个网络往往过于庞大且大部分数据对用户不感兴趣；二是现有方法依赖于人为制定的规则来衡量社区成员资格，然而由于社区对于不同的查询顶点是灵活变化的，很难定义有效的规则。<br />ICS-GNN提出了一种基于图神经网络（GNN）的解决方案，将社区成员资格问题重新定义为顶点分类问题。通过结合内容和结构特征，GNN能够捕捉图顶点与查询顶点之间的相似性，并在用户标签的指导下灵活地进行。文章提出了使用kMG（k大小的最高GNN分数）社区来描述目标社区，并通过迭代和交互式地发现目标社区。在每次迭代中，利用查询顶点和已标记顶点的指导来构建候选子图，使用在子图上训练的GNN模型推断顶点分数，并发现kMG社区，然后由最终用户评估以获取更多反馈。此外，文章还提出了两种优化策略，将排名损失结合到GNN模型中，并在目标社区定位中搜索更广泛的空间。通过在离线和在线真实数据集上进行实验，证明了ICS-GNN能够在通信、计算和用户标记方面以低开销产生有效的社区。 |
| 时间 | 2022                                                         |
| 参考 | https://www.bilibili.com/video/BV19b4y1U7tL/?vd_source=5d4070cc138983fa1babce80b5a31622<br />https://link.springer.com/article/10.1007/s00778-022-00754-0 |



## Pre-问题：

1. 为什么其他模型不能捕捉到结构关系弱，内容相似程度高的社区
2. 社区检测和社区搜索的区别

3. 数据抓取和社区搜索的区别：

> 社区检测是在一个图中找到所有共享一般模式的社区[4]，而社区搜索依赖于给定的查询顶点来定位具有特定模式的社区。
>
> Compared with community detection which finds all communities in a graph sharing general patterns [4], community search relies on the given query vertex to locate the communities with specific patterns.







一些社区具有密集的结构关系，现有的社区搜索模型可以捕获这些结构关系[2,9,20]，但对结构关系弱、内容相似度高的社区进行定位是一个挑战。

Some communities have the dense structural relationships, which can be captured by the existing community search models [2, 9, 20], but it is challenging to locate communities with weak structural relationships and high content similarities.

## 感受

首先是之前的工作有一些问题：没有办法解决关系不那么紧密的，区分度不高社交网络结构。其次是社区搜索和数据抓取不一样，爬虫更多的是有什么爬什么，但是社区搜索还是要精细。

再然后，社区检测是：在一个图里面，对社区进行分类，找到所有类别的社区和他们社区当中的节点

社区搜索是，最后就找到那一个社区，查找的依据是给定的查询顶点。



模型不是无脑直接开始做，大概是4个流程

1. BFS先找到离查询顶点（query vertex，下面记作q）最接近的k个顶点（也就是KGM的K的来源）（这里有一个对BFS的边缘增强策略，防止对所有节点用bfs会导致点太多了。就是先对q用完bfs，然后可以标记一个点z，只有这个点z做bfs的时候才可以把新的点加入社区，其他的点bfs的时候只能对已经在社区当中的点添加边）也就是说只有有标记的顶点（也包括q）才能做完整的bfs，这些标记顶点的邻居也只能添加关系而不是新顶点。
2. 训练一个GNN模型，去评价这K个顶点在与q相同社区的概率（也就是GNN评价）
3. 然后在保证连通性的基础上，把概率很低的点交换出社区。
4. 用户评价效果，用来指导GNN学习
5. 回到1



# Semi-supervised Classification with Graph Convolutional Networks

| 项目 |                                                              |
| ---- | ------------------------------------------------------------ |
| 综述 | 否                                                           |
| 代码 |                                                              |
| 地址 | https://arxiv.org/abs/1609.02907                             |
| 亮点 | GCN开山之作<br />我的理解是，本文是基于两个问题展开的。<br />把卷积思想带入了图当中，并且本文的卷积是发生在频域当中的（应该说频域只是一种方法，前面的卷积思想才是关键）<br />关键的地方就是，对于GNN来说，总的是三个步骤，聚合，更新和多层。卷积就是对聚合做了一种处理。通过拉普拉斯矩阵和拉普拉斯算子，用矩阵运算来进行卷积操作 |
| 时间 | 2017                                                         |
| 参考 | [拉普拉斯算子和拉普拉斯矩阵](https://zhuanlan.zhihu.com/p/67336297)<br />[GCN的解释](https://www.zhihu.com/question/54504471/answer/630639025)<br />https://baijiahao.baidu.com/s?id=1678519457206249337&wfr=spider&for=pc |

## 感受

本文数学上推导很多，看不懂。比较重要的就是首先拉普拉斯算子是导数的散数。实际上**衡量了在空间中的每一点处，该函数梯度是倾向于增加还是减少**，也就是梯度的梯度吧

第二个就是对于GNN的研究方法。聚合，更新和多层。聚合当中会有很多操作，比如卷积，在频域卷积，卷积的时候可能也会有不同的函数，求和或者都是1或者加权。但是都是这个套路（也许）

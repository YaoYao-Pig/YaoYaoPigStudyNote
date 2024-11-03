# GNN

从同质图到异质图，从静态图到动态的序列



## 面对的问题：

1. 异质图：

   1. 最开始是同质图，异质关系。RGCN，方法就是简单的对不同关系下的同质节点GCN，然后再聚合
   2. 拆分图+元路径聚合：现在的通用方法，可以看作一种分类过滤模式。通过不同的元路径，构成某种条件下的子图。子图内部GAT训练然后聚合子图信息。然后相同节点在不同类型的元路径下聚合的信息最后再聚合（HAN）。
   3. 多视角图：item-item，user-user，user-item，item-user（GraphRec）

2. 稀疏性：

   1. 数据增强：根据不同类型的物品，分成不同的子图。然后对子图内部做聚合：聚合分为两种，一种是GAT更新子图当中的每一个物品的feature embedding。（相当于每一个物品都有子图全局的信息）。第二种是对子图整体做GAT，拿到可以代表一个子图的feature embedding。

   2. 图对比学习：提高模型对于非正节点的认知

3. 不平衡，某些影响力很大的节点带来影响：
   1. 节点归一化

4. 冷启动

5. 用户兴趣变化：
   1. 基于时序Session/Seqeunce的图推荐：
      1. 通过Session提取用户的动态兴趣。（user-item）
      2. 拿到动态兴趣之后，然后再根据社交图，考虑邻居节点的影响（话句话说，就是根据当前的兴趣，得到之后的兴趣）（Time-Aware Service Recommendation With Social-Powered Graph Hierarchical Attention Network）（加上社交图）
      3. 通过Session拿到动态兴趣（一段时间内的喜好变化），然后根据用户和物品的静态图，再捕捉静态图信息。最后动态静态合在一起（**Graph Neural Networks with Dynamic and Static Representations for Social Recommendation**）
      4. 根据全局每一个人的Session，做一个K聚类。（也就是说有K种爱好），然后根据类型聚合爱好，最后拿到一个最后的用户的embeding，根据这个embedding推理下一个（Graphical contrastive learning for multi-interest sequential recommendation）
      5. 不单单考虑用户兴趣变化，也考虑商品含义的动态变化：（ipad再2017年是高级产品，但是再2021年可能就是评价的消费品）——全局动态图+局部动态图（全局动态图就是两个时间戳当中，全局图的变化）（局部动态就是指的对于一个用户来说，他的session的变化）（Evolving intra-and inter-session graph fusion for next item recommendation）
   2. 基于会话的图：
      1. 用户的feature是用他们的交互Sequence决定的
   3. 多兴趣推荐
   4. 用户偏置：不同用户对于评分系统本身就有不同认知，偏置项用来平衡用户对于打分系统的不同标准（GDSRec）
   
6. 特征提取：
   1. 简单的聚合效果不好：Attention

   2. 聚合到的信息没法代表全局：
      1. 分层网络本身就是一层往外聚合一跳
      1. 但是多跳会有限制，很容易梯度爆炸。所以可以用Shortcut ，相当于图上的跳跃链接了，在高阶邻居上加边。

   3. 如何处理一个用户它自身的时序数据？如何从交互序列（sequence）当中得到可以代表一个用户兴趣的特征？
      1. 最基础的是，用最近的一个兴趣代表用户喜好（**An Efficient and Effective Framework for Session-based Social**

         **Recommendation**）

      2. 进一步，有的论文把序列过一个RNN，然后用Attention聚合RNN的隐变量（Time-Aware Service Recommendation With Social-Powered Graph Hierarchical Attention Network）

      3. 根据时间的远近来对序列的embedding做加权:(CCFC)

      4. 多兴趣，先根据用户的交互序列对物品做Kmeans，然后每种兴趣单独算出注意力。用户最后的特征，是通过聚合每种兴趣下的物品特征的方式来的。也就是说，最后是根据，用户对于全局分类后的K种物品上的兴趣，来预测它下一个会接收的物品（Graphical contrastive learning for multi-interest sequential recommendation）

   4. Relation Transformer：把节点之间的不同”关系“也加入到Attention 的计算当中，不同类型的关系就会得到不同的Attention权重（RELATIONAL ATTENTION: GENERALIZING TRANSFORMERS FOR GRAPH-STRUCTURED TASKS）

   5. 知识图谱+推荐：

      1. 现根据用户的交互Sequence构建一个知识图谱（**An Efficient and Effective Framework for Session-based Social Recommendation**）

7. Embedding

   1. 用Session/Seqeunce来表示
   2. Node2Vec+打分来表示
   3. IDembedding
   4. Node2Vec

8. 

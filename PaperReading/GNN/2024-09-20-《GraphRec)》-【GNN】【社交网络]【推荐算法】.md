# GraphRec：**Graph Neural Networks for Social Recommendation**

| 项目 |                                                              |
| ---- | ------------------------------------------------------------ |
| 综述 | 否                                                           |
| 代码 | https://github.com/wenqifan03/GraphRec-WWW19                 |
| 地址 | [Graph Neural Networks for Social Recommendation (arxiv.org)](https://arxiv.org/pdf/1902.07243) |
| 亮点 |                                                              |
| 时间 | 2019                                                         |
| 级别 | A                                                            |
| 参考 | [论文《Graph Neural Networks for Social Recommendation》阅读 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/409412868) |

## 解决三个挑战：

1. 用户-物品图既编码了交互行为也编码了与之相关的意见；
1. 社交关系具有不同强度的异质性；（社交关系也有强有弱）（还没看，感觉是attention）
1. 用户同时参与两个图

## 

它们（GNN）的主要思想是如何使用神经网络迭代地聚合来自局部图邻域的特征信息

![image-20240920093511976](../../../../AppData/Roaming/Typora/typora-user-images/image-20240920093511976.png)

## 学习

1. 使用了两个视角来进行学习

一个是用户视角：用户视角包括了【用户对于Item的评价】的部分以及【社交网络对于用户的影响】

第二个是物品视角：我的理解这个视角，从物品的角度获得了不同用户对于某个物品的喜爱程度，并且做了一个聚合，因此应该也反应了整体，平均的对于某个物品的喜好。（按照文章的里说法，是为了捕捉物品的潜藏特征因素）

![image-20240920094709494](../../../../AppData/Roaming/Typora/typora-user-images/image-20240920094709494.png)

### user Modeling

使用了两种聚合方式，第一种是Item Aggregation

item Aggregation的时候，第一步是融合物品信息和评价信息（embedding之后），融合之后，过一个MLP获得用户对于交互物品的融合感知信息。

下一步就是聚合了，最开始选择用均值聚合，但是不同的评价信息对于用户的贡献可能不一样（？有点道理但是不多），所以在确定聚合权重的时候缝了注意力

第二种social aggregation也是差不多的，也是embedding之后跟attention



然后User Aggregation那一部分其实也差不多，只是输入和聚合的输入不太一样，也是先找出一个concat之后的vector，然后再注意力。

最后预测的时候就是一个MLP



【损失函数和优化】用的是一个改进的SGD => RMSprop。用了dropout，防止过拟合

## 总结：

大力出奇迹，异质图这边也是，分开两种图的学习，然后分别训练，学习出一个特征的factor之后，再聚合。

聚合的时候可以平均，但是也可以用attention。attention出来的factor再加权平均，然后再聚合。

最后两部分contact起来，然后过一个FC，最后输出一个rating就是预测的结果了。





# GraphRec+：A Graph Neural Network Framework for Social Recommendations

| 项目 |      |
| ---- | ---- |
| 综述 | 否   |
| 代码 |      |
| 地址 |      |
| 亮点 |      |
| 时间 | 2020 |
| 级别 | A    |
| 参考 |      |

是上一篇的续作，GraphRec+

## 问题：

1. 异质图的问题
2. 用户-item图包括了用户和物品的交互（有没有购买）以及评分
3. 社交图的关系影响强度是异构的

其实这三个问题都和GraphRec是一样的，在模型结构上也大概是类似的，但是额外的是GraphRec+多了一张图：Item-Item。

也就是物品物品之间也是有影响的。比如一个人买了Ipad，那么他也许就会更倾向于购买苹果的其他产品，也就是对于商品来说它并不是完全相互独立的，一个产品的购入可能会促进用户购买类似的产品。



Item-Item这一部分直接是对于一个物品j，把与他有关联的物品的，item-user的那个向量聚合了，加了个注意力。

![image-20240920135655833](../../../../AppData/Roaming/Typora/typora-user-images/image-20240920135655833.png)

这里要注意，预测的问题是这样的：给定一个用户i，和一个商品j，问用户i对商品j的评分是多少。

而用户i就会做user modeling

对于商品j，就会做item modeling。

也就说我们是已知用户i和哪些已知的商品有联系。也知道都有哪些用户买过商品j。而用户i和商品j的联系是未知的，因此做推荐。

![image-20240920151254569](../../../../AppData/Roaming/Typora/typora-user-images/image-20240920151254569.png)


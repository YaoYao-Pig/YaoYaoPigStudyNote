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


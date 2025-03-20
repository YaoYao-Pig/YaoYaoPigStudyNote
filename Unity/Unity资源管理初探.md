# Unity资源管理初探

https://blog.uwa4d.com/archives/USparkle_Addressable1.html

https://blog.uwa4d.com/archives/USparkle_inf_UnityEngine.html

https://mp.weixin.qq.com/s/0XFQt8LmqoTxxst_kKDMjw?

## 资源分类

我们显然需要一个资源管理方案，来适配游戏开发的流程。而我们只有先了解Uinty自身的资源管理方案才能指定自己的资源管理器

对于Unity，我们可以把资源分类为这几类

+ 资源文件
+ 代码文件
+ 序列化文件
+ 文本文档
+ 非序列化文件
+ Meta文件

资源文件很好理解，比如模型文件，图片文件这种，当美术完成导入之后，我们不会在Unity当中进行修改了。

序列化文件则是Unity自身管理的一类文件，比如最典型的Prefab文件，场景文件，Asset文件，材质球，如果打开过Prefab就会发现，Prefab的本质其实就是一个YAML配置文件。类似的文件的本质，是在运行的时候由Unity根据它的配置信息，反序列化为对应类的实例

非序列文件是Unity无法识别的文件，比如一个文件夹也会被认为是一个文件，但是无法识别。

## Meta文件

对于Unity的文件管理，我们先从Meta文件开始。。Meta文件实质上是一个文本文档，只是采用的是一种叫做YAML的格式来写的。

Unity当中，会为每一个文件分配一个GUID，这个GUID由Unity保证了全局唯一性。

这里我们在讨论Object，我们知道一个Asset可能对应多个子文件，比如一个Prefab下可能有多个子文件。一个图集下有多个图片。那么这个GUID只能对应一个文件，而对于一个文件下有多个文件的情况，就需要另外一个ID来表示，这就是LocalID。更习惯用Meta文件中的名字FileID。

## AB包

https://blog.uwa4d.com/archives/USparkle_Addressable3.html



## UMT-AB包

**![340a0e9d-bdaa-48da-8420-5ebb4188ac1d](./assets/340a0e9d-bdaa-48da-8420-5ebb4188ac1d.png)**

第一组图的核心逻辑是：

**AB包是以包为单位的**：加载和卸载都是针对整个包。

**加载B包是因为a.prefab被访问**：a.prefab依赖B包中的资源，导致B包被自动加载。

**释放B包需要A包被释放**：仅释放a.prefab的实例不足以卸载B包，必须卸载A包才能解除对B包的依赖。

**a.prefab的释放跟随包**：a.prefab的资源定义是A包的一部分，只有A包卸载时它才会被从内存中移除。

第二组图的核心逻辑是：

- 基于资产（asset）的依赖关系，通过管理每个资产的引用计数来决定AB包的卸载时机。
- 当B包内的所有资产的引用计数都降为0时，说明这些资产不再被任何对象使用，此时B包可以被安全卸载。
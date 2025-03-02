# Lua与C#

本质上，Lua与C#的交互都是依靠虚拟的托管堆进行的。

为了方便讨论，我们先对Lua和C#可能存在的交互行为进行分类：

1. C#调用Lua代码，Lua调用C#代码
2. 类
3. 函数
   1. 静态函数
   2. 类内静态函数
   3. 类内非静态函数



## C#调用Lua代码

先定义什么叫做C#调用Lua代码：我们定义，从C#启动一段Lua的代码逻辑，叫做用C#调用Lua代码。

我们可以先思考一下在C#当中是如何使用XLua调用Lua代码的：

```lua
LuaEnv luaEnv;
string str = string.Format("require('{0}')", fileName);
luaEnv.DoString(str);
```

我们先看LuaEnv是个什么，关注LuaEnv的构造函数

```lua
// Create State
】rawL = LuaAPI.luaL_newstate();
```







1. 大模型接口
2. 爬虫Readme
   1. Readme里有网址，然后大模型对应的去找网址。
   2. Discussions，问答对。
3. 代理查ClickHouse对应项目的Issue+Pr
   1. Issue的提问，作为一个排除的方法，比如有的人想提issue，然后可以问是否有人提过了（本地查询然后提交类似Issue交给大模型判断）
4. 喂给大模型做prompt工程



要做的事情：

1. 选几个仓库作为demo演示的仓库
2. 获取对应仓库的dissussion
3. 获取对应仓库的Issue和状态
4. 设计prompt和对应的功能
   1. 对项目的介绍（获取discussions）
   2. 提问issue的排除和查询


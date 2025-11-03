# XLua穿梭成本Ⅱ

这一篇文章讨论这个问题

我一直好奇，都说穿梭成本高穿梭成本高，可是，细想一下好像也没那么高啊，Lua也运行在C#的非托管堆上，切换上下文也没有那么大的开销吧，仔细研究了下发现还是有很多门道的。

总的来说：

> C#向Lua的穿梭成本其实不高。涉及的主要是：
>
> 1. class类型其实没啥开销，就是封装一个userdata
> 2. struct类型穿梭开销比较大，涉及到值的转换和lua侧table的封装
> 3. 常规的值类型，格式转换也会有开销
> 4. string的开销，涉及到编码的转换，还会涉及到lua侧重新new一个string字符串的开销
>
> lua向C#的穿梭成本比较高：
>
> 1. lua创建一个C#侧的新的对象，要涉及到table的反复查表，这个时候Lua创建的C#对象，如果在C#侧生命周期结束了，就会添加一个等待GC的对象，如果高频创建，开销就非常大

并且，这里要区分：传递的成本和调用的成本这二者，下面都会一一解释

## 1. C#向Lua的穿梭成本

## 1.1 class类型的传递成本

我们知道，class类型作为一种**引用类型**，为了保证C#侧的引用对象，在Lua侧被修改的时候，能同步到C#侧，所以传过去的就是一个id，这个id被封装成userdata。这个可以看我的第一篇文章。

这个开销非常低，几乎和传递一个整数一样廉价。

## 1.2 struct类型穿梭开销

这个就大了，因为struct值类型，需要保证副本独立，所以一个c#侧的struct传递给Lua的时候，是要在Lua上重新创建一个新的副本，具体就是创建一个table，然后赋值进去。这个开销就是class引用要高了。

> 因此，当我们高频的把C#侧的struct传给Lua，都会导致Lua创建出的新的table，变为GC对象。GC压力会很大，这个问题其实和下面的2.1是镜像的
>
> ```lua
> -- Lua 侧的 Update
> function Update()
>     -- 假设 myTransform.position 返回一个 C# 的 Vector3 (struct)
>     -- 每次调用，都会在 Lua 堆上 new 一个新 table
>     local pos = myTransform.position 
> 
>     -- ...使用 pos.x, pos.y ...
> 
>     -- 当 Update 函数结束时，pos 变量销毁
>     -- 它引用的那个 table 很快就变成了“垃圾”
> end
> ```
>
> 

### 1.3 常规的值类型

常规的值类型（如 `int`, `float`, `bool`）的开销**是最低的**。它们只需要进行简单的格式转换（如 `int` 转为 `double`），开销几乎为零。

C# 必须调用 `lua_pushnumber(L, 10)`。这会把 C# 的 `int` 转换为 Lua 的 `lua_Number` (通常是 `double`) 并压入 Lua 栈。

### 1.4 特殊的引用类型string

string作为一个引用类型很特殊，

作为引用类型，C#这边相同的string用的是同一个副本，但是这个副本本身是不可更改的，

所以string在跨越边界的时候，lua侧要重新创建一个lua管理的string

并且，C#用的是utf-16编码，Lua侧用的是UTF-8，这又是转换成本

## 2. Lua向C#的穿梭成本

### 2.1 class类型的穿梭成本

lua创建一个C#侧的新的对象，要涉及到table的反复查表，这个时候Lua创建的C#对象，如果在C#侧生命周期结束了，就会添加一个等待GC的对象，如果高频创建，开销就非常大。比如下面这个例子：

C# 侧代码 (接收方):

```c#
// 假设我们有一个描述玩家数据的 C# class
public class PlayerStats
{
    public string playerName;
    public int level;
    public float health;
}

public class GameManager : MonoBehaviour
{
    // 这个函数期望一个 C# class 实例
    public void CreatePlayer(PlayerStats stats)
    {
        // 这里的 stats 是一个从 Lua table “翻译”过来的 全新 C# 对象
        Debug.Log($"Creating player: {stats.playerName} at level {stats.level}");
        // ...
    }
}
```

**Lua 侧代码 (发送方):**

```lua
-- 1. 在 Lua 中创建一个 table
local playerInfo = {
    playerName = "LuaHero",
    level = 20,
    health = 100.0
}

-- 2. 将这个 table 传递给 C#
-- gameManager 是一个 C# 的 GameManager 实例 (UserData)
gameManager:CreatePlayer(playerInfo)
```

在Lua侧，这个table自然是引用类型，但问题是，当这个table传递给C#，并且把它经过Wrap类的转换变成一个class的引用对象之后，这个C#侧的引用对象并不和Lua侧的引用对象绑定。他们实际上是两个分开的对象。因此可以预见的是，如果上面的C#侧的函数CreatePlayer离开生命周期，stats就会变成一个等待GC的对象。

Lua侧的gameManager:CreatePlayer(playerInfo)函数调用多少次，就会创建多少个新的C#侧的对象，会造成很大的GC压力。并且，还要考虑特殊的String类型。

这是最大的开销

> 结合1.2 我们总结为:**这两种操作（C# `struct` -> Lua 和 Lua `table` -> C# `class`）是开销最大的两个场景**，它们是镜像关系，分别在 **Lua 侧**和 **C# 侧**制造了巨大的 GC 压力。

# 3. 调用成本

如果我们看上面的1.1和1.2，也许会觉得传递class比传递struct开销小，但是在真正调用的时候，比如gameobject.position.x，引用类型的开销是很大的

其实是class“引用”的这个性质导致了它的使用的时候就必须频繁穿梭。为的是保证每次修改都能修改到C#侧的真实数据上。而Struct，因为是值类型，所以传递的时候要展开+深拷贝。因此传递的开销比class大，但是使用的时候是不需要额外穿梭

> `class` 引用导致频繁穿梭，`struct` 传递开销大但使用开销小

这里还有一个值得注意的是
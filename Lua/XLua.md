# XLua

https://zhuanlan.zhihu.com/p/441169478

https://www.cnblogs.com/iwiniwin/p/15307368.html

https://www.bilibili.com/video/BV15N411S755/?spm_id_from=333.337.search-card.all.click&vd_source=5d4070cc138983fa1babce80b5a31622

https://zhuanlan.zhihu.com/p/571902078

https://blog.csdn.net/weixin_45218342/article/details/135899213

## Lua弱引用

https://cloud.tencent.com/developer/article/2317271

https://zhuanlan.zhihu.com/p/638008523

https://zhuanlan.zhihu.com/p/97322052

## Lua访问C#代码

Lua和C#通过一些API进行通信。对于基础数据类型，可以直接通过API通信。而对于那些C#自定义的类，C#会先把对象放进一个对象池，然后获得其对应的index。然后把index传递到Lua栈上，作为一个userdata。

Lua调用C#对象的函数，首先C#会生成一个胶合函数，注册到一个注册表里面，然后通过Lua栈获取调用的参数



CS其实是个表

然后定义了CS这个表的元表行为

```lua
private string init_xlua = @" 
            local metatable = {}
            local rawget = rawget
            local setmetatable = setmetatable
            local import_type = xlua.import_type
            local import_generic_type = xlua.import_generic_type
            local load_assembly = xlua.load_assembly

            function metatable:__index(key) 
                local fqn = rawget(self,'.fqn')
                fqn = ((fqn and fqn .. '.') or '') .. key
                --fqn就是类型和命名空间名,通过import_type去获取对应的udata并且入栈
                local obj = import_type(fqn)
                if obj == nil then
                    -- It might be an assembly, so we load it too.
                    obj = { ['.fqn'] = fqn }
                    setmetatable(obj, metatable)
                elseif obj == true then
                    --从自身这个类的元表中获取key对应的值
                    return rawget(self, key)
                end
                -- Cache this lookup
                rawset(self, key, obj)
                return obj
            end

            function metatable:__newindex()
                error('No such type: ' .. rawget(self,'.fqn'), 2)
            end

            function metatable:__call(...)
                local n = select('#', ...)
                local fqn = rawget(self,'.fqn')
                if n > 0 then
                    local gt = import_generic_type(fqn, ...)
                    if gt then
                        return rawget(CS, gt)
                    end
                end
                error('No such type: ' .. fqn, 2)
            end

            CS = CS or {}
            setmetatable(CS, metatable)

            typeof = function(t) return t.UnderlyingSystemType end
            //...
```

如果访问到了CS表中不存在的元素,则会调用其元表,在元表中通过映射到c#的StaticLuaCallbacks.ImportType方法完成查找



在XLua中，Lua栈确实可以存储C#函数，但并不直接存储C#函数本身，而是通过特定的机制将C#函数封装成Lua可执行的形式，并通过`userdata`类型存储在栈中。在Lua中，C#函数被视为一种可以通过Lua调用的对象，这些对象通常会绑定在Lua栈上。以下是更详细的解释：

### **C#函数与Lua栈的交互**

1. **C#函数封装为Lua闭包**：
   - 在XLua中，当你将C#函数暴露给Lua时，这些函数通常被包装成Lua闭包。Lua闭包是一种可以在Lua中执行的函数，它本质上是一个包含函数体和自由变量的函数。在XLua中，C#函数被转换成Lua闭包，这些闭包会被压入Lua栈中，并且可以像Lua函数一样被调用。
2. **存储C#函数的`userdata`**：
   - 当C#函数传递给Lua时，XLua会将C#函数作为`userdata`类型存储到Lua栈中。`userdata` 是Lua中用来表示外部数据（如C#对象或C函数）的一个类型，因此C#函数并不会直接存储在栈中，而是存储为一个`userdata`，这个`userdata`在Lua调用时会被解封装并转化为可以执行的Lua闭包XLua框架原理(一) - 知乎。
3. **索引和缓存**：
   - C#函数在Lua栈中并不会直接存储其索引。C#函数在XLua中的处理涉及到两部分：C#代码和Lua栈。首先，C#函数被映射到Lua中的`userdata`，然后通过`ObjectTranslator`类的机制，函数会在栈中以封装的形式出现。
   - 当这些C#函数被第一次访问时，它们会被缓存，并在后续调用中直接通过缓存进行调用。因此，C#函数并不会每次都被重新解析，而是通过已缓存的引用快速访问。
4. **通过元表访问C#函数**：
   - 访问C#函数时，Lua会使用元表的机制。元表中定义了如何处理`userdata`类型的C#函数。例如，当Lua调用一个通过XLua暴露的C#函数时，Lua栈中的`userdata`对象会通过元表转化为可调用的Lua闭包，从而执行对应的C#函数XLua框架原理(二) - 知乎。

### **总结**

在XLua中，C#函数在Lua栈中的存储并不是直接存储函数本身，而是通过`userdata`类型存储其引用。`userdata`是Lua与C#之间的数据桥梁，它将C#函数封装为可以通过Lua调用的形式。函数会被缓存并且通过元表进行访问，确保C#函数可以高效地在Lua中被调用。







##### 您说：

这个userdata里存储的到底是什么，是闭包还是索引？



##### 

**核实内容**

我正在调查userdata字段到底是闭包还是索引，待进一步确认。这让我想到，内容的具体性质是关键。

**理解userdata字段**

目前正在探讨userdata字段是闭包还是索引。要充分利用这些理解，必须深入探究它们的区别和联系人。

在XLua中，`userdata` 存储的并不是闭包或索引，而是一个**指向C#对象的引用**，或者说是**一个指针**，用来表示C#对象或者C#函数在Lua中的表示。具体来说，`userdata` 存储的是一个内存地址，指向C#中的一个对象实例或方法。

### **`userdata` 在 XLua 中的作用**

1. **存储C#对象的引用**：
   - 在XLua中，`userdata` 主要用于存储C#对象的引用。在C#与Lua之间交互时，`userdata` 用来承载C#对象的地址和其他元数据。Lua中的 `userdata` 本质上是一个可以在Lua中使用的封装，它存储的是C#对象在内存中的位置，而不是C#对象本身。
   - 例如，如果你在Lua中创建了一个C#的 `GameObject` 对象，XLua会将这个对象封装为 `userdata` 类型存储在Lua栈上。这些 `userdata` 对象会通过 `ObjectTranslator` 进行转换和访问。
2. **存储C#方法的引用（封装成闭包）**：
   - 当C#函数暴露给Lua时，XLua 会将C#的函数封装为 **Lua闭包**，并将这些闭包存储为 `userdata`。Lua闭包可以被看作是一个被封装的C#函数，它可以被Lua调用。`userdata` 在这里存储的仍然是对C#函数的引用，而不是直接的C#函数代码。
   - 具体来说，`userdata` 内部存储的是C#函数的指针，并且通过元表（metatable）来指示如何调用这个C#函数。这意味着，当Lua需要调用这个C#函数时，实际上是通过 `userdata` 访问C#函数的内存地址，然后通过元表的`__call` 方法来执行。
3. **`userdata` 和闭包的关系**：
   - 当我们说 `userdata` 是闭包时，实际上是指 `userdata` 本身不是闭包，而是封装了一个**C#函数的引用**，这个引用被元表转换为 Lua 闭包。当Lua调用这个`userdata`时，元表会将其解包并执行对应的C#方法。因此，`userdata` 在栈中不直接存储闭包，而是通过元表机制使得 `userdata` 看起来像是一个闭包。
4. **`userdata` 存储的内容**：
   - 具体地，`userdata` 存储的内容取决于它封装的对象类型。如果是C#对象，`userdata` 存储的是该对象在内存中的引用（或者说是C#中的指针）。如果是C#函数，`userdata` 存储的是C#方法的引用（函数指针）。这些信息通过`ObjectTranslator`进行管理和转换，使得Lua能够理解如何与这些C#对象和方法进行交互XLua框架原理(一) - 知乎XLua框架原理(二) - 知乎。

### **总结**

`userdata` 存储的既不是闭包，也不是索引，而是对C#对象或C#函数的引用。它本质上是一个内存地址，指向C#对象或C#方法的指针。Lua通过 `userdata` 访问C#对象或函数时，元表的机制使得`userdata`能够表现出类似闭包的行为，因此Lua可以像调用Lua函数一样调用C#函数。







##### 您说：

也就是说，Lua调用C#代码用到了一种线程通信机制？共享了一块内存？



##### 

**探讨线程通信机制**

Lua调用C#代码涉及线程通信机制和共享内存。线程通信是指多个线程之间的交互，而共享内存是多个线程之间共享的数据区域，一般用于多线程程序中。这样可以同步数据访问，避免竞态条件，提高程序的效率。

**深入探讨**

Lua调用C#代码使用线程通信机制，并共享内存。这意味着C#和Lua之间的交互依赖于线程管理和内存共享，确保多线程程序的同步与数据一致性。

可以这么理解，但具体的机制并不是传统意义上的“线程通信”或者“共享内存”，而是一种通过**内存映射**和**代理机制**实现的**跨语言调用**。XLua 通过 `userdata` 和 **元表** 等机制，允许 Lua 代码调用 C# 代码，并在内存中以某种方式共享对象的引用。这种机制不涉及多线程的直接通信，而是依赖于 **Lua虚拟机与C#环境之间的交互桥梁**。

### **具体的工作原理**：

1. **内存映射与 `userdata`**：
   - 在 XLua 中，当 C# 对象或者方法暴露给 Lua 时，`userdata` 作为一种特殊的 Lua 数据类型，在 Lua 栈中保存一个指向 C# 对象或方法的 **内存引用**。
   - Lua 本身并不直接操作 C# 对象的内存，它只是通过 `userdata` 存储对象的引用，而 `userdata` 内部存储的是 C# 对象在内存中的地址。C# 对象和 Lua 之间通过 `ObjectTranslator` 类来完成这个转换和映射XLua框架原理(一) - 知乎XLua框架原理(二) - 知乎。
2. **C# 与 Lua 的数据共享**：
   - 通过 `userdata`，Lua 可以访问 C# 中的对象和方法。虽然说是“共享内存”，但它本质上是 **C#对象的引用**，而不是将 C# 对象的实际内存直接映射到 Lua 中。
   - 当 Lua 中的 `userdata` 被访问时，XLua 会根据其中存储的 C# 对象引用来执行相应的 C# 代码。例如，在 Lua 中调用一个通过 `userdata` 映射的 C# 函数时，Lua 通过元表的 `__call` 方法触发 C# 函数的执行。这相当于 Lua 中的“调用”通过内部映射转化为 C# 的“调用”XLua框架原理(二) - 知乎。
3. **元表和方法代理**：
   - 元表（metatable）是 XLua 中的一个重要机制，它用来定义如何访问和操作 `userdata`。当 Lua 中访问一个 `userdata` 对象时，实际上是通过元表来代理这些操作的。元表定义了在 Lua 中访问 `userdata` 对象时应该如何去查找和执行 C# 中的对应方法XLua框架原理(一) - 知乎。
   - 比如，当 Lua 代码调用 `CS.UnityEngine.GameObject()` 创建一个 C# 的 `GameObject` 对象时，XLua 会为这个 C# 对象创建一个 `userdata`，并将该对象的引用传递给 Lua。Lua 中对该对象的访问通过元表的 `__index` 和 `__newindex` 方法来转发到 C# 对象的实际方法和属性。
4. **对象生命周期管理**：
   - 对象的生命周期也是通过这种机制管理的。`userdata` 是一种弱引用，意味着当 Lua 中的 `userdata` 对象不再被引用时，Lua 的垃圾回收器会自动清理这些对象，并触发 C# 端的 `__gc` 回调，从而释放 C# 端的资源XLua框架原理(二) - 知乎。

### **总结**：

XLua 使用的并不是传统的“线程通信”或“共享内存”，而是一种 **内存引用的传递** 机制，通过 `userdata` 类型封装 C# 对象的引用，并通过元表来实现 Lua 与 C# 之间的交互。Lua 中的 `userdata` 只是一个对 C# 对象或方法的引用，而不直接涉及内存共享或线程通信。

# HotFix

加特性

在设置里加入宏

HotFix注入，替换C#函数的逻辑


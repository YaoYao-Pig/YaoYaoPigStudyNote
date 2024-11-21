# C#数据结构

> C\# 标准库的设计主要是面向对象编程（OOP）的而 C++ 的 STL（标准模板库）主要是泛型编程（GP）的

## List\<T\>

在C#中，`List<T>` 是一个通用集合，封装了动态数组的功能，用于存储一组可变数量的对象。

`List<T>` 的底层实现是一个动态数组，而不是链表。它的核心是一个数组，能够根据需要动态扩展容量。这种实现方式使得 `List<T>` 在支持随机访问的同时，提供了灵活的容量调整功能。（所以其实更像std::vector)

> 多线程不安全

List\<T>继承自两个主要的接口：IList\<T>, IReadOnlyList\<T>

```c#
public class List<T> : IList<T>, IReadOnlyList<T>, ICollection<T>, IReadOnlyCollection<T>, IEnumerable<T>, IEnumerable
{
    // List<T> 实现了所有这些接口
}

```

### IList\<T>

`IList<T>` 是一个泛型接口，继承自 `ICollection<T>` 和 `IEnumerable<T>`。提供了集合元素的读取、写入、插入和删除操作。

```c#
public interface IList<T> : ICollection<T>, IEnumerable<T>, IEnumerable
{
    T this[int index] { get; set; } // 索引器：通过索引访问或修改元素
    int IndexOf(T item);            // 查找元素索引
    void Insert(int index, T item); // 在指定位置插入元素
    void RemoveAt(int index);       // 删除指定索引的元素
}
```

#### `IEnumerable<T>`

以下是 `IEnumerable<T>` 接口的定义：

```c#
public interface IEnumerable<out T> : IEnumerable
{
    IEnumerator<T> GetEnumerator();
}
```

**关键点**

1. **泛型参数 `T`**
   - 表示集合中元素的类型。
   - `out` 表示协变，允许 `IEnumerable<Derived>` 被用作 `IEnumerable<Base>`。
2. **`GetEnumerator` 方法**
   - 返回一个 `IEnumerator<T>` 对象，用于遍历集合。
   - 通过这个方法，可以实现 `foreach` 循环。

#####  与 `IEnumerator<T>` 的关系

- **`IEnumerable<T>`** 定义了集合的枚举能力。
- **`IEnumerator<T>`** 实现了具体的枚举逻辑。

**`IEnumerator<T>` **

```c#
public interface IEnumerator<out T> : IDisposable, IEnumerator
{
    T Current { get; }  // 当前元素
    bool MoveNext();    // 移动到下一个元素
    void Reset();        // 重置到起始位置（不推荐使用）
}
```

### IReadOnlyList\<T>

- `IReadOnlyList<T>` 是一个泛型接口，继承自 `IReadOnlyCollection<T>` 和 `IEnumerable<T>`。
- 提供只读集合的功能，只允许读取元素而不允许修改。

**关键特性**

- 支持随机访问，通过索引读取元素。
- 不支持任何写操作（添加、移除、修改元素）。

```c#
public interface IReadOnlyList<T> : IReadOnlyCollection<T>, IEnumerable<T>, IEnumerable
{
    T this[int index] { get; } // 索引器：只能通过索引读取元素
}
```

### ICollection\<T>

`ICollection<T>` 是一个泛型接口，它为集合类型提供了一组通用的操作方法和属性，是 `IList<T>` 和 `ISet<T>` 等接口的基础

```c#
public interface ICollection<T> : IEnumerable<T>, IEnumerable
{
    int Count { get; }                 // 集合中元素的数量
    bool IsReadOnly { get; }           // 集合是否只读
    void Add(T item);                  // 添加一个元素
    void Clear();                      // 清空所有元素
    bool Contains(T item);             // 检查集合是否包含某个元素
    void CopyTo(T[] array, int arrayIndex); // 复制元素到数组
    bool Remove(T item);               // 移除某个元素
}

```

ICollection\<T>不支持索引访问，IList支持

`ICollection<T>` vs `IList<T>`

| 特性         | `ICollection<T>`                 | `IList<T>`                           |
| ------------ | -------------------------------- | ------------------------------------ |
| **索引支持** | 不支持索引访问                   | 支持按索引访问和修改集合元素         |
| **常用方法** | 提供 `Add`, `Remove`, `Clear` 等 | 提供 `Insert`, `RemoveAt` 等索引操作 |
| **功能范围** | 更通用，适用于无序集合           | 针对有序集合，功能更强大             |

`ICollection<T>` vs `IEnumerable<T>`

| 特性         | `IEnumerable<T>`               | `ICollection<T>`               |
| ------------ | ------------------------------ | ------------------------------ |
| **基本功能** | 只支持遍历                     | 支持遍历并添加、删除等集合操作 |
| **功能范围** | 更基础，所有集合类都实现该接口 | 更高级，提供集合管理的基本功能 |
| **常用场景** | 遍历和延迟执行                 | 需要增删改集合时使用           |

### `List<T>`和`std::vector`

1. 底层都是数组，都是顺序存储
2. `List<T>` Add的时候也是两倍扩容
3. `List<T>` 的初始容量：如果输入的数组容量为 `0`，则新容量为 `4`，而vector如果输入0默认则是1

## `Dictionary<TKey, TValue>`

`Dictionary<TKey, TValue>`是基于哈希表的

### `Object.GetHashCode()` 

`GetHashCode()` 是 `System.Object` 类中的一个虚方法，所有 C# 对象都继承它。Object.GetHashCode()的默认实现使用对象的内存地址或运行时标识符生成哈希值。

但是当我们Equals的逻辑是基于内容而不是根据引用时，默认的方法就不好用了。可以使用HashCode.Combine(),这时候就要遵循以下原则：

> **相等性一致性：**
>
> - 如果 `Equals()` 返回 `true`，则 `GetHashCode()` 必须返回相同的哈希值。
> - 如果 `Equals()` 返回 `false`，则哈希值可以不同（但不强制）。
>
> **不可变性：**
>
> - 对象的哈希值应仅依赖不可变的状态。如果对象的状态发生变化，会导致哈希值改变，可能破坏哈希表的完整性。
>
> **性能和均匀性：**
>
> - 计算哈希值应尽量高效，同时避免过多的冲突。

比如：

```c#
public class Point
{
    public int X { get; }
    public int Y { get; }

    public Point(int x, int y)
    {
        X = x;
        Y = y;
    }

    public override bool Equals(object obj)
    {
        return obj is Point other && this.X == other.X && this.Y == other.Y;
    }

    public override int GetHashCode()
    {
        return HashCode.Combine(X, Y); // 使用标准方法组合字段的哈希值
    }
}
```


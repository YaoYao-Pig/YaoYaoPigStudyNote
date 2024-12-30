# 边边角角

## 迭代相关

### foreach

#### foreach的原理

foreach的本质是要求支持对象实现`IEnumberable<T>`接口，在`IEnumberable<T>`接口当中，会提供一个GetEnumerator()函数，函数返回一个IEnumrator，这是一个结构体。

```c#
//非泛型
public interface IEnumerator
{
    object Current { get; }     // 返回当前元素
    bool MoveNext();            // 移动到下一个元素
    void Reset();               // 重置到集合的初始位置
}
```

```c#
//泛型
public interface IEnumerator<out T> : IDisposable, IEnumerator
{
    new T Current { get; }   // 返回当前元素，类型为 T
}
```

自定义枚举器：

```C#
using System;
using System.Collections;

public class MyCollection : IEnumerable
{
    private int[] items;

    public MyCollection(int[] items)
    {
        this.items = items;
    }

    public IEnumerator GetEnumerator()
    {
        return new MyEnumerator(items);
    }

    private class MyEnumerator : IEnumerator
    {
        private int[] _items;
        private int _position = -1;

        public MyEnumerator(int[] items)
        {
            _items = items;
        }

        public object Current
        {
            get
            {
                if (_position < 0 || _position >= _items.Length)
                    throw new InvalidOperationException();
                return _items[_position];
            }
        }

        public bool MoveNext()
        {
            if (_position < _items.Length - 1)
            {
                _position++;
                return true;
            }
            return false;
        }

        public void Reset()
        {
            _position = -1;
        }
    }
}

```

#### foreach的问题

1. 可以看到，每次使用foreach的时候，都会从GetEnumerator()函数中new 一个IEnumerator的实例，如果foreach次数很多会造成垃圾内存碎片(.Net 4.0后修复)
2. 在 C# 中，`IEnumerator` 和 `IEnumerator<T>` 的设计初衷是为了实现**只读遍历**集合的迭代器模式，而不是对集合进行增删操作。如果增删了，可能会导致迭代器状态和集合本身的状态不匹配，导致错误和混乱。

> 即便你尝试在迭代器中直接操作集合，C# 设计上也会防止这种行为。例如，`List<T>` 在迭代时会通过版本控制防止非法操作：
>
> 在迭代过程中，`List<T>` 等集合会维护一个 `_version` 字段。每当集合发生增删改操作时，`_version` 会自增。在迭代过程中，如果检测到版本号发生变化，就会抛出 `InvalidOperationException`。

## 非虚函数调用虚函数

在C++种，非虚函数调用虚函数是静态绑定的。因此无法多态

但是在C#中，非虚函数调用虚函数是动态绑定的。
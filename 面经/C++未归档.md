C++中的委托构造函数

C++中的尾返回类型

```cpp
#include <iostream>

template<typename T, typename U>
auto add(T a, U b) -> decltype(a + b) {
    return a + b;
}

int main() {
    std::cout << add(3, 4.5) << std::endl;  // 输出: 7.5
    return 0;
}

```



 C++中如何实现一个简单的状态模式？



C++异常处理

https://zhuanlan.zhihu.com/p/656940263



## 智能指针

为什么更推荐用make_share而不是裸指针直接赋值

```cpp
template<typename T, typename... Args>
shared_ptr<T> make_shared(Args&&... args) {
    // 分配一块连续内存，同时容纳控制块和对象
    void* mem = allocate_shared_memory(sizeof(control_block) + sizeof(T));
    
    // 在内存中构造控制块
    control_block* cb = new (mem) control_block();
    
    // 在控制块后的内存中构造对象
    T* obj = new (static_cast<char*>(mem) + sizeof(control_block)) T(std::forward<Args>(args)...);
    
    // 配置 shared_ptr 的内部数据
    shared_ptr<T> sp;
    sp._ptr = obj;           // 指向对象
    sp._control_block = cb;  // 指向控制块
    cb->set_ptr(obj);        // 控制块记录对象地址
    
    return sp;
}
```

因为make_share是只有一次内存分配（先申请内存，然后构造控制块，再构造指针），裸指针要两次（控制块一次，指针一次）。在内存是连续的，而且内存命中率比较高。

第二个问题是，因为两次分配，第一次是new ptr申请了一个临时的对象，第二步申请控制块的内存，但是如果第二次申请失败，那么第一次申请的裸指针的临时对象就会丢失，导致内存泄漏。

## 类的大小由什么因素决定？ 

https://www.bilibili.com/video/BV1akQ5YwEYt/?spm_id_from=333.1007.tianma.2-3-6.click&vd_source=5d4070cc138983fa1babce80b5a31622

1. 非静态成员变量的大小（静态成员变量存储在.data和.bss区）
2. 内存对齐（alligon）和填充（padding）

> 类中所有的成员变量的内存大小必须填充为最大内存的成员变量的数据类型的大小的整数倍

3. 虚函数表指针（一个指针大小）
4. 虚继承的影响

> 虚基类指针。虚基类指针指向什么，虚基类表吗
>
> #### 不同编译器的实现
>
> - **GCC**：虚基类指针通常直接指向虚基类子对象的地址，不依赖虚基类表。
> - **MSVC**：虚基类指针指向虚基类表（vbtable），表中记录了偏移量，派生类通过偏移量找到虚基类子对象。
>
> #### 回答用户的问题

补充：MSVC编译器，子类虚继承父类的时候，会在子类当中保存一份父类大小的缓冲空间，g++是没有的。

5. 空类大小至少为1

> C++中必须区别每个类的地址，所以对于多个空类，如果容量都为零，就没法区分开每个空类了。所以每个空类会具体的占一个字节的大小，来保存每个空类的内存地址，确保地址独立
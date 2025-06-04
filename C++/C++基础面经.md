

# static关键字

1. 链接性

链接性分三种：外部链接，内部链接和无链接函数内部的局部变量，无论是否用 `static` 修饰，都具有**无连接性**。它们的作用域仅限于函数内部。

默认的函数和变量都是外部链接，而由static修饰之后，变量和函数是内部链接，只在当前的编译单元可见

2. 存储期

static修饰的变量都存储在静态存储区，程序运行期间一直存在

注意，类内的static变量必须要在类外部显式初始化

# C++里面四种强制类型转换

首先转换这个概念有几个分类：

显式/隐式

向上/向下

const_cast：

> 只能用于指针或者引用，只能改变对象的底层const（指向对象const）

static_cast：

> 隐式类型转换，可以实现内置基本类型之间的相互转换（比如void\*转ptr\*），也可以在类的层次之间向上或者向下转换。但是向下转换不安全（因为没有动态类型检查）。不能进行无关类型指针之间的转换，转换的指针也不能作用与包含底层const的对象（指向的对象是const的，也就是说）

dynamic_cast：

> 依赖于RAII方法，也就是依赖虚函数，其实是type_info。一般来说type_info存储的位置是在虚函数表的前面的位置。
>
> ```
>             内存地址低 <-----------------------> 内存地址高
> 
>                           [ ... other RTTI info ... ] (可选)
> vptr[-2] (或其他负偏移) -> [ pointer to std::type_info object for this class ]
> vptr[-1] (或其他负偏移) -> [ base offset / other info ]
> ---------------------------------------------------------------------
> vptr[0]  (vtable_start) -> [ address of virtual_function_1 ]
> vptr[1]                 -> [ address of virtual_function_2 ]
> vptr[2]                 -> [ ...                             ]
>                           [ ... more virtual function pointers ... ]
> ```
>
> 

reinterpret_cast: 

> 重新解释数据的二进制含义，但是不改变其数值。**重新解释**一个对象的底层位模式 (bit pattern)，就好像它是另一种完全不同的类型。
>
> reinterpret_cast不能直接转换变量的值的解释（比如直接把一个float转换为int）
>
> `reinterpret_cast` 的核心在于改变对内存地址的类型解释，而不是进行值的语义转换。
>
> reinterpret_cast可以：
>
> 1. 指针之间的转换（指针变量存储的地址是不变的，变得是解释（比如float* 转换为int* ，改变的是对于指针指向对象的解释）
> 2. 转换引用类型
> 3. 指针转换和足够大的整形之间转换（比如一个指针转换为一个很大的int)

问题：static_cast和reinterpret_cast的区别是什么？

A：static_cast是类型转换，最直接的例子是，比如一个float f =100.0f转换为一个int。

显然，float f是浮点数存储，遵循的是IEEE的浮点数存储，包括了符号位，什么M还有E，经过计算之后才会被解释为100.0。因此，我们期望它经过static_cast隐式类型转换之后变成int是一个截断的100。

但如果直接对二进制位进行重新解释呢？那就会把一个float的bit直接解释为int，而不是遵循隐式类型转换的逻辑

```c++
	float f = 100.0;
	cout << "static_cast: " << static_cast<int>(f) << endl;
	cout << "reinterpret: " << *reinterpret_cast<int*>(&f) << endl;
	// 对于 100.0f (0x42C80000)，这里会输出 1120403456
    // 0x42C80000 (hex) = 1120403456 (decimal)
	
	// 输出：
	// static_cast: 100
	// reinterpret: 1120403456
```

# C++指针和引用的区别

引用只是一个别名，而指针是一个变量，既然指针是一个变量，那么指针自己就有专门的内存地址

指针可以被初始化为指向nullptr，而引用必须指向一个已有的对象

指针可以多级，而引用最多一级()

# 空类大小：

为什么空类至少是1个字节？

原因很简单，是因为为了保证每一个对象在程序中都有唯一的地址

考虑这样的场景：

```c++
class A{}; //空类
int main(){
    A a1; //地址0x1000
    A a2; //地址0x1001
}
```

如果空类不至少有1的大小，那么a1和a2的地址应该是一样的

# 智能指针

## Unique_ptr：

Unique_ptr实现对于指针资源的独占，实现是通过delete它的拷贝构造函数和拷贝赋值运算符来实现的（但是注意，允许移动构造和移动赋值）



## Shared_ptr

### 1. 线程安全：

一句话总结：shared_ptr里，对于控制块内部的引用计数器的增减是线程安全的，但是shared_ptr（作为一种类）本身，不是线程安全的。展开说，就是在多线程条件下，shared_ptr的传递是值传递的，那么它的多线程读写是线程安全的，但是如果是引用传递或者是指针传递，那么是线程不安全的。

原因：

对于值传递，相当于创建了一个新的智能指针副本，这里的副本的意思是，有一个新的shared_ptr对象，它的地址是新的，并且创建了一个新的指向管理资源的指针和一个新的指向控制块的指针（但是指向的控制块和管理的资源是和原来一样的）

因此，在多线程下，相当于只有这一个副本的位置才能通过这一套地址去增减控制块和修改资源（路径是单一的）相当于实现了一种“伪原子性”



而对于引用或者指针传递的多线程操纵，如果有很多个线程同时读写。比如说:

```c++
shared_ptr<T> a;


//线程1：
void f1(shared_ptr<T>& a){
    shared_ptr<T> b;
    a = b;
}

//线程2：
void f1(shared_ptr<T>& a){
    shared_ptr<T> c;;
    a = c;;
}
```

注意这里的赋值过程，分为这几个流程：

> 1. 引用计数修改
> 2. 改变控制块指针指向
> 3. 改变管理资源指针的指向

引用计数的修改是原子的，但是这三个操作之间都不是原子的。

所以存在的问题就是，线程1和2交替执行，那么本身期望的状态时：减少一次a的资源的引用计数，a重新指向b，然后减少一次b的资源的引用计数，然后再指向c；

但是，如果竞争了，就会导致可能a的资源的引用计数减少两次，也有可能会导致，存在一个中间状态指向a的资源，但是控制块是b的。

如果我们使用的是拷贝的，那么很显然，会首先增加两次引用计数（因为是值传递）。并且也只会有一个线程去操作一个独立的对象。

![e40dae5d325869379fef92e3a715fb9](./assets/e40dae5d325869379fef92e3a715fb9.jpg)

### 2. make_shared

为什么尽量使用make_shared有两个目的：

1. 性能优势

因为`std::shared_ptr<T>(new T(args...))` 这种方式创建 `std::shared_ptr` 时，实际上会发生两次内存分配：

> 第一次是分配了T(new T)
>
> 第二次是创建了`std::shared_ptr` 的控制块

但是如果使用make_shared只需要一次内存分配，它会一次性地分配一块足够大的内存，同时容纳 `T` 对象和 `shared_ptr` 的控制块（其实是使用了placement_new)

![image-20250530234610576](assets/image-20250530234610576.png)

并且这也**提高了缓存局部性 (Cache Locality)**：由于对象和其管理数据（控制块）在内存中是相邻的，当访问对象时，其控制块（例如更新引用计数时）很可能已经在CPU缓存中。这减少了从主内存读取数据的次数，从而提高了访问速度。

2. 异常安全

考虑以下：

```c++
void some_function(std::shared_ptr<Widget> spw, int priority)
{
    //...
}
int main(){
    some_function(std::shared_ptr<Widget>(new Widget()), compute_priority());
}
```

编译器可能会按照以下顺序执行：

1. 执行 `new Widget()`：分配内存并构造 `Widget` 对象。假设这里成功了，我们得到一个裸指针 `Widget* raw_ptr`。
2. 执行 `compute_priority()`：这个函数可能会抛出异常。
3. 执行 `std::shared_ptr<Widget>(raw_ptr)` 的构造：如果 `compute_priority()` 抛出异常，这一步将永远不会执行。

如果在第一步 `new Widget()` 执行成功后，第二步 `compute_priority()` 抛出了异常，那么 `std::shared_ptr` 的构造函数将不会被调用。这意味着，第一步分配的 `Widget` 对象的内存将会**泄漏**，因为没有任何 `shared_ptr` 来管理它

而如果这样：

```c++
some_function(std::make_shared<Widget>(), compute_priority());
```

因为make_shared会一次性分配尽可能大的一块内存来同时满足控制块和数据的内存。因此他有能力处理分配和构造中可能会出现的异常（都是相对原子的）

make_shared是线程安全的

> 原因：
>
> 1. make_shared每次都会一次性分配内存，而内存分配，C++是保证线程安全的，
>
> 2. 并且创造出的内存块彼此之间都是独立的，所以不同线程操纵的实际上是不同的make-shared的创建的内存块（操作的独立性）
>



# C++的分段

1. .text 代码段，存储机器码（只读且可以执行）
2. .rodata 只读数据段 存储的是明确定义的常量数据，比如字符串字面量，以及const修饰的全局变量和静态变量（这些量在编译的时候就已知）（只读且不可执行）
3. .bss + .data 用来存储数据的，但是这两个段的用处不一样：
   1. .bss段存储的是未被显式初始化或者初始化为0的全局变量，静态变量（包括全局和局部），和静态成员变量（注意，这部分数据在文件中，只存储了一些元数据，而具体的值并没有存储，省空间，只有在加载到内存的时候直接赋值）
   2. .data段存储的是在编译时就已经被初始化赋值，且非0的全局变量，静态变量（包括静态和局部和类内成员变量）（注意，这部分数据会从可执行文件中加载到内存中)

# C++的数据存储

C++的数据存储在哪里呢？

1.  .bss和.data段，这些存储的是全局的和静态的数据，因为这些数据无论是否被显式初始化，都能在编译时确定，并且他们的生命周期比较特殊，所以需要单独管理

2. .rodata 编译时可以确定不会被修改的数值(const的全局或静态变量)，以及字符串字面量（.rodata里的数据也是有地址的，比如`const char* str = "hello";`，变量 `str` 实际上存储的就是字符串 `"hello"` 在 `.rodata` 段中的起始地址）

3. 堆/栈区，这些数据是运行时加载创建出来的，在函数（其机器指令位于 `.code` 段）执行时，函数中定义的**局部变量**（非 `static` 的，无论是 `int`, `double`, `MyClass obj`, 还是 `std::string local_str` 对象本身）会在运行时在该函数的**栈帧中创建**。

   1. 而对于字符串类型的对象，`local_str` 对象本身（包含指向字符数据的指针、长度、容量等成员）是在栈上。

      它所管理的实际字符串数据，如果很短，可能会有小字符串优化（SSO）直接存在 `local_str` 对象内部（仍在栈上）；如果较长，则通常是在堆上动态分配的。 

4. .code里，可能会有比如int i = 42 这样的局部数据，这些数据会以“立即数”的形式，被编码到.code存储的机器码指令当中，（其实是存在与寄存器上）并不是作为一个独立的变量存在的。只有当需要给这个变脸比如i一个内存地址，例如（&i）的时候，编译器通常会分配它在stack上

# STL

## std::sort

https://feihu.me/blog/2014/sgi-std-sort/

原理，优点和缺点

1. 原理

[Introspective Sorting](http://www.cs.rpi.edu/~musser/gp/index_1.html)(内省式排序)，它是一种混合式的排序算法，集成了前面提到的三种算法各自的优点：

- 在数据量很大时采用正常的快速排序，此时效率为O(logN)。
- 一旦分段后的数据量小于某个阈值，就改用插入排序，因为此时这个分段是基本有序的，这时效率可达O(N)。
- 在递归过程中，如果递归层次过深，分割行为有恶化倾向时，它能够自动侦测出来，使用堆排序来处理，在此情况下，使其效率维持在堆排序的O(N logN)，但这又比一开始使用堆排序好。

2. 优化点

   1. **递归调用优化：**

      看代码：

      ```c++
      template <class RandomAccessIterator>
      inline void sort(RandomAccessIterator first, RandomAccessIterator last) {
          if (first != last) {
              __introsort_loop(first, last, value_type(first), __lg(last - first) * 2); //<-- sort
              __final_insertion_sort(first, last); //<---插入排序
          }
      }
      
      template <class RandomAccessIterator, class T, class Size>
      void __introsort_loop(RandomAccessIterator first,
                            RandomAccessIterator last, T*,
                            Size depth_limit) {
          while (last - first > __stl_threshold) { //最小分段阈值 __introsort_loop
              if (depth_limit == 0) {
                  partial_sort(first, last, last);
                  return;
              }
              --depth_limit;
              RandomAccessIterator cut = __unguarded_partition   //分割算法（根据pivot的大小划分两部分）
                (first, last, T(__median(*first, *(first + (last - first)/2),
                                         *(last - 1)))); //三点中值
              __introsort_loop(cut, last, value_type(first), depth_limit);
              last = cut;
          }
      }
      ```

      这是C++ sort实现的一个版本

      ```c++
      function quicksort(array, left, right)
          // If the list has 2 or more items
      
          if left < right
              // See "#Choice of pivot" section below for possible choices
      
              choose any pivotIndex such that left ≤ pivotIndex ≤ right
              // Get lists of bigger and smaller items and final position of pivot
      
              pivotNewIndex := partition(array, left, right, pivotIndex)
              // Recursively sort elements smaller than the pivot (assume pivotNewIndex - 1 does not underflow)
      
              quicksort(array, left, pivotNewIndex - 1)
              // Recursively sort elements at least as big as the pivot (assume pivotNewIndex + 1 does not overflow)
      
              quicksort(array, pivotNewIndex + 1, right)
      ```

      这是正常我们写快排的版本

      有一个显著的区别，正常的sort当中，在完成区间分割之后会分别对左右两边继续递归调用，而\_\_introsort_loop版本当中，似乎只对右边分割了。但实际上，\_\_introsort_loop是把左边的一部分，在当前的递归层就直接展开了（也就是说，对于每一个分割后的区间，递归右侧，而左侧则在循环中分割，为更小的左右，右侧递归。。。）

      ![两种递归调用对比](assets/stl-recursive-call-comparison.png)

   2. #### 三点中值法

   3. #### 分割算法： __unguarded_partition

   ```c++
   template <class RandomAccessIterator, class T>
   RandomAccessIterator __unguarded_partition(RandomAccessIterator first, 
                                              RandomAccessIterator last, 
                                              T pivot) {
       while (true) {
           while (*first < pivot) ++first;
           --last;
           while (pivot < *last) --last;
           if (!(first < last)) return first;
           iter_swap(first, last);
           ++first;
       }
   } 
   ```

   4. 深度恶化处理：partial_sort(其实是堆排序)`__introsort_loop`的最后一个参数`depth_limit`是前面所提到的判断分割行为是否有恶化倾向的阈值，即允许递归的深度，调用者传递的值为`2logN`。注意看`if`语句，当递归次数超过阈值时，函数调用`partial_sort`，它便是堆排序:

   ```c++
   template <class RandomAccessIterator, class T, class Compare>
   void __partial_sort(RandomAccessIterator first, RandomAccessIterator middle,
                       RandomAccessIterator last, T*, Compare comp) {
       make_heap(first, middle, comp);
       for (RandomAccessIterator i = middle; i < last; ++i)
           if (comp(*i, *first))
               __pop_heap(first, middle, i, T(*i), comp, distance_type(first));
       sort_heap(first, middle, comp);
   }
   
   template <class RandomAccessIterator, class Compare>
   inline void partial_sort(RandomAccessIterator first,
                            RandomAccessIterator middle,
                            RandomAccessIterator last, Compare comp) {
       __partial_sort(first, middle, last, value_type(first), comp);
   }
   ```

   4. 插入排序，在sort大小小于_stl_threadhold的时候开始调用__final_insertion_sort(插入排序优化看上面的文章)

```c++
template <class RandomAccessIterator>
void __final_insertion_sort(RandomAccessIterator first, 
                            RandomAccessIterator last) {
    if (last - first > __stl_threshold) {
        __insertion_sort(first, first + __stl_threshold); //保证最小值一定在序列的最左边
        __unguarded_insertion_sort(first + __stl_threshold, last); //剩下的就可以用不带边界检测的版本排序
    }
    else
        __insertion_sort(first, last);
}
```

插入排序这部分优化非常巧妙，总结来说是这样的：

STL有一个__stl_threshold，快排的时候划分划分到这个threshold就可以停止了。这时候剩下的就是一个每个块之间有序，但是块内部无序的状态

然后接下来，使用插入排序，插入排序对于第一小部分（也就是first - > first+threshold)这部分，使用基本的插入排序。然后对于第一小部分之后的部分，因为已经保证了最小值一定在区间之前，因此可以使用没有边界检测的插入排序，这样更快。

对于特别小的序列，使用插入排序优化，对于递归特别深的部分，直接使用堆排序方式最坏情况
# std::bind和lambda.md

std::bind是一个适配器，所谓的适配器，可以理解为一种重新打包，让原来有的东西，经过打包之后发挥出新的作用。

std::bind顾名思义，绑定，它绑定的对象是一个函数（或者仿函数）和参数。通过绑定之后，可以提前指定一些传入的参数：

```cpp
int Add(int i,int j) {
    cout << "i=" << i << endl;
    cout << "j=" << j << endl;
    return i + j;
}
int main() {
    std::function<int(int)> add10 = std::bind(Add, std::placeholders::_1, 10);
    cout << add10(1);
    
    return 0;
}
//
// i=1
// j=10
// 11
//
```

`std::placeholders::_1`是占位符，意思是第一个位置`int i`的位置空着，而`10`则是指定了第二个参数`inr j`的值：10

std::bind的返回值是一个std::function。

那么std::bind是如何做到的呢？

很简单，std::bind的行为，就是生成了一个函数对象，函数对象也是一种对象。我们知道，函数对象一个很重要的性质就是：`可以携带状态`。

因此，绑定参数的行为，实际上是在新生成的对象当中，生成了成员变量，成员变量存储的就是指定的数值。而在后续调用的时候，就会直接将绑定的数值使用进去。

这让我想到了lambda表达式的`捕获列表`

我们知道：

```cpp
int main(void){
	int x=10;
    auto f1=[x](){}
	};
	auto f2=[&x](){}
	};
}
```

具体的语法不赘述，但是让我们思考捕获列表捕获的时候到底做了什么？

我们知道，lambda表达式实际上是生成了一个匿名的函数对象。而捕获列表的本质，就是在这个匿名的函数对象当中，生成了对应的成员变量。而不同的捕获方式（比如值捕获或者引用捕获），则是指定了我们以何种方式存储这个成员变量。

如此对比就会发现，这俩还是有点类似的。
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
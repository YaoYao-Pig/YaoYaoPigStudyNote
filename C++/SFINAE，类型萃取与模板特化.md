# SFINAE，类型萃取与模板特化

## 模板特化

在 **类模板**,（注意，一定是类模板）模板在编写的时候，直接将基础的**通用模板**的模板参数替换为具体类型的就叫做模板特化。特化分为部分特化和完全特化

```c++
// 通用模板
template<typename T, typename U>
class MyClass {};

// 完全特化
template<>
class MyClass<int> {};

// 部分特化，当第一个类型参数是 int 时
template<typename U>
class MyClass<int, U> {};
// 部分特化，当第二个类型参数是 double 时
template<typename T>
class MyClass<T, double> {};
// 部分特化：当 F 是指针类型时
template <typename T, typename U>
struct MyClass<T, F*> {};
```

对于类模板编译的时候，编译器会遵循这样一个顺序去匹配实例化参数和通用模板以及特化模板：从通用模板开始，到特化模板进行匹配。如果通用模板和特化模板都能匹配时，特化模板的优先级更高，就会选择特化模板来作为实例化的依据。（特化模板中，完全特化的比部分特化的优先级更高）

> ### 类模板特化的匹配顺序和优先级
>
> 在类模板的特化过程中，编译器会按照以下规则，从最具体的到最通用的模板版本进行匹配和选择：
>
> 1. **完全特化优先**：
>    - 如果有一个完全特化版本（即所有模板参数都已明确指定，且完全匹配），编译器会优先选择这个完全特化版本。
>    - 完全特化是最具体的特化，因为它针对特定类型组合进行了显式定义。
> 2. **部分特化次之**：
>    - 如果没有完全特化，编译器会选择满足条件的部分特化模板。
>    - 部分特化的优先级比主模板高，但低于完全特化。多个部分特化时，编译器会选择最具体的那个，即匹配条件最严格的部分特化版本。
>    - 部分特化匹配规则更复杂，因为编译器需要比较多个部分特化，判断哪个特化更具体、限制更多。
> 3. **主模板最后**：
>    - 如果没有完全特化或部分特化满足条件，编译器会回退到使用主模板。
>    - 主模板是最通用的匹配，用于没有其他匹配特化的情况。
> 4. **SFINAE 和匹配失败**：
>    - 在类模板的部分特化匹配过程中，如果某个部分特化因为 SFINAE 检查失败导致不匹配，这个特化会被忽略。
>    - 编译器会在剩下的候选特化和主模板中继续寻找最佳匹配。
> 5. **二义性错误**：
>    - 如果多个部分特化都能匹配同一个模板参数组合，且编译器无法确定哪个更具体，则会报出二义性错误（ambiguity）。
>    - 解决二义性错误通常需要添加更具体的特化版本，或重构部分特化的设计，使匹配条件更加明确。

## 函数模板重载

对于模板函数来说，类似的操作也是允许的，但是并不是一种特化，而是对于**函数模板的重载**（C++只支持函数的完整特例化）

```c++
// 通用模板
template<class T, class F>
void func(T t, F f) { /* ... */ }

// 重载，而非部分特化
template<class T, class F>
void func(T t, F* ) { /* ... */ }
```

具体的函数模板重载的匹配规则如下：

> ### 函数模板重载的匹配顺序和优先级
>
> 1. **普通函数优先**：
>    - 如果存在与函数模板同名的普通函数，编译器首先会优先选择普通函数，而不是函数模板。
> 2. **完全匹配的重载优先**：
>    - 如果没有普通函数或者普通函数不匹配，编译器会根据参数类型寻找与之“完全匹配”的重载版本。
>    - 重载解析会优先选择参数类型与传入参数最接近的重载（通常是可以直接匹配的类型，而非经过隐式转换或模板参数推导的类型）。
> 3. **模板参数推导**：
>    - 如果没有完全匹配的重载，编译器会对所有候选的函数模板进行模板参数推导，尝试匹配传入的参数。
>    - 若存在多个候选模板，编译器会选择“最特化”的匹配（即推导后类型最接近传入参数类型的模板）。
> 4. **SFINAE 和匹配失败**：
>    - 在模板匹配过程中，如果某些模板重载由于类型不兼容导致匹配失败（例如不满足 SFINAE），这些模板会被忽略。
>    - 编译器会在剩余的候选模板中寻找最佳匹配。
> 5. **二义性错误**：
>    - 如果有两个或多个候选模板具有相同的匹配优先级且都能与传入参数匹配，编译器将报出二义性错误。

## SFINAE

注意到上述两种匹配原则当中，都会有SFINAE的出现。SFINAE（Substitution Failure Is Not An Error）是一种模板实例化过程中的特殊机制。在模板实例化过程中，如果某些模板参数的替换导致了编译失败，那么编译器不会报错，而是会尝试使用其他可行的模板特化或者重载（这里很重要的是要和上面的优先级关联起来）

### 方法查找

我们这里实现这样一个需求：在模板编程中，如何确定一个类里是否有size()方法

```c++
//只用SFINAE机制实现：
template<typename T>
auto hasSize(T t)->std::decltype(t.size(),bool()){
	std::cout<<"Has Size"<<std::endl;
	return true;
}

template<typename ...Args>
bool hasSize(Args... args){
	std::cout<<"Not Size"<<std::endl;
	return false;
}

//测试用类：
class A{};
class B{
public:
	void size();
};

int main(){
    hasSize(A());
    hasSize(B());
}
```

顺序如下：

当调用hasSize<>()的时候，实际上两种模板都可以匹配。但是单一的模板比起可变参数模板更特化。因此会优先选择单一参数的模板进行匹配。

这时候，对于类型A来说，decltype是一种编译时确定类型的**萃取方法**。因此在编译时，使用decltype就会尝试调用A的size()方法，但是由于A没有size()方法，因此编译失败，这时候触发SFINAE机制。编译器不会把这作为一个错误，而是会 尝试去实例化别的可匹配的重载模板函数。也就是`template<typename ...Args> bool hasSize(Args... args){}`

这次匹配成功，因此对于A来说，最终匹配的就是第二个多参数的模板函数。

对于类型B来说，匹配的优先级也是一样的。和第一个单参数函数匹配之后，std::decltype(expression)会根据内部`expression`的类型确定返回值类型。而`std::decltype(t.size(),bool())`是一个逗号表达式，因此bool是推断的类型。匹配成功，实例化第一个函数



### 类型查找与enable_if



```c++
// 仅在 T 是整型时启用此模板
template<typename T>
std::enable_if_t<std::is_integral_v<T>, void> printType(T value) {
    std::cout << "Integral type: " << value << std::endl;
}

// 仅在 T 是浮点型时启用此模板
template<typename T>
std::enable_if_t<std::is_floating_point_v<T>, void> printType(T value) {
    std::cout << "Floating-point type: " << value << std::endl;
}

int main() {
    printType(42);         // 输出：Integral type: 42
    printType(3.14);       // 输出：Floating-point type: 3.14

    return 0;
}
```

`std::enable_if_t`会在编译器判断类型，根据SFINAE来避免冲突。如果没有`enable_if_t`的话，那么上述的两种方法的参数列表实际上完全一致，没有办法区分，编译器会报错。

但是加上了`std::enable_if_t<B, T>` 会在 **布尔值 `B` 为 `true`** 时定义为类型 `T`（通常默认为 `void`），如果 `B` 为 `false`，则会导致该模板的实例化失败。



```c++
// 辅助模板：判断是否具有 size 成员函数
template <typename T, typename = void>
struct has_size : std::false_type {};

template <typename T>
struct has_size<T, std::void_t<decltype(std::declval<T>().size())>> : std::true_type {};

// 第一个模板函数：当 has_size<T> 为 true 时启用
template <typename T>
typename std::enable_if<has_size<T>::value, bool>::type
has_size_function(const T& obj) {
    cout << "Has size()" << endl;
    return true;
}
// 第二个模板函数：当 has_size<T> 为 false 时启用
template <typename T>
typename std::enable_if<!has_size<T>::value, bool>::type
has_size_function(const T&) {
    cout << "Don't have size()" << endl;
    return false;
}
```



### void_t

```c++
template <typename... T> 
struct make_void { using type = void; };
template <typename... T> 
using void_t = typename make_void<T...>::type;
//或者：
template <typename...>
using void_t = void;
```

看得出来，void_t<...>其实就是一个别名，它的作用就是把给出的所有类型都转换为void类型

在此基础上，我们就可以用类型萃取+void_t来判断模板类型的成员函数是否存在了

```c++
template <typename T, typename = void>
struct has_get : std::false_type {}; //继承自std::false_type类型，value变量为false

template <typename T>
struct has_get<T, void_t<decltype(std::declval<T&>().get())>> : std::true_type {};


int main(){
    static_assert(has_get<A>()::value);
    static_assert(has_get<B>()::value);
}
```

使用的时候也是，两种模板都可以匹配。同时，`struct has_get<T, void_t<decltype(std::declval<T&>().get())>>`的特化程度更高，因此会先尝试实例化第二个。std::declval()会尝试在**不需要实际构造对象** 的情况下生成类型 `T` 的右值引用表达式。在这个右值上调用get。如果调用失败（也就是没有），那么就会直接触发SFINAE，下一步就会找到通用模板`template <typename T, typename = void> struct has_get`。

如果成功了，那么会首先推断一个类型，然后再把这个类型转换为void，完成实例化。

### is_detected

可以多做一步封装：

```C++
template <typename, template <typename...> class Op, typename... T>
struct is_detected_impl : std::false_type {};
template <template <typename...> class Op, typename... T>
struct is_detected_impl<void_t<Op<T...>>, Op, T...> : std::true_type {};

template <template <typename...> class Op, typename... T>
using is_detected = is_detected_impl<void, Op, T...>;\
    
//使用：
// 定义一个操作 Op
template <typename T>
using has_type_t = typename T::type; //类型T有type这个类内别名

// 测试类型
struct WithType { using type = int; };
struct WithoutType {};

int main() {
    std::cout << std::boolalpha;
    std::cout << "WithType has type: " << is_detected<has_type_t, WithType>::value << std::endl;  // 输出 true
    std::cout << "WithoutType has type: " << is_detected<has_type_t, WithoutType>::value << std::endl; // 输出 false
    return 0;
}
```



`template <typename...> class Op`表示，op是一个模板类，template<...,template <typename...> class Op...>表示传入的模板参数本身是一个模板类型







> Reference: https://zhuanlan.zhihu.com/p/26155469
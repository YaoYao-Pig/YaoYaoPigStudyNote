# C++踩坑：switch在分支中的声明

## 1.坑：

在《C++程序设计语言 4th》的9.4.2.1当中有这样一个问题：

![image-20240711171757157](https://raw.githubusercontent.com/YaoYao-Pig/YaoYaoZhu-Pic/main/image-20240711171757157.png)

“C++允许在switch语句的块内声明变量，但是不能不初始化。”这句话给我看蒙了，但是对照原文翻译的也没问题：

```
It is possible, and common, to declare variables within the block of a switch-statement. However, it is not possible to bypass an initialization. 
```

最后发现，是我把“声明”和初始化搞混了，但是这里依然有点绕——对于在switch的case当中的变量声明问题。

首先，现在版本的C++允许在Switch的case分支当中声明临时的变量。这里就牵扯一个问题，在这个例子当中：

```c++
void f(int i)
{
switch (i) {
case 0:
	int x;
	int y = 0; 
	string s; 
case 1:
	++x; 
	++y;
	s = "nasty!";
}
}
```

如果我们f()的时候，i==1了，那么就会跳转到case 1。这时候对于int x，int y，string s的初始化就会被跳过。这种行为显然是非法的。但是问题是，我觉得C++编译器没法对这种行为做出正确的处理。

众所周知，对于一个非静态（non-static）、非全局的（local）、内置的（built-int）类型，声明了之后是不会自动进行初始化的。比如上述例子的int x；他其实就是一个非初始化状态。而int y=3则是进行了显示的初始化。对于string s，它提供了默认的构造函数，进行了隐式的初始化。其实这三种行为都是不合理的，但是编译器只会对y和s变量在case1当中的行为报错：

![image-20240711172600718](https://raw.githubusercontent.com/YaoYao-Pig/YaoYaoZhu-Pic/main/image-20240711172600718.png)

这里我的理解是，对于一个变量的声明，在switch当中的作用域是**从它声明的位置开始到switch结束**。所以在switch这里，不存在跳过声明这一说。但是初始化行为是确实会被跳过的。也因此，当存在跳过初始化行为的时候，编译器就会报错：`crosses initialization of`。也是，是cross initialization也不是crosses declaration。对于int这种，就算不初始化就使用也是可以的，应该是一个UD行为。

## 2.解决方法：

### 2.1 人为确定对象作用范围：

最合理的解决方法，变量的声明，在switch当中的作用域是**从它声明的位置开始到switch结束**。那就直接限定死，某个变量如果要在哪个分支里面被使用那就直接限定死：

```c++
void f(int i){
	switch(i){
		case 0:
			{//限定作用范围
				int x=0;
				x++;
				//....
			}
		case 1:
			//...
	}
}
```

### 2.2初始化前置：

既然在switch里面可能会被跳过初始化，那就直接让变量的初始化不能被跳过就行了

```c++
void f(int i){
	int x=0; //初始化前置
	switch(i){
		case 0:{
				x++;
				//....
			}
		case 1:
			//...
	}
}
```

### 2.3 邪道：初始化后置

一种核合法但有病的解决方法：就是让变量的初始化被所有用不到变量的地方全都跳过：

```C++
void f(int i){
	
	switch(i){
		case 1:
			//...
		case 0://最后一条分支
			//...
			int x=0;
			x++;
	}
}
```

case 0当中要使用到x，那就直接把case 0调整到最后的位置，这样其他分支就不能访问到x了，因此x的初始化也没法被跳过了。但这种方法大部分情况就是图一乐，前两种就是最合理的方法了。
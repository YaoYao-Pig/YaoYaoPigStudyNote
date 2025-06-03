# 生命周期

Unity的生命周期的基本原则是：先物理，后输入事件，再逻辑，最后渲染（先场景后GUI）。协程根据各自的恢复时机不同插入在不同的阶段之后

## 协程的生命周期





## FixedUpdate的生命周期

https://zhuanlan.zhihu.com/p/30335370

https://zhuanlan.zhihu.com/p/682884720

Unity是如何实现FixedUpdate按照固定时间调用的？

我们知道Update的调用时一个主循环里面调用Update。

```c#
double lastTime = Timer.GetTime();
...

while (!quit()){

  double currentTime = Timer.GetTime();
  double frameTime = currentTime - lastTime ;

  UpdateWorld(frameTime);
  RenderWorld();

  lastTime = currentTime;
}
```

也是因此，每次的UpdateWorld，RenderWorld的运算时间可能都不一样，也因此每次Update更新的间隔实际上并不固定

当机器十分快时，引擎可能通过线程休眠来保证固定的FPS。

```c#
while(timeout > 0.001 && deltaTime<timeout)
{
    //...休眠后计算新的增量时间
}
```

而神奇的是，FixedUpdate可以做到恒量时间增加。

首先解决的是为什么？为什么需要恒量时间增加？原因是物理模拟的需要，实际上，我们希望的是，每次物理模拟的更新都是按照一个固定的数值。

我们真正需要的不是每次间隔固定的时间段调用一次FixedUpdate，而是每次调用FixedUpdate都传入一个固定的时间。

那么如何实现呢？

Unity采用了双重循环的方式，在主循环内部套了一个循环，内部的循环用于更新物理模拟

```c#
double  simulationTime = 0;
double  fixedTime = 20;

while (!quit()){

  double  realTime = Timer.GetTime();

  while (simulationTime < realTime){
         simulationTime += fixedTime; 
         UpdateWorld(fixedTime);
  }

  RenderWorld();
}
```

![img](https://picx.zhimg.com/v2-44fb57d9ffd7e571bf1b56460e5c7481_1440w.jpg)

fiexdupdate会进行补针，比如update卡住了10s，我们的物理设置间隔1s，当轮到fixedupdate的时候会把之前的10次调用都补上才往下轮
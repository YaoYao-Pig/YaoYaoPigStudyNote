# [计算机图形学]透明测试、Early-Z与Z-Test

## PipeLine

DX12：

[应用阶段（CPU）]

> 准备顶点数据，索引数据，图元，PSO

[GPU]

> [IA]->[VS]->[光栅化]->[FS/PS]->[混合（Blend）和测试]

Alpha Test

Stencil Test

Depth Test

## 为什么Alpha Test会降低渲染效率

Alpha Test=clip(x)

```
void clip(condition){
	if(condition){
		discord;
	}
}
```



## Depth-Test发生的时间

[IA]->[VS]->[光栅化]->[Z-Test]->[FS/PS]->[混合（Blend）和测试]
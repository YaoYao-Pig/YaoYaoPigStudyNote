# Dx12-CPU和GPU

我们讨论两个东西：

1. 描述符（描述符堆）
2. GPU的Heap

**描述符堆本身存储在由驱动程序分配和管理的GPU内存中**。根据其是否需要被着色器直接访问，这块内存会被赋予不同的属性和访问权限。

一堆数据（例如一张贴图），在Dx12中的存储如下：



Dx12的流程是i这样的，以一张贴图为例子：

1. CPU提交：

   1. 这张贴图最开始是存储在硬盘中，需要加载到内存当中。加载到内存后，需要提交到GPU显存才能被利用。

   2. 因此，CPU需要先创建两个Resouce，一个是最终的存储位置，这个位置应该是只有GPU只读的，所以是DefualtHeap：在GPU上创建一个`Committed Resource`，其类型为 **`D3D12_HEAP_TYPE_DEFAULT`**

   3. 第二个是中转的Heap,这里应该是UploadHeap。

   4. 然后CPU需要完成数据的提交，提交的方式是：通过memcpy等方法，把数据直接拷贝到UploadHeap当中，然后再在CommandList当中指挥GPU把数据从UploadHeap拷贝到DefualtHeap

      > 为了能把数据从cpu内存拷贝到GPU的UploadHeap，需要把GPU这里的物理地址映射到CPU的虚拟地址里

   5. 然后CPU为这个Resouce创建一个Descriptor以及一个DescriptorHeap。这个DescriptorHeap是在显存上分配的空间。CPU为了操控对应的Descriptor，把DescriptorHeap的这段物理显存地址，映射到自己的虚拟空间当中，并且获取一个句柄（D3D12_CPU_DESCRIPTOR_HANDLE），描述符是为最终资源创建的，不是为上传资源创建的。

2. GPU利用：为了获取贴图然后绑定到pipeline上，这里需要CPU告诉GPU如何获取贴图。

   1. 首先为了更好的驱动Shader中描述的“参数”和具体数据的绑定，需要首先在命令列表中设置好当前使用的**根签名**

      > <img src="assets/image-20250618154505019.png" alt="image-20250618154505019" style="zoom: 50%;" />

   2. CPU需要在CommandList里面写入读取的命令，这时候要站在GPU的角度，让他从D3D12_GPU_DESCRIPTOR_HANDLE，这个GPU角度的偏移句柄找到对应的Descriptor，把这个Descriptor绑定到Pipeline上。

3. Pipeline执行的时候，比如在FragementShader，这时候GPU就通过这个Descriptor找到Resouce，根据Resource的采样和滤波显存里的贴图数据。



## 根签名：

Shader，其实就是Shader文件（或者是由Shader文件定义的）。所以，在执行渲染的时候，GPU会根据Shader文件（比如.hlsl）编译后的二进制文件来进行渲染，因此也需要Shader文件中声明的：变量“（比如常量，比如贴图；Texture2D MyTex : register(t0);` 或 `cbuffer MyCb : register(b0); ）。这里的Shader文件中的声明变量，就相当于是”函数的形参“。而根签名，相当于是，为了把函数实参（也就是具体的Resource）绑定到形参上。而出现的中间层

> 通过将“函数声明”（根签名）从“函数调用”（命令列表中的绑定）中分离出来并预先编译，DX12的驱动程序可以在PSO创建时就完成绝大部分的验证和优化工作。这使得真正的“函数调用”（每帧的资源绑定）变得极其轻量和快速，从而实现了巨大的性能提升。

# Resource

资源本身也是有状态的。

对于一个资源，可能是target，也可能是Shader的输入。Dx12需要自己控制资源状态的变换

![image-20250619161801974](assets/image-20250619161801974.png)

很自然的疑问：Resource的状态和Heap的种类有什么关系呢？

![image-20250619161857388](assets/image-20250619161857388.png)
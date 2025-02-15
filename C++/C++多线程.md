thread本质上就是对pthread的封装。

如果thread默认构造，id也是默认的，为0。

join的时候其实会把id赋值为默认

thread禁止拷贝行为。但不禁止移动行为。并且移动的时候会判断id是否为默认（也就是是否join过了，或者是否一方是默认的，因为id为0，代表一个thread对象没有实际“控制”一个线程资源，因而才可以接受新的移动，否则就会有丢失控制的资源）

https://www.bilibili.com/video/BV1DH4y1g7gS/?spm_id_from=333.788.top_right_bar_window_history.content.click&vd_source=5d4070cc138983fa1babce80b5a31622
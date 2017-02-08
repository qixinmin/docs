下面的链接是对android官方的总结，包括Android的渲染机制，内存与GC，电量优化， OOM， Overdraw等主要需要结合试验来理解先记录下来
每天进步一点点
http://hukai.me/android-performance-patterns/

#Render Perfermance

大多数用户感知到的卡顿等性能问题的最主要根源都是因为渲染性能，现在的app做的都比较绚丽， 动画，图片等用的比较多，但android系统很可能来不及渲染那么复杂的渲染操作。如果每次渲染都成功，这样就能够达到流畅的画面所需要的60fps，为了能够实现60fps，这意味着程序的大多数操作都必须在16ms内完成。
（开发app的性能目标就是保持60fps，这意味着每一帧你只有16ms=1000/60的时间来处理所有的任务。）

有很多原因可以导致丢帧，也许是因为你的layout太过复杂，无法在16ms内完成渲染，
有可能是因为你的UI上有层叠太多的绘制单元，  =>这里可以通过DDMS工具查看layout的层次是否可以减少或者打开Overdraw查看绘图的颜色

还有可能是因为动画执行的次数过多。  =>这里可以通过**BlockCanary**能及时发现卡顿的地方，然后及时修复
这些都会导致CPU或者GPU负载过重。

这里有Overdraw的因素，也有可能在逻辑上的不合理，在UI线程中做了不合理的计算，需要具体情况具体分析

##Understanding Overdraw
Overdraw(过度绘制)描述的是屏幕上的某个像素在同一帧的时间内被绘制了多次。在多层次的UI结构里面，如果不可见的UI也在做绘制的操作，这就会导致某些像素区域被绘制了多次。这就浪费大量的CPU以及GPU资源。

我们可以通过手机设置里面的开发者选项，打开Show GPU Overdraw的选项，可以观察UI上的Overdraw情况
![](https://github.com/qixinmin/docs/blob/master/pics/overdraw_options_view.png)

蓝色，淡绿，淡红，深红代表了4种不同程度的Overdraw情况，我们的目标就是尽量减少红色Overdraw，看到更多的蓝色区域。

Overdraw有时候是因为你的UI布局存在大量重叠的部分，还有的时候是因为非必须的重叠背景。例如某个Activity有一个背景，然后里面的Layout又有自己的背景，同时子View又分别有自己的背景。仅仅是通过移除非必须的背景图片，这就能够减少大量的红色Overdraw区域，增加蓝色区域的占比。这一措施能够显著提升程序性能。
已之前做的项目为例：
![](https://github.com/qixinmin/docs/blob/master/pics/home.jpg)
然后进入搜索页面发现一片红
![](https://github.com/qixinmin/docs/blob/master/pics/problem.jpg)明显能看出来一片红呀
也能清晰的看出其实上后面的内容没有隐藏，导致后面的重绘问题优化后的结果如下：

优化自己app的例子然后贴出优化后结构 

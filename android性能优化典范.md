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
<br>
<img src="https://github.com/qixinmin/docs/blob/master/pics/home.jpg" width="400" height="600" alt="主页显示"/><br>

然后进入搜索页面发现一片红<br>
<img src="https://github.com/qixinmin/docs/blob/master/pics/problem.jpg" width="400" height="600" alt="overdraw出现了"/><br>

明显能看出来一片红呀，并且跟主页有关系。

也能清晰的看出其实上主页的内容没有隐藏，导致后面的重绘问题优化后的结果如下：<br>
<img src="https://github.com/qixinmin/docs/blob/master/pics/problem_fix.jpg" width="400" height="600" alt="overdraw出现了"/><br>

效果还是很明显滴，当然后面还有优化的空间

#Memory Churn and performance
Android系统里面有一个Generational Heap Memory的模型，系统会根据内存中不同的内存数据类型分别执行不同的GC操作。例如，最近刚分配的对象会放在Young Generation区域，这个区域的对象通常都是会快速被创建并且很快被销毁回收的，同时这个区域的GC操作速度也是比Old Generation区域的GC操作速度更快的。
![](https://github.com/qixinmin/docs/blob/master/pics/memory_mode_generation.png)<br>
除了速度差异之外，执行GC操作的时候，所有线程的任何操作都会需要暂停，等待GC操作完成之后，其他操作才能够继续运行。 这个暂停用户是能感觉到的
<br>
![](https://github.com/qixinmin/docs/blob/master/pics/gc_event_thread_stop.png)<br>
通常来说，单个的GC并不会占用太多时间，但是大量不停的GC操作则会显著占用帧间隔时间(16ms)。如果在帧间隔时间里面做了过多的GC操作，那么自然其他类似计算，渲染等操作的可用时间就变得少了(16ms).<br><br>
导致GC频繁执行有两个原因：
+ Memory Churn内存抖动，内存抖动是因为大量的对象被创建又在短时间内马上被释放。
+ 瞬间产生大量的对象会严重占用Young Generation的内存区域，当达到阀值，剩余空间不够的时候，也会触发GC。即使每次分配的对象占用了很少的内存，但是他们叠加在一起会增加Heap的压力，从而触发更多其他类型的GC。这个操作有可能会影响到帧率，并使得用户感知到性能问题。
<br>一些常见的例子：
例如，
+ 你需要避免在for循环里面分配对象占用内存，需要尝试把对象的创建移到循环体之外，
+ 自定义View中的onDraw方法也需要引起注意，每次屏幕发生绘制以及动画执行过程中，onDraw方法都会被调用到，避免在onDraw方法里面执行复杂的操作，避免创建对象。对于那些无法避免需要创建对象的情况，
+ 我们可以考虑对象池模型，通过对象池来解决频繁创建与销毁的问题，但是这里需要注意结束使用之后，需要手动释放对象池中的对象。
<br> <br>


#Garbage Collection in Android

JVM的回收机制给开发人员带来很大的好处，不用时刻处理对象的分配与回收，可以更加专注于更加高级的代码实现。相比起Java，C与C++等语言具备更高的执行效率，他们需要开发人员自己关注对象的分配与回收，但是在一个庞大的系统当中，还是免不了经常发生部分对象忘记回收的情况，这就是内存泄漏。
<br><br>
原始JVM中的GC机制在Android中得到了很大程度上的优化。Android里面是一个三级Generation的内存模型，最近分配的对象会存放在Young Generation区域，当这个对象在这个区域停留的时间达到一定程度，它会被移动到Old Generation，最后到Permanent Generation区域。
<br>
前面提到过每次GC发生的时候，**所有的线程都是暂停状态的**。GC所占用的时间和它是哪一个Generation也有关系，Young Generation的每次GC操作时间是最短的，Old Generation其次，Permanent Generation最长。执行时间的长短也和当前Generation中的对象数量有关，遍历查找20000个对象比起遍历50个对象自然是要慢很多的



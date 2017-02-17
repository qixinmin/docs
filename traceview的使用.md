本篇参考网友的资料整理而来，省下摸索的时间 http://blog.csdn.net/u011240877/article/details/54347396

**最近重点关注cpu消耗的优化**
# traceView
TraceView 是 Android SDK 中内置的一个工具，它可以加载 trace文件，用图形的形式展示代码的执行时间、次数及调用栈，便于我们分析。<br>
trace 文件是 log 信息文件的一种，可以通过代码，Android Studio，或者 DDMS 生成。<br>

生成 trace 文件有三种方法：<br>
1. 使用代码 <br>
2. 使用 Android Studio =》使用过程中有点卡顿<br>
3. 使用 DDMS <br>

## 使用代码生成 trace 文件
```
Debug.startMethodTracing("shixintrace");//开始 trace，保存文件到 "/sdcard/shixintrace.trace"
// ...
Debug.stopMethodTracing();//结束
```
代码很简单，当你调用开始代码的时候，系统会生产 trace 文件，并且产生追踪数据，当你调用结束代码时，会将追踪数据写入到 trace 文件中。<br>
下一步使用 adb 命令将 trace 文件导出到电脑：
adb pull /sdcard/shixintrace.trace /tmp

使用代码生成 trace 方式的好处是容易控制追踪的开始和结束，缺点就是步骤稍微多了一点。
　
 ## 使用 Android Studio 生成 trace 文件
Android Studio 内置的 Android Monitor 可以很方便的生成 trace 文件到电脑。
在 CPU 监控的那栏会有一个闹钟似的的按钮，未启动应用时是灰色：
启动应用后，这个按钮会变亮，点击后开始追踪，相当于代码调用 startMethodTracing：

当要结束追踪时再次点击这个按钮，就会生成 trace 文件了。
生成 trace 后 Android Studio 自动加载的 traceview 图形如下：

从这个图可以大概了解一些方法的执行时间、次数以及调用关系，也可以搜索过滤特定的内容。
左上角可以切换不同的线程，这其实也是直接用 Android Studio 查看 trace 文件的缺点：无法直观地对比不同线程的执行时间。
鼠标悬浮到黄色的矩形上，会显示对应方法的开始、结束时间，以及自己占用和调用其他方法占用的时间比例：

　## 使用 DDMS 生成 trace 文件
　　DDMS 即 Dalvik Debug Monitor Server ，是 Android 调试监控工具，它为我们提供了截图，查看 log，查看视图层级，查看内存使用等功能，可以说是如今 Android Studio 中内置的 Android Monitor 的前身。
　　双击 shift 弹出全局搜索，搜索 "Android Device Monitor"：
　　
　　或者直接在 设置里设置 Android Device Monitor 的快捷键：
　　
　　打开 Android Device Monitor，在 DDMS 中打开 trace 文件，DDMS 会启动 TraceView 加载 trace 文件：
　　
　　上图介绍了 TraceView 的大致内容：
上半部分显示了 不同线程的执行时间
其中不同的颜色表示不同的方法
同一个颜色越长，说明执行时间越久，如图中的主线程 main
空白表示这个时间段内没有执行内容
下半部分展示了不同方法的执行时间信息，关键指标有三个：
Cpu Time/Call ：该方法平均占用 CPU 的时间
Real Time/Call ：平均执行时间，包括切换、阻塞的时间，>= Cpu Time
Calls + Recur Calls/Total ：调用、递归次数
　　点击下面的任意一个方法，可以看到它的详细信息：
Parents：选中方法的调用处
Children：选中方法调用的方法
　　根据 TraceView 显示内容定位问题
　　定位问题时 TraceView 的使用方式：
从上半部分查看哪些线程执行时间长？什么时候开始执行？与主线程交错时间？
哪些方法的执行需要花费很长时间
点击 TraceView 中的 Cpu Time/Call，按照占用 CPU 时间从高到低排序
哪些方法调用次数非常频繁
点击 TraceView 中的 Calls + Recur Calls/Total ，按照调用次数从高到底排序
　　排序后，然后逐个排查是否有项目代码或者依赖库代码，有的话点击查看详情，查看是这个方法还是调用的子方法的问题，进一步定位问题。
　　解决 DDMS 中的 TraceView 无法搜索，find 无法使用的问题
　　Traceview 中信息太多，想要查找可以使用最下方的 find：
　　
　　但是目前 DDMS 中的 TraceView 有 bug，find 无法使用，许多人给 Google 提 issue 提了 5 年也没有解决 ╮(╯_╰)╭ ：
　　
　　（图片截自：https://code.google.com/p/android/issues/detail?id=38825）
　　*解决办法就是直接使用 SDK 中的 TraceView: *
　　直接打开 SDK 中的 TraceView :
　　
　　然后打开之前生成的 trace 文件：
　　
　　如果直接打开 traceview 有问题，可以通过命令行 traceview 打开：
　　
　　虽然提示 deprecated，但起码在搜索上还是比 Android Device Monitor 中好用。
　　TraceView 的使用场景
在发现某个页面或者操作会卡顿时，可以使用 TraceView 定位问题代码。
比如启动，加载图片列表卡顿等情况。
　　总结
　　Android SDK 中提供了许多工具帮助我们发现问题，在学会使用工具之余，还是要加强自身对性能要求的意识。


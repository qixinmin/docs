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
下一步使用 adb 命令将 trace 文件导出到电脑：<br>
adb pull /sdcard/shixintrace.trace /tmp

使用代码生成 trace 方式的好处是容易控制追踪的开始和结束，缺点就是步骤稍微多了一点。<br>
有大神写了一个简单blog **移动性能测试 TraceView 自动化抓取方案实践https://testerhome.com/topics/3505?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io <br><br>
可以尝试特定抓取指定的函数与类，但是限定为android5.0以上才支持，*
最终目标为：抓取对应数据，存放到数据库，这样在今后的迭代中可进行周期性比对，从而体现出函数优化情况 ！！大赞**
　
## 使用 Android Studio 生成 trace 文件
Android Studio 内置的 Android Monitor 可以很方便的生成 trace 文件到电脑。<br>
在 CPU 监控的那栏会有一个闹钟似的的按钮，未启动应用时是灰色：<br>
启动应用后，这个按钮会变亮，点击后开始追踪，相当于代码调用 startMethodTracing：<br>

当要结束追踪时再次点击这个按钮，就会生成 trace 文件了。<br>

从这个图可以大概了解一些方法的执行时间、次数以及调用关系，也可以搜索过滤特定的内容。<br>
左上角可以切换不同的线程，这其实也是直接用 Android Studio 查看 trace 文件的缺点：无法直观地对比不同线程的执行时间。<br>

## 使用 DDMS 生成 trace 文件
　　DDMS 即 Dalvik Debug Monitor Server ，是 Android 调试监控工具，它为我们提供了截图，查看 log，查看视图层级，查看内存使用等功能，可以说是如今 Android Studio 中内置的 Android Monitor 的前身。<br>
　　
Traceview 中信息太多，想要查找可以使用最下方的 find：<br>
　　
但是目前 DDMS 中的 TraceView 有 bug，find 无法使用，许多人给 Google 提 issue 提了 5 年也没有解决 <br>
　
图片截自：https://code.google.com/p/android/issues/detail?id=38825）<br>
**解决办法就是直接使用 SDK 中的 TraceView: **<br>
使用命令直接打开 SDK 中的 TraceView :<br>
/Users/xinmin/Library/Android/sdk/tools/traceview /Users/xinmin/codes/an********17_14.59.trace <br>
　　
然后就可以愉快的搜索了。。。。　　
虽然提示 deprecated，但起码在搜索上还是比 Android Device Monitor 中好用。<br>
  
还有一个问题就是DDMS的traceview放大后就很难缩小了，在使用上很不方便，只能放大时间轴不能缩小，造成没办法回复初始状态。<br>
发现了两种方法可以，分享一下：

1. 双击timeline panel上面的时间轴，即可回复到初始状态

2. 将鼠标移动到最左边，点击鼠标开始拖曳，可以做相应的压缩时间操作。
  
## TraceView 的使用场景
在发现某个页面或者操作会卡顿时，可以使用 TraceView 定位问题代码。
比如启动，加载图片列表卡顿等情况。


## 总结

Android SDK 中提供了许多工具帮助我们发现问题，在学会使用工具之余，还是要加强自身对性能要求的意识。


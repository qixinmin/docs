#Apk 反编译分析工具

| 工具名称| 下载地址 | 功能简介|
| ------- | ------- | ----| 
| apktool	| Github	| 还原 Apk 中所包含的 resources.arsc, classes.dex, 9.png 和 xml 等文件，也可以用于二次打包|
|SmaliIdea	|Github	 | Smail调试插件，用于对反编译后的内容进行调试 |
| dex2jar	| Github	| 将 Apk 安装包中的 dex 文件还原成为 jar 文件 |
|jd-gui	|Github	|查看 dex 文件还原成为 jar 文件代码，并且能将 jar 文件中所有的 class 文件转换成为 java 文件|
|jadx	|Github	|与 jd-gui 功能类似，由于 jd-gui 作者停止更新一段时间了，所以 jadx 会是一个不错的选择|
|enjarify	|Github	|Google出品，可直接将 apk 文件还原成为 jar 文件，也可以和 dex2jar 一样，直接操作某个dex|
|Procyon	|Bitbucket	|将反编译得到 jar 包还原成 java 文件，同时能够更强的还原代码的逻辑结构|
|ClassyShark	|Github	|Google出品，可直接浏览 Apk，支持对.dex, .aar, .so，.apk, .jar, .class等文件的操作|


#网络抓包分析工具

|工具名称	|下载地址	|功能简介|
| ----- | ------ | ----| 
|TcpDump	|官网	|支持对手机进行多种协议网络抓包并生成pcap文件，前提是手机已经获取 Root 权限|
|WireShark	|官网	|配合 Tcpdump 生成打 pcap 文件，进行分析|
|Fiddler	|官网	|支持对 HTTP 和 HTTPS 两种协议进行抓包，相比 TcpDump 工具，它不需要手机 Root 权限，同时有自带独立的 GUI|
|Charles	|官网	|和 Fiddler 功能类似，但是 Mac 开发环境下的童鞋，用此工具会胜于 Fiddler|
|Stetho	|官网	|Facebook研发的安卓APP网络诊断和数据监控的框架，使用教程请戳**-->qxm已查看，查看ui结构，db的ui和网络接口，网络监控是在原生app请求的网络都可以在inspector来监控，比较厉害,使用了一天后发现，网络请求只是加入了addNetworkInterceptor的才能被过滤到inspector，如果调用其他sdk的网络，如果加入不了addNetworkInterceptor，则不能被inspector捕获，还是需要类似fiddler或charles来过滤**|


#静态代码分析工具

|名称	| 功能介绍|
|----|--------|
|Android Lint	|AndroidStudio自带的代码扫描工具，用于帮助我们识别代码结构存在的问题，官方使用教程请戳这里 (需要VPN)|
|Checkstyle	|帮助开发者实现常用JAVA代码规范的自动化检查。它的功能比较丰富，相对配置起来比较复杂，你需要根据自己的需求配置你想检查的东西，比如Annotations，Block Checks，Class Design，Coding，Duplicate Code，Headers，Imports，Javadoc Comments，Metrics，Miscellaneous，Modifiers，Naming Conventions，Regexp，Size Violations，Whitespace。使用教程请戳这里|
|FindBugs	|findbugs是一个分析bytecode并找出其中可疑部分的一个工具。它给项目字节码做一个全面扫描，通过一些通用规则去判断可能潜在的一些问题，比如性能，多线程安全等等。使用教程请戳这里|
|PMD	|PMD也是一个静态代码分析工具，它主要用来分析一些常见问题。PMD和Findbugs功能上有很多重叠的地方，二者区别主要体现在分析对象上，Findbugs扫描的是字节码，所以找到问题的级别有可能不一样。使用教程请戳这里|
|Infer	|FaceBook 开源的静态代码分析工具。官方使用教程请戳这里|

#性能分析工具
|名称	|功能介绍|
|----|--------|
|Debug GPU Overdraw|	系统自带功能UI渲染检测功能（打开Settings，然后到 Developer Options -> Debug GPU Overdraw 选择 Show overdraw areas)。 用来检测UI的重绘次数，开发者可以用来优化UI的性能|
|Profile GPU Rendering	|系统自带功能UI渲染检测功能（打开Settings，然后到 Developer Options -> Profile GPU Rendering. 选择 On screen as bars )。用来检测UI绘制帧的速率和耗时，同样开发者可以用来优化UI的性能。|
|Hierarchy Viewer	|SDK自带工具（打开Settings，然后到 Developer Options -> Profile GPU Rendering. 选择 On screen as bars )，同样是检测UI渲染用的|
|Memory Monitor、Heap Viewer、Allocation Tracker	|AndroidStudio自带的内存检测分析工具|
|Traceview、Systrace	|AndroidSDK自带的CPU使用分析的工具|
|Battery Historian	|Google IO大会上的推荐的耗电分析工具，地址 https://github.com/google/battery-historian|
|WakeLock Detector	|对手机的运行状态进行探测记录，能统计那些应用触发了CPU运行消耗cpu，那些应用触发了屏幕点亮。同时还可以对运行时间进行统计，可以查看应用内使用细节。手机需要root|
|GSam Battery Monitor|	检测手机电池电量的消耗去向，能够用折线图进行统计展示。手机需要root|
|Trepn Profiler	|高通出品，用于分析检测手机CPU的消耗。手机需要root，且只支持手机的CPU是高通的。|
|GT	|腾讯MIG专项测试组开发出来的狂拽酷炫屌炸天的神器，官网地址：http://gt.tencent.com/index.html|
|iTest	|科大讯飞出品的测试工具，直接安装使用，是一款服务于Android测试人员的专业手机自动化性能监控工具。官网地址：http://itest.iflytesting.com|
|Emmagee	|网易出品的测试工具，和iTest差不多，最大的好处在于，能够对应用的常用性能指标进行检测，并以csv的格式保存方便查看应用的各项参数。测试结果看起来更加直观，还有很重要一点是，它开源!官网地址：https://github.com/NetEase/Emmagee|


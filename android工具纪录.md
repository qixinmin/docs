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
|Stetho	|官网	|Facebook研发的安卓APP网络诊断和数据监控的框架，使用教程请戳|

#问题
webview在加载网页过程中通常做法是给用户一个进度条来显示当前的进度，
但是有时候会发现网页内容已经显示出来了，为什么进度条还在加载中，这是因为在看webview源码中发现其逻辑中，
onPageFinish的逻辑特别晚，一般情况下进度条消失的回调就在onPageFinish中

比较的体验应该在当内容显示出来就应该让进度条消失，但是如何做呢
#解决方法
这个分2中情况，
* 如果webview的渲染引擎也就是我们常说的浏览器内核自己能够控制， 那么比较简单
只需要添加一个回调上来的接口，在webview的framework添加相应的接口即可使用，方法是didFirstVisuallyNonEmptyLayout

* 无核版本的webview想使用上面的接口是不可能的啦，因为接口处于c++层的接口上，但是还有一种办法可能接近上面的办法
在使用chrome的inpsect中无意中发现有个叫DOMContentLoaded的事件，在网络加载页面中，总是在网络加载完之前就
上报出来这个事件了，于是把DOMContentLoaded好好调查了一番， 原来DomContentLoaded是网页DOM树建立完成而发出来的
一个回调，一般情况下这个回调出来其实离绘制内容到显示都已经很接近了，实际试验中也证明了这一点

问题是怎么使用它呢？
因为webview支持javascript回调，而且DomContentloaded方法也是javascript方法，所以在
```
webView.addJavascriptInterface(this, "check2345domcontentloaded");
```
加入javascript的事件处理
在网页回调的onprogresschanged方法中加入
```
public void onProgressChanged(int newProgress) {
        .......
        if (mBrowserWebView != null && mIsDomLoaded == false) {
            String funName = "_2345listener" + mDomFunctionIndex++;
            **mBrowserWebView.loadUrl("javascript:" + "document.addEventListener('DOMContentLoaded',\n" +**
                    **" function  " + funName + "(){window.check2345domcontentloaded.checkDomCallback()}, true);");**
            mDomFunctionList.add(funName);
        }
    }
```
在java中加入代码
```
 @JavascriptInterface
    public void checkDomCallback() {
        if(mIsRedirect)
            return;

        mIsDomLoaded = true;
        injectRecommendNews();
    }
```
这样能保证DomContentLoaded时java端能处理

在实际测试中发现只有50%的成功率，而且在一段时间后突然失效，然后又突然起作用
问题出在那里呢？？
问了前端的同事才知道， 
```
document.addEventListener('DOMContentLoaded',
```
document加入eventlistener是有个数上限的，而且用完最好销毁掉（removeEventListener）这样才能保证每个网页用的都是新的

根据上面2条，我加入限制，
```
    public void onDestory() {
        for (String str : mDomFunctionList) {
            mBrowserWebView.loadUrl("javascript:" + "document.removeEventListener(\"DOMContentLoaded\", " + str + ");");
        }
```
果然成功率上升到95%以上， 基本上满足需要。

在这个试验中也发现，如果频繁的创建webview和销毁webview， android应该对webview做了优化，使用了上一次销毁的webview，导致
某些奇怪的现象



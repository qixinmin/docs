最近2天在解决一个问题，是webview的输入框有时候不弹出系统输入法问题
首先想到的是对webview是否走到了正常的调用流程，通过debug发现的确走到了显示流程
```
public boolean showSoftInput(View view, int flags, ResultReceiver resultReceiver) {
        checkFocus();
        synchronized (mH) {
            if (mServedView != view && (mServedView == null
                    || !mServedView.checkInputConnectionProxy(view))) {
                return false;
            }

            try {
                return mService.showSoftInput(mClient, flags, resultReceiver);
            } catch (RemoteException e) {
            }
            
            return false;
        }
    }
```
问题很奇怪， 如下图，
![调用堆栈](https://github.com/qixinmin/docs/blob/master/pics/webview_pop_stack.png)
是一个正常的调用关系，但是在
```
 public boolean showSoftInput(View view, int flags, ResultReceiver resultReceiver) {
        checkFocus();
        ''''
```

checkfocus中含有
```
private boolean checkFocusNoStartInput(boolean forceNewFocus, boolean finishComposingText) {
        // This is called a lot, so short-circuit before locking.
        if (mServedView == mNextServedView && !forceNewFocus) {
            return false;
        }
```
mServedView与mNextServiedView都是空值null， 所以怀疑是有逻辑对这2个值进行了操作
然后又发现InputMethodManager是单例的，更加做实了这种想法,但是需要验证一下是否正确
发现里面有个focusin方法
```
 /**
     * Call this when a view receives focus.
     * @hide
     */
    public void focusIn(View view) {
        synchronized (mH) {
            focusInLocked(view);
        }
    }
    
     void focusInLocked(View view) {
        if (DEBUG) Log.v(TAG, "focusIn: " + view);

        if (mCurRootView != view.getRootView()) {
            // This is a request from a window that isn't in the window with
            // IME focus, so ignore it.
            if (DEBUG) Log.v(TAG, "Not IME target window, ignoring");
            return;
        }

        mNextServedView = view;
        scheduleCheckFocusLocked(view);
    }
```
其中   mNextServedView = view;进行赋值，应该对输入法弹框对象进行服务的对象, 但是由于是@Hide的方法
所以**不能在SDK的接口中直接调用**，需要在**反射**的方法来验证我们的想法是否可行
所以在WebView的

```
public boolean onTouchEvent(MotionEvent event) {
  Log2345.d("test", "start");
        Class<?> c;
        try {
            c = Class.forName("android.view.inputmethod.InputMethodManager");
            Method[] ms = c.getMethods();
            for (Method m : ms) {
                Log2345.d("test", m.getName());
                Class<?>[] cx = m.getParameterTypes();
                for (Class<?> clas : cx) {
                    String parameterName = clas.getName();
                    Log2345.d("test:" , parameterName);
                }
            }

            Method m = c.getMethod("focusIn", new Class[] {View.class });
            m.setAccessible(true);

            Method imm = c.getMethod("getInstance");
            imm.setAccessible(true);
            InputMethodManager imm2 = (InputMethodManager) imm.invoke(null);

            Object s = m.invoke(imm2, new Object[] {BrowserWebView.this});
            Log2345.d("test", "end");
        } catch (Exception e) {
            e.printStackTrace();
        }
//                    }

```

进行反射的验证，保证在webview点击的时候可以调用到正常的View，所以在webview中进行上面的反射处理
实验证明的确可以正常显示输入法的， 但是是谁对mNextServedView 或对mServedView进行清理工作呢

只需要加入断点调试即可，如图所示, 

![调用堆栈](https://github.com/qixinmin/docs/blob/master/pics/clearStack.png)


在函数

```
@Override
    public void onPause() {
        super.onPause();
        mActivity.getWindow().setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_RESIZE
                | WindowManager.LayoutParams.SOFT_INPUT_ADJUST_PAN);
        if (isEnterFolder) {
            exitFolder();
        }
        mViewPager.setCurrentItem(0);
        urlInputView.clearFocus();
    }
    
``` 

有clearFocus的操作，所以判断发出清除操作，导致webview的焦点进行获得然后被失去，
去掉这行代码就可以了
    
### 总结：
相关联的类：
1. android.view.inputmethod.InputMethodManager
2. android.view.View/ViewGroup
3. android.webkit.WebView

解决这个问题采用从framework层反推(和反射)的方法找到解决方案，希望在以后解决问题中有借鉴意义

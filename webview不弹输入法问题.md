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
mServedView与mNextServiedView都是空值， 所以怀疑是有逻辑对这2个值进行了操作
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
其中   mNextServedView = view;进行赋值，应该对输入法弹框对象进行服务的对象但是是@Hide的方法所以
    

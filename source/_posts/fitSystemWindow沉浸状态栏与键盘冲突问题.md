---
title: fitSystemWindow沉浸状态栏与键盘冲突问题
date: 2020-05-31 21:57:56
tags: Android 
summary: fitSystemWindow沉浸状态栏与键盘冲突问题
---

<!-- toc -->

## 前言 

在将状态栏改为沉浸时遇到了如下一个问题：fitsSystemWindows设置为true后，界面就无法全屏，因为顶部有一个状态栏高度的padding；不设置fitsSystemWindows，adjustResize模式无法用于沉浸全屏界面，导至输入框无法跟随键盘。

沉浸状态栏使用的工具：[ImmersionBar](https://github.com/gyf-dev/ImmersionBar)

## 问题分析

1. **fitSystemWindow**

   如果多个View设置了`fitsSystemWindows=”true”`,只有最外层view起作用，从最外层设置了`fitsSystemWindows`的view开始计算padding，如果在布局中不是最外层控件设置`fitsSystemWindows=”true”`, 那么设置的那个控件高度会多出一个状态栏高度。若有多个view设置了，因第一个view已经消耗掉insect，其他view设置了也会被系统忽略。

   *详细内容见[全屏、沉浸式、fitSystemWindow使用及原理分析：全方位控制“沉浸式”的实现](https://www.jianshu.com/p/28f1954812b3?from=groupmessage)*

2. **键盘挡住输入框问题** 

   方法一、在AndroidManifest.xml对应的Activity里添加`windowSoftInputMode`属性

   **adjustResize**：调整activity主窗口的尺寸来为屏幕上的软键盘腾出空间

   **adjustPan**：自动平移窗口的内容，使当前焦点永远不被键盘遮盖，让用户始终都能看到其输入的内容。只有关闭软键盘  才能看到因平移被遮盖的内容。

   方法二、在布局最外层使用ScrollView

   *详细内容见[android全屏／沉浸式状态栏下，各种键盘挡住输入框解决办法](https://blog.csdn.net/xueand/article/details/73293809)*

## 解决方案

去除fitSystemWindow，然后使用adjustResize以保证输入框跟随软键盘，所以现在只要解决一个问题：*adjustResize在全屏时失效*。

最后通过监听`addOnGlobalLayoutListener`在输入法弹出时改变rootView或DecorView的高度，使view高度=屏幕高度-输入法高度。代码如下：

```java


import android.app.Activity;
import android.graphics.Rect;
import android.os.Build;
import android.view.View;
import android.view.ViewGroup;
import android.view.ViewTreeObserver;


public class FullScreenInputWorkaround {

    // For more information, see https://code.google.com/p/android/issues/detail?id=5497
    // To use this class, simply invoke assistActivity() on an Activity that already has its content view set.
    private static final String TAG = "AndroidBug5497Workaround";

    public static FullScreenInputWorkaround assistActivity(Activity activity, View contentView, InputShowListener inputShowListener) {
        return new FullScreenInputWorkaround(activity, contentView, inputShowListener);
    }

    private Activity activity;
    private View mChildOfContent;
    private int usableHeightPrevious;
    private ViewGroup.LayoutParams layoutParams;
    private ViewTreeObserver.OnGlobalLayoutListener listener;
    private FullScreenInputWorkaround(Activity activity, View contentView, InputShowListener inputShowListener) {
        this.activity = activity;
        this.inputShowListener = inputShowListener;
        mChildOfContent = contentView;
        mChildOfContent.getViewTreeObserver().addOnGlobalLayoutListener(listener = new ViewTreeObserver.OnGlobalLayoutListener() {
            public void onGlobalLayout() {
                possiblyResizeChildOfContent();
            }
        });
        layoutParams = mChildOfContent.getLayoutParams();
    }

    private void possiblyResizeChildOfContent() {
        int usableHeightNow = computeUsableHeight();
        if (usableHeightNow != usableHeightPrevious) {
            int usableHeightSansKeyboard = mChildOfContent.getRootView().getHeight();

            int heightDifference = usableHeightSansKeyboard - usableHeightNow;
            if (heightDifference > (usableHeightSansKeyboard / 4)) {
                // keyboard probably just became visible
                layoutParams.height = usableHeightSansKeyboard - heightDifference;
                if (inputShowListener != null) {
                    inputShowListener.inputShow(true);
                }
            } else {
                // keyboard probably just became hidden
                layoutParams.height = usableHeightSansKeyboard;
                if (inputShowListener != null) {
                    inputShowListener.inputShow(false);
                }
            }
            mChildOfContent.requestLayout();
            usableHeightPrevious = usableHeightNow;
        }
    }

    private int computeUsableHeight() {
        Rect frame = new Rect();
        activity.getWindow().getDecorView().getWindowVisibleDisplayFrame(frame);
        int statusBarHeight = frame.top;

        Rect r = new Rect();
        mChildOfContent.getWindowVisibleDisplayFrame(r);

        //这个判断是为了解决19之后的版本在弹出软键盘时，键盘和推上去的布局（adjustResize）之间有黑色区域的问题
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
            return (r.bottom - r.top) + statusBarHeight;
        }

        return (r.bottom - r.top);
    }

    public void finish() {
        if(mChildOfContent != null && Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN) {  
        	mChildOfContent.getViewTreeObserver().removeOnGlobalLayoutListener(listener);
        }
    }

    private InputShowListener inputShowListener;

    public interface InputShowListener {
        void inputShow(boolean show);
    }
}
```

*详细内容见[Android全屏状态下弹出输入法adjustResize无效的修复方案及踩坑指南](https://blog.csdn.net/xyq046463/article/details/85103713)*
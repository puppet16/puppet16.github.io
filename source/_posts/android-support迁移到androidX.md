---
title: android.support迁移到androidX
date: 2020-05-31 21:33:00
tags: Android Library
summary: android.support迁移到androidX
---

<!-- toc -->

# 一、前言
 Google 2018 IO 大会推出了 Android新的扩展库 AndroidX，用于替换原来的 Android扩展库，将原来的android.*替换成androidx.*；只有包名和Maven工件名受到影响，原来的类名，方法名和字段名不会更改。官方早就推荐将support库迁移到androidx，因为后续support库不会再做更新。  
 迁移时只需要3.2版本及以上的android studio，在菜单refactor中点击migrate to androidx即可，该向导会提示需要做的更新。其中包括gradle版本至少3.2以上，compileSdkVersion 版本28以上。   
 **1. 常用依赖库对比**  

| Old build artifact | AndroidX build artifact |
|:----|:---|
| com.android.support:appcompat-v7:28.0.2 | androidx.appcompat:appcompat:1.0.0 |  
| com.android.support:design:28.0.2 | com.google.android.material:material:1.0.0 |  
| com.android.support:support-v4:28.0.2 | androidx.legacy:legacy-support-v4:1.0.0 |  
| com.android.support:recyclerview-v7:28.0.2 | androidx.recyclerview:recyclerview:1.0.0 |  
| com.android.support.constraint:constraint-layout:1.1.2 | androidx.constraintlayout:constraintlayout:1.1.2 |  
 
**2. 常用支持库类对比**  
| Support Library class	| AndroidX class|
|:--|:--|
| android.support.v4.app.Fragment | androidx.fragment.app.Fragment | 
| android.support.v4.app.FragmentActivity | androidx.fragment.app.FragmentActivity | 
| android.support.v7.app.AppCompatActivity | androidx.appcompat.app.AppCompatActivity | 
| android.support.v7.app.ActionBar | androidx.appcompat.app.ActionBar | 
| android.support.v7.widget.RecyclerView | androidx.recyclerview.widget.RecyclerView | 

了解androidX可以看这一篇:[androidX了解一下](https://blog.csdn.net/qq_17766199/article/details/81433706)  
更多androidX详细内容看:[官方文档](https://developer.android.google.cn/jetpack/androidx/migrate)  

 关于gradle低版本升级可以看这篇文章:[https://blog.csdn.net/u013183608/article/details/89428563](https://blog.csdn.net/u013183608/article/details/89428563)
# 二、替换成androidx后遇到的问题
## 1. 对findViewById的引用不明确
**描述：** Activity 中的方法 findViewById(int) 和 AppCompatActivity 中的方法 <T>findViewById(int) 都匹配其中, T是类型变量:T扩展已在方法 <T>findViewById(int)中声明的View  

**解决方法：** compileSdkVersion不一致，应该进行统一  

## 2. v4jar包keyeventcompat不存在

**描述：** 错误: 找不到符号 符号:  类 KeyEventCompat 位置: 程序包 androidx.core.view

**原因：** KeyEventCompat类被取消了hasNoModifiers方法，而该方法已经被KeyEvent实现了
**原代码：**
```
if (KeyEventCompat.hasNoModifiers(event)) {
    handled = arrowScroll(FOCUS_FORWARD);
} else if (KeyEventCompat.hasModifiers(event, KeyEvent.META_SHIFT_ON)) {
    handled = arrowScroll(FOCUS_BACKWARD);
}
```
**修改后代码:**
```
if (event.hasNoModifiers()) {
    handled = arrowScroll(FOCUS_FORWARD);
} else if (event.hasModifiers(KeyEvent.META_SHIFT_ON)) {
    handled = arrowScroll(FOCUS_BACKWARD);
}
```
## 3. ERROR: [TAG] Failed to resolve variable '${junit.version}'
**解决方法：** file->Invalidate Caches / restart

## 4. Error:error: resource previously defined here. 

**原因：** 自定义View时自定义属性使用了系统的属性名称  
**解决方案：**   
 1. 重命名自已定义的属性，以避免与系统的属性名称重复  
    ```
    <declare-styleable name="TestTextView">  
        <attr name="text_color" format="color"/>
        <attr name="text_size" format="float" />
    </declare-styleable>
    ```
 2. 不对textColor进行重命名，直接引用系统的textColor，然后在xml里面使用 时，就不能使用自定义的命名空间了（例如：app:），得用使用原生的引用（android:）
    ```
    <declare-styleable name="TestTextView">
        <attr name="textColor"/>
        <attr name="textSize"/>
    </declare-styleable>
    ```



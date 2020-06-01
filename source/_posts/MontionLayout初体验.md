---
title: MontionLayout 初体验一
date: 2020-06-01 16:39:21
tags: [Android, MotionLayout, ConstraintLayout, Animate]
summary: MontionLayout 初体验一
---

<!-- toc -->

# 前言

`MotionLayout`是一种布局类型，为`ConstraintLayout`子类，可以帮助管理应用程序中的运动和组件动画。他缩小了布局切换和复杂动作处理之间的距离，提供了具有两者混合特征的属性动画框架`TransitionManager`和`CoordinatorLayout`。
除了描述布局切换之间的过渡外，`MotionLayout`还可以为布局属性设置动画。而且还支持可定位的过渡，这意味着可以根据某些条件（例如触摸输入）立即显示过渡中的任何点。
**注意：**

1. MotionLayout 只支持第一层它包裹的子View，不支持再深层嵌套的布局层次结构或 activity 转换。
2. Android API 要大于等于**14**，即android4.0。

# 使用

## 1. 添加依赖

将`ConstraintLayout2.0`依赖项添加到应用程序的`build.gradle`文件中。

* 如果使用的是`AndroidX`，请添加以下依赖项：

    ```java
    dependencies {
        implementation 'androidx.constraintlayout:constraintlayout:2.0.0-beta6'
    }
    ```

* 如果不使用 AndroidX，请添加以下支持库依赖项：

    ```java
    dependencies {
        implementation 'com.android.support.constraint:constraint-layout:2.0.0-beta6'
    }
    ```

***以下示例均为`AndroidX`版本***  

## 2. 创建MotionLayout布局  

1. `MotionLayout`是`ConstraintLayout`的子类，所以你可以将任何现有的`ConstraintLayout`换成`MotionLayout`,如下：

    ```java
    <!-- before: ConstraintLayout -->
    <androidx.constraintlayout.widget.ConstraintLayout .../>
    <!-- after: MotionLayout -->
    <androidx.constraintlayout.motion.widget.MotionLayout .../>
    ```

2. MotionLayout完整布局：

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <androidx.constraintlayout.motion.widget.MotionLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        android:id="@+id/motionLayout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layoutDescription="@xml/scene_01"
        tools:showPaths="true">

        <View
            android:id="@+id/button"
            android:layout_width="64dp"
            android:layout_height="64dp"
            android:background="@color/colorAccent"
            android:text="Button" />

    </androidx.constraintlayout.motion.widget.MotionLayout>
    ```

    其中，`app:layoutDescription`属性引用一个`MotionScene`。`MotionScene`是XML资源文件，其中包含相应布局的所有运动描述。为了使布局信息与动作描述分开，每个动作都`MotionLayout`引用一个单独的`MotionScene`。请注意，`MotionScene`中的定义优先于中的任何类似定义`MotionLayout`。
    完整`MotionScene`示例如下：

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <MotionScene xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:motion="http://schemas.android.com/apk/res-auto">

        <Transition
            motion:constraintSetStart="@+id/start"
            motion:constraintSetEnd="@+id/end"
            motion:duration="1000">
            <OnSwipe
                motion:touchAnchorId="@+id/button"
                motion:touchAnchorSide="right"
                motion:dragDirection="dragRight" />
        </Transition>

        <ConstraintSet android:id="@+id/start">
            <Constraint
                android:id="@+id/button"
                android:layout_width="64dp"
                android:layout_height="64dp"
                android:layout_marginStart="8dp"
                motion:layout_constraintBottom_toBottomOf="parent"
                motion:layout_constraintStart_toStartOf="parent"
                motion:layout_constraintTop_toTopOf="parent" />
        </ConstraintSet>

        <ConstraintSet android:id="@+id/end">
            <Constraint
                android:id="@+id/button"
                android:layout_width="64dp"
                android:layout_height="64dp"
                android:layout_marginEnd="8dp"
                motion:layout_constraintBottom_toBottomOf="parent"
                motion:layout_constraintEnd_toEndOf="parent"
                motion:layout_constraintTop_toTopOf="parent" />
        </ConstraintSet>
    </MotionScene>
    ```
## 3、属性说明：  

* `<Transition>` 包含了动作的基本定义。
    * `motion:constraintSetStart`和`motion:constraintSetEnd`对动作的端点引用。这些端点是在`<ConstraintSet>`中定义。
    * `motion:duration` 指定运动完成所需的**毫秒数**。
* `<OnSwipe>` 可以让用户以触摸控制动作。  
    * `motion:touchAnchorId` 指定可以滑动和拖动的视图。
    * `motion:touchAnchorSide` 表示我们正在从右侧拖动视图。
    * `motion:dragDirection`指拖动的进度方向。例如，这 `motion:dragDirection="dragRight"`意味着随着向右拖动，进度会增加。
* `<ConstraintSet>`可以在其中定义描述运动的各种约束。在以上示例中，`ConstraintSet`为动作的每个端点定义一个约束。这些端点垂直居中（通过 `app:layout_constraintTop_toTopOf="parent"`和 `app:layout_constraintBottom_toBottomOf="parent"`）。在水平方向上，端点在屏幕的最左侧和右侧。

有关`MotionScene`支持的各种元素的详细信息，请参见 [MotionLayout示例](https://developer.android.com/training/constraint-layout/motionlayout/examples)。

### 1. 插值属性  

在`MotionScene`文件中，`ConstraintSet`元素可以包含在过渡期间内插的其他属性。除了位置和界限外，还通过以下方式插入以下属性`MotionLayout`：

* alpha
* visibility
* elevation
* rotation, rotationX, rotationY
* translationX, translationY, translationZ
* scaleX, scaleY

### 2. 自定义属性

在标签`<Constraint>`中，可以使用`<CustomAttribute>`标签为与**位置**或**View属性**相关的属性指定过渡。

```xml
<Constraint
    android:id="@+id/button" ...>
    <CustomAttribute
        motion:attributeName="backgroundColor"
        motion:customColorValue="#D81B60"/>
</Constraint>
```

一个`<CustomAttribute>`标签包含它自己的两个属性：

* `motion:attributeName`，是必需的，**并且必须有`getter`和`setter`方法匹配。** 例如，`backgroundColor`由于我们的视图具有基础`getBackgroundColor()`和`setBackgroundColor()`方法，因此受支持。

* 另一个必须提供的属性基于值类型。从以下受支持的类型中进行选择：

  * `motion:customColorValue` 用于颜色
  * `motion:customIntegerValue` 对于整数
  * `motion:customFloatValue` 用于浮点数
  * `motion:customStringValue` 对于字符串
  * `motion:customDimension` 尺寸，大小
  * `motion:customBoolean` 对于布尔
    
  **注意:** 指定自定义属性时，必须在开始和结束<ConstraintSet>元素中都定义端点值。  

以上面例子为基础，将视图更改颜色作为其运动的一部分。给每个`ConstrainSet`标签添加一个`CustomAttribute`标签，如下：

```xml
<ConstraintSet android:id="@+id/start">
    <Constraint
        android:id="@+id/button"
        android:layout_width="64dp"
        android:layout_height="64dp"
        android:layout_marginStart="8dp"
        motion:layout_constraintBottom_toBottomOf="parent"
        motion:layout_constraintStart_toStartOf="parent"
        motion:layout_constraintTop_toTopOf="parent">
        <CustomAttribute
            motion:attributeName="backgroundColor"
            motion:customColorValue="#D81B60" />
    </Constraint>
</ConstraintSet>

<ConstraintSet android:id="@+id/end">
    <Constraint
        android:id="@+id/button"
        android:layout_width="64dp"
        android:layout_height="64dp"
        android:layout_marginEnd="8dp"
        motion:layout_constraintBottom_toBottomOf="parent"
        motion:layout_constraintEnd_toEndOf="parent"
        motion:layout_constraintTop_toTopOf="parent">
        <CustomAttribute
            motion:attributeName="backgroundColor"
            motion:customColorValue="#9999FF" />
    </Constraint>
</ConstraintSet>
```

### 3. 其它MotionLayout属性  

除了上面示例中的`MotionLayout`属性外，还可以设置其他属性：

* `app:applyMotionScene="boolean"`是否应用`MotionScene`。此属性的默认值为**true**。
* `app:showPaths="boolean"`是否在动作运行时显示运动路径。此属性的默认值为**false**。
* `app:progress="float"`可以明确指定过渡进度。可以使用从0（转换开始）到1 （转换结束）的任何浮点值。
* `app:currentState="reference"`可以指定特定的`ConstraintSet`。
* `app:motionDebug`可以显示有关动作的其他调试信息。值可为`“ SHOW_PROGRESS”`，`“ SHOW_PATH”`或`“ SHOW_ALL”`。

>参考：
>* [https://developer.android.com/reference/androidx/constraintlayout/motion/widget/MotionLayout](https://developer.android.com/reference/androidx/constraintlayout/motion/widget/MotionLayout)
>* [https://developer.android.com/training/constraint-layout/motionlayout#androidx](https://developer.android.com/training/constraint-layout/motionlayout#androidx)
>* [使用MotionLayout对Android Apps进行动画处理（codelab）](https://codelabs.developers.google.com/codelabs/motion-layout/#2)


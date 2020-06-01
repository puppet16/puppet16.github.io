---
title: 使用IntDef注解代替ENUM
date: 2020-05-31 21:01:29
tags: [Android, 注解]
summary: 注解代替枚举
---

<!-- toc -->

## 一、前言  
在android系统中，不推荐使用ENUM类型，因为他占用内存较大，所以一般使用静态常量来代替枚举，但是有些场景我们只需要某几个固定的或一个范围内的值。此时静态常量就没有办法用来检查我们传递的是不是自己想要的值，这个场景下可以使用这两个注解来完成，它会在编译的时候检查我们的赋值是否符合要求，提前发现错误。

## 二、依赖  
这两个注解实现需要`com.android.support:support-annotations`依赖包  


## 三、实例 
StringDef用法与IntDef相同，以下为一个IntDef例子：  

```
import android.support.annotation.IntDef;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

public class IntDefDemo {
    public static final int TEST_ONE = 1;
    public static final int TEST_TWO = 2;
    public static final int TEST_THREE = 3;

    @IntDef({TEST_ONE, TEST_TWO, TEST_THREE})
    @Retention(RetentionPolicy.SOURCE)
    public @interface Test{}

    private @Test int mTestValue;

    public void setTestValue(@Test int value) {
        mTestValue = value;
    }

    public @Test int getTestValue() {
        return mTestValue;
    }
}

```
## 四、Retention说明 
### 其中Retention有三种模式：  
- **`RetentionPolicy.SOURCE`**：注解只保留在源码文件，当Java文件编译成class文件的时候，注解信息会被丢弃，不会保留在编译好的class文件中。生命周期对应**Java源文件(.java文件)**  
- **`RetentionPolicy.CLASS`**：注解被保留到class文件，但jvm加载class文件时候被遗弃，***默认值***。生命周期对应 **.class文件**  
- **`RetentionPolicy.RUNTIME`**：注解不仅被保存到class文件中，jvm加载class文件之后，仍然存在，因此可以通过反射机制读取注解的信息。生命周期对应 **内存中的字节码dex**
<br>
### 如何选择合适的注解生命周期

1. 要明确生命周期长度 SOURCE < CLASS < RUNTIME ，所以前者能作用的地方后者一定也能作用。  
2. 一般如果需要在运行时去动态获取注解信息，那只能用 RUNTIME 注解  
3. 如果要在编译时进行一些预处理操作，比如生成一些辅助代码（如 ButterKnife），就用 CLASS注解  
4. 如果只是做一些检查性的操作，比如 @Override 和 @SuppressWarnings，则可选用 SOURCE 注解。  

### RetentionPolicy源码

```
/*
 * Copyright (c) 2003, 2004, Oracle and/or its affiliates. All rights reserved.
 * DO NOT ALTER OR REMOVE COPYRIGHT NOTICES OR THIS FILE HEADER.
 *
 * This code is free software; you can redistribute it and/or modify it
 * under the terms of the GNU General Public License version 2 only, as
 * published by the Free Software Foundation.  Oracle designates this
 * particular file as subject to the "Classpath" exception as provided
 * by Oracle in the LICENSE file that accompanied this code.
 *
 * This code is distributed in the hope that it will be useful, but WITHOUT
 * ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or
 * FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public License
 * version 2 for more details (a copy is included in the LICENSE file that
 * accompanied this code).
 *
 * You should have received a copy of the GNU General Public License version
 * 2 along with this work; if not, write to the Free Software Foundation,
 * Inc., 51 Franklin St, Fifth Floor, Boston, MA 02110-1301 USA.
 *
 * Please contact Oracle, 500 Oracle Parkway, Redwood Shores, CA 94065 USA
 * or visit www.oracle.com if you need additional information or have any
 * questions.
 */

package java.lang.annotation;

/**
 * Annotation retention policy.  The constants of this enumerated type
 * describe the various policies for retaining annotations.  They are used
 * in conjunction with the {@link Retention} meta-annotation type to specify
 * how long annotations are to be retained.
 *
 * @author  Joshua Bloch
 * @since 1.5
 */
public enum RetentionPolicy {
    /**
     * Annotations are to be discarded by the compiler.
     */
    SOURCE,

    /**
     * Annotations are to be recorded in the class file by the compiler
     * but need not be retained by the VM at run time.  This is the default
     * behavior.
     */
    CLASS,

    /**
     * Annotations are to be recorded in the class file by the compiler and
     * retained by the VM at run time, so they may be read reflectively.
     *
     * @see java.lang.reflect.AnnotatedElement
     */
    RUNTIME
}

```

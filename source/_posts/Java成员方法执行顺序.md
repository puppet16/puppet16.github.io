---
title: Java成员方法执行顺序
date: 2020-05-31 21:49:39
tags: Java
summary: Java成员方法执行顺序
---

<!-- toc -->

## 一、三个成员方法：

1. **普通初始化块**：直接在类里写花括号，初始化块只在**创建Java对象时**隐式执行，而且在执行构造器之前执行。例子如下：

   ```
    public class XXX {
        {
            //普通初始化块的可执行性代码
        }
    }
   ```

2. **静态初始化块**：就是定义普通初始化块时使用了static修饰符，静态初始化块只在**在类初始化阶段执行**，而不是在创建对象时才执行，所以只会调用一次。例子如下：

   ```
    public class XXX {
        static {
            //静态初始化块的可执行性代码
        }
    }
   ```

3.  **构造方法**：与类名同名的方法，，在创建Java对象时执行。例子如下:

    ```
    public class A {
        A () {
            //构造方法的可执行性代码
        }
    }
    ```

## 二、代码实例

```
public class FatherClass {

    int value = 11;

    {
        value = 12;
        Log.d("Extends","普通初始化块-Father");
    }

    static {
         Log.d("Extends","静态初始化块-Father");
    }

    FatherClass () {
        Log.d("Extends","构造方法-Father");
    }
}
```

```
public class ChildClass extends FatherClass {

    {
        Log.d("Extends","普通初始化块-ChildClass");
    }

    static {
         Log.d("Extends","静态初始化块-ChildClass");
    }

    ChildClass() {
        Log.d("Extends","构造方法-ChildClass");
    }
}
```

执行：

```
ChildClass childClass = new ChildClass();
Log.d("Extends","value值："+childClass.value);
```

输出结果为：

```
D/Extends: 静态初始化块-Father
D/Extends: 静态初始化块-ChildClass
D/Extends: 普通初始化块-Father
D/Extends: 构造方法-Father
D/Extends: 普通初始化块-ChildClass
D/Extends: 构造方法-ChildClass
D/Extends: value值：12
```

## 三、结论

1. 单个类执行顺序为：静态初始化块—>普通初始化块—>构造方法。但是静态初始化块只会执行一次。

2. 继承父类后调用子类时执行顺序为：父类静态初始化块—>子类静态初始化块—>父类普通初始化块—>父类构造方法—>子类普通初始化块—>子类构造方法。

3. 关于普通赋值：普通初始化块、声明实例变量指定的默认值都可以是对象的初始化代码，执行顺序与代码中的排列顺序相同。如FatherClass中，先执行int value = 11;再执行了value = 12；所以value结果为12。若是将int value == 11;放于普通初始化块后，如

   ```
   
   {
      value = 12;
      Log.d("Extends","普通初始化块-Father");
   }
   int value = 11;
   ```

   则value的值为11。

4. 关于静态变量赋值：与普通赋值一致。



*参考：[https://www.cnblogs.com/wft1990/p/7998353.html](https://www.cnblogs.com/wft1990/p/7998353.html)* 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*[https://blog.csdn.net/qq_34626097/article/details/83352293](https://blog.csdn.net/qq_34626097/article/details/83352293)*

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*[https://blog.csdn.net/china_songlei/article/details/79696583](https://blog.csdn.net/china_songlei/article/details/79696583)*


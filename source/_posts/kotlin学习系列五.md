---
title: Kotlin学习系列五：类型进阶
date: 2021-02-02 10:24:05
tags: [Kotlin, 类型]
summary: Kotlin 类型进阶
---

<!-- toc -->

# 一、前言

1. 本文主要讲述**Kotlin 类型进阶**
2. **Kotlin官网：[https://kotlinlang.org/](https://kotlinlang.org/)**
3. **Kotlin中文官网：[https://www.kotlincn.net/](https://www.kotlincn.net/)**
4. Kotlin 学习系列文章：
    * {% post_link kotlin学习系列一 kotlin学习系列一：内置类型 %}
    * {% post_link kotlin学习系列二 kotlin学习系列二：类与接口初解 %}
    * {% post_link kotlin学习系列三 kotlin学习系列三：表达式 %}
    * {% post_link kotlin学习系列四 kotlin学习系列四：函数进阶 %}

# 二、 类的构造器

## 1. 基本写法

将构造器写到类的定义上

```kotlin
class Person constructor(var age:Int, name:String) {
    
}
```

**说明：**
1. 如上定义一个类`Person`，其中关键字`constructor`可省略
2. 同时定义了类内的属性`age`,类内全局可见
3. 同时定义了一个形参`name`，构造器内可见(init块，属性初始化)

**init块：**

```kotlin
class Person(var age:Int, name:String) {
    var name: String
    init {
        this.name = name
    }
    val firstName = name.split(" ")[0]
    init {
        println("firstName:$firstName")
    }
}
```

**说明：**

1. `init`块类似于**主构造器** 的方法体，`init`块可以有多个
2. 多个`init`块会按**从上到下** 的顺序合并执行，上面示例执行顺序如下：

    ```kotlin
    class Person(var age: Int, name: String) {
        var name: String
        var firstName: String

        init {
            this.name = name
            firstName = name.split(" ")[0]
        }
    }
    ```


`Java`中有类似功能，名为构造块

```java
public static class Person {
    private String name;
    private int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    {
        System.out.println("Person-构造块");
    }

    static {
        System.out.println("Person-静态构造块");
    }
}
```

**说明：**

1. `java`中执行顺序为：静态构造块-->构造块-->构造方法
2. 构造块访问不到构造方法内的形参


## 2. 类的继承

`Kotlin`中类的继承需要在子类后而加`:父类构造器`

```kotlin
class Person(var age: Int,name: String) :Animal(){
    var personName: String
    var firstName: String

    init {
        this.personName = name
        firstName = name.split(" ")[0]
    }
}

abstract class Animal(var name: String) {
    constructor():this("unknown")
}

```

## 3. 副构造器

定义了主构造器后，在类内部再定义的构造器都被称为**副构造器**

```kotlin
fun main() {
    val person = Person()
}
class Person(var age: Int,name: String){
    constructor():this(20, "unknown")
}
```

**说明：** 

1. 副构造器必须调用主构造器
2. 若子类没有主构造器，则不必在继承父类时调用构造器，可在**副构造器时调用父类的构造器**

    ```kotlin
    class Person :Animal{
        var personName: String
        constructor(personName: String): super("unknown"){
            this.personName = personName
        }
    }

    abstract class Animal(var name: String) {
    }
    ```

3. 建议创建类时采用**主构造器+默认参数**的方式，这样构造路径较少，减少复杂度

**@JvmOverloads**注解

**作用：** 主构造器的**默认参数** 在`Java`代码中可以以重载的形式调用，即可省略

```kotlin
//kotlin

class Person @JvmOverloads constructor(var age: Int = 18,name: String){
    constructor():this(20, "unknown")
}
```

```java
//java
Person person = new Person("lee");
```

如上，若`kotlin`中主构造器前不加`@JvmOverloads`注解，则`java`中的调用语句会报错。

## 3. 构造同名的工厂函数

在`Kotlin`中，定义了一个类情况下，还可以定义同名的函数，该同名函数一般用于构建同名类对象

```kotlin
val str = String()
val str1 = String(charArrayOf('1','2'))
```

如上，第一个语句调用的是类`String`的构造器来创建一个`String`对象，第二条语句是调用的函数`String()`来创建一个`String`对象，该函数入参为`CharArray`。还可以自定义同名的函数，如下

```kotlin
fun String(ints: IntArray): String {
    return ints.contentToString()
}
```

# 三、 类与成员的可见性

## 1. 可见性对比

|可见性类型|Java|Kotlin|
|--|--|--|
|public|公开|与java相同，默认类型|
|internal|X|模块内可见|
|default|包内可见，默认|X|
|protected|包内及子类可见|类内及子类可见|
|private|类内可见|类或文件内可见|


## 2. 修饰对象

|可见性类型|顶级声明|类|成员|
|--|--|--|--|
|public|可以|可以|可以|
|internal|可以|可以|可以|
|protected|不可|不可|可以|
|private|可以|可以|可以|

**说明：**

1. `internal`虽然所有都可以修饰，但是被修饰的对象是模块内可见
2. `private`虽然所有都可以修饰，但是修饰顶级声明和类时，只是文件内可见；修饰成员的话只是类内可见

**模块**：在一次调用命令`kotlinc`的文件当中，`internal`可见，直观的讲，大致可认为一个`Jar`包、一个`aar`

## 3. `internal`与`default`的对比

1. 一般由SDK或公共组件开发者用于隐藏模块内部细节实现
2. `default`可通过外部创建相同包名来访问，访问控制非常弱
3. `default`会导致不同抽象层次的类聚集到相同包之下
4. `internal`可方便处理内外隔离，提升模块代码内聚、减少接口暴露
5. `internal`修饰的`Kotlin`类或成员在`Java`当中可直接访问，此时若想`Java`不能调用，需要借助注解`@JvmName()`为将类或成员添加一个`Java`识别的**不符合java命名规范的名称**

    ```kotlin
    //kotlin
    internal class Person(var name: String = "Lee", var age:Int = 18) {
        internal fun printPerson() {
            println("Person{name=$name, age=$age}")
        }
    }
    ```

    ```java
    //java
    Person person = new Person();
    person.printPerson$ProjectCollection_app();
    ```
    
    如上所示，在`kotlin`中声明了一个`internal`的类`Person`，其中有一个用于打印信息的函数`printPerson()`，正常情况下，在`Java`文件中，创建`Person`对象后，可直接调用该函数，在`java`中该函数名字是*方法名$模块名*
    <br/>

    ```kotlin
    //kotlin
    internal class Person(var name: String = "Lee", var age:Int = 18) {
        @JvmName("123")
        internal fun printPerson() {
            println("Person{name=$name, age=$age}")
        }
    }
    ```

    ```java
    //java
    Person person = new Person();
    person.123();
    ```

    如上所示，为函数`printPerson()`添加了注解`@JvmName`,让`java`识别该函数名为`123`,但因`Java`不支持该命名方式，所以不可以调用该函数

## 4. 构造器的可见性

在声明类时若要为声明的主构造器限制作用域，则不可以省略关键字`constructor`
比如说要将该类写成一个单例或写成工厂，则应该将构造器私有化

```kotlin
class Person private constructor(var age:Int, var name:String)
```

## 5. 属性的可见性

可在声明属性时直接加可见性关键字进行修饰

```kotlin
class Person private constructor(private var age:Int, var name:String)
```

如上所示，将属性`age`私有化，则创建对象`person`后，无法访问`age`属性。

还可以为属性的`setter`添加可见性

```kotlin
class Person(var name: String, private var age:Int) {
     var firstName: String = ""
        private set(value) {
            field = value
        }
}
```

1. 不可以设置`getter`的可见性，因为`getter`可见性要与属性保持一致
2. `setter`的可见性不得大于属性的可见性

## 6. 顶级声明的可见性

1. 顶级声明批文件内直接定义的属性、函数、类等
2. 顶级声明不支持`projected`，在`kotlin`中只有子类可见的含义
3. 顶级声明被`private`修饰表示文件内部可见

# 四、 类属性的延迟初始化

## 1. 延迟初始化原因

1. 类属性必须在构造时初始化
2. 某些成员只有在类构造之后才会被初始化

```java
private TextView mTvName;

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_test);
    mTvName = findViewById(R.id.tv_name);
}
```

如上，`Android`中创建了一个名为`mTvName`的`TextView`，并在`onCreate()`方法中为其赋值。但是转换为`Kotlin`代码时却会在声明变量时报错，因为没有初始化，此时可将变量先定义为可空，然后赋空，再在`onCreate()`方法中为其赋正确的值。

## 2. 延迟初始化实现

### 1. 为需要延迟初始化的成员赋空值

```kotlin
private var mTvName: TextView? = null

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_test)
    mTvName = findViewById(R.id.tv_name);
}
```

如上，将变量`mTvName`初始化为了`null`，再在`onCreate()`方法中为其赋正确的值，虽然解决了要延后初始化的问题，但每次使用`mTvName`的属性时都要将其做判空处理或进行类型强转。为解决以上问题可使用关ß键字`lateinit`

### 2. 使用关键字`lateinit`

可在需要延迟初始化的成员声明前加上关键字`lateinit`

```kotlin
private lateinit var mTvName: TextView

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_test)
    mTvName = findViewById(R.id.tv_name);
}
```

**说明：**

1. 使用关键字`lateinit`修饰的属性必须定义为`var`，允许再次赋值
2. 关键字`lateinit`会让编译器忽略变量的初始化，不支持`Int`等基本类型，只能修饰**非基本类型**
3. 在`Kotlin`的1.2版本之后，为属性添加了扩展方法`isInitialized`，可使用该方法来判断被`lateinit`修饰的属性是否被初始化，已初始化返回`true`
4. 必须在能够完全确定变量值的生命周期下使用`lateinit`
5. 不要在复杂的逻辑中使用`lateinit`，否则可读性会很差，不能很清晰的知道在哪里初始化

### 3. 使用`lazy`

```kotlin
private val mTvName by lazy {
    findViewById<TextView>(R.id.tv_name)
}
```

**说明：**

1. 如上代码使用的是属性代理，`lazy`本身是属性代理的实现，只有`mTvName`首次被访问时才会执行`lambda`表达式内代码
2. 如上将声明和初始化放到了一起，可读性较强

### 4. 方案对比

|方案名称|优缺点|推荐|
|--|--|--|
|可空类型|增代码复杂度；初始化与声明分离；调用处需要做判空处理|不推荐|
|lateinit|初始化与声明分离；调用处虽无需判空处理，但潜在的初始化问题可能被掩盖|一般|
|lazy|初始化与声明内聚；无需声明可空类型|推荐|

# 五、 代理Delegate

**接口代理：** 某对象代替某个类实现某个接口
**属性代理：** 某对象代替某属性实现`getter/setter`方法

## 1. 接口代理

```kotlin

interface TextWatcher {

    fun beforeTextChanged()

    fun onTextChanged()

    fun afterTextChanged();
}
```

如上声明一个接口`TextWatcher`，其中三个方法，用于监听文本改变之前，文本改变时，文本改变之后三个状态。

```kotlin
class TextWatcherImpl : TextWatcher {
    override fun beforeTextChanged() {
        println("beforeTextChanged")
    }

    override fun onTextChanged() {
        println("onTextChanged")
    }

    override fun afterTextChanged() {
        println("afterTextChanged")
    }
}
```

如上所示，正常的一个类实现接口时，需要将接口的所有方法全部实现。此时若有一个业务场景：需要在`afterTextChanged()`方法内做一些其它逻辑，其它两个方法还沿用之前的逻辑，此时应创建一个`wrapper`类，包装一下`TextWathcerImpl`

```kotlin
class TextWatcherImplWrapper(val watcher: TextWatcher) : TextWatcher {
    override fun beforeTextChanged() {
        watcher.beforeTextChanged()
    }

    override fun onTextChanged() {
        watcher.onTextChanged()
    }

    override fun afterTextChanged() {
        watcher.afterTextChanged()
        println("wrapper")
    }
}

```

如上所示，创建一个类`TextWatcherImplWrapper`除了继承`TextWatcherImpl`类的功能外，在`afterTextChanged()`方法里又做了其它操作，相较于`TextWatcherImpl`类，除了在`afterTextChanged()`方法做了修改，另外两个方法没有修改，但是还必须创建出来

```kotlin
class TextWatcherImplWrapper(val watcher: TextWatcher) : TextWatcher by watcher {
    
    override fun afterTextChanged() {
        watcher.afterTextChanged()
        println("wrapper")
    }
}

```

**说明：**

1. `by`关键字可以看作是委托，将某操作委托给某对象
2. `:TextWatcher by watcher`作用是使对象`watcher`代替类`TextWatcherImplWrapper`实现接口`TextWatcher`
3. 对于对象`watcher`的唯一要求是实现了被代理的接口也就是`TextWatcher`
4. 使用此方法后，只需要在包装类内实现需要修改的方法即可，编译器会在编译时将其它方法添加到该类内，编译后的`TextWatcherImplWrapper`代码如下：

    ```java
    public final class TextWatcherImplWrapper implements TextWatcher {
        @NotNull
        private final TextWatcher watcher;

        public TextWatcherImplWrapper(@NotNull TextWatcher watcher) {
            this.watcher = watcher;
        }

        @NotNull
        public final TextWatcher getWatcher() {
            return this.watcher;
        }

        public void beforeTextChanged() {
            this.watcher.beforeTextChanged();
        }

        public void onTextChanged() {
            this.watcher.onTextChanged();
        }

        public void afterTextChanged() {
            this.watcher.afterTextChanged();
            String var1 = "wrapper";
            System.out.println(var1);
        }
    }
    ```

## 2. 属性代理

### 1. lazy

**lazy的源码如下：**

```kotlin
@file:kotlin.jvm.JvmName("LazyKt")
import kotlin.reflect.KProperty

public actual fun <T> lazy(initializer: () -> T): Lazy<T> = SynchronizedLazyImpl(initializer)

private class SynchronizedLazyImpl<out T>(initializer: () -> T, lock: Any? = null) : Lazy<T>, Serializable {
    private var initializer: (() -> T)? = initializer
    @Volatile private var _value: Any? = UNINITIALIZED_VALUE
    // final field is required to enable safe publication of constructed instance
    private val lock = lock ?: this

    override val value: T
        get() {
            val _v1 = _value
            if (_v1 !== UNINITIALIZED_VALUE) {
                @Suppress("UNCHECKED_CAST")
                return _v1 as T
            }

            return synchronized(lock) {
                val _v2 = _value
                if (_v2 !== UNINITIALIZED_VALUE) {
                    @Suppress("UNCHECKED_CAST") (_v2 as T)
                } else {
                    val typedValue = initializer!!()
                    _value = typedValue
                    initializer = null
                    typedValue
                }
            }
        }

    override fun isInitialized(): Boolean = _value !== UNINITIALIZED_VALUE

    override fun toString(): String = if (isInitialized()) value.toString() else "Lazy value not initialized yet."

    private fun writeReplace(): Any = InitializedLazyImpl(value)
}

@kotlin.internal.InlineOnly
public inline operator fun <T> Lazy<T>.getValue(thisRef: Any?, property: KProperty<*>): T = value

```

**说明：**

1. `lazy`实际上是一个函数，它接收了一个函数，即`lazy`为高阶函数
2. `lazy`函数返回一个`lazy`对象，该对象代理了**某个对象的实例的某属性的`getter`**
3. `lazy`只能代理只读属性 *(即用`val`修饰的属性)* 的`getter`
4. 其`lazy`的扩展函数`getValue()`,其中入参`thisRef: Any?`表示属性所在类的实例；`property: KProperty<*>`表示所代理的属性的引用


```kotlin
class Person(val name: String) {
    val firstName by lazy { name.split(" ")[0] }
}
```

如上，接口`lazy`的实例代理了类`Person`的实例的属性`firstName`的`getter`方法，每次访问`firstName`时都会将`lazy`内的值返回出来，只有第一次调用时会计算，之后会存储起来。经过编译器编译后如下：

```kotlin

public final class Person {
   @NotNull
   private final Lazy firstName$delegate;
   @NotNull
   private final String name;

   @NotNull
   public final List getFirstName() {
      Lazy var1 = this.firstName$delegate;
      return (List)var1.getValue();
   }

   @NotNull
   public final String getName() {
      return this.name;
   }

   public Person(@NotNull String name) {
      this.name = name;
      this.firstName$delegate = LazyKt.lazy((Function0)(new Function0() {
         // $FF: synthetic method
         // $FF: bridge method
         public Object invoke() {
            return this.invoke();
         }

         @NotNull
         public final List invoke() {
            return StringsKt.split$default((CharSequence)Person.this.getName(), new char[]{" ".charAt(0)}, false, 0, 6, (Object)null);
         }
      }));
   }
}
```


### 2. observable

`observable`源码如下：

```kotlin
package kotlin.properties
import kotlin.reflect.KProperty

public object Delegates {

public inline fun <T> observable(initialValue: T, crossinline onChange: (property: KProperty<*>, oldValue: T, newValue: T) -> Unit):
        ReadWriteProperty<Any?, T> =
    object : ObservableProperty<T>(initialValue) {
        override fun afterChange(property: KProperty<*>, oldValue: T, newValue: T) = onChange(property, oldValue, newValue)
    }
}
...
```

```kotlin
package kotlin.properties
import kotlin.reflect.KProperty

public abstract class ObservableProperty<V>(initialValue: V) : ReadWriteProperty<Any?, V> {
    private var value = initialValue

    protected open fun beforeChange(property: KProperty<*>, oldValue: V, newValue: V): Boolean = true

    protected open fun afterChange(property: KProperty<*>, oldValue: V, newValue: V): Unit {}

    public override fun getValue(thisRef: Any?, property: KProperty<*>): V {
        return value
    }

    public override fun setValue(thisRef: Any?, property: KProperty<*>, value: V) {
        val oldValue = this.value
        if (!beforeChange(property, oldValue, value)) {
            return
        }
        this.value = value
        afterChange(property, oldValue, value)
    }
}
```

```kotlin
package kotlin.properties
import kotlin.reflect.KProperty

public interface ReadWriteProperty<in T, V> : ReadOnlyProperty<T, V> {

    public override operator fun getValue(thisRef: T, property: KProperty<*>): V

    public operator fun setValue(thisRef: T, property: KProperty<*>, value: V)
}
```

**说明：**

1. `Delegates.observable`返回了一个对象`ObservableProperty`,该对象实现了接口`ReadWriteProperty`，因此有`getValue()`方法和`setValue()`方法，因此所代理的属性应为**可读写属性** *(即用var声明的属性)*
2. 接口`ReadWriteProperty`的`getValue()`方法和`setValue()`方法第一个参数为该属性所在类，第二个参数为所要代理的属性
3. `ObservableProperty`的实例代理了属性的`getter`和`setter`
4. 在`setValue()`方法中会执行`afterChange()`函数，`afterChange()`函数包含三个入参`property`表示当前属性，`oldValue`表示改变之前的值，`value`表示改变之后的值

示例代码如下：

```kotlin
class Person(var name: String) {
    //代理getter
    val firstName by lazy { name.split(" ")[0] }
    var secondName: String by Delegates.observable(name.split(" ")[1]) { property, oldValue, newValue ->
        println("value changed from $oldValue -> $newValue")
    }
}
```

**说明：**

1. 定义属性时使用关键字`by`定义要代理属性的对象
2. `Delegates.observable`对象后跟的`lambda`表达式即为上面提到的`afterChange()`函数,它可以获取当前属性、改变之前的值、改变之后的值


### 3. 自定义属性代理类

经过看`lazy`和`observale`源码可以发现，他们之所以可以代理属性，是因为实现了`getVaue()`、`setValue()`这两个运算符，所以只要实现了这两个运算符可即代理属性

```kotlin
fun main() {
    var str:String by ProxyX("321")
    println(str)
    str = "123"
    println(str)
}

class ProxyX(private var initialValue: String) {
    private var value = initialValue

    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$value-Proxy"
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        this.value = this.value + value
    }
}
```

打印结果为

```kotlin
321-Proxy
321123-Proxy
```

**说明：**

1. 声明了一个代理类`ProxyX`，代理可读写属性，传入一个初始值，实现`getValue()`方法和`setValue()`方法
2. `getValue()`方法功能为：取值时返回一个字符串，该字符串是值加一个常量
3. `setValue()`方法功能为：将新添加的值拼接到原值的后面

### 4. 自定义属性代理

**实现效果：** 将`.properties`文件中的内容以变量的方式展示，以方便用户读写，文件中包含`author`、`version`、`desc`等信息

在工程里的`resources`文件夹下创建`Config.properties`文件，如下图所示：

![Config.properties位置](/kotlin学习系列五/kotlin_proxy_properties_1.png)


```kotlin
class PropertiesDelegate(private val path: String, private val defaultValue: String = "") {
    private lateinit var url: URL
    private val properties: Properties by lazy {
        val prop = Properties()
        url = try {
            javaClass.getResourceAsStream(path).use {
                prop.load(it)
            }
            javaClass.getResource(path)!!
        } catch (e: Exception) {
            try {
                ClassLoader.getSystemClassLoader().getResourceAsStream(path).use {
                    prop.load(it)
                }
                ClassLoader.getSystemClassLoader().getResource(path)!!
            } catch (e: Exception) {
                FileInputStream(path).use {
                    prop.load(it)
                }
                URL("file://${File(path).canonicalPath}")
            }
        }
        prop
    }

    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return properties.getProperty(property.name, defaultValue)
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        properties.setProperty(property.name, value)
        File(url.toURI()).outputStream().use {
            properties.store(it, "Hello")
        }
    }
}

abstract class AbsProperties(path: String) {
    protected val prop = PropertiesDelegate(path)
}

class Config: AbsProperties("Config.properties") {
    var author by prop
    var version by prop
    var desc by prop
}

fun main() {
    val config = Config()
    println(config.author)
    println(config.version)
    config.version = "1.1.0"
    println(config.desc)
    println(config.version)
}
```

**说明：**

1. `resources`文件夹下内容编译完成后会被复制到`classpath`下，所以使用`ClassLoader`可以读取到文件
2. `resources`文件夹内容编译后位置如下图所示：

    ![confg.properties编译后位置](/kotlin学习系列五/kotlin_proxy_properties_2.png)

3. 上述代码创建一个`Properties`的代理类`PropertiesDelegate`，入参为一个地址，功能为：获取该地址文件里的所有属性值。
4. 注意创建的变量名要与`Config.properties`文件中的键值对里的键名一致，否则无法读取出属性
5. 代理类`PropertiesDelegate`也可以重新设置属性值，设置的属性值保存在编译后的文件内，并没有改变源文件，在修改时会自动添加所写的`comments`即`Hello`，并且自动添加了修改时间，效果上看上去就像是一个提交纪录

    ![修改后的config.properties](/kotlin学习系列五/kotlin_proxy_properties_3.png)

# 六、 单例 `object`

## 1. 定义

`Kotlin`中使用关键字`object`来定义**饿汉式单例**

```kotlin
object Singleton {

}
```

如上代码所示，只需要关键字`object`后加一个类名即可创建一个饿汉式单例，它等价于`Java`中如下所示的单例定义：

```java
public class Singleton {
    public static final Singleton INSTANCE = new Singleton();
}
```

## 2. 访问`object`的成员

`object`定义的单例可以当做类一样去添加成员属性，成员函数

```kotlin
object Singleton {
    var value: Int = 26
    fun plus() {
        value++
    }
}

fun main() {
    Singleton.plus()
    println("value${Singleton.value}")
}
```

```java
public static void main(String[] args) {
    Singleton.INSTANCE.plus();
    System.out.println("value:"+ Singleton.INSTANCE.getValue());
}
```

如上所示，在`kotlin`定义了一个单例`Singleton`，在`kotlin`中调用时直接使用单例的名字即可调用，但是在`java`调用时需要加一个`INSTANCE`，上述定义单例代码经过编译反后的`class`文件代码如下：

```java
public final class Singleton {
   private static int value;
   @NotNull
   public static final Singleton INSTANCE;

   public final int getValue() {
      return value;
   }

   public final void setValue(int var1) {
      value = var1;
   }

   public final void plus() {
      int var10000 = value++;
   }

   private Singleton() {
   }

   static {
      Singleton var0 = new Singleton();
      INSTANCE = var0;
      value = 26;
   }
}
```

## 3. 静态成员`@JvmStatic`

因`kotlin`是跨平台语言，其中`C`语言、`JavaScript`等都没有静态成员概念，所在`kotlin`没有静态成员的概念，但可以使用注解`JvmStatic`来模拟。

### 1. 在单例中使用`@JvmStatic`

```kotlin
object Singleton {
    @JvmStatic var value: Int = 26
    @JvmStatic fun plus() {
        value++
    }
}

fun main() {
    Singleton.plus()
    println("value${Singleton.value}")
}
```

```java
public static void main(String[] args) {
    Singleton.plus();
    System.out.println("value:"+ Singleton.getValue());
}
```

**说明：**

1. 如上所示，在`kotlin`中定义单例时为其成员添加了注解`@JvmStatic`。
2. 在`kotlin`中调用单例成员时没有任何变化
3. 在`java`中调用单例成员时可省略`INSTANCE`，可直接视同其调用静态成员
   

上述定义添加注解的单例代码经过反编译后的`class`文件代码如下：

```java
public final class Singleton {
   private static int value;
   @NotNull
   public static final Singleton INSTANCE;

   public static final int getValue() {
      return value;
   }

   public static final void setValue(int var0) {
      value = var0;
   }

   @JvmStatic
   public static final void plus() {
      int var10000 = value++;
   }

   private Singleton() {
   }

   static {
      Singleton var0 = new Singleton();
      INSTANCE = var0;
      value = 26;
   }
}
```

### 2. 在普通类中使用`@JvmStatic`

**注解`JvmStatic`只能使用在单例或普通类的伴生对象里的成员上**
**伴生对象 *(companion objects)*：** 是一个类的伴生对象，定义一个类时可以同时在类内定义定义一个`object`

```kotlin
class Foo {
    companion object {
        @JvmField var value:String = "Test"
        @JvmStatic fun y() {
            println("Test")
        }
    }
}

fun main() {
    Foo.y()
    println("value:${Foo.value}")
}
```

```java
public static void main(String[] args) {
    Foo.y();
    System.out.println("value:"+ Foo.value);
}
```


**说明：**

1. 如上所示，在`kotlin`中定义伴生对象时使用`companion object`
2. 在`kotlin`中调用类的伴生对象中的静态成员时，直接使用类名即可
3. 在`java`中调用类的伴生对象中的静态成员时，直接使用类名即可
   

上述定义伴生对象的类代码经过反编译后的`class`文件代码如下：

```java
public final class Foo {
   @JvmField
   @NotNull
   public static String value = "Test";
   @NotNull
   public static final Foo.Companion Companion = new Foo.Companion((DefaultConstructorMarker)null);

   @JvmStatic
   public static final void y() {
      Companion.y();
   }

   public static final class Companion {
      @JvmStatic
      public final void y() {
         String var1 = "Test";
         boolean var2 = false;
         System.out.println(var1);
      }

      private Companion() {
      }

      // $FF: synthetic method
      public Companion(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
}
```


## 4. `@JvmField`

**注解`@JvmField`作用：** 不生成`getter/setter`，但不会给所修饰的属性添加静态属性，只是该属性变成`public`

### 1. 在单例中使用`@JvmField`

```kotlin
object Singleton {
    @JvmField var value: Int = 26
    @JvmStatic fun plus() {
        value++
    }
}

fun main() {
    Singleton.plus()
    println("value${Singleton.value}")
}
```

```java
//java

public static void main(String[] args) {
    Singleton.plus();
    System.out.println("value:"+ Singleton.value);
}
```

**说明：**

1. 如上所示,为单例中的属性`value`添加注解`@JvmField`
2. `kotlin`中调用并没有改变
3. `java`中调用时直接使用该变量名，它成为了一个全局静态变量

上述定义添加注解的单例代码经过反编译后的`class`文件代码如下：

```java
public final class Singleton {
   @JvmField
   public static int value;
   @NotNull
   public static final Singleton INSTANCE;

   @JvmStatic
   public static final void plus() {
      int var10000 = value++;
   }

   private Singleton() {
   }

   static {
      Singleton var0 = new Singleton();
      INSTANCE = var0;
      value = 26;
   }
}
```

### 2. 在普通类中使用`@JvmField`

```kotlin
class Foo {
    @JvmField var x: Int = 22
}

fun main() {
    val foo = Foo()
    println(foo.x)
}
```

```java
public static void main(String[] args) {
    Foo foo = new Foo();
    System.out.println(foo.x);
}
```

**说明：**

1. 如上所示，在普通类中使用注解`@JvmField`修饰一个属性
2. 注解`@JvmField`并没有给属性静态属性，所以必须要声明一个对象再调用它


上述定义添加注解的普通类经过反编译后的`class`文件代码如下：

```java
public final class Foo {
   @JvmField
   public int x = 22;
}
```

## 5. `object`的构造器

`kotlin`中使用`object`定义的单例不能自定义构造器，系统会自动生成一个无参构造器，但是可以自定义若干个`init`块

`kotlin`中单例可以与普通类一样继承或实现其它的类或接口

```kotlin
object Singleton {
    @JvmField var value: Int = 26
    init {
        value *= 2
    }
}
```

上述代码经过反编译后的`class`文件代码如下：

```java
public final class Singleton {
   @JvmField
   public static int value;
   @NotNull
   public static final Singleton INSTANCE;

   private Singleton() {
   }

   static {
      Singleton var0 = new Singleton();
      INSTANCE = var0;
      value = 26;
      value *= 2;
   }
}
```

# 七、内部类

## 1. 定义

在`Java`中定义内部类，只需在一个类内再定义一个类，类内的类即为内部类，内部类分为一般内部类、静态内部类。一般内部类持有外部类的引用，所以有可能引起内存泄露，静态内部类没有外部类的引用，所以不能直接调用外部的方法。

```java
public class Outer {
    class Inner {

    }
    static class StaticInner {

    }
}
```

在`kotlin`中定义**非静态内部类** 使用关键字`inner`，而不加关键字的类内类是**静态内部类**

```kotlin
class Outer {
    inner class Inner
    class StaticInner
}
```

## 2. 实例化

对于内部类的实例化，`java`和`kotlin`操作相同。
初始化非静态内部类时需要**先构造外部类对象** ，然后再拿对象去初始化内部类
初始化静态内部类时，只引用了**外部类的类名** ，不需要初始化外部类


```kotlin
//kotlin --非静态内部类
val inner = Outer().Inner()
```

```java
//java --非静态内部类
Outer.Inner inner = new Outer().new Inner();
```

```kotlin
//kotlin --静态内部类
val staticInner = Outer.StaticInner()
```

```java
//java --静态内部类
Outer.StaticInner inner = new Outer.StaticInner();
```

## 单例`object`的内部`object`

```kotlin
object OuterObject {
    object InnerObject
}
```
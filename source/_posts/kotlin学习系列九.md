---
title: kotlin学习系列九：协程初解
date: 2021-04-15 10:03:09
tags: [Kotlin, 协程]
summary: Kotlin 协程初解
---

<!-- toc -->

# 一、前言

1. 本文主要讲述**Kotlin 协程**
2. *本文是对[Bennyhuo老师](https://github.com/enbandari)讲解的`Kotlin`系列视频的总结笔记*
3. **Kotlin官网：[https://kotlinlang.org/](https://kotlinlang.org/)**
4. **Kotlin中文官网：[https://www.kotlincn.net/](https://www.kotlincn.net/)**
5. **Kotlin源码：[https://github.com/JetBrains/kotlin](https://github.com/JetBrains/kotlin)**
6. Kotlin 学习系列文章：
    * {% post_link kotlin学习系列一 kotlin学习系列一：内置类型 %}
    * {% post_link kotlin学习系列二 kotlin学习系列二：类与接口初解 %}
    * {% post_link kotlin学习系列三 kotlin学习系列三：表达式 %}
    * {% post_link kotlin学习系列四 kotlin学习系列四：函数进阶 %}
    * {% post_link kotlin学习系列五 kotlin学习系列五：类型进阶 %}
    * {% post_link kotlin学习系列六 kotlin学习系列六：泛型 %}
    * {% post_link kotlin学习系列七 kotlin学习系列七：反射 %}
    * {% post_link kotlin学习系列八 kotlin学习系列八：注解 %}
    * {% post_link kotlin学习系列十 kotlin学习系列十：协程进阶 %}
    * {% post_link kotlin学习系列十一 kotlin学习系列十一：协程应用 %}

# 二、协程基本概念

协程源码库：[https://github.com/hltj/kotlinx.coroutines-cn](https://github.com/hltj/kotlinx.coroutines-cn)

## 1. 什么是协程`Coroutine`

**协程** 通过将复杂性放入库来简化异步编程。程序的逻辑可以在协程中顺序地表达，而底层库会为我们解决其异步性。该库可以将用户代码的相关部分包装为回调、订阅相关事件、在不同线程 *（甚至不同机器）* 上调度执行，而代码则保持如同顺序执行一样简单

**协程** 就像 **非常轻量级的线程**。线程是由系统调度的，线程切换或线程阻塞的开销都比较大。而协程依赖于线程，但是协程挂起时不需要阻塞线程，几乎是无代价的，协程是由开发者控制的。所以协程也像用户态的线程，非常轻量级，**一个线程中可以创建任意个协程**

**协程** 很重要的一点就是当它挂起的时候，**不会阻塞其他线程**。协程底层库也是异步处理阻塞任务，但是这些复杂的操作被底层库封装起来，**协程代码的程序流是顺序的**，不再需要一堆的回调函数，就像同步代码一样，也便于理解、调试和开发。它是可控的，线程的执行和结束是由操作系统调度的，而协程可以手动控制它的执行和结束

* 协程是可以由程序自行控制 **挂起、恢复** 的程序
* 协程可以用来实现多任务的协作执行
* 协程可以用来解决异步任务控制流的灵活转移
* 协程可以让异步代码同步化
* 协程可以降低异步程序的设计复杂度

## 2. `kotlin`协程的示例

<span id="jumpNetSample"></span>

### 1. 常规网络接口请求处理流程

```kotlin

//网络接口请求返回数据
data class User(val id: String, val name: String, val url: String)

//网络请求API
interface GitHubApi {
    @GET("users/{login}")
    fun getUserCallback(@Path("login") login: String): Call<User>
}

//构建retrofit网络请求实例
val githubApi by lazy {
    val retrofit = retrofit2.Retrofit.Builder()
            .client(OkHttpClient.Builder().addInterceptor(Interceptor {
                it.proceed(it.request()).apply {
                    println("request: $code")
                }
            }).build())
            .baseUrl("https://api.github.com")
            .addConverterFactory(GsonConverterFactory.create())
            .build()

    retrofit.create(GitHubApi::class.java)
}


//请求结果处理函数
fun showUser(user:User) {
    println(user)
}

//请求结果处理函数
fun showError(t: Throwable) {
    t.printStackTrace(System.out)
}

fun main() {

    val call = githubApi.getUserCallback("puppet16")
    call.enqueue(object : Callback<User> {
        override fun onFailure(call: Call<User>, t: Throwable) {
            showError(t)
        }
        override fun onResponse(call: Call<User>, response: Response<User>) {
            response.body()?.let(::showUser) ?: showError(NullPointerException())
        }
    })
}
```

**说明：**

1. 创建了一个名为`GitHubApi`的网络请求`API`，其中有一个`getUser()`方法用于获取`GitHub`的用户信息，该方法返回一个带泛型`User`的`Call`。若要拿到`User`数据，则有两种方法：一是使用`enqueue`方法，也就是传进一个`Callback`回调；二是阻塞的调用，但是在主线程是明显不合适。
2. 通过`retrofit`创建网络请求`API`的实例`githubApi`，其中添加拦截器用于打印网络请求返回码，设置了网络请求地址域名为`https://api.github.com`，再设置了返回的结果使用`Gson`解析。
3. 再创建了两个请求结果处理函数`showUser()`、`showError()`，用于处理接口请求成功及请求失败的逻辑
4. 最后在`main()`函数中，通过`githubApi`获取网络请求返回的`Call`，再调用`Call`的`equeue`方法，传入一个`Callback`，再在`Callback`的两个方法中分别调用请求结果处理函数`showUser()`、`showError()`
5. `call.equeue()`方法是个异步方法，该异步方法回调到主流程后会调用`showUser()`或是调用`showError()`，如此异步方法将主流程分裂成了两个分支

**问题：** 如果传入多个`GitHub`的用户名称，使用`forEach`方法每个名称调用一次网络请求，可以获取多个对应`User`的结果，但是传入的是名称的数组，无法将多个`User`放到一个数组中

### 2. 使用协程改造异步程序

#### 1. 网络请求`API中添加挂起函数

在网络请求`API`，添加如下方法：

```kotlin

interface GitHubApi {
    @GET("users/{login}")
    suspend fun getUserSuspend(@Path("login") login: String): User
}
```

**说明：**

1. 添加了一个`getUserSuspend()`方法，该方法使用了关键字 **`suspend`** 修饰，表示该方法为 **挂起函数**
2. 该函数返回结果为`User`对象

#### 2. 替换`call.equeue`方法

```kotlin
try {
    val user = githubApi.getUserSuspend(name)
    showUser(user)
} catch (e: Exception) {
    showError(e)
}
```

**说明：**

1. 使用如上方法替换之前的网络请求
2. `githubApi.getUserSuspend()`是一个挂起函数，调用到该条语句时该函数会挂起，直到返回结果再回到该线程中。此时已经获得了网络请求的结果，若请求正常返回了`user`，则直接显示信息，否则显示错误信息
3. 如此网络请求的后的异步操作变成了同步操作

#### 3. 通过多个用户名获取多个用户的数组

```kotlin
val names = arrayOf("abreslav","udalov", "yole")
val users = names.map { name ->
    githubApi.getUserSuspend(name)
}
```

**说明：**

1. 通过`map`方法，每个用户名都进行一次网络操作，之后将网络操作的结果组合成一个列表返回

# 三、 协程的常见实现

## 1. 协程分类

* 按调用栈
  * 有栈协程：每个协程会分配单独的调用栈，类似线程的调用栈，可以在任意函数嵌套中挂起
  * 无栈协程：不会分配单独的调用栈，挂起点状态通过闭包或对象保存，只能在当前函数中挂起，无法嵌套函数实现挂起
* 按调用关系
  * 对称协程：调度权可以转移给任意协程，协程之间是对等关系
  * 非对称协程：调度权只能转移给调用自己的协程，协程存在父子关系

**`Kotlin`中的是无栈的非对称协程**

## 2. 其它语言的常见协程实现

**协程的关键：** 挂起、恢复

<span id="jumpGenerator"></span>

### 1. `Python`的`Generator`

```py
def numbers():
    i = 0
    while True:
        yield i
        i += 1
        time.sleep(1)
```

```py
gen = numbers()
print(f"[0]{next(gen)}")
print(f"[1]{next(gen)}")
```

打印结果：

```py
[0]0
[1]1
```

**说明：**

1. `Python`因其语法简单，不需要额外配置环境等而号称是最简单的语言
2. `Python`通过缩进定义作用域
3. 定义了一个函数`numbers()`，函数中创建一个死循环，`yield`函数有挂起的含义，表示不再执行下面的代码，该条语句后面是将`i`自加1
4. 调用函数`numbers()`返回的是一个`generator`，`generator`有`next()`方法，在`python`中`next(gen)`语句表示调用`generator`的`next()`方法，调用这条语句时才会真正执行`numbers()`里的代码
5. 执行到第一条打印语句时，才会真正开始执行`numbers()`里的代码：先给`i`赋值为0，再进入死循环，之后执行到`yield i`，将这个函数挂起，再将`i`的值抛给了调用的位置，即`next(gen)`得到的是`i`的值，此时`i`的值为0
6. 之后开始执行第二条打印语句，如此又开始执行`numbers()`函数里的`yield i`语句后的代码，将`i`值加一，因在列循环中，所以再次执行到了`yield i`，如此第二条打印语句的`next(gen)`得到了`i`的值，此时`i`的值为1
7. `Python`的`Generator`是无栈的非对称协程

<span id="jumpLua"></span>

### 2. `Lua`的`Coroutine`

* 创建协程：`coroutine.create(<function>)`，传入一个函数
* 查询协程：`coroutine.status(<Coroutine-Object>)`，传入一个协程对象
* 挂起协程：`coroutine.yield(<Values-to-Yield>)`，可以传入一些值
* 恢复协程：`coroutine.resume(<Coroutine-Object>)`，传入协程对象

```lua
function producer()
    for i = 0,3 do
        print("send"..i)
        coroutine.yield(i)
    end
    print("End Producer")
end

function consumer(value)
    repeat
        print("receive"..value)
        value = coroutine.yield()
    until(not value)
    print("EndConsumer")
end

producerCoroutine = coroutine.create(producer)
consumerCoroutine = coroutine.create(consumer)
repeat
    status,product = coroutine.resume(producerCoroutine)
    coroutine.resume(consumerCoroutine,product)
until(not status)
print("End Main")
```

**打印结果：**

```lua
send0
receive0
send1
receive1
send2
receive2
send3
receive3
End Producer
EndConsumer
End Main
```

**说明：**

1. 定义了函数`producer()`，其中创建一个循环四次的`for`循环`yeild`值`i`
2. 定义了函数`consumer()`，该函数接收值`value`，创建一个条件循环当没有`value`值时停止循环，循环体内打印该值，然后将协程的`yield()`携带的值赋值给`value`
3. 主调用中操作：
   1. 先根据那两个函数创建出两个协程：`producerCoroutine`、`consumerCoroutine`
   2. 创建一个条件循环，直到`status`为`false`时停止
   3. 循环体内先执行协程`producerCoroutine`中的函数`producer`：执行函数`producer`中的`for`循环，每次循环都打印信息，再通过`coroutine.yield(i)`方法将当前协程挂起并将`i`值丢出去
   4. 此时因为协程`producerCoroutine`还没执行完，所以`status`为`true`，而`product`值是`yield`出来的`i`值
   5. 之后执行协程`consumerCoroutine`中的函数`consumer`，并将`product`值作为入参传入函数中：进入函数内的循环，打印接收的值，之后将当前协程挂起
   6. 此时`status`值还是`true`，再次开始主调用循环：即再次执行协程`producerCoroutine`中的函数`producer`。走到函数`producer`中的循环执行完毕，`status`的值变为`false`，主调用结束
4. 调用`resume`时，会将`resume`的其它参数作为入参传入到协程的函数中
5. `Lua`中`..`有拼接字符串的意思
6. `Lua`的协程是 **有栈的非对称协程**

<span id="jumpGoRoutine"></span>

### 3. `Go`的`routine`

>计算机科学领域的任何问题都可以通过增加一个间接的中间层来解决
非对称协程的调度权只能转移给调用自己的协程，而对称协程的调度权可以转移给任意协程。
对称协程是基于非对称协程的实现：在两个非对称协程中添加一个中间层调度中心就可以实现对称协程

1. `Go`语言的`Coroutine`是对称协程，它实现对称协程的中间层`channel`。
2. `Channel`是`Go`中的一个核心类型，可以把它看成一个管道，通过它并发核心单元就可以发送或者接收数据进行通讯。
3. `Channel`的操作符是箭头 **<-**。该操作符代表channel的方向。如果没有指定方向，那么`Channel`就是双向的，既可以接收数据，也可以发送数据
4. 使用 `go` 语句开启一个新的运行期线程，即 `goroutine`，以一个不同的、新创建的 `goroutine` 来执行一个函数。 同一个程序中的所有 `goroutine` 共享同一个地址空间

    ```go
    ch <- v    // 发送值v到Channel ch中
    v := <-ch  // 从Channel ch中接收数据，并将数据赋值给v
    ```

```go
package main

import (
 "fmt"
 "time"
)

func main() {
    channel := make(chan int)
    var readChannel <-chan int = channel
    var writeChannel chan <- int = channel

    go func() {
        for i:= 0; i < 3; i++ {
            fmt.Println("write",i)
            writeChannel <- i
        }
        close(writeChannel)
    }()

    go func() {
        fmt.Println("wait for read")
        for i := range readChannel {
            fmt.Println("read", i)
        }
        fmt.Println("read end")
    }()

    now := time.Now()
    time.Sleep(5 * time.Second)
    latency := time.Since(now).Seconds()
    fmt.Println(latency)
}
```

**打印结果：**

```go
wait for read
write 0
write 1
read 0
read 1
write 2
read 2
read end
5.000133976
```

**说明：**

1. `:=`是赋值符号。`make(chan int)`语句创建了一个`channel`，该`channel`类型是`int`，即`channel`可以传递的元素是`int`，没有指定方向，这个`channel`是双向的
2. 声明一个变量`readChannel`，类型是一个单向的`channel`，`channel`只可以发送数据，即只能取到`channel`里的数据
3. 声明一个变量`writeChannel`，类型是一个单向的`channel`，`channel`只可以接收数据，即只能往`channel`里放数据
4. 定义了两个函数，并直接使用`go`语句开启两个新的运行期线程来分别执行这两个函数
5. 第一个函数功能：定义一个`for`循环，循环体先打印信息，再将`i`的值放到`writeChannel`中，最后关闭
6. 第二个函数功能：先打印信息。再定义了一个`for`循环，循环内打印从`writeChannel`取出的数据
7. 因为主程序退出的太快了，所以最后添加了一个5秒的延时，否则两个协程的信息打印不出来。
8. 结论：
   1. 每个`go routine` 都是并发或并行执行
   2. 无`Buffer`的`Channel`写时会挂起，直到读取，反之亦然
   3. **`go routine` 可以认为是一种有栈对称协程的实现**

<span id="jumpJsAsync"></span>

### 4. `async/await`关键字

> 支持这两个关键字的语言有：JavaScript Es 2016(ES7)、C# 5.0、Python 3.5、Rust 1.39.0等等

#### 1. JavaScript的使用

```js
function getUser(name) {
    return axios.get(`https://api.github.com/users/${name}`)
}

//正常调用该网络请求
function main() {
    const userPromise = getUser("Lee")
    userPromise.then(user => {
        console.log(user);
    }).catch(e => {
        console.error(e)
    })
}

//使用这两个关键字调用网络请求
async function main() {
    const user = await getUser("Lee")
    console.log(user)
}
```

**说明：**

1. 定义一个名为`getUser()`的函数，入参为`name`，用于获取`GitHub`上某用户的公开信息
2. `Axios` 是一个基于 `promise` 的 `HTTP` 库，返回类型为`Promise<AxiosResponse<T>>`，类似于`Retrofit`。官网：[http://www.axios-js.com/](http://www.axios-js.com/)
3. 若要正常调用网络请求的函数`getUser()`，需要使用`.then()`和`.catch()`添加回调，这是一个异步的调用
4. 而使用关键字`async`和`await`调用网络请求函数`getUser()`，这是个同步的操作，不需要添加回调
5. 关于`async/await`的总结：
   1. 可以多层嵌套，但必须为`async function`
   2. **`async/await`是一种无线非对称的协程实现**
   3. `async/await`是目前各类语言支持最广泛的特性

#### 2. `async/await`和`suspend`的区别

```js
//js中使用
async function main() {
    const user = await getUser("Lee")
    console.log(user)
}
```

```kotlin
//kotlin中使用
suspend fun coroutine(){
    try {
        val user = githubApi.getUserSuspend(name)
        showUser(user)
    } catch (e: Exception) {
        showError(e)
    }
}
```

**说明：**

1. `kotlin`中使用修饰于函数上的关键字`suspend`实现调用网络请求时的同步操作，而`sync/await`需要两个关键字

# 四、`Kotlin`协程的基本要素

## 1. 协程的挂起和恢复

### 1. 相关概念说明

**挂起函数：** 即使用`suspend`关键字修饰的函数。

1. `suspend` 是有暂停的意思，但在协程中应该理解为：当线程执行到协程的 `suspend` 函数的时候，暂时不继续执行协程代码了。
2. 挂起函数只能在 **其他挂起函数** 或 **协程** 中调用
3. 协程在执行到某一个`suspend`函数的时候，这个**协程会被挂起 *(即挂起的对象是协程)***，也就是说 **这个协程从正在执行它的线程上脱离**，此时当前线程不再关心协程进行什么操作
4. 挂起函数调用时包含了协程 **挂起**的语义
5. 挂起函数返回时包含了协程 **恢复**的语义
6. 挂起函数的类型是在一般函数类型前加一个`suspend`
   |函数|类型|
   |--|--|
   |suspend fun foo(){}| suspend () -> Unit|
   |suspend fun bar(a:Int):String { </br> &nbsp;&nbsp;&nbsp;&nbsp; return "Hello"</br>}|suspend (Int) -> String|

**挂起点：** 挂起函数的调用处称为挂起点，在`Android Studio`中，挂起点位置有如下图标显示：![avar](/kotlin学习系列九/starting_point_pic.png)

**协程：** 可以使用 `launch` 或者 `async` 函数启动一个协程，协程其实就是这两个函数中 **闭包的代码块**

### 2. 在挂起时线程和协程各自操作

**在挂起点时线程操作：**

1. 线程执行到了协程的代码块中的 `suspend` 函数时，就暂时不再执行剩余的协程代码，跳出协程的代码块。
2. 此时若线程是一个后台线程，则跟 `Java` 线程池里的线程在工作结束之后是完全一样的：回收或者再利用。
3. 若线程是一个`Android`主线程，那它接下来就会继续执行本职工作：也就是一秒钟60次的界面刷新任务

**在挂起点时协程操作：**

1. 线程的代码在到达 `suspend` 函数的时候被掐断，接下来协程会从这个 `suspend` 函数开始继续往下执行，不过是在 `suspend` 函数指定的线程中执行。
2. `suspend` 函数通过`Dispatchers` 调度器来指定线程。`Dispatchers`调度器，它可以将协程限制在一个特定的线程执行，或者将它分派到一个线程池，或者让它不受限制地运行
3. 常用的 `Dispatchers`有以下三种
   * `Dispatchers.Main`：`Android` 中的主线程
   * `Dispatchers.IO`：针对磁盘和网络 `IO` 进行了优化，适合 `IO` 密集型的任务，比如：读写文件，操作数据库以及网络请求
   * `Dispatchers.Default`：适合 `CPU` 密集型的任务，比如计算
4. 在 `suspend` 函数执行完成之后，**协程会自动帮我们把线程再切回来**

**挂起时线程和协程操作流程总结：**

1. 我们的协程原本是运行在某线程的。当代码遇到 `suspend` 函数的时候，会被 `suspend` 也就是被挂起，而所谓的被挂起，就是线程切换，根据 `Dispatchers` 切换到其它线程执行`suspend`函数。当这个`suspend`函数执行完毕后，协程会重新切换回一开始的线程
2. 切回原线程的操作在 `Kotlin` 里叫做 **恢复 *(resume)***， 也就是协程会帮我们再 `post` 一个 `Runnable`，让协程中剩下的代码继续回到一开始的线程中执行
3. 简单来讲，在 `Kotlin` 中所谓的 **挂起，就是一个稍后会被自动切回来的线程调度操作**
4. 挂起之后需要恢复，而 **恢复这个功能是协程的**，如果你不在协程里面调用，恢复这个功能没法实现，所以挂起函数必须在协程或者另一个挂起函数里被调用
5. 因为一个挂起函数要么在协程里被调用，要么在另一个挂起函数里被调用，所以它其实直接或者间接地，总是会在一个协程里被调用
6. 所以要求 `suspend` 函数只能在协程里或者另一个 `suspend` 函数里被调用，是为了要让协程能够在 `suspend` 函数切换线程之后再切回来，若在一般函数中调用了`suspend`函数，`IDE`会报错：`Suspend function 'XXX' should be called only from a coroutine or another suspend function`
7. 挂想和恢复可以控制执行流程的转移
8. 异步逻辑可以用同步代码的形式写出
9. 同步代码比异步代码更灵活，更容易实现复杂业务

### 3. 挂起实现原理

```kotlin
suspend fun suspendingPrint() = withContext(Dispatchers.IO) {
    println("Thread: ${Thread.currentThread().name}")
}
```

**说明：**

1. 如上声明了一个挂起函数`suspendingPrint()`，函数功能是打印当前线程
2. `withContext()`函数是一个挂起函数，接收一个 `Dispatcher` 参数，依赖这个 `Dispatcher` 参数的指示，协程被挂起，然后切到别的线程
3. 真正挂起协程是 `Kotlin` 的协程框架帮我们实现的。所以我们想要自己写一个挂起函数，除了加上 `suspend` 关键字还要函数内部直接或间接地调用到 `Kotlin` 协程框架自带的 `suspend` 函数才行

### 4. `suspend`挂起总结

1. `suspend` 关键字并不是用来操作挂起的，挂起的操作依赖的是挂起函数里面的实际代码。该关键字其实是函数的创建者对函数的使用者的提醒：这是一个耗时函数，它被创建者用挂起的方式放在后台运行，所以请在协程里调用该函数
2. 若创建了 `suspend` 函数但它内部不包含真正的挂起逻辑，`IDE`会给你提示一个警告：`redundant suspend modifier` *(冗余的“redundant”修饰符)*
3. `suspend` 关键字限制函数只能在协程里被调用，如果在非协程的代码中调用，就会编译不通过
4. 创建一个 `suspend` 函数，为了让它包含真正挂起的逻辑，要在它内部直接或间接调用 `Kotlin` 自带的 `suspend` 函数，自定义的 `suspend` 函数才是有意义的
5. `suspend` 函数用于处理比较耗时也就是要等的操作。耗时操作一般分为两类：`I/O` 操作和 `CPU` 计算工作。比如文件的读写、网络交互、图片的模糊处理，都是耗时的，通通可以把它们写进 `suspend` 函数里
6. `suspend` 函数的另一个应用场景是操作本身做起来并不慢，但它需要等待，比如 5 秒钟之后再做这个操作
7. `withContext()`函数是在挂起函数里功能最简单直接：把线程自动切走和切回
8. `delay()`函数等待一段时间后再继续往下执行代码
9. **挂起，就是一个稍后会被自动切回来的线程调度操作**
10. 参考文章：[kotlin协程的挂起suspend](https://blog.csdn.net/cpcpcp123/article/details/111724079)

## 2. `Continuation`

因为 **`Kotlin`中的协程是无栈的**，没有栈去保存挂起点的状态，而是通过`Continuation`接口实现了挂起点状态的保存，实现这个接口的类，具有可以暂停和继续的能力。

### 1. `Continuation`部分源码

```kotlin
package kotlin.coroutines

@SinceKotlin("1.3")
public interface Continuation<in T> {

    public val context: CoroutineContext

    public fun resumeWith(result: Result<T>)
}

@SinceKotlin("1.3")
@InlineOnly
public inline fun <T> Continuation<T>.resume(value: T): Unit =
    resumeWith(Result.success(value))

@SinceKotlin("1.3")
@InlineOnly
public inline fun <T> Continuation<T>.resumeWithException(exception: Throwable): Unit =
    resumeWith(Result.failure(exception))

```

**说明：**

1. `Continuation` 定义了一个回调方法：`resumeWith()`：恢复执行相应的协程，将成功或失败的结果传递给挂起点返回值
2. `Continuation`接口有两个扩展方法：
   1. `resume()`：恢复执行相应的协程，传递对应值作为挂起点的返回值。
   2. `resumeWithException()`：恢复执行相应的协程，以便在挂起点之后立即重新抛出错误
   3. `resume()`与`resumeWithException()`都调用的`resumeWith()`方法
3. 该源码是`1.3`版本之后的

### 2. 挂起函数与`Continuation`关系

每个挂起函数编译后会增加一个 `Continuation` 类型的参数。每个挂起函数都有一个回调自己的 `Continuation` 实现类，并且这个类会被传递给这个挂起函数所调用的其它挂起函数，这些子方法可以通过 `Continuation` 回调父方法以恢复暂停的程序

```kotlin
suspend fun foo() {}
suspend fun bar(a:Int):String {
    return "Hello"
}
```

上述代码中的两个函数本质上其内容如下：

```kotlin
fun foo(continuation:Continuation<Unit>):Any{}
fun bar(a:Int, continuation:Continuation<String>):Any{
    return "Hello"
}
```

**说明：**

1. 一开始两个函数的类型分别为`suspend () -> Unit`、`suspend (Int) -> String`
2. 如上挂起函数在编译后会增加一个 `Continuation`类型的入参，`Continuation`中的泛型是挂起函数的返回值类型，而挂起函数的返回值类型变为了`Any`，`Any`有两层含义：
   1. 若该挂起函数没有挂起，跟普通函数没有区别，则`Any`是承载真正的返回值结果的
   2. 若该挂起函数挂起，需要通过回调返回结果，则`Any`返回的是一个挂起标志：`COROUTINE_SUSPENDED`
3. 如此调用挂起函数时一定要传入一个`Continuation`参数，而传递`Continuation`参数是编译器完成的，所以当前调用挂起函数的位置一定要有一个`Continuation`参数实例才可被作为参数传入，因此 **挂起函数只能在挂起函数或协程中调用**

### 3. 将回调转为挂起函数

通过将 [常规网络接口请求处理流程章节](#jumpNetSample) 中的异步网络请求改为同步来直观展示`Continuation`的使用

```kotlin

suspend fun getUserSuspend(name: String) = suspendCoroutine<User> { continuation ->
    githubApi.getUserCallback(name).enqueue(object: Callback<User> {
        override fun onFailure(call: Call<User>, t: Throwable) =
                continuation.resumeWithException(t)
        override fun onResponse(call: Call<User>, response: Response<User>) =
                response.takeIf { it.isSuccessful }?.body()?.let(continuation::resume)
                        ?: continuation.resumeWithException(HttpException(response))
    })
}
```

**说明：**

1. `githubApi.getUserCallback()`方法返回的是一个`Call<User>`，它可以通过`equeue`方法来实现网络的请求。
2. `getUserSuspend()`函数有`suspend`修饰符，则该函数是有一个`Continuation`对象的，正常使用是无法使用该`Continuation`对象，可以借助`suspendCoroutine`
3. `suspendCoroutine`函数可以获取当前挂起函数中的`Continuation`实例，并且在该函数中可以调用`Continuation`的`resume()`和`resumeWithException()`方法
4. 在回调成功的分支中使用`Continuation.resume(value)`，在回调失败的分支中使用`Continuation.resumeWithException(e)`
5. **真正的挂起** 必须异步调用`resume`，包括：
   * 切换到其他线程`resume`
   * 单线程事件循环异步执行
6. **没有真正挂起** 的情况：一开始直接`return`；在`suspendCoroutine`中直接调用`resume`。以上两种情况都没有切换线程
7. 而`Call<User>`的`equeue`方法会将线程切换到`IO`线程去请求网络，所以`getUserSuspend()`函数实现了真正的挂起

## 3. 协程的创建

### 1. `createCoroutine()`方法

协程是一段可执行的程序，它的创建通常需要一个函数即挂起函数，且需要使用`createCoroutine()`来进行创建

```kotlin

fun <T> (suspend () -> T).createCoroutine(completion: Continuation<T>): Continuation<Unit> {

}

fun <R, T> (suspend R.() -> T).createCoroutine(receiver: R, completion: Continuation<T>): Continuation<Unit> {

}
```

**说明：**

1. `createCoroutine()`方法是挂起函数的扩展方法，也就是说只有挂起函数才能创建协程
2. 提供了两个`createCoroutine()`方法，一个带`receiver`，一个不带`receiver`，这取决于调用该方法的挂起函数是否有`receiver`
3. `createCoroutine()`方法中传入了一个`Continuation`类型的参数`completion`，这个`Continuation`是作为协程执行完毕后返回的`Continuation`
4. `createCoroutine()`方法返回结果是一个`Continuation`，这个`Continuation`是协程创建出来的本体，该`Continuation`中的所有`resume()`方法执行完毕之后，就会调用入参传入的那个`Continuation`类型参数`completion`
5. 一个协程需要两个`Continuation`：
   * 一个是协程的本体即挂起函数本身执行需要一个`Continuation`实例在恢复时调用，即`createCoroutine()`方法中传入的参数`completion`
   * `createCoroutine()`方法的返回值`Continuation<Unit>`，是创建出来的协程的载体，该方法的`receiver`即挂载函数会被传给该实例作为协程的实际执行体

## 4. 协程启动

### 1. `resume`

```kotlin
suspend {

}.createCoroutine(object: Continuation<Unit> {
    override val context = EmptyCoroutineContext

    override fun resumeWith(result: Result<Unit>) {
        Log.d(TAG,"Coroutine End with $result")
    }
}).resume(Unit)
```

**说明：**

1. `suspend {}`创建了一个匿名的挂起函数，该匿名挂起函数又调用了`createCoroutine()`方法创建一个该匿名挂起函数包装的`Continuation`
2. 而`createCoroutine()`方法中传入的`Continuation`是在协程执行完成后调用
3. 调用`resume(Unit)`方法启动协程

### 2. `startCoroutine`

```kotlin
fun <T> (suspend () -> T).startCoroutine(completion: Continuation<T>): Unit {

}

fun <R, T> (suspend R.() -> T).startCoroutine(receiver: R, completion: Continuation<T>): Unit {

}
```

**说明：**

1. `startCoroutine()`方法作用是创建协程，并在创建后启动协程，相当于`createCoroutine()`加上`resume(Unit)`
2. 如此上一节代码可写为如下样式：

   ```kotlin
    suspend {

    }.startCoroutine(object: Continuation<Unit>{
        override val context = EmptyCoroutineContext

        override fun resumeWith(result: Result<Unit>) {
            Log.d(TAG,"Coroutine End with $result")
        }
    })
   ```

3. 协程在启动和执行结束的时候都会调用一次`resume`方法，协程中如果有N个挂起点，即调用了N个挂起函数且 **挂起函数真正实现了挂起 *(即切换了线程)***，则总共调用了 **N + 2** 次`resume`方法

## 5. 协程上下文

* **协程上下文** 存放了协程执行过程中需要携带的数据
* **协程上下文** 是一组用于定义协程行为的元素。由如下几项构成：
  * Job: 控制协程的生命周期
  * CoroutineDispatcher: 调度器如Dispatchers.IO
  * CoroutineName: 协程的名称，调试的时候很有用
  * CoroutineExceptionHandler: 用于处理未被捕捉的异常

* **协程上下文** 简单来说是一个数据集合，索引是`CoroutineContext.Key`，元素是`CoroutineContext.Element`，实现接口`CoroutineContext.Element`的类，即可存放于协程上下文中，实现该接口的类如下图：
  ![avar](/kotlin学习系列九/continuation_context_impl_class.png)

### 1. `ContinuationInterceptor`

拦截器`ContinuationInterceptor`是协程上下文的一类元素，它可以对协程上下文所在协程的`Continuation`进行拦截、篡改

```kotlin
@SinceKotlin("1.3")
public interface ContinuationInterceptor : CoroutineContext.Element {

    companion object Key : CoroutineContext.Key<ContinuationInterceptor>

    public fun <T> interceptContinuation(continuation: Continuation<T>): Continuation<T>
}
```

**说明：**

1. `ContinuationInterceptor`接口中的`interceptContinuation()`方法，传入了一个`Continuation`，返回了一个`Continuation`
2. 该拦截器与`OkHttp`里面`addInterceptor`的时候在这里面更改`Interceptor`然后在返回是一样的原理

## 6. `Continuation`执行示意

### 1. `SuspendLambda`

```kotlin
suspend {
...
}.startCoroutine(...)
```

**说明：**

1. 通过`startCoroutine()`创建出一个协程，而协程真正执行的逻辑就是该方法的`receiver`，即匿名挂起函数`suspend {...}`，这个匿名挂起函数创建成功后会生成一个类`SuspendLambda`，这是一个`Continuation`的实现类，如下图：
   ![avar](/kotlin学习系列九/continuation_run_1.png)

### 2. `SafeContinuation`

```kotlin
suspend {
    a()
}.startCoroutine(...)

suspend fun a() = suspendCoroutine<Unit> {
    thread {
        it.resume(Unit)
    }
}
```

**说明：**

1. 若该匿名挂起函数内有挂起函数的调用，则会用`SafeContinuation`将`SuspendLambda`进行包装
2. 只要使用了`suspendCoroutine`，就会将`SuspendLambda`包装起来，如下图：
   ![avar](/kotlin学习系列九/continuation_run_2.png)
3. `suspendCoroutine()`函数中有一个参数`Continuation`，可以用`it`来表示该参数。而`it`代表的`Continuation`是`SafeContinuation`的一个实例。则`it.resume()`实际上调用的是`SafeContinuation`的`resume`，最终会调到`SuspendLambda`的`resume`，如此恢复到匿名挂起函数的执行
4. `SafeContinuation`的作用就是确保：
   1. `SuspendLambda`只被调用一次
   2. 如果在当前线程调用栈上直接调用则不会挂起

### 3. 加入`Intetcepted`拦截器

```kotlin
suspend {
    a()
}.startCoroutine(...)
```

**说明：**

1. 拦截器其实是将`SuspendLambda`包装了一次，如下图：
    ![avar](/kotlin学习系列九/continuation_run_3.png)
2. `SafeContinuation`真正调用`resume`时，会调用了拦截器`Intetcepted`中返回的`Continuation`的`resume`，在拦截器返回的`Continuation`的`resume`方法中可以做切换线程操作，切换线程后，再调用`SuspendLambda`，如此实现了线程切换
3. 拦截器最重要的一个功能就是线程切换
4. 协程体是一个用`suspend`关键字修饰的一个无参、无返回值的函数类型。
5. 调用顺序注意点：
   1. `SafeContinuation`仅在挂起点时出现
   2. 拦截器在每次 *(恢复)* 执行协程体时调用
   3. **`SuspendLambda`是协程函数体**

### 4. 协程挂起恢复执行示意

```java
//java
public class ContinuationImpl implements Continuation<Object> {

    private int mLabel = 0;

    private final Continuation<Unit> mCompletion;

    public ContinuationImpl(Continuation<Unit> completion) {
        mCompletion = completion;
    }

    @NotNull
    @Override
    public CoroutineContext getContext() {
        return EmptyCoroutineContext.INSTANCE;
    }

    @Override
    public void resumeWith(@NotNull Object o) {
        try {
            Object result = o;
            switch (mLabel) {
                case 0: {
                    Logger.debug(1);
                    result = ConsoleMainKt.returnSuspend(this);
                    mLabel++;
                    if (isSuspended(result)) return;
                }
                case 1: {
                    Logger.debug(result);
                    Logger.debug(2);
                    result = DelayKt.delay(1000, this);
                    mLabel++;
                    if (isSuspended(result)) return;
                }
                case 2: {
                    Logger.debug(3);
                    result = ConsoleMainKt.returnImmediately(this);
                    mLabel++;
                    if (isSuspended(result)) return;
                }
                case 3: {
                    Logger.debug(result);
                    Logger.debug(4);
                }
            }
            mCompletion.resumeWith(Unit.INSTANCE);
        } catch (Exception e) {
            mCompletion.resumeWith(e);
        }
    }

    private boolean isSuspended(Object o) {
        return o == IntrinsicsKt.getCOROUTINE_SUSPENDED();
    }
}
```

```kotlin
//kotlin
Logger.debug(1)
Logger.debug(returnSuspend())
Logger.debug(2)
delay(1000)
Logger.debug(3)
Logger.debug(returnImmediately())
Logger.debug(4)
```

**介绍：**

1. 如上`Java`代码是仿照`Kotlin`中协程的调用逻辑写出的一个类`ContinuationImpl`，其本质上就是`SuspendLambda`：
   1. `ContinuationImpl#resumeWith()`方法执行的就是`SuspendLambda`中`Lambda`表达式真正传进去的东西
   2. `ContinuationImpl#isSuspended()`方法用于判断某个对象是否为挂起标记，
2. 如上`Kotlin`代码是协程启动后挂起函数执行的逻辑即`SuspendLambda`中`Lambda`表达式内容，启动的协程就是上述的`Java`代码类`ContinuationImpl`，每遇到一个挂起点都会有一个`SafeContinuation`，而`SafeContinuation` `resume`之后，会继续执行挂起函数里后续逻辑
3. `Kotlin`代码中有三个挂起点：
   * `Logger.debug(returnSuspend())`
   * `delay(1000)`
   * `Logger.debug(returnImmediately())`
4. 假定：
   1. 挂起函数`returnSuspend()`有切换线程的操作
   2. 挂起函数`returnImmediately()`直接在该挂起函数中`return`、或没有切换线程即没有调用`SafeContinuation`的`resume`

**执行逻辑说明：**

1. 执行`Kotlin`中第一行代码：`Logger.debug(1)`，纯打印语句没有执行挂起之类的操作时，就是执行对应到`Continu  ationImpl#resumeWith()`方法，第一次调用`resumeWith()`时，`mLabel`为0，所以执行值为0的分支：执行`Logger.debug(1)`
2. 之后执行第二行代码：`Logger.debug(returnSuspend())`，因为上一条语句没有挂起之类的操作，所以还是在分支0中：
   1. 调用挂起函数`ConsoleMainKt.returnSuspend(this)`，该挂起函数有切换线程的操作，所以要实现真正的挂起
   2. `mLabel`自加1
   3. 再通过函数`isSuspended()`判断该挂起函数的返回值`result`是一个 **挂起标记**，直接`return`，这就是非阻塞式的挂起，不卡当前线程。此时
   4. 此时等待该挂起函数恢复
   5. 当挂起函数`returnSuspend()`恢复时会再次调用`ContinuationImpl#resumeWith()`方法，因`mLabel`之前已经自加，所以此时走到了`mLabel`值为1的分支中，而恢复时传递回来的真正结果的打印也是在该分支中进行
3. 之后执行第三行代码：`Logger.debug(2)`，纯打印语句，直接执行
4. 之后执行第四行代码：`delay(1000)`，是`Kotlin`提供的挂起函数，功能是延时一秒，此时继续执行`mLabel`值为1的分支：
   1. 执行`DelayKt.delay(1000, this)`
   2. `mLabel`自加1
   3. 通过函数`isSuspended()`判断该挂起函数的返回值`result`是一个 **挂起标记**，直接`return`
   4. 等待延时结束后恢复
   5. 当延时恢复后，会再次调用`ContinuationImpl#resumeWith()`方法，此时`mLabel`已经自加到了2
5. 之后执行第五行代码：`Logger.debug(3)`，纯打印语句，直接执行
6. 之后执行第六行代码：`Logger.debug(returnImmediately())`，继续执行`mLabel`值为2的分支：
   1. 调用挂起函数`ConsoleMainKt.returnImmediately(this);`，该挂起函数没有切换线程之类的操作，不需要实现真正的挂起
   2. `mLabel`自加1
   3. 通过函数`isSuspended()`判断该挂起函数的返回值`result` **不是** **挂起标记**，**此时不会`return`**，而是继续执行下个分支的语句
   4. 此时执行`mLabel`值为3的分支：打印出`returnImmediately()`函数的返回值
7. 之后执行第七行代码：`Logger.debug(4)`，纯打印语句，直接执行
8. 如此，创建协程时返回的`Contiuation`中`resume`全部执行完毕，此时执行创建协程时传入的`Continuation`即`mCompletion`的`resumeWith()`方法，将`Unit`传入其中
9. 若是以上流程有错误的话，则会执行`catch(Execption e)`分支中的`mCompletion.resumeWith(e);`，即将错误信息返回
10. 如此以`Kotlin`代码中的几条代码为`SuspendLambda`中`Lambda`而创建的协程执行完毕

**协程挂起恢复的要点总结：**

* 协程体内的代码都是通过`Continuation.resumeWith`调用来实现恢复
* 每调用一次`resumeWith`则`mLabel`加1，`mLabel`表示挂起次数，每个分支是挂起`mLabel`次后执行的代码，即每一个挂点对应一个`case`分支
* 挂起函数返回`COROUTINE_SUSPENDED`时才是真正的挂起

<span id="jumpThreadSwitch"></span>

## 7. 协程的线程调度

`suspend`函数会将整个协程挂起，而不仅仅是这个`suspend`函数，也就是说一个协程中有多个挂起函数时，它们是顺序执行的。而协程每次`resume`都会`resume`一开始的`SuspendLambda`，如此只要将这个`resume`拦截就可以进行线程调度

```kotlin
suspend {
...
}.startCoroutine(object: Continuation<Unit>{
    override val context = DispatcherContext(HandlerDispatcher)
    ...
})
```

```kotlin
interface Dispatcher {
    fun dispatch(block: ()->Unit)
}

open class DispatcherContext(private val dispatcher: Dispatcher = DefaultDispatcher) : AbstractCoroutineContextElement(ContinuationInterceptor), ContinuationInterceptor {
    override fun <T> interceptContinuation(continuation: Continuation<T>): Continuation<T>
            = DispatchedContinuation(continuation, dispatcher)
}

private class DispatchedContinuation<T>(val delegate: Continuation<T>, val dispatcher: Dispatcher) : Continuation<T>{
    override val context = delegate.context

    override fun resumeWith(result: Result<T>) {
        dispatcher.dispatch {
            delegate.resumeWith(result)
        }
    }
}

```

**说明：**

1. 整体实现原理是：添加拦截器，拦截了`SuspendLambda`并返回了自己的`Continuation`，在自己的`Continuation`被调用时先切换线程，再去调用`SuspendLambda`的`resume`。如此实现了线程切换。这也是自定义类`DispatcherContext`实现的功能

# 五、示例--使用标准库的序列生成器`Sequence`实现`Generator`

`Generator`是`Python`中的协程特性，`Generator`的使用请看之前的章节：[Pyhton的Generator](#jumpGenerator)

## 1. 代码实现

```kotlin
abstract class GeneratorScope<T> internal constructor(){
    protected abstract val parameter: T

    abstract suspend fun yield(value: T)
}

sealed class State {
    class NotReady(val continuation: Continuation<Unit>): State()
    class Ready<T>(val continuation: Continuation<Unit>, val nextValue: T): State()
    object Done: State()
}

interface Generator<T> {
    operator fun iterator(): Iterator<T>
}

class GeneratorImpl<T>(private val block: suspend GeneratorScope<T>.(T) -> Unit, private val parameter: T): Generator<T> {
    override fun iterator(): Iterator<T> {
        return GeneratorIterator(block, parameter)
    }
}

fun <T> generator(block: suspend GeneratorScope<T>.(T) -> Unit): (T) -> Generator<T> {
    return { parameter: T ->
        GeneratorImpl(block, parameter)
    }
}

class GeneratorIterator<T>(private val block: suspend GeneratorScope<T>.(T) -> Unit, override val parameter: T)
    : GeneratorScope<T>(), Iterator<T>, Continuation<Any?> {
    override val context: CoroutineContext = EmptyCoroutineContext

    private var state: State

    init {
        val coroutineBlock: suspend GeneratorScope<T>.() -> Unit = { block(parameter) }
        val start = coroutineBlock.createCoroutine(this, this)
        state = State.NotReady(start)
    }

    override suspend fun yield(value: T) = suspendCoroutine<Unit> {
        continuation ->
        state = when(state) {
            is State.NotReady -> State.Ready(continuation, value)
            is State.Ready<*> ->  throw IllegalStateException("Cannot yield a value while ready.")
            State.Done -> throw IllegalStateException("Cannot yield a value while done.")
        }
    }

    private fun resume() {
        when(val currentState = state) {
            is State.NotReady -> currentState.continuation.resume(Unit)
        }
    }

    override fun hasNext(): Boolean {
        resume()
        return state != State.Done
    }

    override fun next(): T {
        return when(val currentState = state) {
            is State.NotReady -> {
                resume()
                return next()
            }
            is State.Ready<*> -> {
                state = State.NotReady(currentState.continuation)
                (currentState as State.Ready<T>).nextValue
            }
            State.Done -> throw IndexOutOfBoundsException("No value left.")
        }
    }

    override fun resumeWith(result: Result<Any?>) {
        state = State.Done
        result.getOrThrow()
    }

}

fun main() {
    val nums = generator { start: Int ->
        for (i in 0..5) {
            yield(start + i)
        }
    }

    val seq = nums(10)

    for (j in seq) {
        println(j)
    }
}
```

**打印结果：**

```kotlin
10
11
12
13
14
15
```

**代码说明：**

1. 定义抽象类`GeneratorScope`：
   1. 将该抽象类的构造函数声明为模块内可见
   2. 定义一个泛型`T`类型的参数`parameter`
   3. 定义一个抽象的挂起函数`yield()`，接收一个泛型`T`类型的参数
   4. 该抽象类用于限制挂起函数`yield()`的使用范围：仅限于以该抽象类为`receiver`的`Lambda`表达式
2. 定义密封类`State`，用于标识协程运行情况：
   1. **不带值的状态建议使用枚举，带值的状态建议使用密封类**
   2. `NotReady`状态：还没有准备好值，需要一个协程继续执行协程体
   3. `Ready`状态：已经有值了，此时内部已经`yield`，所以需要协程挂起，并将值传入
   4. `Done`状态：协程体已执行完毕不会有新的值了
3. 定义类`GeneratorIterator`：
   1. 主构造器上添加两个类内属性：一个是名为`block`的挂起函数——该挂起函数的`receiver`是`GeneratorScope`类、入参是泛型、返回结果是`Unit`；另一个是名为`parameter`的泛型参数
   2. 创建一个记录状态的属性`state`
   3. 添加一个初始化代码块：
      1. 启动协程时的`SuspendLambda`是没有参数的，但是该类传入的是一个带参数的挂起函数，所以声明一个名为`coroutineBlock`的无参挂起函数，在挂起函数体中将该类传入的泛型类型的参数作为入参调用传入的那个挂起函数
      2. 通过`coroutineBlock`创建协程，因不需要马上启动，所以使用`createCoroutine()`方法创建，该方法需要传入两个参数：第一个参数是`receiver`，因该类实现了`GeneratorScope`抽象类，所以写`this`；第二个参数是协程执行完毕后的回调，因为该类实现了`Continuation`接口，所以写`this`
      3. 状态初始化为`NotReady`，并传入上一条语句创建的协程
   4. 实现接口`Iterator`用于迭代：
      1. 定义一个`resume()`方法，用于判断如果当前状态是`NotReady`，调用一下状态中协程的`resume()`方法，以此来判断协程是否能`yield`值
      2. `hasNext()`方法：用于判断是否有下一个值。先执行一下自定义的`resume()`方法——若状态是`NotReady`去调用协程的`resume()`方法，之后判断若状态为`Done`，表示协程执行完毕没有值了返回`false`，其它状态返回`true`
      3. `next()`方法返回下一次`yield()`返回的值。将当前状态`state`赋值给另一个属性`currentState`，判断`currentState`：
         1. 若`currentState`是`NotReady`，则调用一下自定义的`resume()`方法，看一下是否还有值，之后再调用一下`next()`方法
         2. 若`currentState`是`Ready`，先将`state`切换为`NotReady`状态，之后将`curentState`强转为`Ready`状态，之后调用`Ready#nextValue()`方法，得到`yield()`的值
         3. 若`currentState`是`Done`，抛出异常：下标越界
   5. 实现接口`Continuation`用于协程：
      1. 因为本次实现的功能不需要`CoroutineContext`，所以为`CoroutineContext`类型的属性`context`赋值为`EmptyCoroutineContext`对象
      2. `resumeWith()`方法会在协程结束时调用，此时将状态置为`Done`，`result.getOrThrow()`语句表示若有异常则抛出异常，否则不做操作
   6. 继承抽象类`GeneratorScope`：实现`yield()`方法，该方法是一个挂起函数，需要通过`suspendCoroutine`拿到它的`Continuation`，之后根据当前状态`state`重新为`state`赋值:
      1. 若当前状态是`NotReady`，调用了`yield()`方法表示值已经准备好了，所以将当前置为`Ready`状态。创建`Ready`实例时将`yield()`之后继续执行的`Continuation`即通过`suspendCoroutine`拿到的`Continuation`传入进去，也将`yield()`方法的入参`value`传入进去
      2. 若当前状态是`Ready`*(判断时`Ready`后加星投影是因为运行时这个类型不会真正存在)*，抛出异常：在`Ready`状态不能`yield`值了
      3. 若当前状态是`Done`状态，抛出异常：不能在`Done`状态`yield`值
4. 定义接口`Generator`，该接口中定义了一个重载运算符的函数`iterator()`、返回一个迭代器
5. 定义实现类`GeneratorImpl`，它实现了接口`Generator`：
   1. 主构造器上添加两个类内属性：一个是名为`block`的挂起函数——该挂起函数的`receiver`是`GeneratorScope`类、入参是泛型、返回结果是`Unit`；另一个是名为`parameter`的泛型参数
   2. 在实现`iterator()`方法中直接返回`GeneratorIterator`类的对象
6. 定义一个生成器函数`generator`：
   1. 入参是一个挂起函数——该挂起函数的`receiver`是`GeneratorScope`类、入参是泛型、返回结果是`Unit`
   2. 返回结果是一个函数——该函数有一个泛型入参、返回结果是接口`Generator`类型对象
   3. 函数体直接`return`了一个`Lambda`表达式，入参是泛型`T`类型的`parameter`，返回是一个接口`Generator`的实现类`GeneratorImpl`的对象

**执行流程说明：**

1. `generator`函数创建一个`Lambda`表达式即创建一个新函数，新函数接收一个参数返回一个`GeneratorImpl`实例
2. `GeneratorImpl`实现了`iterator`运算符即可以迭代了
3. 每次迭代先通过`hasNext()`函数判断是否有值，这个值就是`yield()`函数中的入参，其中若状态是`NotReady`则 **会恢复协程**
4. 恢复协程会执行执行`yield()`函数，若状态是`NotReady`状态会将状态置为`Ready`并 **将协程挂起** 后交给状态对象`state`
5. 之后执行`next()`函数取值，若是`Ready`状态，则将状态置为`NotReady`并将状态对象中的协程传给`NotReady`以便于下次该状态下恢复协程，如此实现了协程的挂起和恢复

## 2. 使用`sequence`实现

```kotlin
fun main() {
    val sequence = sequence {
        yield(1)
        yield(2)
        yield(3)
        yield(4)
        yieldAll(listOf(1,2,3,4))
    }

    for(seq in sequence){
        println(seq)
    }
}
```

**打印结果：**

```kotlin
1
2
3
4
1
2
3
4
```

**说明：**

1. 直接使用`sequence`实现了`Python`中的`Generator`功能
2. 每次`yield`都会挂起，也可以对`sequence`迭代
3. 但是上一节自定义的代码中可以输入一个初始值，而`sequence`不行

<span id="jumpLuaAPI"></span>

# 六、 示例-- 仿 `Lua` 协程实现非对称协程 `API`

非对称协程：非对称协程的调度权只能转移给调用自己的协程，[Lua的Coroutine](#jumpLua)就是非对称协程

```kotlin

sealed class Status {
    class Created(val continuation: Continuation<Unit>): Status()
    class Yielded<P>(val continuation: Continuation<P>): Status()
    class Resumed<R>(val continuation: Continuation<R>): Status()
    object Dead: Status()
}

@RequiresApi(Build.VERSION_CODES.N)
class Coroutine<P, R> (
    override val context: CoroutineContext = EmptyCoroutineContext,
    private val block: suspend Coroutine<P, R>.CoroutineBody.(P) -> R
): Continuation<R> {

    companion object {
        fun <P, R> create(
            context: CoroutineContext = EmptyCoroutineContext,
            block: suspend Coroutine<P, R>.CoroutineBody.(P) -> R
        ): Coroutine<P, R> {
            return Coroutine(context, block)
        }
    }

    inner class CoroutineBody {
        var parameter: P? = null

        suspend fun yield(value: R): P = suspendCoroutine { continuation ->
            val previousStatus = status.getAndUpdate {
                when(it) {
                    is Status.Created -> throw IllegalStateException("Never started!")
                    is Status.Yielded<*> -> throw IllegalStateException("Already yielded!")
                    is Status.Resumed<*> -> Status.Yielded(continuation)
                    Status.Dead -> throw IllegalStateException("Already dead!")
                }
            }

            (previousStatus as? Status.Resumed<R>)?.continuation?.resume(value)
        }
    }

    private val body = CoroutineBody()

    private val status: AtomicReference<Status>

    val isActive: Boolean
        get() = status.get() != Status.Dead

    init {
        val coroutineBlock: suspend CoroutineBody.() -> R = { block(parameter!!) }
        val start = coroutineBlock.createCoroutine(body, this)
        status = AtomicReference(Status.Created(start))
    }


    override fun resumeWith(result: Result<R>) {
        val previousStatus = status.getAndUpdate {
            when(it) {
                is Status.Created -> throw IllegalStateException("Never started!")
                is Status.Yielded<*> -> throw IllegalStateException("Already yielded!")
                is Status.Resumed<*> -> {
                    Status.Dead
                }
                Status.Dead -> throw IllegalStateException("Already dead!")
            }
        }
        (previousStatus as? Status.Resumed<R>)?.continuation?.resumeWith(result)
    }

    suspend fun resume(value: P): R = suspendCoroutine { continuation ->
        val previousStatus = status.getAndUpdate {
            when(it) {
                is Status.Created -> {
                    body.parameter = value
                    Status.Resumed(continuation)
                }
                is Status.Yielded<*> -> {
                    Status.Resumed(continuation)
                }
                is Status.Resumed<*> -> throw IllegalStateException("Already resumed!")
                Status.Dead -> throw IllegalStateException("Already dead!")
            }
        }

        when(previousStatus){
            is Status.Created -> previousStatus.continuation.resume(Unit)
            is Status.Yielded<*> -> (previousStatus as Status.Yielded<P>).continuation.resume(value)
        }
    }
    //为了下一章节对称协程调用API时特殊添加
    suspend fun <SymT> SymCoroutine<SymT>.yield(value: R): P {
        return body.yield(value)
    }
}

class Dispatcher: ContinuationInterceptor {
    override val key = ContinuationInterceptor

    private val executor = Executors.newSingleThreadExecutor()

    override fun <T> interceptContinuation(continuation: Continuation<T>): Continuation<T> {
        return DispatcherContinuation(continuation, executor)
    }
}

class DispatcherContinuation<T>(val continuation: Continuation<T>, val executor: Executor): Continuation<T> by continuation {

    override fun resumeWith(result: Result<T>) {
        executor.execute {
            continuation.resumeWith(result)
        }
    }
}

@RequiresApi(Build.VERSION_CODES.N)
suspend fun main() {
    val producer = Coroutine.create<Unit, Int>(Dispatcher()) {
        for(i in 0..3){
            log("send", i)
            yield(i)
        }
        200
    }

    val consumer = Coroutine.create<Int, Unit>(Dispatcher()) { param: Int ->
        log("start", param)
        for(i in 0..3){
            val value = yield(Unit)
            log("receive", value)
        }
    }

    while (producer.isActive && consumer.isActive){
        val result = producer.resume(Unit)
        consumer.resume(result)
    }
}

fun log(vararg msg: Any?) = println("Coroutine: ${msg.joinToString(" ")}")

```

**打印结果：**

```kotlin
Coroutine: send 0
Coroutine: start 0
Coroutine: send 1
Coroutine: receive 1
Coroutine: send 2
Coroutine: receive 2
Coroutine: send 3
Coroutine: receive 3
Coroutine: receive 200
```

**说明：**

1. 创建密封类`Status`，用于协程的状态：
   1. `Created`：创建状态
   2. `Yielded`：挂起状态，`yield`时的值定义为泛型类型`<P>`
   3. `Resumed`：恢复状态，
   4. `Dead`：销毁状态
2. 创建类`Coroutine`：
   1. 该类有两个泛型参数：`P`表示需要的参数；`R`表示返回值，也就是`yield`时传出的值
   2. 创建伴生对象，在伴生对象中创建可静态调用的方法`create()`,用于创建`Coroutine`类的对象
   3. 创建内部类`CoroutineBody`，用于约束`yield`方法只能在该协程`lambda`中使用：
      1. 创建一个泛型`P`类型的参数`parameter`，初始值为`null`
      2. 创建一个挂起函数`yield`，该函数接收的入参为泛型`R`类型`value`，返回值为泛型`P`类型
   4. 该类主构造器中有两个参数：一个是带默认参数的`CoroutineContext`类型的`context`，若不需要在多线程下运行不传该值直接使用默认值`EmptyCoroutineContext`；一个是挂起函函数`block`，该函数必须以`CoroutineBody`为`receiver`、以泛型`P`为入参、以泛型`R`为返回值
   5. 该类实现`Continuation`的接口，用于监听协程结束，在协程结束后会调用`resumeWith()`方法，在该方法中更新状态：
      1. 创建状态属性`previousStatus`，调用`status`的`getAndUpdate()`方法，该方法使用给定函数的结果以原子方式更新当前值，并返回先前的值，该方法中的函数里的`it`即为当前状态，也就是更新之前的状态
      2. 添加`when`语句进行状态判断
      3. 若为`Created`状态，抛非法异常：从没开始过
      4. 若为`Yielded`状态，抛非法异常：已经`yield`了
      5. 若为`Resumed`状态，返回`Dead`状态
      6. 若为`Dead`状态，抛非法异常：已经销毁
      7. 切换完状态后，将`previousStatus`强转为`Resumed`状态，之后每步都加非空判断，调用该状态的协程的`resumeWith()`方法将`result`传入
   6. 创建名为`body`的`CoroutineBody`对象
   7. 创建名为`status`的`Status`对象，但是使用了`AtomicReference`将对象包裹。**`AtomicReference`是可以原子更新的对象引用**，以保证线程安全
   8. 创建布尔型的属性`isActive`，用于判断协程是否还在活动，覆写该属性的`get()`方法，比较`status`是否是`Dead`状态
   9. 创建初始代码块：
      1. 创建名为`coroutineBlock`的用于创建协程的`SuspendLambda`，它的类型为以`CoroutinBody`为`receiver`的无入参、返回值为泛型`R`类型的挂起函数，其函数体是将强转为非空的`parameter`参数为入参调用的挂起函数`block`
      2. 通过`coroutineBlock`创建名为`start`的协程，因不需要马上启动，所以使用`createCoroutine()`方法创建。`createCoroutine()`方法需要传入两个参数：第一个参数是`receiver`即`SuspendLambda`的`receiver`，所以写`CoroutinBody`的实例`body`；第二个参数是协程执行完毕后的回调，因为类`Coroutine`实现了`Continuation`接口，所以写`this`
      3. 初始化状态`status`，为其赋值为`Created`状态，并将创建的`start`传入
   10. 创建`resume()`方法，该方法是一个挂起函数，传入泛型`P`类型的参数，返回泛型`R`，需要通过`suspendCoroutine`拿到它的`Continuation`，然后进行状态的流转:
       1. 创建状态属性`previousStatus`，调用`status`的`getAndUpdate()`方法，该方法使用给定函数的结果以原子方式更新当前值，并返回先前的值，该方法中的函数里的`it`即为当前状态，也就是更新之前的状态
       2. 添加`when`语句进行状态判断
       3. 若为`Created`状态，将`body`中的`parameter`赋值为`value`，作为函数调用的第一个参数。之后状态流转为`resume`,并将控制外部流程的`continuation`传递进去
       4. 若为`Yielded`状态，状态流转为`resume`,并将控制外部流程的`continuation`传递进去
       5. 若为`Resumed`状态，抛非法异常：已经`resumed`了
       6. 若为`Dead`状态，抛非法异常：已经销毁
       7. 切换完状态后，判断`previousStatus`状态：
          1. 若为`Created`状态，将之前的挂起点执行一下，之所以不在上一条的`when`语句中执行，是因为`getAndUpdate`中的`lambda`可能会执行多次。因此，等状态流转成功后再执行。这是协程刚被创建，第一次执行`resume`方法的场景，所以`resume()`方法不需要传进去参数
          2. 若为`Yielded`状态，将`previousStatus`强转为`P`泛型的`Yielded`类型，之后执行它的`continuation`的`resume()`方法，并将`value`值传递进去
   11. 创建`yield()`方法，该方法是一个挂起函数，传入泛型`R`类型的参数，返回泛型`P`，需要通过`suspendCoroutine`拿到它的`Continuation`，然后进行状态的流转。
3. 创建类`DispatcherContinuation`，用于进行`continuation`的分发
   1. 其主构造器中传入两个参数，一个是要执行的`continuation`，一个是要执行`continuation`的`executor`
   2. 该类实现使用`Continuation`代理，如果只需要实现需要修改的方法即可，其它与`Continuation`一致
4. 创建类`Dispatcher`，实现`ContinuationInterceptor`接口，用于切换线程：
   1. 其中`key`若是实现协程拦截器，则赋值`ContinuationInterceptor`
   2. 在拦截协程方法中返回分发协程类对象，其中传入的执行器是使用`newSingleThreadExecutor()`创建的新执行器
5. 创建`main()`函数验证代码：
   1. 创建一个名为`producer`的`Coroutine`，其中
      1. 第一个形参是`CoroutineContext`类型，传入的是`Dispatcher`类实例，该类实现`ContinuationInterceptor`接口，而该接口继承自`CoroutineContext`
      2. 第二个形参是`receiver`为`CoroutineBody`的`lambda`表达式，该表达式入参为`Unit`即无入参，返回结果为`Int`类型。此时`lambda`表达式功能为执行`fori`循环，执行四次，从0开始到3结束，每次打印日志发送出去的值，并将`i`通过`yield()`方法传递出去
   2. 创建名为`consumer`的`Coroutine`，其中
      1. 第一个形参是`CoroutineContext`类型，传入的是`Dispatcher`类实例，该类实现`ContinuationInterceptor`接口，而该接口继承自`CoroutineContext`
      2. 第二个形参是`receiver`为`CoroutineBody`的`lambda`表达式，该表达式入参为`Int`类型，返回结果为`Unit`类型。此时`lambda`表达式功能为打印传递进来的参数，之后执行`fori`循环，执行四次，从0开始到3结束，每次通过`yield()`方法取到生产者发送的值，并打印接收到的值
   3. 创建一个循环，如果生产者和消费者都处于活跃状态则执行循环体：通过生产者的`resume()`方法取到生产的值，将通过消费者的`resume()`方法将值传递给消费者
6. 问题：使用`Executor`的`newSingleThreadExecutor()`方法创建的线程不是幽灵线程，该线程会一直在后台等待`execute()`方法传递的任务。所以主线程执行完毕后，`executor`创建的线程还在后台运行，因此不能正常结束

# 七、 示例-- 基于非对称协程`API`实现对称协程

## 1. 实现思路

基于非对称协程实现对称协程只需要 **添加一个中间层调度中心**即可实现。

![avatar](/kotlin学习系列九/symmetric_coroutine.png)

如上图所示：

1. 若从协程A转到协程B，若是非对称协程直接在协程A中`yield()`或是`resume(coroutineB)`即可
2. 而若是对称协程，协程A`yield()`之后将调度权转给调度中心，而调度中心再调用`resume(coroutineB)`,如此来通过非对称实现对称。
3. **调度中心可以是单独的一个协程，它只负责分发转移调度权。** 而调度协程如何知道调度权转移给谁呢？只需要调用`yield()`方法时将要转到的协程作为参数传入调度协程即可，如下图所示：</br>
    ![avatar](/kotlin学习系列九/symmetric_coroutine2.png)

## 2. 代码实现

```kotlin

//sym全拼为symmetric
class SymCoroutine<T>(
    override val context: CoroutineContext = EmptyCoroutineContext,
    private val block: suspend SymCoroutine<T>.SymCoroutineBody.(T) -> Unit
) : Continuation<T> {

    companion object {
        lateinit var main: SymCoroutine<Any?>

        suspend fun main(block: suspend SymCoroutine<Any?>.SymCoroutineBody.() -> Unit) {
            SymCoroutine<Any?> { block() }.also { main = it }.start(Unit)
        }

        fun <T> create(
            context: CoroutineContext = EmptyCoroutineContext,
            block: suspend SymCoroutine<T>.SymCoroutineBody.(T) -> Unit
        ): SymCoroutine<T> {
            return SymCoroutine(context, block)
        }
    }

    class Parameter<T>(val coroutine: SymCoroutine<T>, val value: T)

    val isMain: Boolean
        get() = this == main

    private val body = SymCoroutineBody()

    private val coroutine = Coroutine<T, Parameter<*>>(context) {
        Parameter(this@SymCoroutine, suspend {
            block(body, it)
            if(this@SymCoroutine.isMain) Unit else throw IllegalStateException("SymCoroutine cannot be dead.")
        }() as T)
    }

    inner class SymCoroutineBody {
        private tailrec suspend fun <P> transferInner(symCoroutine: SymCoroutine<P>, value: Any?): T{
            if(this@SymCoroutine.isMain){
                return if(symCoroutine.isMain){
                    value as T
                } else {
                    val parameter = symCoroutine.coroutine.resume(value as P)
                    transferInner(parameter.coroutine, parameter.value)
                }
            } else {
                this@SymCoroutine.coroutine.run {
                   return yield(Parameter(symCoroutine, value as P))
                }
            }
        }

        suspend fun <P> transfer(symCoroutine: SymCoroutine<P>, value: P): T {
           return transferInner(symCoroutine, value)
        }
    }

    override fun resumeWith(result: Result<T>) {
        throw IllegalStateException("SymCoroutine cannot be dead!")
    }

    suspend fun start(value: T){
        coroutine.resume(value)
    }
}

object SymCoroutines {
    val coroutine0: SymCoroutine<Int> = SymCoroutine.create<Int> { param: Int ->
        log("coroutine-0", param)
        var result = transfer(coroutine2, 0)
        log("coroutine-0 1", result)
        result = transfer(SymCoroutine.main, Unit)
        log("coroutine-0 1", result)
    }

    val coroutine1: SymCoroutine<Int> = SymCoroutine.create { param: Int ->
        log("coroutine-1", param)
        val result = transfer(coroutine0, 1)
        log("coroutine-1 1", result)
    }

    val coroutine2: SymCoroutine<Int> = SymCoroutine.create { param: Int ->
        log("coroutine-2", param)
        var result = transfer(coroutine1, 2)
        log("coroutine-2 1", result)
        result = transfer(coroutine0, 2)
        log("coroutine-2 2", result)
    }
}

suspend fun main() {
    SymCoroutine.main {
        log("main", 0)
        val result = transfer(SymCoroutines.coroutine2, 3)
        log("main end", result)
    }
}
```

**代码说明：**

1. 该章节代码是基于[上一章节非对称协程代码](#jumpLuaAPI)开发的
2. **对称协程要求在运行结束后主动将调度权转移，而不是执行完毕**。即不允许在不确定的协程里运行结束，以避免意想不到的结果。
3. 创建类`SymCoroutine`：
   1. 该类只有一个泛型参数：`T`表示需要的参数；因为是对称协程没有返回值的说法
   2. 创建伴生对象，在伴生对象中做如下操作：
      1. 创建`SymCoroutine`类型的属性`main`，用于保存调度中心的协程，以判断当前协程是否为调度中心
      2. 创建挂起函数`main`，该函数入参类型为以`SymCoroutinBody`为`receiver`的无入参、返回值为`Unit`类型的挂起函数，`main`函数体将传入的挂起函数启动
      3. 创建可静态调用的方法`create()`，用于创建`SymCoroutine`类的对象
   3. 创建类`Parameter`，用于包含`yield()`和`resume()`方法中所有入参，即`SymCoroutine`类型的协程对象，和该协程对象所需要的类型为泛型`T`的参数`value`，也是为了复用[上一章节非对称协程代码](#jumpLuaAPI)
   4. 创建属性`isMain`，返回值类型为`Boolean`，用于判断当前协程是否为调度中心协程即是是否为`main`协程
   5. 创建`SymCoroutineBody`类型对象`body`
   6. 创建非对称协程对象`coroutine`，即为[上一章节非对称协程代码](#jumpLuaAPI)中的`Coroutine`类对象，其中需要的参数为泛型`T`，返回的参数为`Parameter`。传入真正执行的`lambda`表达式，该`lambda`表达式返回一个`Parameter`实例对象。
      1. `parameter`对象第一个入参是要转移的目标协程，传当前协程即可
      2. `parameter`对象另一个参数是目标协程所需要参数，创建一个`suspend lambda`表达式，在表达式中运行`block`表达式以得到结果，`block`表达式中首先传入上一步创建的`body`对象，再传入泛型参数，因为[上一章节非对称协程代码](#jumpLuaAPI)中的`Coroutine`类对象中的语句`private val block: suspend Coroutine<P, R>.CoroutineBody.(P) -> R`入参中`CoroutineBody`有一个入参`P`,因此可以直接使用`it`来代指。
      3. 执行完`block`表达式后，判断当前协程是否为`main`，如果是`main`协程则返回一个`Unit`，否则抛出异常：`SymCoroutine`不能执行完毕
   7. 创建内部类`SymCoroutineBody`，用于约束`transfer`方法只能在该协程`lambda`中使用：
      1. 创建一个挂起函数`transfer`：
         1. 一个入参类型为`SymCoroutine`的协程对象，即转移的目标协程，该协程接收一个泛型为`P`类型的参数
         2. 另一个入参类型为泛型`P`的`value`值，该值为上一个入参协程对象所需要的参数
         3. 返回值为泛型`T`类型，用于流转到该协程时所要传入参数
      2. 创建挂起函数`transferInner`，用于实现函数`transfer`的功能：
         1. 满足尾递归条件的函数前面加`tailrec`修饰，编译器会优化该递归成一个快速而高效的基于循环的版本，减少栈消耗。尾递归条件：1. 最后一条语句是函数调用语句；2. 调用的函数是自身
         2. 第一个入参类型与`transfer()`函数一致
         3. 第二个入参类型为`Any?`
         4. 先判断当前`SymCoroutineBody`的`SymCoroutine`是否为`main`协程
         5. 若是`main`协程，因为每次转换调度权都会回到`main`协程中，因此要再判断转移目标协程是否为`main`协程
            1. 若转移目标协程是`main`协程，直接将`value`值返回
            2. 若转移目标协程不是`main`协程，即不是转移给`main`协程的，则需要获取`yield()`出来的两个参数，通过`symCoroutine.coroutine`取到非对称协程的对象并调用其`resume()`方法获取该协程返回的结果，其中`resume()`方法入参就是该协程需要的参数，即`value`,将其强转为泛型`P`。之后再调用`transferInner()`方法，将转移的目标协程及目标协程需要的参数传递进去
         6. 若当前`SymCoroutineBody`的`SymCoroutine`不是`main`协程，即协程A到协程B的过程中，协程A还没有`yield`,因此需要执行一下协程A的`yield()`方法，但是`this@SymCoroutine.coroutine`中只有它的`CoroutineBody`里才有`yield()`方法，因此我们需要改造[上一章节的代码](#jumpLuaAPI)在`Coroutine`类中添加如下方法:

            ```kotlin
            suspend fun <SymT> SymCoroutine<SymT>.yield(value: R): P {
                return body.yield(value)
            }
            ````

            之后调用`this@SymCoroutine.coroutine`的`run`方法，并将`symCoroutine`和`value`拼装成`Parameter`对象，再将该对象作为入参调用`yield()`方法

   8. 创建方法`start()`，用于启动该调度中心的协程`main`，在该方法体中调用非对称协程的`resume()`方法。该方法在`SymCoroutine`的伴生对象里的`main()`方法中被调用
   9. 创建带名伴生对象`SymCoroutines`，在该伴生对象中创建三个对称协程，在各自的函数中调用`transfer()`方法进行协程的切换，并打印日志
   10. 创建程序入口`main()`函数，在该函数中调用`SymCoroutine.main()`方法，用于创建并启动调度中心的协程`main`，并在该`main`协程中开始进行协程的切换并打印日志

**打印结果：**

```kotlin
Coroutine: main 0
Coroutine: coroutine-2 3
Coroutine: coroutine-1 2
Coroutine: coroutine-0 1
Coroutine: coroutine-2 1 0
Coroutine: coroutine-0 1 2
Coroutine: main end kotlin.Unit
```

**执行流程说明：**

上述例子中协程调用流程如下图所示：</br>
   ![avatar](/kotlin学习系列九/symmetric_coroutine3.png)
    </br>

   1. 程序开始首先启动调度中心的协程即协程`main`
   2. 打印信息`main 0`然后再将调度权转给协程2，并传入值3
   3. 之后进入协程2，打印信息`coroutine-2 3`，接着将调度权转移给协程1，并传入值2
   4. 之后进入协程1，打印信息`coroutine-1 2`，接着将调度权转移给协程0，并传入值1
   5. 之后进入协程0，打印信息`coroutine-0 1`，接着将调度权转移给协程2，并传入值0
   6. 之后进入协程2，运行调用其第一个挂起函数`transfer()`之后的语句，打印信息`coroutine-2 1 0`，接着将调度权转移给协程0，并传入值2
   7. 之后进入协程0，打印信息`coroutine-0 1 2`，接着将调度权转移给了调度中心的协程`main`，表示要结束协程调度
   8. 之后进入协程`main`，运行调用其挂起函数`transfer()`之后的语句，打印信息`main end kotlin.Unit`，程序结束
   9. 若第6步，打印信息后，将调度权转移给协程1，即`result = transfer(coroutine0, 2)`替换为`result = transfer(coroutine1, 2)`，则会进入协程1，运行调用其挂起函数`transfer()`之后的语句，打印信息`coroutine-1 1 2`，之后协程里`lambda`运行完毕了，并会走到如下语句`if(this@SymCoroutine.isMain) Unit else throw IllegalStateException("SymCoroutine cannot be dead.")`，因为该协程不是协程`main`，则程序会触发断言，抛出异常。由此将协程切换限制到在协程`main`开始，最后到协程`main`里结束

# 八、 示例--仿`Go`的`channel`实现协程通信

```kotlin
@file:RequiresApi(Build.VERSION_CODES.N)
package cn.ltt.projectcollection.kotlin.coroutinesLab.go

import android.os.Build
import androidx.annotation.RequiresApi
import cn.ltt.projectcollection.kotlin.coroutinesLab.lua.log
import cn.ltt.projectcollection.kotlin.coroutinesLab.primary.dispatcher.DispatcherContext
import java.util.*
import java.util.concurrent.atomic.AtomicReference
import kotlin.coroutines.*

interface Channel<T> {
    suspend fun send(value: T)

    suspend fun receive(): T

    fun close()
}

class ClosedException(message: String) : Exception(message)

class SimpleChannel<T> : Channel<T> {

    sealed class Element {
        override fun toString(): String {
            return this.javaClass.simpleName
        }

        object None : Element()
        class Producer<T>(val value: T, val continuation: Continuation<Unit>) : Element()
        class Consumer<T>(val continuation: Continuation<T>) : Element()
        object Closed : Element()
    }

    private val state = AtomicReference<Element>(Element.None)

    override suspend fun send(value: T) = suspendCoroutine<Unit> { continuation ->
        val prev = state.getAndUpdate {
            when (it) {
                Element.None -> Element.Producer(value, continuation)
                is Element.Producer<*> -> throw IllegalStateException("Cannot send new element while previous is not consumed.")
                is Element.Consumer<*> -> Element.None
                Element.Closed -> throw IllegalStateException("Cannot send after closed.")
            }
        }
        (prev as? Element.Consumer<T>)?.continuation?.resume(value)?.let { continuation.resume(Unit) }
    }


    override suspend fun receive(): T = suspendCoroutine<T> { continuation ->
        val prev = state.getAndUpdate {
            when (it) {
                Element.None -> Element.Consumer(continuation)
                is Element.Producer<*> -> Element.None
                is Element.Consumer<*> -> throw IllegalStateException("Cannot receive new element while previous is not provided.")
                is Element.Closed -> throw IllegalStateException("Cannot receive new element after closed.")
            }
        }
        (prev as? Element.Producer<T>)?.let {
            it.continuation.resume(Unit)
            continuation.resume(it.value)
        }
    }

    override fun close() {
        val prev = state.getAndUpdate { Element.Closed }
        if (prev is Element.Consumer<*>) {
            prev.continuation.resumeWithException(ClosedException("Channel is closed."))
        } else if (prev is Element.Producer<*>) {
            prev.continuation.resumeWithException(ClosedException("Channel is closed."))
        }
    }
}

fun plainChannelSample() {
    val channel = SimpleChannel<Int>()

    go("producer"){
        for (i in 0..6) {
            log("send", i)
            channel.send(i)
        }
    }

    go("consumer", channel::close){
        for (i in 0..5) {
            log("receive-i:$i")
            val got = channel.receive()
            log("got", got)
        }
    }
}

fun go(name: String = "", completion: () -> Unit = {}, block: suspend () -> Unit){
    block.startCoroutine(object : Continuation<Any> {
        override val context = DispatcherContext()

        override fun resumeWith(result: Result<Any>) {
            log("end $name", result)
            completion()
        }
    })
}

fun main() {
    plainChannelSample()

    //blockingChannelSample()
}
```

**说明：**

1. `go routine`的详细内容可以看前而讲到的[Go routine示例](#jumpGoRoutine)
2. 因工程里设置的最小`API`版本为16，而`AtomicReference#getAndUpdate()`方法需要的`API`版本为**24**， 所以需要添加注解来标识，因此使用**`@file`** 来为整个文件添加注解而不需要再在每个方法上添加
3. 定义接口`Channel`，用于规范协程间通信的管道。在接口中定义三个方法：
   1. 挂起函数`send()`方法，用于发送参数
   2. 挂起函数`receive()`方法，用于接收参数
   3. 函数`close()`，用于关闭管道
4. 定义类`CosedException`，用于显示关闭管道异常信息
5. 定义类`SimpleChannel`，实现接口`channel`，该类 **不支持队列**，只能发送接收，不能发送发送再接收接收：
   1. 创建密封类`Element`，用于规范管道的几种状态：
      1. `None`状态，即没有发送的也没有接收的
      2. `Producer`状态，即已经发送了，但还没有接收
      3. `Consumer`状态，即已经开始接收了，但还没有发送
      4. `Closed`状态，管道关闭
   2. 创建状态属性`state`，类型为`AtomicReference`类型的`Element`，初始值为`Element.None`
   3. 实现挂起函数`send()`，通过`suspendCoroutine`函数可以获取当前挂起函数中的`Continuation`实例，并且在该函数中可以调用`Continuation`方法。
      1. `send()`方法是发送数据，不需要返回数据，所以`suspendCoroutine`后的泛型写`Unit`
      2. 创建状态属性`prev`，调用`state`的`getAndUpdate()`方法，该方法使用给定函数的结果以原子方式更新当前值，并返回先前的值，该方法中的函数里的`it`即为当前状态，也就是更新之前的状态
         1. 添加`when`语句进行状态判断
         2. 若为`None`状态，则将`state`更新为`Producer`状态，并将传入的`value`和当前的`coroutine`作为参数传进去，等接收人将`value`取走，并调用这个`coroutine`来将这个协程恢复执行
         3. 若为`Producer`状态，抛非法异常。因为此时已经是发送待接收状态了不能再发送了
         4. 若为`Consumer`状态，意味着已经在等待发送数据，将数据发送出去，再将`state`更新为`None`状态
         5. 若为`Closed`状态，抛非法异常，关闭了就不能再发送了
      3. 切换完状态后，将`prev`判空强转为`Consumer`状态，意味着已经在等待发送数据，之后每步都加非空判断，调用该状态的协程的`resume()`方法将`value`发送出去，之后调用`continuation`的`resume()`方法恢复当前协程
   4. 实现挂起函数`receive()`，依然是通过通过`suspendCoroutine`函数获取当前挂起函数中的`Continuation`实例，并且在该函数中调用`Continuation`方法。
      1. `receive()`方法用于接收数据，该方法返回接收到的数据，所以`suspendCoroutine`后的泛型写`T`
      2. 创建状态属性`prev`，调用`state`的`getAndUpdate()`方法，该方法使用给定函数的结果以原子方式更新当前值，并返回先前的值，该方法中的函数里的`it`即为当前状态，也就是更新之前的状态
         1. 添加`when`语句进行状态判断
         2. 若为`None`状态，则将`state`更新为`Consumer`状态，并将当前的`coroutine`作为参数传进去，
         3. 若为`Producer`状态，意味着已经是发送待接收状态，因此要将数据接收，再将`state`更新为`None`状态
         4. 若为`Consumer`状态，抛非法异常。因为此时已经是等待发送状态了，不能接收了。不支持队列
         5. 若为`Closed`状态，抛非法异常，关闭了就不能再发送了
      3. 切换完状态后，将`prev`判空强转为`Producer`状态，意味着已经发送数据等待接收，之后每步都加非空判断，因为该状态是发送者，不需要返回结果，调用该状态的协程的`resume()`方法将`Unit`发送回去，之后调用`continuation`的`resume()`方法将发送过来的值传递进去来获取
   5. 实现挂起函数`close()`：
      1. 创建状态属性`prev`，调用`state`的`getAndUpdate()`方法，将`state`切换为`closed`状态，并将切换之前的状态值赋值给`prev`
      2. 判断切换之前的状态值：如果是`Consumer`状态或是`Producer`状态，调用该状态的`continuation`的`resumeWithException()`方法。也就是在关闭过程中管道还在发送或是接收数据，抛出异常管道已经关闭啦不要再用啦
6. 定义函数`go()`，用于模仿`go`语言中的`routine`样式，
   1. 有三个入参：
      1. 第一个参数是一个`String`类型，默认为空，只做打印日志使用
      2. 第二个参数是无入参返回值为`Unit`的函数`completion`，默认是空函数
      3. 第三个参数是无入参返回值为`Unit`的挂起函数`block`
   2. 函数体内调用`block`的`startCoroutine()`方法来运行挂起函数：
      1. 每次运行挂起函数都会创建一个`DispatcherContext()`，由此实现每个协程都单独运行在自己的线程上。具体实现见[协程间的线程调度](#jumpThreadSwitch)
      2. 在`resumeWith()`方法中打印信息，并执行传进来的方法`completion()`
7. 定义函数`plainChannelSample()`，用于非队列非阻塞式管道示例：
   1. 定义属性`channel`，赋值为`SimpleChannel`类的实例，泛型传入`Int`
   2. 调用`go()`函数
      1. 第一个参数传入`producer`
      2. 第二个参数不传
      3. 第三个参数传入一个挂起函数，挂起函数体为一个`fori`循环，从0到6执行7次，每次循环都打印信息，并通过调用`channel.send()`方法将循环下标值传递出去
   3. 调用`go()`函数：
      1. 第一个参数传入`consumer`
      2. 第二个参数传入匿名函数，函数中调用`channel::close`，该方法用于关闭管道，即接收完所有数据后调用该方法关闭管道
      3. 第三个参数传入一个挂起函数，挂起函数体为一个`fori`循环，从0到5执行6次，每次循环打印信息，并通过调用`channel.receive()`方法获取管道发送的值，之后打印该值
8. 创建`main()`入口函数，在函数中调用`plainChannelSample()`方法

**打印结果：**

```kotlin
16:49:16:913 [DefaultDispatcher-worker-1] receive
16:49:16:913 [DefaultDispatcher-worker-0] send 0
16:49:16:946 [DefaultDispatcher-worker-0] send 1
16:49:16:946 [DefaultDispatcher-worker-2] got 0
16:49:16:946 [DefaultDispatcher-worker-2] receive
16:49:16:946 [DefaultDispatcher-worker-2] got 1
16:49:16:946 [DefaultDispatcher-worker-3] send 2
16:49:16:946 [DefaultDispatcher-worker-2] receive
16:49:16:946 [DefaultDispatcher-worker-2] got 2
16:49:16:946 [DefaultDispatcher-worker-2] receive
16:49:16:946 [DefaultDispatcher-worker-4] send 3
16:49:16:947 [DefaultDispatcher-worker-4] send 4
16:49:16:947 [DefaultDispatcher-worker-5] got 3
16:49:16:947 [DefaultDispatcher-worker-5] receive
16:49:16:947 [DefaultDispatcher-worker-5] got 4
16:49:16:947 [DefaultDispatcher-worker-5] receive
16:49:16:947 [DefaultDispatcher-worker-6] send 5
16:49:16:947 [DefaultDispatcher-worker-6] send 6
16:49:16:947 [DefaultDispatcher-worker-7] got 5
16:49:16:948 [DefaultDispatcher-worker-7] end consumer Success(kotlin.Unit)
16:49:16:949 [DefaultDispatcher-worker-8] end producer Failure(cn.ltt.projectcollection.kotlin.coroutinesLab.go.ClosedException: Channel is closed.)
```

**打印结果说明：**

1. 最后在`go()`函数中，协程执行完毕后会打印信息，结果显示`consumer`协程成功结束，但是`producer`协程是异常结束，之所以异常是因为`consumer`只循环了6次，且在最后一次循环后将管道关闭了，而在`producer`中循环了7次，最后一次循环时管道已经关闭了，抛出异常，因此`producer`是异常结束

**定义使用队列的可阻塞式的管道**

```kotlin

class QueueChannel<T> : Channel<T> {
    class Producer<T>(val value: T, val continuation: Continuation<Unit>)
    class Consumer<T>(val continuation: Continuation<T>)

    sealed class ElementList {
        object Nil : ElementList()

        abstract class AbsElementList<T> : ElementList() {
            private val list = mutableListOf<T>()

            protected abstract fun new(): AbsElementList<T>

            fun offer(element: T): ElementList {
                return new().also { it.list += this.list + element }
            }

            open fun take(): ElementList {
                val newList = this.list - this.list.first()
                if (newList.isEmpty()) return Nil
                return new().also { it.list += newList }
            }

            fun peek(): T = list.first()

            fun elements(): List<T> = Collections.unmodifiableList(list)
        }

        class ProducerList<T> : AbsElementList<Producer<T>>() {
            override fun new() = ProducerList<T>()
        }

        class ConsumerList<T> : AbsElementList<Consumer<T>>() {
            override fun new() = ConsumerList<T>()
        }

        object Closed : ElementList()
    }

    private val state = AtomicReference<ElementList>(ElementList.Nil)

    override suspend fun send(value: T) = suspendCoroutine<Unit> { continuation ->
        val prev = state.getAndUpdate {
            when (it) {
                ElementList.Nil -> ElementList.ProducerList<T>().offer(Producer(value, continuation))
                is ElementList.ProducerList<*> -> {
                    (it as ElementList.ProducerList<T>).offer(Producer(value, continuation))
                }
                is ElementList.ConsumerList<*> -> {
                    (it as ElementList.ConsumerList<T>).take()
                }
                ElementList.Closed -> throw IllegalStateException("Cannot send after closed.")
                else -> throw IllegalStateException()
            }
        }
        (prev as? ElementList.ConsumerList<T>)?.peek()?.continuation?.resume(value)?.run { continuation.resume(Unit) }
    }

    override suspend fun receive(): T = suspendCoroutine<T> { continuation ->
        val prev = state.getAndUpdate {
            when (it) {
                ElementList.Nil -> ElementList.ConsumerList<T>().offer(Consumer(continuation))
                is ElementList.ProducerList<*> -> {
                    (it as ElementList.ProducerList<T>).take()
                }
                is ElementList.ConsumerList<*> -> {
                    (it as ElementList.ConsumerList<T>).offer(Consumer(continuation))
                }
                ElementList.Closed -> throw IllegalStateException("Cannot receive after closed.")
                else -> throw IllegalStateException()
            }
        }
        (prev as? ElementList.AbsElementList<Producer<T>>)?.peek()?.let {
            it.continuation.resume(Unit)
            continuation.resume(it.value)
        }
    }

    override fun close() {
        val prev = state.getAndUpdate { ElementList.Closed }
        if (prev is ElementList.ConsumerList<*>) {
            prev.elements().forEach {
                it.continuation.resumeWithException(ClosedException("Channel is closed."))
            }
        } else if (prev is ElementList.ProducerList<*>) {
            prev.elements().forEach {
                it.continuation.resumeWithException(ClosedException("Channel is closed."))
            }
        }
    }
}

fun blockingChannelSample() {
    val channel = QueueChannel<Int>()
    for (n in 0..5) {
        go("producer $n"){
            for (i in n..n + 5) {
                log("send", i)
                channel.send(i)
            }
        }
    }

    go("consumer", channel::close) {
        for (i in 0..11) {
            val got = channel.receive()
            log("got", got)
        }
    }
}

```

**说明：**

1. 定义实现接口`Channel`的实现类`QueueChannel`，用于实现队列的可阻塞式的管道
   1. 定义类`Producer`，有两个入参
   2. 定义类`Consumer`，有一个入参
   3. 定义密封类`ElementList`，通过列表实现队列功能：
      1. 定义`Nil`状态，
      2. 定义抽象类`AbsElementList`:
         1. 定义可变列表属性`list`
         2. 定义抽象方法`new()`，返回`AbsElementList`对象
         3. 定义方法`offer()`，入参是泛型`T`类型的元素，返回结果是一个`ElementList`对象，函数体中返回的是一个新创建的列表，新列表添加了之前列表的所有内容，并添加了传入的元素
         4. 定义开放方法`take()`，返回一个`ElementList`对象，函数体操作：先创建属性`newList`用于存储之前列表去除头部元素后的列表，如果新列表是空的，则返回`Nil`，否则返回一个新创建的列表，该列表添加了`newList`中所有元素
         5. 定义`peek()`方法，用于返回当前列表的头部元素
         6. 定义`elements()`方法，用于返回一个只读的元素列表
   4. 定义类`ProducerList`，继承抽象类`AbsElementList`，重写`new()`方法，返回一个`ProducerList`对象
   5. 定义类`ConsumerList`，继承抽象类`AbsElementList`，重写`new()`方法，返回一个`ConsumerList`对象
   6. 定义单例类`Closed`
   7. 定义属性`state`，用于保存管道状态，初始化为`Nil`状态
   8. 重写挂起函数`send()`方法，通过`suspendCoroutine`函数可以获取当前挂起函数中的`Continuation`实例，并且在该函数中可以调用`Continuation`方法。
      1. `send()`方法是发送数据，不需要返回数据，所以`suspendCoroutine`后的泛型写`Unit`
      2. 创建状态属性`prev`，调用`state`的`getAndUpdate()`方法，该方法使用给定函数的结果以原子方式更新当前值，并返回先前的值，该方法中的函数里的`it`即为当前状态，也就是更新之前的状态
         1. 添加`when`语句进行状态判断
         2. 若为`Nil`状态，则将`state`更新为`ProducerList`状态，先将传入的`value`和当前的`coroutine`组合为对象`Producer`，再调用密封类`ElementList`的子类`ProducerList`的`offer()`方法，将组合好的对象`Producer`作为参数传进去
         3. 若为`ProducerList`状态，不切换状态，当前虽然已经处于已发送待接收状态，但因为使用了列表可以继续发送。先强转为`ProucerList`状态，然后再调用`ProducerList`的`offer()`方法，将传入的`value`和当前的`coroutine`这两个参数 组合的对象`Producer`作为参数传进去
         4. 若为`ConsumerList`状态，当前是等待发送状态，需要将生产者列表中的头元素发送出去，因此需要更新消费者列表去掉头元素，若去掉头元素后列表变为空了，则将状态切换为`Nil`
         5. 若为`Closed`状态，抛非法异常，关闭了就不能再发送了
         6. 若为其它状态，则抛非法异常
      3. 切换完状态后，将`prev`判空强转为`ConsumerList`状态，意味着已经在等待发送数据，之后每步都加非空判断，通过`peek()`方法取出消费者列表中的第一个消费者，再调用该消费者的协程的`resume()`方法将`value`发送出去，之后调用`continuation`的`resume()`方法恢复当前协程
   9. 重写挂起函数`receive()`，依然是通过`suspendCoroutine`函数获取当前挂起函数中的`Continuation`实例，并且在该函数中调用`Continuation`方法。
      1. `receive()`方法用于接收数据，该方法返回接收到的数据，所以`suspendCoroutine`后的泛型写`T`
      2. 创建状态属性`prev`，调用`state`的`getAndUpdate()`方法，该方法使用给定函数的结果以原子方式更新当前值，并返回先前的值，该方法中的函数里的`it`即为当前状态，也就是更新之前的状态
         1. 添加`when`语句进行状态判断
         2. 若为`Nil`状态，则将`state`更新为`ConsumerList`状态，先以当前的`coroutine`为参数创建对象`Consumer`，再调用密封类`ElementList`的子类`ConsumerList`的`offer()`方法，将组合好的对象`Consumer`作为参数传进去
         3. 若为`ProducerList`状态，当前处于已发送待接收状态，此时可以接收生产者列表中的头元素，然后更新消费者列表去掉头元素，若去掉头元素后列表变为空，则将状态切换为`Nil`
         4. 若为`ConsumerList`状态，不切换状态，当前虽然是等待发送状态，但还可以继续添加消费者。先强转为`ConsumerList`状态，然后再调用`ConsumerList`的`offer()`方法，将以当前的`coroutine`为参数创建对象`Consumer`，再调用`offer()`方法，将对象`Consumer`放到消费者列表中
         5. 若为`Closed`状态，抛非法异常，关闭了就不能再接收了
         6. 若为其它状态，抛非法异常
      3. 切换完状态后，将`prev`判空强转为以`Producer`为元素的抽象类`AbsElementList`的子类列表，此时意味着已经在等待接收数据，之后每步都加非空判断，通过`peek()`方法取出生产者列表中的第一个生产者，因生产者不需要返回结果，所以调用该生产者的协程的`resume()`方法时传递的是`Unit`，之后调用当前`continuation`的`resume()`方法将发送过来的值传递进去来获取
   10. 重写函数`close()`：
       1. 创建存储状态的属性`prev`，调用`state`的`getAndUpdate()`方法，将`state`切换为`closed`状态，并将切换之前的状态值赋值给`prev`
       2. 判断切换之前的状态值：如果是`ConsumerList`状态或是`ProducerList`状态，通过`prev`的`elements`方法取到该状态中元素列表，使用`foreach`循环调用每个元素的`continuation`的`resumeWithException()`方法。也就是在关闭过程中管道还在发送或是接收数据，抛出异常管道已经关闭啦不要再用啦
2. 定义函数`blockingChannelSample()`，用于非队列非阻塞式管道示例：
   1. 定义属性`channel`，赋值为`QueueChannel`类的实例，泛型传入`Int`
   2. 创建一个`fori`循环，循环下标`n`从0到5共执行6次，每次循环都会调用`go()`函数：
      1. `go()`函数第一个参数传入`producer`加上当前循环下标值
      2. 第二个参数不传
      3. 第三个参数传入一个挂起函数，挂起函数体为一个`fori`循环，循环下标`i`是从外层循环下标当前值`n`开始，到`n+5`结束，共执行6次，每次循环都打印信息，并通过调用`channel.send()`方法将内循环下标`i`值传递出去
   3. 调用`go()`函数：
      1. 第一个参数传入`consumer`
      2. 第二个参数传入匿名函数，函数中调用`channel::close`，该方法用于关闭管道，即接收完所有数据后调用该方法关闭管道
      3. 第三个参数传入一个挂起函数，挂起函数体为一个`fori`循环，从0到11执行12次，每次循环打印信息，并通过调用`channel.receive()`方法获取管道发送的值，之后打印该值
3. 最后在`main()`入口函数中调用`blockingChannelSample()`方法

**打印结果：**

```kotlin
17:39:56:691 [DefaultDispatcher-worker-1] send 1
17:39:56:691 [DefaultDispatcher-worker-5] send 5
17:39:56:691 [DefaultDispatcher-worker-3] send 3
17:39:56:691 [DefaultDispatcher-worker-4] send 4
17:39:56:691 [DefaultDispatcher-worker-2] send 2
17:39:56:691 [DefaultDispatcher-worker-0] send 0
17:39:56:721 [DefaultDispatcher-worker-1] send 2
17:39:56:721 [DefaultDispatcher-worker-7] got 1
17:39:56:721 [DefaultDispatcher-worker-5] send 6
17:39:56:721 [DefaultDispatcher-worker-8] got 5
17:39:56:721 [DefaultDispatcher-worker-8] got 4
17:39:56:722 [DefaultDispatcher-worker-9] send 5
17:39:56:722 [DefaultDispatcher-worker-8] got 3
17:39:56:722 [DefaultDispatcher-worker-10] send 4
17:39:56:722 [DefaultDispatcher-worker-8] got 2
17:39:56:722 [DefaultDispatcher-worker-11] send 3
17:39:56:722 [DefaultDispatcher-worker-8] got 0
17:39:56:722 [DefaultDispatcher-worker-12] send 1
17:39:56:722 [DefaultDispatcher-worker-8] got 2
17:39:56:722 [DefaultDispatcher-worker-13] send 3
17:39:56:722 [DefaultDispatcher-worker-8] got 6
17:39:56:722 [DefaultDispatcher-worker-14] send 7
17:39:56:723 [DefaultDispatcher-worker-8] got 5
17:39:56:723 [DefaultDispatcher-worker-15] send 6
17:39:56:723 [DefaultDispatcher-worker-8] got 4
17:39:56:723 [DefaultDispatcher-worker-15] send 5
17:39:56:723 [DefaultDispatcher-worker-8] got 3
17:39:56:723 [DefaultDispatcher-worker-4] send 4
17:39:56:723 [DefaultDispatcher-worker-8] got 1
17:39:56:723 [DefaultDispatcher-worker-4] send 2
17:39:56:724 [DefaultDispatcher-worker-8] end consumer Success(kotlin.Unit)
17:39:56:725 [DefaultDispatcher-worker-7] end producer 2 Failure(cn.ltt.projectcollection.kotlin.coroutinesLab.go.ClosedException: Channel is closed.)
17:39:56:725 [DefaultDispatcher-worker-5] end producer 0 Failure(cn.ltt.projectcollection.kotlin.coroutinesLab.go.ClosedException: Channel is closed.)
17:39:56:725 [DefaultDispatcher-worker-3] end producer 1 Failure(cn.ltt.projectcollection.kotlin.coroutinesLab.go.ClosedException: Channel is closed.)
17:39:56:725 [DefaultDispatcher-worker-0] end producer 5 Failure(cn.ltt.projectcollection.kotlin.coroutinesLab.go.ClosedException: Channel is closed.)
17:39:56:725 [DefaultDispatcher-worker-2] end producer 3 Failure(cn.ltt.projectcollection.kotlin.coroutinesLab.go.ClosedException: Channel is closed.)
17:39:56:725 [DefaultDispatcher-worker-8] end producer 4 Failure(cn.ltt.projectcollection.kotlin.coroutinesLab.go.ClosedException: Channel is closed.)
```

**打印结果说明：**

1. 因是多协程操作，每次运行的打印结果都不一样
2. 最后在`go()`函数中，协程执行完毕后会打印信息，结果显示`consumer`协程成功结束，但是5个`producer`协程都是异常结束，之所以异常是因为`consumer`只循环了12次，且在最后一次循环后将管道关闭了，而有6个生产者且每个生产者发送了6条数据给`consumer`，而`consumer`只循环了12次，不能把各生产者发送的值都接收到，所以抛出异常，因此`producer`是异常结束，将`consumer`协程里的循环改为到35结束，即循环36次，将生产者们发送的值都接收到，如此生产者们就不会异常结束了

# 八、 示例--仿`Js`实现`async await`

```kotlin

interface AsyncScope

suspend fun <T> AsyncScope.await(block: () -> Call<T>) = suspendCoroutine<T> {
    continuation ->
    val call = block()
    call.enqueue(object : Callback<T>{
        override fun onFailure(call: Call<T>, t: Throwable) {
            continuation.resumeWithException(t)
        }

        override fun onResponse(call: Call<T>, response: Response<T>) {
            if(response.isSuccessful){
                response.body()?.let(continuation::resume) ?: continuation.resumeWithException(NullPointerException())
            } else {
                continuation.resumeWithException(HttpException(response))
            }
        }
    })
}

fun async(context: CoroutineContext = EmptyCoroutineContext, block: suspend AsyncScope.() -> Unit) {
    val completion = AsyncCoroutine(context)
    block.startCoroutine(completion, completion)
}

class AsyncCoroutine(override val context: CoroutineContext = EmptyCoroutineContext): Continuation<Unit>, AsyncScope {
    override fun resumeWith(result: Result<Unit>) {
        result.getOrThrow()
    }
}

fun main() {
    async() {
        val user = await { githubApi.getUserCallback("puppet16") }
        log(user)
    }
}
```

**打印结果：**

```kotlin
request: 200
19:39:56:395 [OkHttp https://api.github.com/...] User(id=14159272, name=Puppet16, url=https://api.github.com/users/puppet16)
```

**说明：**

1. `js`的`async await`的详细内容可以看前面讲到的[`async/await`关键字](#jumpJsAsync)
2. 定义一个接口`AsyncScope`，用于约束`async()`、`await()`这两个函数的使用范围
3. 创建类`AsyncCoroutine`，实现接口`Continuation`和接口`AsyncScope`：
   1. 重写`ConroutineContext`类型的`context`，默认值为`EmptyCoroutinContext`
   2. 重写`resumeWith()`方法，在该方法中如果结果有异常则抛出异常
4. 定义函数`async()`：
   1. 第一个入参是`CoroutineContext`类型的变量`context`，它的默认值是`EmptyCoroutineContext`
   2. 第二个入参是挂起函数的`lambda`表达式`block`，必须是限制必须是`AsyncScope`的扩展函数
   3. 创建名为`completion`的`AsyncCoroutine`类的实例，将`context`传入其中
   4. 之后调用`startCoroutine()`启动`block`协程，该方法需要的`receiver`和协程结束时调用的`completion`都传入上一步创建的`completion`
5. 定义挂起函数`await()`，该函数传入一个`lambda`表达式，该表达式没有入参、出参是一个`Call`对象。之后通过`suspendCoroutine`函数获取当前挂起函数中的`Continuation`实例
   1. 创建属性`call`，用于保存`block()`返回的结果
   2. 调用`call`的异步请求方法`enqueue()`，该方法需要有一个`Callback`类对象，因此使用 **`object:`关键字创建`Callback`类的匿名内部对象**，之后重写该类中方法
      1. 在`onFailure()`方法中通过`resumeWithException()`方法给协程抛出异常
      2. 在`onResponse()`方法中，先判断网络请求是否成功，若未成功则抛出异常；若成功则取`response`的`body`对象，若有`body`对象则通过函数引用，直接调用协程的`resume()`方法；若没有`body`对象则调用`resumeWithException()`方法抛出空指针异常
6. 由此`async`本身没有异步的能力，异步是由`call`的`enqueue()`方法提供的

# 九、延伸--揭秘 `suspend fun main`

## 1. 普通的`main`函数

```kotlin
fun main(args: Array<String>) {
    ...
}
```

自`Kotlin 1.3`之后可省略参数

```kotlin
//since kotlin 1.3
fun main() {
    ...
}
```

## 2. 可挂起的`main`函数

```kotlin
suspend fun main() {
    ...
}
```

挂起`main`函数看起来函数类型为：`suspend () -> Unit`。但其实该函数传入了一个`Continuation`，并且返回了一个`Any?`，即实际上类型为`(Continuation<Unit>) -> Any?`，本质上挂起`main`函数为如下样式：

```kotlin
fun main(continuation: Continuation<Unit>): Any? {
    ...
}
```

而定义为如上形式也就不是入口函数了，可以使用`runSuspend()`方法运行如上形式函数：

```kotlin
fun main1(continuation: Continuation<Unit>): Any? {
    return Unit;
}

fun main() {
    runSuspend(::main1 as suspend () -> Unit)
}
```

```kotlin

package kotlin.coroutines.jvm.internal

import kotlin.coroutines.Continuation
import kotlin.coroutines.CoroutineContext
import kotlin.coroutines.EmptyCoroutineContext
import kotlin.coroutines.startCoroutine

/**
 * Wrapper for `suspend fun main` and `@Test suspend fun testXXX` functions.
 */
@SinceKotlin("1.3")
internal fun runSuspend(block: suspend () -> Unit) {
    val run = RunSuspend()
    block.startCoroutine(run)
    run.await()
}

private class RunSuspend : Continuation<Unit> {
    override val context: CoroutineContext
        get() = EmptyCoroutineContext

    var result: Result<Unit>? = null

    override fun resumeWith(result: Result<Unit>) = synchronized(this) {
        this.result = result
        @Suppress("PLATFORM_CLASS_MAPPED_TO_KOTLIN") (this as Object).notifyAll()
    }

    fun await() = synchronized(this) {
        while (true) {
            when (val result = this.result) {
                null -> @Suppress("PLATFORM_CLASS_MAPPED_TO_KOTLIN") (this as Object).wait()
                else -> {
                    result.getOrThrow() // throw up failure
                    return
                }
            }
        }
    }
}

```

**说明：**

1. 在入口`main()`函数中使用`runSuspend()`方法运行`main1()`方法，其中将`main1()`函数类型引用强转为`suspend () -> Unit`
2. 函数`runSuspend()`是标准库中的源码函数，该函数被`internal`，没办法直接调用，因此直接将该函数代码从源码中提取出来
3. `runSuspend()`函数，入参为标准的无参返回值为`Unit`的挂起函数`block`，在函数体中创建`RunSuspend`类的对象`run`，调用`block`的`startCoroutine()`方法开启协程，并将`run`作为`completion`传递进去，最后再调用`await()`方法
4. `RunSuspend`类实现接口`Continuation`:
   1. `context`为`EmptyCoroutineContext`
   2. 定义结果属性`result`，设置初始值为`null`，`Result`作为特殊类型，编译器不允许它作为返回值的类型，而声明为`var`的属性有`getter`，所以该行会报错：`'kotlin.Result' cannot be used as a return type`。
   而要解决该报错，只需要在`module`的`build.gradle`文件中`kotlinOptions`中配置如下代码：`freeCompilerArgs = ["-Xallow-result-return-type"]`，如下图所示：
    ![attar](/kotlin学习系列九/coroutine_freecompilerargs.png)
   3. 重写`resumeWith()`方法，将协程返回的结果赋值给属性`result`
   4. 定义`await()`方法，添加`synchronized`关键字保证线程安全，添加一个`while`死循环，在循环体内判断如果`result`为`null`，则调用`object`的`wait()`方法继续等待；而如果`result`不为`null`，则直接将其取到并返回
5. `wait()、notify/notifyAll()`方法延伸：
   1. `wait()`使当前线程阻塞，前提是 **必须先获得锁**，一般配合`synchronized` 关键字使用，即一般在`synchronized` 同步代码块里使用 `wait()、notify/notifyAll()` 方法
   2. 由于 `wait()、notify/notifyAll()` 在`synchronized` 代码块执行，说明当前线程一定是获取了锁
   3. 当线程执行`wait()`方法时候，会释放当前的锁，然后让出`CPU`，进入等待状态。只有当 `notify/notifyAll()` 被执行时候，才会唤醒一个或多个正处于等待状态的线程，然后继续往下执行，直到执行完`synchronized` 代码块的代码或是中途遇到`wait()` ，再次释放锁。
    也就是说，**`notify/notifyAll()` 的执行只是唤醒沉睡的线程，而不会立即释放锁，锁的释放要看代码块的具体执行情况**。所以在编程中，尽量在使用了`notify/notifyAll()` 后立即退出临界区，以唤醒其他线程让其获得锁
   4. `wait()` 需要被`try catch`包围，以便发生异常中断也可以使`wait`等待的线程唤醒
   5. `notify` 和`wait` 的顺序不能错，如果 **A** 线程先执行`notify`方法，**B** 线程在执行`wait`方法，那么 **B** 线程是无法被唤醒的
   6. `notify` 和 `notifyAll`的区别：
      1. `notify`方法只唤醒一个等待（对象的）线程并使该线程开始执行。所以如果有多个线程等待一个对象，这个方法只会唤醒其中一个线程，选择哪个线程取决于操作系统对多线程管理的实现
      2. `notifyAll` 会唤醒所有等待(对象的)线程，尽管哪一个线程将会第一个处理取决于操作系统的实现。如果当前情况下有多个线程需要被唤醒，推荐使用`notifyAll` 方法
6. 如果先调用的`resumeWith()`，再调用的`await()`，则`result`值不会为`null`，则会在`await()`方法中将`result`值抛出去；如果先调用`await()`，再调用`resumeWith()`，此时`result`值为`null`，会调用`wait`方法，将线程阻塞在这里，直到其它线程调用了`resumeWith()`方法，并在该方法中调用了`notifyAll`,将刚才阻塞的线程唤醒，又执行了一次循环，此时`result`不为`null`，如此将`result`值抛出去

# 十、参考文章

1. [kotlin协程的挂起suspend](https://blog.csdn.net/cpcpcp123/article/details/111724079)
2. [Kotlin协程简介](https://www.jianshu.com/p/6e6835573a9c)
3. [Kotlin协程-协程的内部概念Continuation](https://blog.csdn.net/weixin_42063726/article/details/106198212)
4. [AtomicReference源码详解](https://blog.csdn.net/z974656361/article/details/110247463)

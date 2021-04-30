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

# 五、应用--`Generator`与标准库的序列生成器


















































# 十三、参考文章

1. [kotlin协程的挂起suspend](https://blog.csdn.net/cpcpcp123/article/details/111724079)
2. [Kotlin协程简介](https://www.jianshu.com/p/6e6835573a9c)
3. [Kotlin协程-协程的内部概念Continuation](https://blog.csdn.net/weixin_42063726/article/details/106198212)


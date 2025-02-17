# Android面试题

## 前言

> 以下是我整理的Android面试题合集文章。

[Android面试题-Android基础篇](https://mexzt5s81r.feishu.cn/docx/doxcnjhlHildTdcsj8Dxdyu7QFh)

[Android面试题-Java基础&多线程&集合篇](https://mexzt5s81r.feishu.cn/docx/doxcnvI3VwyOdPE7gdVUjV3lmug)

[Android面试题-Kotlin&设计者模式&第三方源码篇](https://mexzt5s81r.feishu.cn/docs/doccnx9wZA3G9I30VCDOg2nGmSa#LBK0pM)

[Android面试题-网络&算法&项目篇](https://mexzt5s81r.feishu.cn/docx/doxcnwwQFMEo0TjSItVgHPZOWgf)

## Kotlin

### 高阶函数

**高阶函数的一个重要特征就是参数类型包含函数，或者该函数的返回值类型是一个函数类型，那么该函数就被称为是高阶函数。**

我们常见的高阶函数有：`let、run、apply、also、use`

### data class

在使用`Java`的时候，我们经常会重写类的`equals()、hashCode()、toString()`方法。这些方法往往都是模板化的。在`Kotlin`中提供了更为简便的方法让我们使用一行代码搞定这些工作。这就是 `data class`。

### sealed class

`sealed class`是一种同时拥有枚举类`enum`和普通类`class`特性的类，叫做密封类。

- 是一个继承结构固定的抽象类，即在编译时已经确定了子类数量，不能在运行时动态新增；
- 它的子类只能声明在同一个包名下；
- 是一个抽象类，且构造方法是私有的，它的孩子是`final`类，而且孩子声明必须嵌套在`sealed class`内部；
- 枚举的局限性：限制枚举每个类型只允许有一个实例，限制所有枚举常量使用相同的类型的值。

### infix

在 Kotlin 中通过 infix 关键字修饰一个函数就表示这个函数是允许使用中缀的形式去调用，简称中缀表达式。

### lateinit、lazy区别

`lateinit`用作非空类型的初始化：

- 在使用前需要初始化；
- 如果使用时没有初始化内部会抛出`UninitializedPropertyAccess Exception`；
- 可配合`isInitialized`在使用前进行检查。

`lazy`用作变量的延迟初始化：

- 定义的时候已经明确了`initializer`函数体；
- 使用的时候才进行初始化，内部默认通过同步锁和双重校验的方式返回持有的实例；
- 还支持设置`lock`对象和其他实现`mode`。

### inline、noinline、crossinline区别

内联就是，调用的函数在编译的时候会变成代码内嵌的形式，好处就是调用栈减少了，坏处就是导致编译生成的字节码膨胀。

`inline`不止可以内联自己的内部代码，还可以内联自己内部的内部的代码，也就是参数中`lambda`代码，`Java`并没有对函数类型的变量的原生支持，`Kotlin`需要想办法来进行兼容，就是用一个新创建的对象来作为函数类型的变量的实际载体，让这个对象去执行实际的代码。因此就产生了一个问题，如果这种函数被放在循环里执行，那么就面临中频繁创建对象的问题，`inline`就是用来解决这个问题的。

- `inline`: 声明在编译时，不仅将函数的代码拷贝到调用的地方(内联)，而且会把它内部的函数类型的参数(`lambda`)也内联过来；
- `noinline`: 声明`inline`函数的形参中，不希望内联的`lambda`；
- `crossinline`: 表明`inline`函数的形参中的`lambda`不能有`return`。

### let、with、run、apply、also区别

| **函数** | **参数个数** | **内部访问上下文对象** | **返回值** |
| :---: | :---: | :---: | :---: |
| `let` | `1` | `it` | `lambda最后一行代码返回值` |
| `with` | `2` | `this 或者 省略` | `lambda最后一行代码返回值` |
| `run` | `1` | `this 或者 省略` | `lambda最后一行代码返回值` |
| `apply` | `1` | `this 或者 省略` | `返回调用者对象` |
| `also` | `1` | `it` | `返回调用者对象` |

### Sequence

- 是惰性的：中间操作不会被执行，只有终端操作才会(如 `toList()` )；
- 计算顺序和迭代器不同：`sequence`是对一个元素应用全部的操作，然后第二个元素应用全部操作，而迭代器是对列表所有元素应用第一个操作，然后对列表所有元素应用第二个操作。

### Kotlin空安全

- 编译成`Java`后是通过`if`判空实现空安全的；
- `Kotlin`和`Java`交互的时候空安全被破坏，可以通过在`Java`代码添加`@NotNull`注解进行非空约束。

### LiveData、Flow、RxJava

`LiveData、Flow、RxJava`三者都属于 **可观察的数据容器类**，**观察者模式**是它们相同的基本设计模式。

**LiveData：**

- **生命周期感知型容器；**
- **LiveData 只能在主线程更新数据；**
- **LiveData 数据重放问题；**
- **LiveData 不防抖：**重复`setValue`相同的值，订阅者会收到多次`onChanged()`回调；
- **LiveData 不支持背压：**在数据生产速度 > 数据消费速度时，`LiveData`无法正常处理。比如在子线程大量`postValue`数据但主线程消费跟不上时，中间就会有一部分数据被忽略。

**RxJava：**

- **功能强大：**支持大量丰富的操作符；
- **支持线程切换、背压**；
- **学习门槛过高**，对开发反而是一种新的负担，也会带来误用的风险。

**Flow：**

- **Flow 支持协程：**`Flow`基于协程基础能力，能够以结构化并发的方式生产和消费数据，能够实现线程切换(使用`flowOn`方法)；
- **Flow 支持背压：**`Flow`的子类`SharedFlow`支持配置缓存容量，可以应对数据生产速度 > 数据消费速度的情况；
- **Flow 支持数据重放配置：**`Flow`的子类`SharedFlow`支持配置重放`replay`，能够自定义对新订阅者重放数据的配置；
- **Flow 相对 RxJava 的学习门槛更低：**`Flow`的功能更精简，学习性价比相对更高。不过`Flow`是基于协程，在协程会有一些学习成本，但这个应该拆分来看。
- **本身不支持生命周期感知：**可以使用`flowWithLifecycle`来实现。

### 冷流、热流

**普通`Flow`(冷流)：** 冷流是**不共享的，也没有缓存机制**。冷流**只有在订阅者**`collect`数据时，才按需执行发射数据流的代码**。**冷流和订阅者是一对一的关系**，多个订阅者间的数据流是相互独立的，一旦订阅者停止监听或者生产代码结束，数据流就自动关闭。 

`SharedFlow / StateFlow`(热流)：热流是共享的，有缓存机制的。无论是否有订阅者`collect`数据，都可以生产数据并且缓存起来。热流和订阅者是一对多的关系，多个订阅者可以共享同一个数据流。当一个订阅者停止监听时，数据流不会自动关闭。 

### 协程

- `Kotlin`官方文档说**本质上协程是轻量级的线程**，其实我觉得是一个**基于线程的上层框架(内部通过封装线程池去实现)，**可以使用同步的形式写出异步的代码，减少了回调嵌套。
- `withContext`配合`suspend`函数。可以切换到指定的线程运行，并在闭包内的逻辑执行结束之后，自动把线程切回去继续执行。
- 协程就是切线程；
- 挂起就是可以自动切回来的切线程；
- 挂起的非阻塞式指的是它能用看起来阻塞的代码写出非阻塞的操作。

**异常处理：**

- 在`CoroutineScope`中，异常是向上传播的，只要任意一个子协程发生异常，整个`Scope`都会执行失败，并且其余的所有子协程都会被取消掉。
- 在`SupervisorScope`中，异常是向下传播的，一个子协程的异常不会影响整个`Scope`的执行，也不会影响其余子协程的执行；重写了`childCancelled`并返回`false`，`CancellationException`异常总是被忽略。

**取消协程：**

- 每个启动的协程都会返回一个`Job`，调用 `Job.cancel()`会让`Job`处于`canceling`状态，然后在下一个挂起点抛出`CancellationException` ,如果协程中没有挂起点，则协程不能被取消。因为每个`suspend`方法都会检查`Job`是否活跃，若不活跃则抛出`CancellationException` ，这个异常只是给上层一次关闭资源的机会，可以通过`try-catch`捕获。
- 对于没有挂起的协程，需要通过`while(isActive)`来检查`Job`是否被取消或者`yield()`。
- 当协程抛出`CancellationException`后，在启动协程将会被忽略。

**suspend关键字：**

**它其实是一个提醒。**用`suspend`修饰的函数就是挂起函数，**挂起函数不会阻塞其所在线程，而是会将协程挂起，在特定的时候才再恢复执行。**

**Dispatcher调度器：**

`Dispatchers`调度器，它可以将协程限制在一个特定的线程执行，或者将它分派到一个线程池。

- `Dispatchers.Main`：`Android`中的主线程；
- `Dispatchers.IO`：针对磁盘和网络`IO`进行了优化，适合`IO`密集型的任务，比如：读写文件，操作数据库以及网络请求；
- `Dispatchers.Default`：适合`CPU`密集型的任务，比如计算。

**背压的概念：**

- 生产速度大于消费速度；
- 使用缓冲区，阻塞队列；
- 缓冲区大小&缓冲区满之后的策略(丢弃最新，最久，挂起)。

## 设计者模式

### 六大基本原则

- 单一职责原则
- 开闭原则
- 里氏替换原则
- 迪米特法则
- 接口隔离原则
- 依赖倒置原则

### 单例模式

- 饿汉式
- 懒汉式
- 双重检查
- 静态内部类
- 枚举

### 工厂模式

- 简单工厂
- 工厂方法
- 抽象工厂

### 代理模式

- 静态代理
- 动态代理

### 建造者模式

- 链式调用

### 策略模式

- 策略接口
- 策略实现
- 策略上下文

### 责任链模式

- 责任链接口
- 责任链实现
- 责任链上下文

### 观察者模式

- 观察者接口
- 观察者实现
- 观察者上下文

### 装饰者模式

- 装饰者接口
- 装饰者实现
- 装饰者上下文

### 适配器模式

- 适配器接口
- 适配器实现
- 适配器上下文

## 第三方源码

### Glide

### OkHttp

### Retrofit

### LeakCanary

- 通过`ActivityLifecycleCallbacks`监听`Activity`生命周期，在`onActivityDestroy`时获取`Activity`实例，并为其构建弱引用并关联引用队列；
- 开启异步线程，观察`ReferenceQueue`是否有`Activity`的弱引用，如果有则说明回收成功，否则回收失败；
- 回收失败后会手动触发一次`gc`，再监听`ReferenceQueue`，如果还是回收失败，则`dump`内存信息；
- `LeakCanary`通过`contentProvider`进行初始化；
- 当一个`Activity`的`onDestroy`方法被执行后，说明该`Activity`的生命周期已经走完，在下次`GC`发生时，该`Activity`对象应将被回收。

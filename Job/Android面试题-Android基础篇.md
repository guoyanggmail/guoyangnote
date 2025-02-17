# Android 面试题

## 前言

以下是我整理的 Android 面试题合集文章。

[Android 面试题-Android 基础篇](https://mexzt5s81r.feishu.cn/docx/doxcnjhlHildTdcsj8Dxdyu7QFh)

[Android面试题-Java基础&多线程&集合篇](https://mexzt5s81r.feishu.cn/docx/doxcnvI3VwyOdPE7gdVUjV3lmug)

[Android面试题-Kotlin&设计者模式&第三方源码篇](https://mexzt5s81r.feishu.cn/docs/doccnx9wZA3G9I30VCDOg2nGmSa#LBK0pM)

[Android面试题-网络&算法&项目篇](https://mexzt5s81r.feishu.cn/docx/doxcnwwQFMEo0TjSItVgHPZOWgf)

## View绘制流程

**简单总结来说就是测量 `measure`、定位 `layout`、绘制 `draw`。**

1. 测量、定位、绘制都是从 `View` 的根结点开始**从顶自下进行**(都是由父控件驱动子控件进行)，**父控件的测量在子控件件测量之后，但父控件的定位和绘制都在子控件之前**;
2. 父控件测量过程中 `ViewGroup.onMeasure()`，会遍历所有子控件并驱动它们测量自己 `View.measure()`;
3. 父控件在完成自己定位之后，会调用 `ViewGroup.onLayout()` 遍历所有子控件并驱动它们定位自己 `View.layout()`，**子控件总是相对于父控件左上角定位**;
4. 控件按照**绘制背景、绘制自身、绘制孩子的顺序**进行。重写 `onDraw()` 定义绘制自身的逻辑，父控件在完成绘制自身之后,会调用 `ViewGroup.dispatchDraw()` 遍历所有子控件并驱动他们绘制自己 `View.draw()`。

**为什么只能在主线程绘制：因为最终绘制都是从 `ViewRootImpl` 开始，它在每次触发View树遍历时都会调用 `ViewRootImpl.checkThread()`，在该方法中会进行线程检查校验。**

## MeasureSpec

`MeasureSpec` 用于在 `View` 测量过程中描述尺寸，它是一个包含了**布局模式和布局尺寸的int值(32位)**，其中**前2位代表布局模式，后30位代表布局尺寸**;

它包含三种布局模式分别是：

1. `UNSPECIFIED`：父布局没有规定尺寸，一般父控件都是 `AdapterView`;
2. `EXACTLY`：父布局设定了尺寸，一般是我们写死了具体数值，或者 `FILL_PARENT/MATCH_PARENT` 时;
3. `AT_MOST`：父布局规定了最大尺寸，一般是 `WRAP_CONTENT`。

## SurfaceView

- 一个嵌入 `View` 树的**独立绘制表面**，他位于宿主 `Window` 的下方，通过在宿主 `canvas` 上绘制透明区域来显示自己;
- 虽然它在界面上隶属于 `view hierarchy`，但在 `WMS` 及 `SurfaceFlinger` 中都是和宿主窗口分离的，它拥有独立的绘制表面，绘制表面在 `app` 进程中表现为 `Surface` 对象，在系统进程中表现为在 `WMS` 中拥有独立的 `WindowState`，在 `SurfaceFlinger` 中拥有独立的 `Layer`，而普通 `view` 和其窗口拥有同一个绘制表面;
- **因为它拥有独立与宿主窗口的绘制表面，所以它独立于主线程的刷新机制**;
- **它的背景是属于宿主窗口的绘制表面，所以如果背景不透明则会盖住它的绘制内容。**

## TextureView

- `SurfaceView` 拥有独立的绘制表面，而 `TextureView` 和 `View` 树共享绘制表面;
- `TextureView` 通过观察 `BufferQueue` 中新的 `SurfaceTexture` 到来，然后调用 `invalidate` 触发 `View` 树重绘，如果有 `View` 叠加在 `TextureView` 上面，它们的脏区有交集，则会触发不必要的重绘，所以他的刷新操作比 `SurfaceView` 更重;
- `TextureView` 持有 `SurfaceTexture`，它是一个 `GPU` 纹理;
- `SurfaceView` 有双缓冲机制，绘制更加流畅;
- `TextureView` 在 `5.0` 之前在主线程绘制，`5.0` 之后在 `RenderThread` 绘制。

## requestLayout、invalidate 区别

> `requestLayout` 方法会导致 `View` 的 `onMeasure、onLayout、onDraw` 方法被调用；`invalidate` 方法则只会导致 `View` 的 `onDraw` 方法被调用;

**相同点**：

- 其实两个函数都会自底向上传递到顶层视图 `ViewRootImpl` 中;
- `requestLayout()` 会添加两个标记位 `PFLAG_FORCE_LAYOUT、PFLAG_INVALIDATED`，而 `invalidate()` 只会添加 `PFLAG_INVALIDATED`(**所以不会测量和布局**);
- `View.measure()` 和 `View.layout()` 会先检查是否有 `PFLAG_FORCE_LAYOUT` 标记，如果有则进行测量和定位;
- `invalidate()` 的过程和 `requestLayout()` 方法很像，但是 `invalidate()` 方法没有标记 `PFLAG_FORCE_LAYOUT`，所以不会执行测量和布局流程，而只是对需要重绘的 `View` 进行重绘，也就是只会调用 `onDraw` 方法，不会调用 `onMeasure` 和 `onLayout` 方法。

## 界面卡顿

刷新率是屏幕每秒钟刷新次数,即每秒钟去 `buffer` 中拿帧数据的次数, 帧率是 `GPU` 每秒准备帧的速度，即 `GPU` 每秒向 `buffer` 写数据的速度。

卡顿是因为掉帧，掉帧是因为写 `buffer` 的速度慢于取 `buffer` 的速度。对于 **60HZ** 的屏幕，每隔 **16.6ms** 显示设备就会去 `buffer` 中取下一帧的内容，没有取到就掉帧了。

- **双缓冲机制**：用两个 `buffer` 存储帧内容，屏幕始终去 `font buffer` 取显示内容，`GPU` 始终向 `back buffer` 存放准备好的下一帧，每个 `Vsync` 发生就是他们交换之时,若下一帧没有准备好,则就错过一次交换,发生掉帧;
- **三缓冲机制**：为了防止耗时的绘制任务浪费一个 `Vsync` 周期，用三个 `buffer` 存储帧。当前显示 `buffer a` 中内容，正在渲染的下一帧存放在 `buffer b` 中，当 `Vsync` 触发时，`buffer b` 中内容还没有渲染好，此时 `buffer a` 不能清除，因为下一帧需要继续显示 `buffer a`，如果没有第三个 `buffer`，`GPU` 和 `CPU` 要白白等待到下一个 `Vsync` 才有可能可以继续渲染后序帧。

## Binder

- **Linux内存空间 = 内核空间(操作系统+驱动)+用户空间(应用程序)，为了保证内核安全，它们是隔离的，内核空间可访问所有内存空间，而用户空间不能访问内核空间**;
- 系统调用主要通过 `copy_to_user()` 和 `copy_from_user()` 实现，`copy_to_user()` 用于将数据从内核空间拷贝到用户空间，`copy_from_user()` 将数据从用户空间拷贝到内核空间;
- `mmap`：将内核空间与接收方用户空间进行内存映射，意思就是将用户空间一块虚拟内存地址和内核空间虚拟内存地址指向同一块物理地址，这样就只需发送方将数据拷贝到内核就等价于拷贝到了接收方的用户空间;
- `Binder` 通信优点：1. 安全性好：为发送方添加 `UID/PID` 身份信息。2. 性能更佳：传输过程只要一次数据拷贝。而 `Socket`、管道等传统 `IPC` 手段都至少需要两次数据拷贝;
- 通过 `Binder` 实现的跨进程通信是 `C/S` 模式的，**客户端通过远程服务的本地代理像服务端发请求，服务端处理请求后将结果返回给客户端**;

## 进程间通信方式

1. `File`：通过读写同一个文件实现数据传递(复杂对象需要序列化)，并发读可能发生数据不是最新的情况，并发写可以破坏数据，适用于对同步要求低的场景;
2. `ContentProvider`：一对多进程的数据共享，支持增删改查;
3. `Bundle`：仅限于跨进程的四大组件间传递数据，且只能传递`Bundle`支持的数据类型;
4. `Messenger`：不支持RPC，低并发串行通信，并发高可能等待时间长;
5. `AIDL`：支持RPC，一对多并发通信;

### **AIDL进程通信流程**

1. 通过 `IInterface` 定义服务后会自动生成 `stub` 和 `proxy`;
2. 服务器通过实现 `stub` 来实现服务逻辑并将其以 `IBinder` 形式返回给客户端;
3. 客户端拿到 `IBinder` 后生成一个本地代理对象(通过 `asInterface()` )，通过代理对象请求服务，代理会构建两个 `Parcel` 对象，一个 `data` 用来传递参数，另一个 `reply` 用来存储服务端处理的结果，并调用 `BinderProxy` 的 `transact()` 发起 `RPC`(Remote Procedure Call)，同时将当前进程挂起。所以如果远程调用很费时，不能在 `UI` 线程中请求服务。
4. 这个请求通过 `Binder` 驱动传递到远程的 `Stub.onTransact()` 调用了服务真正的实现。
5. 返回结果：将返回值写入 `reply parcel` 并返回。

`AIDL` 在生成的 `stub` 中有一个 `asInterface()`，它用于将服务端的 `IBinder` 对象转换成服务接口，这种转化是区分进程的，如果客户端和服务端处于同一个进程中，此方法返回的是服务端 `Stub` 对象本身，否则新建一个远程服务的本地代理。

`AIDL` 中的 `oneway` 表示异步调用，发起 `RPC` 之前不会构建 `Parcel reply`。

`AIDL` 中的 `in` 表示客户端向服务端传送值并不关心值的变化，`out` 表示服务端向客户端返回值。

## Bundle

`Bundle` 主要用于传递数据，它保存的数据，是以 `key-value` 的形式存在的，**传输大小最大限制1MB**。

- 使用 `ArrayMap` 存储结构，省内存，查询速度稍慢，因为是二分查找，适用于小数据量;
- 使用 `Parcelable` 接口实现序列化，而 `HashMap` 使用 `Serializable`。

## Parcel

将各种类型的数据或对象的引用**进行序列化和反序列化，经过 `mmap` 直接写入内核空间**。另一个进程可以直接读取这个内核空间。

**注意项：** `parcel` 存取数据顺序需要保持一致，因为 `parcel` 在一块连续的内存地址，通过首地址+偏移量实现存取。**

## Parcelable、Serializable区别

`Parcelable`：`Android` 专用序列化接口，**优点性能好，效率高(基于内存)，缺点无法本地化，实现复杂**，需要自己重写序列化和反序列化方法(注意序列化和反序列化参数顺序要保持一致)。

`Serializable`：`Java` 自带序列化接口，**优点使用简单，可以本地化(基于硬盘)，缺点效率低**(使用反射机制，通过 `IO` 流写入磁盘)。

## 本地存储方式

1. **SQLite**
2. **文件**
3. `SharedPreference`
    1. **以**`xml`**文件形式存储在磁盘**;
    2. **安全性差：**因为其使用明文存储;
    3. **读写速度慢需要解析`xml`文件，可能造成 `ANR`;
    4. **是线程安全的，但不是进程安全的：**因为调用写操作后会先写入内存中的 `map` 结构，调用 `commit` 或者 `apply` 才会执行写文件操作。
4. **DataStore**
    1. **基于 `Flow`，提供了挂起方法而不是阻塞方法**;
    2. **基于事物更新数据**;
    3. **支持 `protocol buffer`**;
    4. **不支持多进程。**
5. **MMKV**
    1. 主线程同步高效写入存储组建;
    2. 基于`mmap`写入内存，合适时机同步本地化存储;
    3. 可能面临数据丢失问题。

## Handler消息机制

- `Android` 主线程存在一个无限循环，该循环在不停地**从消息队列中取消息**。消息是**按时间先后顺序**插入到链表结构的消息队列中，最旧的消息在队头，最新的消息在队尾;
- `Looper` 通过无限循环从消息队列中取消息并分发给其对应的 `Handler`，并回收消息;
- `Handler` 是用于串行通信的工具类;
- 消息池：链式结构，静态，所有消息共用，取消息时从头部取，消息分发完毕后头部插入;
- `Android` 消息机制共有三种消息处理方式，它们是互斥的，优先级从高到低分别是 `Runnable.run()-> Handler.callback->Handler.handleMessage()`;
- 若消息队列中消息分发完毕,则调用 `natviePollOnce()` 阻塞当前线程并释放 `cpu` 资源,当有新消息插入时或者超时时间到时线程被唤醒;
- `IdleHandler` 是在消息队列空闲时会被执行的逻辑，每拿取一次消息有且仅有一次机会执行，通过 `queueIdle()` 返回 `true` 表示每次取消息时都会执行,否则执行一次就会被移出;
- **同步消息屏障是一种特殊的同步消息**，他的 `target` 为 `null`，在 `MessageQueue.next()` 中遇到该消息,则会遍历消息队列优先执行所有异步消息,若遍历到队列尾部还是没有异步消息,则阻塞会调用 `epoll` ,直到异步消息到来或者同步屏障被移出;
- **使用 `epoll` 实现消息队列的阻塞和唤醒**，`Message.next()` 是个无限循环，若当前无消息可处理会阻塞在 `nativePollOnce()`，若有延迟消息，则设置超时，没有消息时主线程休眠不会占用 `cpu`;
- `epoll` 是一个 `IO` 事件通知机制，监听多个文件描述符上的事件，`epoll` 通过使用红黑树搜索被监控的文件描述符(内核维护的文件打开表的索引值)；
- `epoll` 最终是在 `epoll_wait` 上阻塞；
- `nativeWake()` 唤醒是通过往管道中写入数据，`epoll` 监听写入事件，`epoll_wait()` 就返回了。

## 同步消息屏障

**总结来说是`Handler`消息分发时优先处理发送的异步消息。**

**同步屏障**也是一个 `Message`，只不过 `target=null`。

`MessageQueue.next()` 取下一条 `message` 的方法中，若遇到同步屏障，则会越过同步消息，向后遍历找第一条异步消息找到则返回，若没有找到则会执行 `epoll` 挂起。

1. `ViewRootImpl` 将遍历 `view` 树包装成一个 `Runnable` 并抛到 `Choreographer`，在抛之前会向主线程消息队列中抛出**同步屏障消息**。
2. `MessageQueue` 轮询消息时，会优先执行 `ViewRootImpl` 发送的**同步屏障消息。**
3. 当执行到遍历 `view` 树的 `runnable` 时，`ViewRootImpl` 会移除同步屏障。

## 触摸事件

- 触摸事件的传递是从根视图自顶向下的过程，触摸事件的消费是自下而上的过程。
- 触摸事件由 `ViewRootImpl` 通过 `ViewRootHandler` 接收到，再分发给 `Activity` 接收到触摸事件后，会传递给 `PhoneWindow`，再传递给 `DecorView`，由 `DecorView` 调用 `ViewGroup.dispatchTouchEvent()` 自顶向下分发 `ACTION_DOWN` 触摸事件。
- `View.dispatchTouchEvent()` 是传递事件的终点，消费事件的起点。它会调用 `onTouchEvent()` 或 `OnTouchListener.onTouch()` 来消费事件。
- 每个层次都可以通过在 `onTouchEvent()` 或 `OnTouchListener.onTouch()` 返回 `true`，来告诉自己的父控件触摸事件被消费。只有当下层控件不消费触摸事件时，其父控件才有机会自己消费。
- `ACTION_MOVE` 和 `ACTION_UP` 会沿着刚才 `ACTION_DOWN` 的传递路径，传递给消费了 `ACTION_DOWN` 的控件，如果该控件没有声明消费这些后序事件，则它们也像 `ACTION_DOWN` 一样会向上回溯让其父控件消费。
- 父控件可以通过在 `onInterceptTouchEvent()` 返回 `true` 来拦截事件向其孩子传递。如果在孩子已经消费了 `ACTION_DOWN` 事情后才进行拦截，父控件会发送 `ACTION_CANCEL` 给孩子。

## 滑动冲突

1. **父控件主动**：父控件只拦截自己滑动方向上的事件，通过重写 `onInterceptTouchEvent()` 方法，在 `ACTION_MOVE` 事件中根据判断自身是否需要消费事件返回 `true/false` 实现，其余事件不拦截继续传递给子控件；
2. **子控件主动**：重写子控件的 `dispatchTouchEvent()` 方法，在 `ACTION_DOWN` 事件中调用 `getParent().requestDisallowInterceptTouchEvent(true)` 方法告知父控件不能拦截，`ACTION_MOVE` 事件中根据判断父控件是否需要消费事件，通过 `getParent().requestDisallowInterceptTouchEvent(true/false)` 来实现，同时父控件需要重写 `onInterceptTouchEvent()` 方法，在 `ACTION_DOWN` 事件中返回 `false` 不能拦截事件，这是因为在 `ACTION_DOWN` 事件时会初始化一些状态和标志位等变量导致 `requestDisllowInterceptTouchEvent(boolean)` 方法会失效。因此，在外层控件的 `ACTION_DOWN` 事件不能拦截。

## RecyclerView

`Recycler`有4个层次用于缓存`ViewHolder`对象，优先级从高到底依次为`ArrayList<ViewHolder> mAttachedScrap`、`ArrayList<ViewHolder> mCachedViews`、`ViewCacheExtension mViewCacheExtension`、`RecycledViewPool mRecyclerPool`。如果四层缓存都未命中，则重新创建并绑定`ViewHolder`对象。

- **缓存性能：**

| **缓存** | **缓存结构** | **重新创建ViewHolder** | **重新绑定数据** |
| --- | --- | --- | --- |
| mAttachedScrap | ArrayList | FALSE | FALSE |
| mCachedViews | ArrayList | FALSE | FALSE |
| mRecyclerPool | SparseArray | FALSE | TRUE |

- **缓存容量：**
    1. `mAttachedScrap`：没有大小限制，但最多包含屏幕可见表项。
    2. `mCachedViews`：默认大小限制为2，放不下时，按照先进先出原则将最先进入的`ViewHolder`存入回收池以腾出空间。
    3. `mRecyclerPool`：对`ViewHolder`按`viewType`分类存储（通过`SparseArray`），同类`ViewHolder`存储在默认大小为5的`ArrayList`中。
- **缓存用途：**
    1. `mAttachedScrap`：用于布局过程中屏幕可见表项的回收和复用。
    2. `mCachedViews`：用于移出屏幕表项的回收和复用，且只能用于指定位置的表项，有点像“回收池预备队列”，即总是先回收到`mCachedViews`，当它放不下的时候，按照先进先出原则将最先进入的`ViewHolder`存入回收池。
    3. `mRecyclerPool`：用于移出屏幕表项的回收和复用，且只能用于指定`viewType`的表项。

## ItemDecoration

用于绘制`ItemView`以外的内容，有两个回调`onDraw`和`onDrawOver`，区别是绘制的顺序有先后，`onDraw()`会在`RecyclerView.onDraw()`中调用，表示绘制`RecyclerView`自身的内容，会在绘制孩子之前，所以出现在孩子下面，而`onDrawOver()`是在`RecyclerView.draw()`中调用，在绘制孩子之后调用，所以会出现在孩子上方。

## ~~View生命周期~~

构造`View->onFinishInflate->onAttachedToWindow->onMeasure->onSizeChanged->onLayout->onDraw->onDetachedFromWindow`。

## Bitmap

`BitmapFactory.Options`
    - `inPreferredConfig`：每个像素占用几个字节

只有当图片是`webp`或者`png`的时候，`inPreferredConfig`才会有效果。

- **ALPHA_8：**图片只有alpha值，没有RGB值，一个像素占用一个字节
- **RGB_565：**2字节
- **ARGB_8888：**一个像素占用4个字节
- `inSampleSize`：为2的幂次，表示压缩宽高为原来的1/2，像素密度不变，按需加载，按`ImageView`大小加载，计算`inSampleSize`的算法是，原始宽高不停除2，`inSampleSize`不停乘2，直到原始宽高小于需求宽高。
- `Bitmap`大小计算：长*宽*像素大小*缩放比例
- 图片的原始宽高
- 解码图片时的`Config`配置(即每个像素占用几个字节，**ALPHA_8占1字节、RGB_565占2字节、ARGB_8888占4字节**)；
- 解码图片时的**缩放比例=设备分辨率 / 资源目录分辨率**，即`scale=inTargetDensity/inDensity`，如：1080x1920的图片显示`xhdpi`中的图片，`scale=480/320=1.5`，图片的宽高会乘以`scale`；

## 图片格式类型

- `jpg`格式：色彩丰富，没有兼容性问题，不支持透明度，和动画，适用于摄影作品。jpg在高对比度的场景下效果不好，比如黑色文字在白色的背景上；
- `png`格式：包括透明度适用于图标，因为大面积的重复颜色，适用于无损压缩；
- `webp`格式：包括有损和无损两个方式，`webp`的浏览器支持不佳，`webp`支持动画和透明度，之前动画只能用`gif`，透明度只能选`png`，**有损压缩后的`webp`的解码速度慢，比`gif`慢2倍。**
- `raw、drawable、sdcard`，这些不带`dpi`的文件夹中的图片被解析时不会进行缩放(`inDensity`默认为160)；
- 获取`Bitmap`宽高：若`inJustDecodeBounds`为`true`，则不会把`Bitmap`图片的像素加载到内存(实际是在`Native`层解码了图片，但是没有生成`Java`层的`Bitmap`)，只是获取该`Bitmap`的原始宽(`outWidth`)和高(`outHeight`)；
- `Bitmap.recycle()`：只是释放图片`native`对象的内存，并且去除图片像素数据的引用，让图片像素数据可以被垃圾回收；
- `BitmapRegionDecoder`用于图片分块加载。

## ANR

- `KeyDispatchTimeout`：`View`的点击事件或者触摸事件在特定的时间(5s)内无法得到响应；
- `BroadcastTimeout`：广播`onReceive()`函数运行在主线程中，在特定的时间(10s)内无法完成处理；
- `ServiceTimeout`：`Service`的各个生命周期函数在特定时间(20s)内无法完成处理。

## 恢复数据

1. `onSaveInstanceState()+onRestoreInstanceState()`：会进行序列化到磁盘，耗时，杀进程依然存在；
2. `Fragment+setRetainInstance()`：数据保存在内存，配置发生变化时数据依然存在，但杀进程后数据不存在；
3. `onRetainNonConfigurationInstance() + getLastNonConfigurationInstance()`:数据保存在内存，配置发生变化时数据依然存在，但杀进程后数据不存在，`ViewModel`能够在`Activity/Fragment`重新构建后保存数据就是通过这个原理。

## 进程优先级

一共有五个进程优先级

1. **前台进程：**
    1. 正在交互的`Activity`，`Activity.onResume()`
    2. 前台服务
    3. `Service.onCreate()、onStart()`正在执行
    4. `Receiver.onReceive()`正在执行
2. **可见进程：**
    1. 正在交互的`Activity`，`Activity.onPause()`
3. **服务进程：**
    1. 后台服务
4. **后台进程：**
    1. 不可见`Activity`，`Activity.onStop()`
    2. 后台进程优先级等于后台服务，所以长时间后台任务最后其服务
5. **空进程：**

不包含任何组件的进程，`Activity`在退出的时候进程不会销毁，会保留一个空进程方便以后启动. 但在内存不足时进程会被销毁。

按下返回键退出应用，此时应用进程变成缓存进程，随时可能被杀掉；

按下`home`键退出应用，此时应用是不可见进程。

## LruCache

- `LruCache`是内存缓存，采用最近最少使用原则，持有一个`LinkedHashMap`实例；
- `LruCache`用`LinkedHashMap`作为存储结构，且`LinkedHashMap`按访问顺序排序，最新的结点在尾部，最老的结点在头部。

## DiskLruCache

- 内部有一个线程池(只有一个线程)用于清理缓存；
- 内部有一个`LinkedHashMap`结构代表内存中的缓存，键是`key`，值是`Entry`实体(`key+file`文件)；
- 有一个`journal`文件缓存操作的日志文件，构造时会读取日志文件并将其转换成`LinkedHashMap`存储在内存；
- 取缓存时，先读取内存中的`Entry`，然后将其转换成`Snapshot`对象，可以从`Snapshot`中拿到输入流；
- 写缓存时，新建`Entry`实体并存在`LinkedHashMap`中，将其转换成`Editor`对象，可以从中拿到输出流；
- 通过`LinkedHashMap`实现`LRU`替换；
- 每一个`Cache`项有四个文件，两个状态(DIRTY,CLEAN),每个状态对应两个文件：一个文件存储`Cache meta`数据，一个文件存储`Cache`内容数据。

## App启动流程

冷启动就是在 `Launcher` 进程中开启用户`App`的过程。这是一个`Launcher`进程、`AMS`、`WMS`、应用进程通信的过程：

- `Launcher`进程和`AMS`说：“我要启动`Activity1`”
- `AMS`创建出`Activity1`对应的`ActivityRecord`以及`TaskRecord`，通知`Launcher`进程执行`onPause()`，`Launcher`执行`onPause()`，并告知`AMS`。
- 启动一个新窗口，`AMS`请求`zygote`进程`fork`一个新进程。
- 在新进程中，构建`ActivityThread`，并调用`main()`，在其中开启**主线程消息循环**。
- `AMS`开始回调`Activity1`的生命周期方法。
- 当执行到`Activity.onAttch()`时，`PhoneWindow` 被构建。
- 当执行到`Activity.onCreate()`时，`setContentView()`会被委托给`PhoneWindow`，并在其中构建`DecorView`。
- 当执行到`Activity.onResume()`时，`DecorView`先被设置为`invisible`，然后将其添加到窗口，此过程中会构建`ViewRootImpl`对象，`ViewRootImpl.requestLayout()`会被调用，**以触发**`**<font style="color:rgb(216,57,49);">View</font>**`**的绘制**。
- `View`树遍历，会被包装成一个任务抛给`Choreographer`，在此之前`ViewRootImpl`会向主线程消息队列抛一个**同步消息屏障(异步消息)，**以达到优先遍历异步消息的效果。
- **异步消息**的执行，即是从顶层视图开始，自顶向下，逐个视图进行`**<font style="color:rgb(216,57,49);">measure，layout，draw</font>**`的过程。
- 当`DecorView`完成渲染后，就会被设置为`visible`，界面展示出来。

## Gradle构建生命周期

Gradle 将构建划分为三个阶段： **初始化->配置->执行** 。

- **初始化阶段**

1. **执行 `Init` 脚本**
2. **实例化 `Settings` 接口实例：** 解析根目录下的 `settings.gradle` 文件，并实例化一个`Settings`接口实例；
3. **实例化 `Project` 接口实例：** `Gradle`会解析`include`声明的模块，并为每个模块`build.gradle`文件实例化`Project`接口实例。

- **配置阶段**

1. **下载插件和依赖：**`Project`通常需要依赖其他插件或`Project`来完成工作，如果有需要先下载；
2. **执行脚本代码：**在`build.gradle`文件中的代码会在配置阶段执行；
3. **构造 Task DAG：** 根据`Task`的依赖关系构造一个有向无环图，以便在执行阶段按照依赖关系执行`Task`。

- **执行阶段**

1. `Task`配置代码在配置阶段执行，而`Task`动作在执行阶段执行；
2. 即使执行一个`Task`，整个工程的初始化阶段和所有`Project`的配置阶段也都会执行，这是为了支持执行过程中访问构建模型的任何部分。

## Gradle生命周期监听

Gradle 提供了一系列监听构建生命周期流程的接口，大部分的节点都有直接的`Hook`点。

**监听初始化阶段、监听配置阶段、监听执行阶段。**

## 编译打包流程

1. **打包资源文件**：`res`下文件转换成二进制、`asset`目录下的资源；
2. **编译`java`文件为`class`文件**；
3. **将`class`文件转换为`dex`文件**；
4. **将资源和`dex`打包到`apk`**；
5. **签名**。

## 动画

1. **补间动画**
    1. 局限于`view`，只支持位移、旋转、透明度、缩放；
    2. 不能监听动画变化的过程；
    3. 响应**原有位置**的触摸事件；
2. **属性动画**
    1. 不仅仅适用于`view`，可用于任何提供了`getter`和`setter`的对象的任何属性上；
    2. 可监听动画变化过程；
    3. `Interpolator`决定值变化的速度；
    4. 若是硬件加速，则直接修改`RenderNode`中的相关属性，不需要重新构建`DisplayList`，若是软件绘制，则会触发`invalidate`。
3. **帧动画**
    1. 就像连环画一样，一张张图片连续播放
    2. `AnimationDrawable`会在动画播放之前将所有帧都加在到内存，**缺点耗内存**。

**播放10张1MB图片的帧动画优化方案**：**可以使用`SurfaceView`突破这个限制，而且它可以将计算帧数据放到独立的线程中进行**。用`HandlerThread`作为独立帧绘制线程，好处是可以通过与其绑定的`Handler`方便地实现“每隔一段时间刷新”，而且在`Surface`被销毁的时候可以方便的调用`HandlerThread.quit()`来结束线程执行的逻辑。

## Lifecycle

- **让任何组件可以作为观察者观察界面生命周期；**
- 通过`LifecycleRegistry`，它持有所有观察者，通过注册`ActivityLifecycleCallbacks`实现生命周期的分发，如果是`Android 29`以下则将`ReportFragment`添加到`Activity`中；
- 监听应用前后台切换：通过`registerActivityLifecycleCallbacks`，然后在维护一个活跃`activity`的数量，`ProcessLifecycleOwner`为我们做了这件事情 用于监听应用前后台切换，`ProcessLifecycleOwner`的初始化通过`ContentProvider`实现。

## LiveData

- **自动绑定生命周期：**`LiveData`的数据观察者在内部被包装成另一个对象(实现了`LifecycleEventObserver`接口)，它同时具备了数据观察能力和生命周期观察能力；
- **页面活跃时更新数据：**`LiveData`内部会将数据观察者进行封装，使其具备生命周期感知能力。当页面可见时，才会通知数据的更新，当生命周期状态为 `DESTROYED`时，自动移除观察者；
- **粘性问题：**`LiveData`的观察者会维护一个“值的版本号”，用于判断上次分发的值是否是最新值，“新观察者”被“老值”通知的现象叫“粘性”。因为新观察者的版本号总是小于最新版号，且添加观察者时会触发一次老值的分发；
- **数据丢失问题：**在高频数据更新的场景下使用`LiveData.postValue()`时，会造成数据丢失。因为“设值”和“分发值”是分开执行的，之间存在延迟。值先被缓存在变量中，再向主线程抛一个分发值的任务。若在这延迟之间再一次调用`postValue()`，则变量中缓存的值被更新，之前的值在没有被分发之前就被擦除了。

## ViewModel

- `ViewModel`实例被存储在`ViewModelStore`的`map`中, 在配置发生变化时`onRetainNonConfigurationInstance`会被调用(`ViewModelStore`的`map`对象会被存储在`NonConfigurationInstances`中)，并会返回这个对象.在恢复`ViewModel`时再`getLastNonConfigurationInstance`中再次获取；
- `Activity`实现了`LifecycleOwner`，等`onDestroy`时会尝试(非配置变化时)调用`store`的`clear`(遍历了`Viewmodel.clear()`)
- `ViewModel`在`Fragment`中不会因配置改变而销毁的原因其实是因为其声明的`ViewModel`是存储在`FragmentManagerViewModel`中的，而`FragmentManagerViewModel`是存储在宿主`Activity`中的`ViewModelStore`中，又因`Activity`中`ViewModelStore`不会因配置改变而销毁，故`Fragment`中`ViewModel`也不会因配置改变而销毁。

## ConstraintLayout 性能

**使用`ConstraintLayout`布局可以减少层级嵌套。**

**原因**：因为`ConstraintLayout`会进行额外的封装，会将容器控件包装成`ConstraintWidgetContainer`，子控件包装成`ConstraintWidget`。在测量布局的时候，会先把所有的`ConstraintWidget`移除，然后遍历所有子控件并重新添加`ConstraintWidget`，如果子控件有约束则将其连接到锚点，构建依赖图，深度遍历依赖图，进行求解(`Cassowary`算法)，最终得到相对于父布局的上下左右。

## Service

- **分类：**

  - `start service`
        1. 生命周期和启动他的组件无关，必须显示调用`stopservice`才能停止。
  - `bind service`
        1. 生命周期和启动他的组件绑定，组件都销毁了，他也销毁。
- `service`默认是后台进程
- `Service`和`Activity`通信
  - 通过`Intent`传入`startService`；
  - 发送广播，或者本地广播；
  - `bind service`，通过方法调用通信。

## HandlerThread

`HandlerThread`本质上就是一个`Thread`，只不过内部已经帮我们开启了一个`Looper`循环，我们可以多次通过`Handler`发消息到`MessageQueue`中，`Handler`会通过`Looper`的循环器将消息传递到`handleMessage`中，这个循环器将会始终开启着，所以我们不使用的时候一定要及时关闭。

- `start()->run()`：在该方法中先调用`Looper.prepare()`初始化，然后调用`Looper.loop()`开启循环；
- `HandlerThread.quit()、HandlerThread.quitSafely()`：关闭`Looper`的循环，这两个的区别就是前者直接停止，后者是等任务执行完在停止。

## IntentService

他是`Service`和消息机制的结合，它适用于后台串行处理一连串任务，任务执行完毕后会自销毁。

- 它启动时会创建`HandlerThread`和`Handler`，并将`HandlerThread`的`Looper`和`Handler`绑定，每次调用`startService()`时，通过`Handler`发送消息到新线程执行；
- 但是它没有将处理结果返回到主线程，需要自己实现(可以通过本地广播)。

## 广播

- 是一种用观察者模式实现的异步通信；
- 有**静态注册和动态注册两种方式**，静态注册生命周期比动态注册长(在应用安装后就处于监听状态)：
  - 静态注册广播接收器时只要定义了`intent-filter`则`android:exported`属性为`true`表示该广播接收器是跨进程的；
  - 静态注册广播容易被攻击，其他`App`可能会针对性的发出与当前`App intent-filter`相匹配的广播，由此导致当前`App`不断接收到广播并处理；
  - 静态注册广播容易被劫持：其他`App`可以注册与当前`App`一致的`intent-filter`用于接收广播，获取广播具体信息；
  - 通过`manifest`注册的广播是静态广播；
  - 静态广播是常驻广播，常驻广播在应用退出后依然可以收到；
  - 动态注册的不是常驻广播，它的生命周期同注册组件一致；
  - 动态注册要等到启动他的组件启动时时才注册；
  - 动态广播优先级比静态高。
- 广播接收者`BroadcastReceiver`通过`Binder`机制向`AMS`进行注册，广播发送者通过`binder`机制向`AMS`发送广播，广播的`Intent`和`Receiver`会被包装在`BroadcastRecord`中，多个`BroadcastRecord`组成队列；
- `onReceive()`回调执行时间超过10s会发生`ANR`，如果有耗时操作需要使用`IntentService`处理，不建议新建线程；
- **分为有序广播和无序广播**：
  - **无序广播**：所有广播接收器接受广播的先后顺序不确定，广播接收者无法阻止广播发送给其他接收者；
    - **有序广播**：广播接收器收到广播的顺序按预先定义的优先级从高到低排列 接收完了如果没有丢弃，就下传给下一个次高优先级别的广播接收器进行处理，依次类推，直到最后。
- **还可以分为本地广播和跨进程广播**：
  - 本地广播仅限于在应用内发送广播，向`LocalBroadCastManager`注册的接收器都存放在本地内存中，跨进程广播都注册到`system_server`进程。

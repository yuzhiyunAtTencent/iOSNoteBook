# 学习资料

## 大神博客

YYCache 作者 <https://blog.ibireme.com/>



# 语言层面

## 编译链接

从源代码到可执行文件全过程：

<https://juejin.im/post/5c22eaf1f265da611b5863b2#heading-9>

源代码 》 预处理 〉 词法分析 》语法分析(AST) 〉生成汇编代码 》 生成机器码输出.o文件 〉linker链接 》Mach-o 可执行文件

### 静态库、动态库链接

链接的共用库分为静态库和动态库：静态库是编译时链接的库，本质就是一组.o文件，需要链接进你的 Mach-O 文件里，如果需要更新就要重新编译一次，无法动态加载和更新；而动态库是运行时链接的库，使用 dyld 就可以实现动态加载。

### 链接是什么意思？

把符号和内存地址进行绑定，符号简而言之就是变量名、函数名

## 源代码阅读

<https://opensource.apple.com/tarballs/objc4/>

在这里下载一个，用xcode 打开，可以看到 runtime的代码，还有比如 NSObject.mm 文件



<https://opensource.apple.com/tarballs/CF/> runloop代码

## 查看oc转换为c代码

```powershell
clang -rewrite-objc main.m
```



## 栈帧

## 内存模型

## property

###使用 assign 修饰 oc 对象导致crash

```objective-c
@property(nonatomic, assign) News *crashNews;
@end

@implementation NewsListTableViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    // 编译器会提示 warning: Assigning retained object to unsafe property; object will be released after assignment
    self.crashNews = [[News alloc] initWithPicUrl:@"oldUrl" title:@"oldTitle"];
  
  ...
    
    //然后在tableview的 didSelect中访问就会crash
 - (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath{
    NSLog(@"self.crashNews = %@", self.crashNews);
}
  
  //根据提示，不会持有对象，赋值后立刻被释放，但是指针不像weak会自动变成nil，后续访问就会crash
  //EXC_BAD_ACCESS (code=1, address=0x3f067b1c0)
```



## weak原理

维护了一个hashmap,key是对象的地址，value是指针数组，在对象销毁的时候，会遍历value数组把所有指针设置为nil

##block原理

### _ _block 的原理

## NSAutoreleasePool

官方文档说明了，每次runloop开始时会创建NSAutoreleasePool，runloop结束时释放NSAutoreleasePool


```objective-c
The Application Kit creates an autorelease pool on the main thread at the beginning of every cycle of the event loop, and drains it at the end, thereby releasing any autoreleased objects generated while processing an event.
```

一个线程的releasepool就是一个指针stack

```objective-c
/***********************************************************************
   Autorelease pool implementation

   A thread's autorelease pool is a stack of pointers. 
   Each pointer is either an object to release, or POOL_BOUNDARY which is 
     an autorelease pool boundary.
   A pool token is a pointer to the POOL_BOUNDARY for that pool. When 
     the pool is popped, every object hotter than the sentinel is released.
   The stack is divided into a doubly-linked list of pages. Pages are added 
     and deleted as necessary. 
   Thread-local storage points to the hot page, where newly autoreleased 
     objects are stored. 
**********************************************************************/


​```objective-c
The Application Kit creates an autorelease pool on the main thread at the beginning of every cycle of the event loop, and drains it at the end, thereby releasing any autoreleased objects generated while processing an event.

```

一个线程的releasepool就是一个指针stack

```objective-c
/***********************************************************************
   Autorelease pool implementation

   A thread's autorelease pool is a stack of pointers. 
   Each pointer is either an object to release, or POOL_BOUNDARY which is 
     an autorelease pool boundary.
   A pool token is a pointer to the POOL_BOUNDARY for that pool. When 
     the pool is popped, every object hotter than the sentinel is released.
   The stack is divided into a doubly-linked list of pages. Pages are added 
     and deleted as necessary. 
   Thread-local storage points to the hot page, where newly autoreleased 
     objects are stored. 
**********************************************************************/
```



## Associated Object


## Associated Object

关联对象 并不是直接保存在 主对象内部的 

关联对象由 AssociationsManager 来管理 

关联对象全部保存在一个全局的 HashMap 里面 

key是 对象 value是 另一个 HashMap，这个 map 存着这个对象的所有关联对象（设置关联对象的时候需要一个key值） 



![image-20190627201924550](/Users/zhiyunyu/Library/Application Support/typora-user-images/image-20190627201924550.png)



### dealloc block

主对象 dealloc的时候会调用一个函数开始销毁 associated object,因此我们也可以做一个工具，这个工具使得我们可以给一个对象追加一个在dealloc 时候执行的  block   

工具也不复杂，创建一个 NSObject 的分类，分类中给 NSObject 添加一个关联对象，然后把 block扔给管理对象的 dealloc去执行。 



## Category

分类的原理是什么？

### extension 和 category区别

extension可以添加实例变量，而category是无法添加实例变量的（因为在运行期，对象的内存布局已经确定，如果添加实例变量就会破坏类的内部布局，这对编译型语言来说是灾难性的）

extension是编译时决议的，他就是类的一部分，和@interface 以及 @implement共同组成一个完整的类，一般来说私有属性会放在这里。

category是运行时决议的，在runtime时期，方法列表会插入主类方法列表的头部。

需要注意的有两点： 

1)、category的方法没有“完全替换掉”原来类已经有的方法，也就是说如果category和原来类都有methodA，那么category附加完成之后，类的方法列表里会有两个methodA 

2)、category的方法被放到了新方法列表的前面，而原来类的方法被放到了新方法列表的后面，这也就是我们平常所说的category的方法会“覆盖”掉原来类的同名方法，这是因为运行时在查找方法的时候是顺着方法列表的顺序查找的，它只要一找到对应名字的方法，就会罢休^_^，殊不知后面可能还有一样名字的方法。

## KVO & KVC



```objective-c
News *news = [[News alloc] initWithPicUrl:@"oldUrl" title:@"oldTitle"];
    
    NSLog(@"%p", [news methodForSelector:@selector(setTitle:)]);
    
    [news addObserver:self forKeyPath:@"title" options:NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld context:nil];
    
    NSLog(@"%p", [news methodForSelector:@selector(setTitle:)]);
    
    [news willChangeValueForKey:@"title"];
    [news didChangeValueForKey:@"title"];

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context {
    NSLog(@"old = %@ , new = %@ ",[change objectForKey:NSKeyValueChangeOldKey], [change objectForKey:NSKeyValueChangeNewKey]);
}
```



通过手动调用  willChangeValueForKey 和  didChangeValueForKey可以触发  observeValueForKeyPath ，两者缺一不可。

## 锁（线程安全）

这里讲的非常好

<http://mrpeak.cn/blog/ios-thread-safety/>

所以总结来说，atomic 只能保证。getter setter自身的安全，并不可以保证整个业务的线程安全，加锁操作应该在业务层完成。

建议再看看 leveldb 是如何控制线程安全的。



## @synchronized

本质上底层维护了一个互斥锁数组，通过object的地址哈希值获取对应的锁。

下方代码 SyncData 中的object 就是我们填进@synchronized的对象， mutex 就是作用于该对象的互斥锁。

从recursive_mutex_t可以看出@synchronized的锁是一个递归互斥锁，可重入

```objective-c
typedef struct alignas(CacheLineSize) SyncData {
    struct SyncData* nextData;
    DisguisedPtr<objc_object> object;
    int32_t threadCount;  // number of THREADS using this block
    recursive_mutex_t mutex;
} SyncData;

int objc_sync_enter(id obj) {
  //找到对象对应的锁，然后上锁
}

int objc_sync_exit(id obj) {
  //找到对象对应的锁，然后解锁
}
```



## pthread_mutex 互斥锁

一个线程获取锁之后，其他线程休眠等待。 并且这个获取了锁的线程，不能再次获取锁，比如这个函数是一个递归函数，那么再次获取锁就会导致死锁。（这也因此引入了递归锁）

NSLock内部封装的就是pthread_mutex，只不过他是oc类，需要经过消息转发

```objective-c
pthread_mutex_t _lock;

pthread_mutex_lock(&_lock);
//do something
pthread_mutex_unlock(&_lock);
```

## 递归锁（可重入锁）

pthread_mutex 可以设置类型PTHREAD_MUTEX_RECURSIVE 就可以支持递归锁。

同一个线程获取锁之后，可以再次获取锁，而不是发生死锁。

其实@synchronized就是一个可重入锁。

NSRecursiveLock内部封装的同样是pthread_mutex，类型是PTHREAD_MUTEX_RECURSIVE

## 自旋锁

### 和互斥锁的区别：

互斥锁的等待线程会进入休眠状态，锁释放后之后被唤醒，

自旋锁的等待线程不会进入休眠状态，而是一直循环检测锁是否可用，busy_waiting，由于状态不变这样就避免了上下文切换，但是一直做着无意义的等待消耗着cpu资源，所以自旋锁通常使用的场景要求持有锁的时间尽量短。

### 含义

一个线程获取锁之后，其他线程会处于忙等状态，一直循环去检查能不能获取到锁。在新版iOS中OSSpinLock已经被弃用，存在优先级反转问题。

具体来说，如果一个低优先级的线程获得锁并访问共享资源，这时一个高优先级的线程也尝试获得这个锁，它会处于 spin lock 的忙等状态从而占用大量 CPU。此时低优先级线程无法与高优先级线程争夺 CPU 时间，从而导致任务迟迟完不成、无法释放 lock。

atomic就是用一个自旋锁控制，还是osspinlock, 但是atomic意义不大，无法实现线程安全，理论上在iOS层面，可以不考虑自旋锁这种东西。

## dispatch_semaphore信号量

四个停车位用于停车，如果有5辆，就有一辆需要等待，这种锁适用于这种特殊业务场景。

## 条件锁

以后总结吧

## 锁的总结

实际开发中，并不是总要选择性能最好的锁来实现，需要根据业务需求和开发成本，代码维护等方面综合选择，这也是@synchronized和NSLock的原因。





## runloop

<https://blog.ibireme.com/2015/05/18/runloop/>

### mode

让一次 runloop 只处理当前 mode 相关的事情（timer source observer ）

### 触摸等事件响应

苹果注册了一个 Source1 (基于 mach port 的) 用来接收系统事件，其回调函数为 __IOHIDEventSystemClientQueueCallback()。

当一个硬件事件(触摸/锁屏/摇晃等)发生后，首先由 IOKit.framework 生成一个 IOHIDEvent 事件并由 SpringBoard 接收。这个过程的详细情况可以参考[这里](http://iphonedevwiki.net/index.php/IOHIDFamily)。SpringBoard 只接收按键(锁屏/静音等)，触摸，加速，接近传感器等几种 Event，随后用 mach port 转发给需要的App进程。随后苹果注册的那个 Source1 就会触发回调，并调用 _UIApplicationHandleEventQueue() 进行应用内部的分发。

_UIApplicationHandleEventQueue() 会把 IOHIDEvent 处理并包装成 UIEvent 进行处理或分发，其中包括识别 UIGesture/处理屏幕旋转/发送给 UIWindow 等。通常事件比如 UIButton 点击、touchesBegin/Move/End/Cancel 事件都是在这个回调中完成的。

### 利用runloop监控卡顿

<https://github.com/ming1016/DecoupleDemo/blob/master/DecoupleDemo/SMLagMonitor.m>

这是戴铭的技术文章对应的代码，就是通过observer获取状态的更新，超过指定时间就可以认为是卡顿了。

```objective-c
dispatchSemaphore = dispatch_semaphore_create(0); //Dispatch Semaphore保证同步
    //创建一个观察者
    CFRunLoopObserverContext context = {0,(__bridge void*)self,NULL,NULL};
    runLoopObserver = CFRunLoopObserverCreate(kCFAllocatorDefault,
                                              kCFRunLoopAllActivities,
                                              YES,
                                              0,
                                              &runLoopObserverCallBack,
                                              &context);
    //将观察者添加到主线程runloop的common模式下的观察中
    CFRunLoopAddObserver(CFRunLoopGetMain(), runLoopObserver, kCFRunLoopCommonModes);
    
    //创建子线程监控
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        //子线程开启一个持续的loop用来进行监控
        while (YES) {
            long semaphoreWait = dispatch_semaphore_wait(dispatchSemaphore, dispatch_time(DISPATCH_TIME_NOW, 20*NSEC_PER_MSEC));
            // dispatch_semaphore_wait 超时后会return 非0数字
            if (semaphoreWait != 0) {
                if (!runLoopObserver) {
                    timeoutCount = 0;
                    dispatchSemaphore = 0;
                    runLoopActivity = 0;
                    return;
                }
                //两个runloop的状态，BeforeSources和AfterWaiting这两个状态区间时间能够检测到是否卡顿
                if (runLoopActivity == kCFRunLoopBeforeSources || runLoopActivity == kCFRunLoopAfterWaiting) {
                    //出现三次出结果
//                    if (++timeoutCount < 3) {
//                        continue;
//                    }
                    NSLog(@"monitor trigger");
                    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), ^{
//                        [SMCallStack callStackWithType:SMCallStackTypeAll];
                    });
                } //end activity
            }// end semaphore wait
            timeoutCount = 0;
        }// end while
    });

// 发送信号
static void runLoopObserverCallBack(CFRunLoopObserverRef observer, CFRunLoopActivity activity, void *info){
    SMLagMonitor *lagMonitor = (__bridge SMLagMonitor*)info;
    lagMonitor->runLoopActivity = activity;
    
    dispatch_semaphore_t semaphore = lagMonitor->dispatchSemaphore;
    dispatch_semaphore_signal(semaphore);
}
```



## 一个著名的 timer 问题



```objective-c
_timer = [NSTimer scheduledTimerWithTimeInterval:0.1f
                                              target:self
                                            selector:@selector(doNetwork)
                                            userInfo:nil
                                             repeats:YES];
```

以上写法创建的 timer 会被添加到当前 runloop 的 NSDefaultRunLoopMode 模式下，Creates a timer and schedules it on the current run loop in the default mode.

这样会受滑动影响，为了避免这个问题可以换成commonMode .

```objective-c
NSTimer *timer = [NSTimer scheduledTimerWithTimeInterval:1.0
     target:self
     selector:@selector(timerTick:)
     userInfo:nil
     repeats:YES];
[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
```

同时注意下 NSTimer 循环引用的问题

```objective-c
- (void)dealloc {
    [_timer invalidate];
    _timer = nil;
}

#pragma mark - Public Methods

- (void)layoutWithNewsList:(NSArray <QNListNewsItem *> *) newslist {
    if (!self.timer) {
        self.timer = [NSTimer qn_scheduledAutoStopTimerWithTimeInterval:2.f
                                                             weakTarget:self
                                                               selector:@selector(_startAnimation)
                                                               userInfo:nil
                                                                repeats:YES];

        [[NSRunLoop currentRunLoop] addTimer:self.timer forMode:NSRunLoopCommonModes];
    }
}
```

事实上，dealloc 方法并不会执行，因为循环引用了，NSTimer > self  && self > NSTimer

可以使用一个中间件来处理该问题。QQNews 中有很好的封装 qn_scheduledAutoStopTimerWithTimeInterval

## GCD

```c
/*
     *GCD需要开发者做的事情是：
     *1、定义想要执行的任务
     *2、把任务加入指定的dispatch queue
     *
     *关于系统提供的队列：
     *1、dispatch_get_main_queue()获取到主线程的队列
     *2、dispatch_get_global_queue获取系统提供的全局队列，有四种优先级：
     * DISPATCH_QUEUE_PRIORITY_HIGH 2
     * DISPATCH_QUEUE_PRIORITY_DEFAULT 0
     * DISPATCH_QUEUE_PRIORITY_LOW (-2)
     * DISPATCH_QUEUE_PRIORITY_BACKGROUND INT16_MIN
     **/
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        //后台线程
        NSLog(@"当前线程为： %@",[NSThread currentThread]);
        dispatch_async(dispatch_get_main_queue(), ^{
            //主线程，可以在此更新UI
            NSLog(@"当前线程为： %@",[NSThread currentThread]);
        });
    });

也可以使用performSelector相关实现上述功能

[self performSelectorInBackground:@selector(checkAndLoadSubMenuWithCallBack:) withObject:nil]; 
[self performSelectorOnMainThread:@selector(registDeviceToken:)
     withObject:[NSString stringWithFormat:@"%@", deviceToken]
     waitUntilDone:[NSThread isMainThread]


显然使用gcd 避免了代码分离，使用@selector导致代码分离 ，书写也更加麻烦
```



### Queue来源





#### 1、通过dispatch_queue_create创建

可以指定是concurrent还是serial

#### 2、获取系统自带Dispatch Queue

系统自带queue 分为Main  Dispatch Queue  和 Global Dispatch Queue 

Main  Dispatch Queue 是串行，主线程执行

Global Dispatch Queue 是并行，分为4个优先级：

 \*  - DISPATCH_QUEUE_PRIORITY_HIGH:         QOS_CLASS_USER_INITIATED

 \*  - DISPATCH_QUEUE_PRIORITY_DEFAULT:      QOS_CLASS_DEFAULT

 \*  - DISPATCH_QUEUE_PRIORITY_LOW:          QOS_CLASS_UTILITY

 \*  - DISPATCH_QUEUE_PRIORITY_BACKGROUND:   QOS_CLASS_BACKGROUND



## dispatch_sync

这里有一个著名的死锁问题：

在主线程执行以下代码：

```objective-c
NSLog(@"1");
dispatch_sync(dispatch_get_main_queue(), ^{
    NSLog(@"2");
}) ;
NSLog(@"3");

/* 
*  只会打印 1，死锁了,报错 EXC_BAD_INSTRUCTION 错误指令
*  使用 dispatch_sync 就是同步，当前线程就阻塞了，也就是说当前任务无法继续执行了，但是新增的任务又得等当前任*  务执行完才开始执行，互相等待就死锁了。
*/
```





## dispatch_barrier_async

并行处理结束后再执行Dispatch_barrier_async追加的处理，最后再恢复为并行处理,就是说通过 dispatch_barrier_async添加的块是在并发队列中仍旧是单独执行的

这个传入的队列必须是自己创建的并发队列，否则没有效果

The queue you specify should be a concurrent queue that you create yourself using the [`dispatch_queue_create`](apple-reference-documentation://hccjgwsoUD) function. If the queue you pass to this function is a serial queue or one of the global concurrent queues, this function behaves like the [`dispatch_async`](apple-reference-documentation://hcVyOLxquW) function.



## dispatch_semaphore_t

信号量 

```c
dispatch_semaphore_t lock; 
_lock = dispatch_semaphore_create(1); 
dispatch_semaphore_wait(_lock, DISPATCH_TIME_FOREVER); 
[_enqueueViews removeAllObjects]; 
dispatch_semaphore_signal(_lock); 
```

dispatch_semaphore_wait 第二个参数也可以指定一个具体的数字，比如3秒，那么超时后该函数会返回一个非0数字，而不是永远阻塞下去。在通过runloop监控卡顿的时候，可以利用这个特性。

## dispatch_set_target_queue

依赖关系

## dispatch_queue_set_specific

## dispatch_after

延迟执行

## dispatch_group

执行结束处理

# App层面

## 渲染

<https://lision.me/ios_rendering_process/>

## CoreAnimation Render PipeLine



## Commit Transaction:

Layout \ display \ prepare\commit



## 界面更新

<https://cloud.tencent.com/developer/article/1030703>

![屏幕快照 2019-10-03 下午5.19.27](/iOS/iOS 笔记/images/屏幕快照 2019-10-03 下午5.19.27.png)

每秒60次vsync 信号，cpu gpu占用时间超过16ms，就会造成掉帧，这就是卡顿的原因

iOS 的显示系统是由 VSync 信号驱动的，VSync 信号由硬件时钟生成，每秒钟发出 60 次（这个值取决设备硬件，比如 iPhone 真机上通常是 59.97）。iOS 图形服务接收到 VSync 信号后，会通过 IPC 通知到 App 内。App 的 Runloop 在启动后会注册对应的 CFRunLoopSource 通过 mach_port 接收传过来的时钟信号通知，随后 Source 的回调会驱动整个 App 的动画与显示。

![屏幕快照 2019-10-03 下午5.30.29](/iOS/iOS 笔记/images/屏幕快照 2019-10-03 下午5.30.29.png)

Core Animation 在 RunLoop 中注册了一个 Observer，监听了 BeforeWaiting 和 Exit 事件。这个 Observer 的优先级是 2000000，低于常见的其他 Observer。当一个触摸事件到来时，RunLoop 被唤醒，App 中的代码会执行一些操作，比如创建和调整视图层级、设置 UIView 的 frame、修改 CALayer 的透明度、为视图添加一个动画；这些操作最终都会被 CALayer 捕获，并通过 CATransaction 提交到一个中间状态去（CATransaction 的文档略有提到这些内容，但并不完整）。当上面所有操作结束后，RunLoop 即将进入休眠（或者退出）时，关注该事件的 Observer 都会得到通知。这时 CA 注册的那个 Observer 就会在回调中，把所有的中间状态合并提交到 GPU 去显示；如果此处有动画，CA 会通过 DisplayLink 等机制多次触发相关流程。

## 启动流程、编译运行



### UIView CALayer (Core Animation Layer)

UIView拥有处理和响应用户事件的能力，CALayer则负责绘制显示内容以及动画效果

事实上，UIView 是 CALayer 的 delegate



#### mask

mask 是 layer的一个属性，

mask本身也是一个layer, 比方说 a.mask = b,那么b相当于a的 sublayer，a中超出 b 范围的部分会被裁减掉，看不见。

所以说mask 可以决定layer 显示的具体区域

#### cornerRadius

/* When positive, the background of the layer will be drawn with

rounded corners. Also effects the mask generated by the

`masksToBounds' property. Defaults to zero. Animatable. */

@property CGFloat cornerRadius;

设置cornerRadius 会作用于通过`masksToBounds 创建的mask. 而且background会带有圆角。

#### maskToBounds

图片UIImageView设置了 layer.corner 还得设置 layer.maskToBounds才可以展示出圆角

```objective-c
A Boolean indicating whether sublayers are clipped to the layer’s bounds


/* When true an implicit mask matching the layer bounds is applied to
 * the layer (including the effects of the `cornerRadius' property). If
 * both `mask' and `masksToBounds' are non-nil the two masks are
 * multiplied to get the actual mask values. Defaults to NO.
 * Animatable. */

```

只是设置  layer.corner，并不会产生mask,不会出现圆角，

只有maskToBounds设置true之后，才会有一个隐含的 mask 应用到layer, 这个mask 带有圆角，作用在 layer 上，所以用户就可以看到圆角了。



#### clipToBounds

```objective-c
// When YES, content and subviews are clipped to the bounds of the view. Default is NO.
```



# 框架

## SDWebimage

### 缓存

磁盘缓存 基于文件系统 NSFileManager

内存缓存 NSCache 

NSCache （没有官方定锤NSCache实现了 LRU 算法，但是YYCache 是真正的使用了双向链表+hash表来实现 LRU 算法）

#### 缓存清理

sd 中的NSCache 在收到内存告急通知后，直接

```objective-c
  [self.memCache removeAllObjects] 
```

进入后台或者程序终止时会清理过期文件，超出1周就删除

```objective-c
static const NSInteger kDefaultCacheMaxCacheAge = 60 * 60 * 24 * 7; *// 1 week*
```

缓存目录是：/var/mobile/Containers/Data/Application/8055DD9B-89C8-43B7-A4C9-9BD8183D6EAE/Library/Caches/default/com.hackemist.SDWebImageCache.default

获取 MD5

CC_MD5(str, (CC_LONG)strlen(str), r);

## AFNetworking

### 断点续传

<https://developer.apple.com/documentation/foundation/nsurlsession/1411598-downloadtaskwithresumedata?language=objc>

A download can be resumed only if it is an HTTP or HTTPS `GET` request, and only if the remote server supports byte-range requests (with the `Range` header) and provides the `ETag` or `Last-Modified` header in its responses. A download may also restart if the file on the server has been modified, or if the temporary file has been deleted because of low disk space.

在QQNews 中，对断点续传的临时文件的管理是我们自己QNNetworing封装的，搜索allowResumeForFileDownloads这个属性大概就懂了，AFNetworking 对断点续传的支持是提供了一个带resumeData参数的获取 downloadTask的接口,在接口中最终是调用了 NSURLSession的 downloadTaskWithResumeData 函数。 （临时文件存储在 [QNFileManager urlSessionResumeDataFolder] 中，用FLEX 搜索 session_resumeData 就可以看到这个临时文件目录）

```objective-c
- (NSURLSessionDownloadTask *)downloadTaskWithResumeData:(NSData *)resumeData
                                                progress:(void (^)(NSProgress *downloadProgress)) downloadProgressBlock
                                             destination:(NSURL * (^)(NSURL *targetPath, NSURLResponse *response))destination
                                       completionHandler:(void (^)(NSURLResponse *response, NSURL *filePath, NSError *error))completionHandler
{
    __block NSURLSessionDownloadTask *downloadTask = nil;
    url_session_manager_create_task_safely(^{
        downloadTask = [self.session downloadTaskWithResumeData:resumeData];
    });

    [self addDelegateForDownloadTask:downloadTask progress:downloadProgressBlock destination:destination completionHandler:completionHandler];

    return downloadTask;
}

```



那么，如何获得这个 resumedata呢？原来系统的 NSURLSessionDownloadTask 提供了 cancelByProducingResumeData 接口可以获取到 resumedata .

```
@interface NSURLSessionDownloadTask : NSURLSessionTask

- (void)cancelByProducingResumeData:(void (^)(NSData * _Nullable resumeData))completionHandler;

@end

```



# 设计模式

- 简单工厂
- 工厂方法
- 抽象工厂
- 

## 享元模式

## 单例模式

## 装饰模式

## 观察者模式

### KVO

把对象的 isa 指向一个新的class，在新的class中重写了 setter / getter.

 

# 6大设计原则

开迪和李毅接单：

## 开闭原则

##迪米特法则

## 里氏代换法则

##依赖倒置原则

## 接口隔离原则

## 单一原则

## 









# 算法

## 算法总结文章

https://mp.weixin.qq.com/s/6BuXwpEwCy85jYGKlRpOGA

## 数组

#### 数字之和

1、两数之和

分两种，一种是给定一个升序排列的数组，找出两数之和为s的，如果有多个，输出乘积最小的，这个可以用双指针来处理，从头尾开始遍历，如果太小，左指针向右移动，如果太大，右指针向左移动

另一种不是按序排列的，就是找出两数之和为s的，并且只有一个结果。

思路：

1、暴力，遍历数组，针对i 搜索数组中存不存在 等于target - array[i] 的元素，O(N^2)

2、用 HashMap 存储数组元素和索引的映射，在访问到 nums[i] 时，判断 HashMap 中是否存在 target - nums[i] ，如果存在说明 target - nums[i] 所在的索引和 i 就是要找的两个数。该方法的时间复杂度为 O(N)，空间复杂度为 O(N)，使用空间来换取时间。

有空的时候看一下c++的map是怎么使用的？

2、三数之和为0

1. 先把数组排序
2. 从小到大遍历这个数组，每次取一个元素，将这个元素的相反数设为`target`
3. 在每次遍历中，使用双指针对当前元素的后面的所有元素进行处理，找到两个元素的和为`target`，这样，三个元素的和就是 0
4. 双指针的具体处理方式为：头尾各一个指针，每次判断两个指针所指的元素的和与`target`的大小，如果和小了，左指针右移；如果和大了，右指针左移，直到两个指针相遇

```java
class Solution {
    public List<List<Integer>> threeSum (int[] nums) {
        List<List<Integer>> ans = new ArrayList<>();
        if (nums.length < 3) {
            return ans;
        }

        Arrays.sort(nums);

        for (int i = 0; i + 2 < nums.length; i++) {
            if (i > 0 && nums[i] == nums[i - 1]) {
                continue; // skip same result
            }
            int j = i + 1;
            int k = nums.length - 1;
            int target = -nums[i];
            while (j < k) {
                if (nums[j] + nums[k] == target) {
                    ans.add(Arrays.asList(nums[i], nums[j], nums[k]));
                    j++;
                    k--;
                    while (j < k && nums[j] == nums[j - 1]) j++; // skip same result
                    while (j < k && nums[k] == nums[k + 1]) k--; // skip same result
                } else if (nums[j] + nums[k] < target) {
                    j++;
                } else {
                   k--;
                }
            }
        }
        return ans;
    }
}
```

3、四数之和

多一层循环而已

### 双指针

#### 调整数组顺序使奇数位于偶数前面

申请一个新的数组，先遍历得出奇数的个数，设置两个指针，奇数指针从0开始，偶数指针从奇数个数的末尾开始遍历，填充到新数组

#### 和为S的连续正整数序列

```java
import java.util.ArrayList;
    
    /**
     * T: 和为S的连续正数序列
     * 
     * 题目描述 
     * 小明很喜欢数学,有一天他在做数学作业时,要求计算出9~16的和,他马上就写出了正确答案是100。
     * 但是他并不满足于此,他在想究竟有多少种连续的正数序列的和为100(至少包括两个数)。
     * 没多久,他就得到另一组连续正数和为100的序列:18,19,20,21,22。
     * 现在把问题交给你,你能不能也很快的找出所有和为S的连续正数序列? Good Luck! 
     *  
     * 输出描述: 
     * 输出所有和为S的连续正数序列。序列内按照从小至大的顺序，序列间按照开始数字从小到大的顺序
     * 
     * date: 2015.12.7  21:44
     * @author SSS
     *
     */
    public class Solution2 {
    
    	/**
    	 * 类似于滑动窗口的概念，总和小于sum,就执行endIndex ++； 大于的话，就执行 startIndex ++;
    	 * @param sum
    	 * @return
    	 */
    	public ArrayList<ArrayList<Integer> > FindContinuousSequence(int sum) {
    		ArrayList<ArrayList<Integer>> resultsList = new ArrayList<ArrayList<Integer>>();
    		
    		if (sum < 3) {
    			return resultsList;
    		}
    		
    		int startIndex = 1;
    		int endIndex = 2;
    		int curSum = 3;
    		
    		int mid = sum / 2;
    		while (endIndex < mid || startIndex < endIndex) {
    			if (curSum == sum) {
    				resultsList.add(this.saveList(startIndex, endIndex));
    			}
    			
    			endIndex ++;
    			curSum += endIndex;
    			
    			while (curSum > sum) {
    				curSum -= startIndex;
    				startIndex ++;
    			}
    		}
    		
    		return resultsList;
        }
    	
    	/**
    	 * 存储到list当中，并返回
    	 * @param startIndex
    	 * @param endIndex
    	 * @return
    	 */
    	public ArrayList<Integer> saveList(int startIndex, int endIndex) {
    		ArrayList<Integer> resList = new ArrayList<Integer>();
    		
    		for (int i = startIndex; i <= endIndex; i++) {
    			resList.add(i);
    		}
    		
    		return resList;
    	}
    }
————————————————
版权声明：本文为CSDN博主「shansusu」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/shansusu/article/details/50211519
```





## 排序

### 快速排序

### 堆排序

### 二分查找

如果是一个循环的有序数组，比如 456789123,

怎么查找呢？其实思路还是二分查找，很简单。只是说进入左边部分的判断条件变了而已，变成了 target >= array[start] && target < array[mid];

```
Int getIndex( int[ ] array, int start, int end, int target ) {

// 左边部分

getIndex( );

// 右边部分

getIndex( );

}
```



## 链表

### 1、反转单链表

### 2、查找公共的parentView

获取到两个subview的superview链条准确的说是数组，然后从根开始往下逆序遍历。直到找到两个数组第一个不是同一个view的index，index - 1就是我们的结果。



建议：写完代码后套用1>2>3代入试试，看看会不会输出有错误。
=======

## 二叉树
### 3、桶排序


ac ca 看作同样的字符串，给一个字符串数组，如何算出每个字符串出现的次数，用到桶排序（拍完序，ac ca就是同一个啦，dictionary的key就可以不用对比了），避免key的多次对比。


### 二叉树非递归遍历

首先，这些递归都需要用到stack

(简而言之，BFS广度优先是使用queue,DFS深度优先是使用stack)



记住，先搞中序，先序和中序差不多，后序最复杂

#### 先序（跟>左>右）

这个相比于中序，逻辑几乎一样，就是打印的语句换个地方，换成在if里面，而不是else里面，就完成了。

#### 中序（左>跟>右）

思路：

第一个打印的肯定是最左下角的那个节点，打印之后，肯定还存在一个慢慢往回退退到整个树的根节点那儿，所以很自然的联想到最开始就顺着左边把左节点都存进栈里面，然后不断的pop出来进行处理。

```java
public static void midOrderNotRecursion(BinaryTreeNode root){
        Stack<BinaryTreeNode> stack=new Stack<>();
        while (root!=null || !stack.empty()) {
            if(root!=null) {
                stack.push(root);
                root=root.left;
            }
            else {
                root=stack.pop();
                System.out.print(root.value+" ");
                root=root.right;
            }
        }
    }
```



#### 后序

https://blog.csdn.net/yuzhiyun3536/article/details/76690473

后序遍历可以创新性的使用两个栈来解决问题。

```java
 /**
     * 非递归后序遍历
     * 用两个栈实现，我希望把最终结果存储在第二个栈stackResult中，然后stackResult 一直pop就可以打印出结果，
     * 后序是左、右、根，于是在对stackResult入栈的时候顺序就得是 根、右、左，
     * 由于我们是不断地把第一个栈stack的top节点 压入stackResult的，
     * 并且在while()循环外面已经把root  push进去了，进入while循环之后，首先就把stack顶部push了，所以之后是先push左孩子，再push右孩子
     * @param root
     */
    public static void postOrderNotRecursion(BinaryTreeNode root){
        if(null==root)
            return;
        Stack<BinaryTreeNode> stack=new Stack<>();
        Stack<BinaryTreeNode> stackResult=new Stack<>();
        stack.push(root);
        while (!stack.empty()) {
            BinaryTreeNode node=stack.pop();
            stackResult.push(node);
            if(node.left!=null)
                stack.push(node.left);
            if(node.right!=null)
                stack.push(node.right);
        }
        while (!stackResult.isEmpty()) {
            System.out.print(stackResult.pop().value+" ");
        }
    }
```

#### 重建二叉树

根据先序和中序重建二叉树

#### 搜索二叉树的第k个节点

就是中序遍历嘛，然后取第k个

#### 判断一个数组是否是搜索二叉树的后序遍历

题目：输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历的结果。如果是则输出Yes,否则输出No。假设输入的数组的任意两个数字都互不相同。 

分析：在后序遍历序列中，最后一个数字是树的根节点的值。数组中前面的数字可以分为两部分：第一部分是左子树节点的值，它们都比根节点的值小；第二部分是右子树节点的值，它们都比根节点的值大。

所以先取数组中最后一个数，作为根节点。然后从数组开始计数比根节点小的数，并将这些记作左子树，然后判断后序的树是否比根节点大，如果有点不满足，则跳出，并判断为不成立。全满足的话，依次对左子树和右子树递归判断。



### 1、层次遍历二叉树

可以删除level 和  len 变量，简化上述代码即可

### 2、分行层次遍历二叉树

追加一下打印即可

### 3、求二叉树深度

上述代码返回值 level就是深度，当然最简单的代码是直接使用递归

```c++
int TreeDepth(TreeNode* pRoot)
    {
     queue<TreeNode*> q;
        if(!pRoot) return 0;
        q.push(pRoot);
        int level=0;
        while(!q.empty()){
            int len=q.size();
            level++;
            while(len--){
                TreeNode* tem=q.front();
                q.pop();
                if(tem->left) q.push(tem->left);
                if(tem->right) q.push(tem->right);
            }
        }
        return level;
    } 
```



以上三个问题，其实可以归结到一种方法，那就是运用队列来解决问题。（求最小深度也可以通过这段代码实现，提前return即可，用队列的就是BFS,齐头并进的）

采用层次遍历的方法，每遍历一层，depth++;每一层，需使用一个变量len记录该层的结点个数，也就是队列的当前长度，然后依次在队列中访问该层的len个结点（将队列中len个元素出队列），并将下一层如队列。

###4、找出二叉树中节点值的和等于输入整数所有的路径

### 5、二叉搜索树的第k个节点

给定一棵二叉搜索树，请找出其中的第k小的结点。例如， （5，3，7，2，4，6，8）    中，按结点数值大小顺序第三小结点的值为4。

```
//思路：二叉搜索树按照中序遍历的顺序打印出来正好就是排序好的顺序。
//     所以，按照中序遍历顺序找到第k个结点就是结果。
```

那么使用递归的中序遍历即可。

```c++
/*
struct TreeNode {
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
    TreeNode(int x) :
            val(x), left(NULL), right(NULL) {
    }
};
*/
class Solution {
public:
    TreeNode* KthNode(TreeNode* pRoot, int k)
    {
        if(k<0 || pRoot == NULL )
            return NULL;
        
        int index = 0;
        return KthSmallNode(pRoot, k, &index);
            
            
    }
    
    TreeNode* KthSmallNode(TreeNode* node, int k, int* index) {
        if(node == NULL)
            return NULL;
        TreeNode* leftNode = KthSmallNode(node->left, k, index);
        if(leftNode != NULL)
            return leftNode;
        
        (*index)++;
        if(*index == k){
            return node;
        }
        
        TreeNode* rightNode = KthSmallNode(node->right, k, index);
        if(rightNode != NULL)
            return rightNode;
        return NULL;
    }
};
```



## 回溯法

### 八皇后

<https://blog.csdn.net/yuzhiyun3536/article/details/76624136>

### 机器人运动范围

### 矩阵中的路径

一个字符矩阵，查找其中是否包含一个路径对应了一个指定的字符串

回溯法的关键是回退。

```objective-c
class Solution {
public:
    bool hasPath(char* matrix, int rows, int cols, char* str){
        bool* visited = new bool[rows * cols];
        memset(visited, 0 , rows * cols);
        
        int pathLength = 0;
        for(int i = 0; i < rows; i++){
            for(int j = 0; j < cols; j++){
                if(hasPathCore(matrix, rows, cols, i, j, str, &pathLength, visited))
                    return true;
            }
        }
        
        return false;
    }
    
    bool hasPathCore(char* matrix, int rows, int cols, 
                     int curRow, int curCol, char* str, int* pathLength, bool* visited){
        if(str[*pathLength] == '\0')
            return true;
        
        if(curRow >= 0 && curRow < rows &&
           curCol >= 0 && curCol < cols &&
           matrix[curRow * cols + curCol] == str[*pathLength] &&
           !visited[curRow * cols + curCol]){
            
            visited[curRow * cols + curCol] = true;
            (*pathLength)++;
            if(hasPathCore(matrix, rows, cols, curRow, curCol - 1, str, pathLength, visited) ||
              hasPathCore(matrix, rows, cols, curRow, curCol + 1, str, pathLength, visited) ||
              hasPathCore(matrix, rows, cols, curRow + 1, curCol, str, pathLength, visited) ||
              hasPathCore(matrix, rows, cols, curRow - 1, curCol, str, pathLength, visited)){
                return true;
            } else {
                // 回溯
                (*pathLength)--;
                visited[curRow * cols + curCol] = false;
            }
        }
        
        return false;
    }
};
```

## 动态规划


### 最长公共子序列（非连续）

x[0~n] 

y[0~m]

先考虑x[n] 和y[m]是否相等。转换为子问题



### 最大子段和

定义b[ j ] 为以j结尾的最大字段和

由于任何子数组必然以数组a 中某一个元素结尾，那么我们就可以把所有子数组按照它的最后一个元素下标来进行分类，按照前面的定义，每一个类别中的子数组最大值就是b[ j ]啊。所以我们要求的最终结果就是b [ j ]的最大值了。

然后是递推关系。显然，b[ j ]=max {b[ j-1 ]+a[ j ], a[ j ]}, 1<=j<=n.  

为了节省空间，我在算法中只用了一个变量 b ,而不是使用数组 b [ j ],因为b[ j-1 ]仅仅是在求b[ j ] 的时候被使用了，暂时没有其他用处，于是可以想办法去掉。

```java
class Solution {
public:
    int FindGreatestSumOfSubArray(vector<int> array) {
        int maxSum = array[0];
        int tem = array[0];
        for(int i = 1; i < array.size(); i++) {
            if(array[i] + tem > array[i]) {
                tem =  array[i] + tem;
            } else {
                tem = array[i];
            }
            if(tem > maxSum) {
                maxSum = tem;
            }
        }
        return maxSum;
    }
};
```


### 找零钱


求最小的纸币数量（假设都是纸币）

贪心算法不是最优的，需要用动态规划。

https://blog.csdn.net/yuzhiyun3536/article/details/77933982>

### 最长公共子序列（非连续）

x[0~n] 

y[0~m]

先考虑x[n] 和y[m]是否相等。转换为子问题

### 找零钱

求最小的纸币数量（假设都是纸币）

贪心算法不是最优的，需要用动态规划。

<https://blog.csdn.net/yuzhiyun3536/article/details/77933982>

# 数据结构

## NSSet 、NSArray、NSDictionary

hash 函数

## NSCache

基于 LRU 算法实现缓存淘汰策略，使用双向链表实现，单链表优缺点

1、线程安全

- You can add, remove, and query items in the cache from different threads without having to lock the cache yourself.

2、不拷贝 key 

- Unlike an [`NSMutableDictionary`](apple-reference-documentation://hc0AfBjV9X) object, a cache does not copy the key objects that are put into it.

# 网络

## https

# 操作系统

## 多线程调度算法

### 先来先服务（FCFSFirst Come First Served）

### 最短作业优先（SJF：Shortest Job First）

### 时间片轮转调度算法





# 工具使用

## time profile

## instrument

## git

## runtime源代码

## Xcode 插件自动生成代码

以前是做安卓的，java 一个类可以先写好成员变量，再快捷键生成getter setter, 于是我自己利用 xcode 插件完成了这个功能。提高了生产效率。

## cocoapods 私有库

自己做一个库，就2个文件，那个时间的，然后再结合正杰的组件化，自己写一篇博客进行总结一下。

实践出真知。

# 实际应用

## 如何全面了解一个应用的动态库静态库都有哪些？

可以使用 那个分析 linkmap 的工具，他会列举出每个动态库，静态库的大小。顺便我们可以看出一共有哪些动态库静态库

应该也可使用 otool 来查看吧

##瘦身

总结：从两方面入手解决：资源文件和二进制文件。

是一个不断摸索演进的结果

 资源层面：

写shell脚本对本地图片进行大小排序，然后把大图改成远程加载。同时流水线上加限制，不再允许提交大于50k的图片。还包括压缩图片之类的，后来觉得不太彻底，索性全部远程加载。资源改为冷启动时候从远程加载zip包，然后解压后使用，同时做了兜底策略，在下载失败后可以通过 URL 得到图片

二进制层面：

Fui 利用字符串匹配，读取每个文件，所以特别慢，objcthin只需要分析otool 命令的输出结果， 比较快。

用网上提供的一些工具定期扫描无用的类，发现效果一般，后来决定以controller为维度线上搜集 pv,然后协商删除对应的业务代码，效果非常好。

Linkmap 解读 <https://blog.cnbang.net/tech/2296/>

同时，通过linkmap可以得出哪些库最大，提供一点参考价值。

后来还了解到 属性动态化方案，但是对项目影响太大，决定放弃。

二进制文件定期扫描删除代码



### 1、shell MR的时候阻止大图提交

同时还写了 githook脚本用于禁止本地提交 warning 代码，防止低级错误

### 2、所有图片远端加载

### 3、动态属性瘦身方案

### 4、不太使用的controller

Fui 是字符串匹配的原理，读取得到所有类名，然后搜索所有import.

fui工具非常垃圾，等很久，而且结果没有参考价值

controller可以顺便带动其他类的删除，是一个很好的方法

### 5、统计每个类和静态库占用大小

<https://github.com/huanxsd/LinkMap>

这个很牛逼

### 6、扫描没用的类

<https://github.com/svenJamy/objcthin/>

一个ruby脚本，通过对二进制文件执行 otool 命令获取到 用到的类和全部类，取差集即可得到结果。

```powershell
otool -v -o /iOS/ipa_file_analyze/Payload_5.8.2_has_armv7/QQNews.app/QQNews  >> otool_result.txt
```

 

<https://github.com/CatchZeng/CATClearProjectTool>

### 7、clang插件

### <https://www.infoq.cn/article/clang-plugin-ios-app-size-reducing>

### 8、删除无用方法

<https://tech.meituan.com/2018/12/06/waimai-ios-optimizing-startup.html>



## 列表页架构

## 底层页架构

## 冷启动时间优化

## XCode 插件的编写

## 时间显示策略代码重构

## AppDelegate 的分拆 ( runtime )

## 组件化

protocol

target-action





## 






# iOS 技术面试问题总结

总结一些在求职经历中的遇到的一些面试问题

## Swift 和 Objc

1. **什么是逃逸闭包**
    - 当闭包作为⼀个实际参数传递给⼀个函数的时候，并且是在函数返回之后调⽤，我们就说这个闭包逃逸了。当我们声明⼀个接受闭 包作为形式参数的函数时，你可以在形式参数前写 `@escaping` 来明确闭包是允许逃逸的。

2. **OC 中属性 nonatomic 和 atomic 的区别**
    - nonatomic 为非原子性，不是线程安全的
    - atomic 叫做原子操作，其本质是在属性的 getter 与 setter 方法内部添加了 `pthread_mutex` 普通同步锁，它并不能保证使用属性的过程是线程安全的。由于atomic的加锁解锁需要消耗额外的cpu资源并不推荐使用。我们在使用属性的时候可以使用多线程同步技术来保证属性的线程安全即可。

3. **Swift 中类和结构体的区别**
    - 类有继承，结构体没有继承，所以也就没有重写、多态等特性
    - 类比结构体复杂，类编译后的汇编代码比结构体多，所以，结构体的运行效率要比类快一点。

      >**提示**
      一些明确的类，结构简单，就只放些属性和方法，没有继承啥的，建议使用结构体代替类。
4. **Swift 中的值类型和引用类型的区别**
    - 值类型，存放在栈区；引用类型存在在堆区
    - 值类型的赋值为`深拷贝`，即新对象和源对象是独立的，当改变新对象的属性，源对象不会受到影响，反之同理。
      引用类型的赋值是`浅拷贝`，即新对象和源对象的变量名不同，但其引用（指向的内存空间）是一样的，因此当使用新对象操作其内部数据时，源对象的内部数据也会受到影响。

5. **Swift 中值类型和引用类型的数据类型**
    - 值类型：整型 浮点型 字符串 元组 数组 字典 集合 结构体
    - 引用类型：类 闭包

6. **OC中的深拷贝和浅拷贝**
    - 深拷贝：拷贝前后两个对象是独立的，改变其中一个对象的值，另一个对象不受干扰。
    - 浅拷贝：拷贝前后的两个对象都指向同一个对象，只复制了对象的地址，修改其中一个对象的值，另一个对象也会改变。

7. **OC 中的 copy 和 mutableCopy**
    - `copy`：当不可变对象调用 copy，生成的对象还是不可变对象，属于浅拷贝；可变对象调用 copy，生成的是不可变对象，属于深拷贝。
    - `mutableCopy`：无论是可变或者不可变对象调用 mutableCopy，生成的对象都是可变对象，都是属于深拷贝
8. **OC 中 NSString 为什么使用 copy 定义属性而不是 strong**
    - NSString 使用 copy 和 strong 都可以定义属性，如果给这个属性赋值的是一个 NSString 对象是没什么影响的，如果使用 strong 定义属性，赋值对象是一个 NSMutableString 就会有影响，当 NSMutableString 对象中的值发生改变的时候，属性中的值也会发生改变，这是因为属性 NSString 直接指向了一个 NSMutableString 的对象。如果使用 copy 则不会有什么影响，copy 对于 NSMutableString 来说是属于深拷贝，拷贝结果是一个不可变对象（NSString），正好符合 NSStirng 不可变的初衷。
9. **OC 中使用 copy 修饰 NSMutableString 会发生什么**
    - 修改内容的时候会发生崩溃，因为 copy 对于 NSMutableStirng 是属于深拷贝，返回的是一个 NSString 类型对象（不可变对象），如果对这个 NSMutableString 所指的对像进行修改就会发生崩溃。

10. **weak 和 assgin 的区别**
    - `weak` 只能修饰对象，`assgin` 即可修饰对象也可修饰基本数据类型
    - `weak` 修饰的对象被释放之后会自动设置成 nil，所以是安全的；而 `assgin` 修饰的对象被释放之后不会自动设置成 nil，会产生也指针，所以不安全。 
11. **weak 的实现原理概括**
    - runtime 维护了一个 weak 表，用于存储指向某个对象的所有weak指针。weak 表其实是一个 hash（哈希）表，Key 是所指对象的地址，Value 是 weak 指针的地址数组（这个地址的值是所指对象的地址）。
    - 在对象被回收的时候，首先根据对象地址获取所有 Weak 指针地址的数组，然后遍历这个数组，把每个地址存储的数据设为 nil ，最后把这个 key-value entry 从 Weak 表中删除。

    - 实现原理可以概括以下三步：
      1. 初始化时：runtime 会调用 objc_initWeak 函数，初始化一个新的weak指针指向对象的地址。
      2. 添加引用时：objc_initWeak 函数会调用 objc_storeWeak() 函数， objc_storeWeak() 的作用是更新指针指向，创建对应的弱引用表。
      3. 释放时，调用 clearDeallocating 函数。clearDeallocating 函数首先根据对象地址获取所有weak指针地址的数组，然后遍历这个数组把其中的数据设为nil，最后把这个 entry从weak 表中删除，最后清理对象的记录。

12. **OC 中 block 的本质**
    - block 本质上是一个 Objc 对象，它内部也有 isa 指针，这个对象封装了函数的调用地址(函数指针)以及函数调用的环境（函数参数、返回值、捕获的外部变量等）。

    - 如下所示 block 的源码实现:

        ```c
        // block类的结构体指针，每个block的这个都是相同的，存储一些block常用的标识等等
        struct __block_impl {
            void *isa;      // 指向所属类的指针，也就是 Block 类型，参考OC的self指针
            int Flags;      // 标志变量，在实现 block 的内部操作时会用到
            int Reserved;   // 保留变量
            void *FuncPtr;  // block 执行时调用的函数指针，其指向block代码块中的地址
        };
        ```
    
11. **block 存放在哪里，分为几种**
    - data 区
    - 栈区
    - 堆区

    按存储区域可以分为三种 block, 分别是 全局 block, 栈 block，堆 block。
    
    - 全局 block 存放在 data 区，类名 `__NSGlobalBlock__`，block 不使用**外部的局部变量**或者只有当block里面仅仅捕获了外部的**静态局部变量**、**全局变量**、**静态全局变量**时就变成全局 block，堆全局 block 做 copy 操作还是全局 block。
    - 栈 block 存放在栈区，类名 `__NSStackBlock__`，使用了外部的局部变量的 block 刚创建出来没有复制给任何变量，也没有做 copy 操作就是栈 block。如果创建出来的 block 使用了外部局部变量，然后将这个 block 赋值给一个使用 `__weak` 修饰的变量，该 block 也是一个栈 block。
    - 堆 block 存放在堆区，类名 `__NSMallocBlock__`，block 创建出来赋值给一个变量并且使用了外部的变量，或者做了 copy 操作就变成了堆block。

      三种 block 的代码演示如下：

      ```c
      // 全局 block
      void (^globalBlock)(void) = ^() {
          NSLog(@"hahah");
      };
      NSLog(@"%@", [globalBlock class]);
      
      // 对全局 block 进行 copy 还是全局 block
      NSLog(@"%@", [[globalBlock copy] class]);
      
      // 堆 block，赋值的时候自动 copy
      NSInteger count = 0;
      void (^mallocBlock)(void) = ^() {
          NSLog(@"%ld", count);
      };
      
      NSLog(@"%@", [mallocBlock class]);
      
      // 栈 block ，block 创建出来没有赋值或者 copy
      __block NSInteger num = 0;
      NSLog(@"%@", [^{
          NSLog(@"%ld", num ++);
      } class]);
      
      // 栈 block，block 创建出来赋值给一个 __weak 修饰的变量，不会 copy 到堆区
      __weak void (^weakBlock)(void) = ^() {
          NSLog(@"%ld", count);
      };
      NSLog(@"%@", [weakBlock class]);
      ```
11. **block 中如何修改局部变量**
    - 在局部变量前面使用 `__block` 修饰

12. **__block 的实现原理**
    - __block 在底层是一个结构体，如下所示
    
      ```objc
      __block int i = 0;
 
      // __block 结构体 
      struct __Block_byref_i_0 {
        void *__isa;  // isa指针
        __Block_byref_i_0 *__forwarding; // 该指针指向自身
        int __flags;  // 标记
        int __size;   // 大小
        int i;        // 变量值
      };
      ```

    - 系统将带有 __block 的变量转换成一个前缀为 `__Block_byref_` 结构体。并且当 block 被 copy 到堆上的时候，一并把 __block 变量的结构体也 copy 一份到堆中。其中堆中 block 结构体的 `__forwarding` 指针指向其变量本身。 这样  block 里面就能截获到 __block 变量的结构体里面的 `__forwarding` 指针，而该指针指向了堆上 __block 变量。这样 block 就能对该变量进行修改了。

13. **block 中的循环引用**
    - 当 block 作为一个对象属性的时候，当前对象会持有 block，然后又在 block 中使用(持有)当前对象，这样就造成两个对象互相持有，这就是 block 中的循环引用。
    
    如何打破循环引用？

    - 让其中一个对象声明成为一个弱引用，使用 `__weak` 修饰。

14. **__weak 是什么**
    - `__weak` 是一个弱引用 

    如何使用：声明一个变量/对象的时候，在前面添加 __weak。
    >**注意**
    弱引用指针不能直接指向创建对象时向堆申请的空间, 只能间接指向一个强引用的空间地址, 间接使用该对象。
15. **__strong 是什么**
    - `__strong` 强引用；相当于声明一个局部变量, 在 block 使用完之后才会释放。也就是说保证在 block 调用完之前, 对象不会被释放。

16. **KVO 和 KVC 是什么**
    - KVO 键值监听，提供了观察某一个属性变化的方法
    - KVC 键值编码，通过字符串去访问或者修改一个属性的值

17. **OC 中 KVO 的原理是什么**
    - KVO 基于 runtime 实现
    - 当一个对象执行 addObserver 之后，对象指向父类的指针 isa 变成了指向一个新的类 `NSKVONotifying_XXX`，当一个类 Person 有一个属性叫 name，Person 类的对象的 name 属性发生改变的时候，`NSNotifying_Person` 的 setName 方法里面调用 [super setName:] [self willChangeValueForKey:@"name"] [self didChangeValueForKey:@"name"]，后面的这个两个方法中会调用监听者内部的 `observeValueForKeyPath` 方法。

18. **OC 中 KVC 的原理是什么**
    ```objc
        Person *p = [[Person alloc] init];
        [p setValue:@"Tom" forKey:@"name"];
        [p valueForKey:@"name"];
    ```
    - 如上述代码：

        1. 首先查找一个对象中和 key 相对应的 setter 或者 getter 方法，有就直接调用
        2. 不存在 setter 或者 getter 方法，就查找是否存在以 `_`开头的成员变量 _key，有就访问
        3. 不存在 `_` 开头的成员变量 _key 就查找是否存在这个属性 key，有就直接使用
        4. 不存在这个属性 key，就调用 `setValue: forUndefineKey:` 或者 `valueForUndefineKey:` 方法抛出异常




## 多线程
 
1. **NSNotificationCenter 发送通知和接受通知在一个线程吗？**
    - 在一个线程中，如果在子线程发送消息，那么接受消息也是在子线程中。
    
    **追问：** 为什么在同一个线程中？

    - 实际上发送通知都是同步的，不存在异步操作。而所谓 的异步发送，也就是延迟发送，在合适的实际发送。

    **追问:** 如何实现异步发送 

    - 让通知的执行方法异步执行即可, 通过NSNotificationQueue，将通知添加到队列当中，立即将控制权返回给调用者，在合适的时机发送通知，从而不会阻塞当前的调用。
2. **iOS 中多线程有哪些**
    - **GCD** 一套使用 C 语言开发的多线程 API，能更好的利用多核多线程处理器，API 更接近底层，高级用法稍显复杂。
    - **NSOpration**  对 GCD 面向对象的封装，使用更简单，使用 NSOprationQueue 队列来管理线程
    - **NSThread**  对 pthread 的面向对象的封装，使用简单，功能单一，需要手动管理线程生命周期。
    - **pthread** 一套 C 语言编写的通用的多线程API，久经考验，Unix 和 Linux 都可用

    在实际开发中，GCD 和 NSOpration 使用的更多一些。
3. **线程同步有哪些方式可以实现**
    - GCD 信号量: DispatchSemaphore
    - GCD 串行队列，依次执行
    - pthread 互斥锁: pthread_mutex_t
      ```swift
      var count = 0
      var lock = pthread_mutex_t()
      pthread_mutex_init(&lock, nil)
      pthread_mutex_lock(&lock)
      pthread_mutex_unlock(&lock)
      count += 1
      pthread_mutex_destroy(&lock)
      ```
    - NSLock 基于 pthread 的互斥锁
    - OSSpinLock 自旋锁，一般不使用
    - @synchronized(self) {} 基于 pthred
    ...
4. **什么是线程锁死**
    - 在同一个串行队列中存在任务1和任务2，任务1等待任务2的完成，任务2等待任务1的完成，这样相互等待就造成锁死

    **案例1**
      ```swift
      print("执行了 任务 1")
      DispatchQueue.main.sync {
          print("执行了 任务 2")
      }
      print("执行了 任务 3")
      ```
    **案例2**
      ```swift
      let queue = DispatchQueue(label: "queue.com");
        print("初始任务")
        queue.async {
            queue.sync {
                print("执行了 A")
            }
            queue.sync {
                print("执行了 B")
            }
            print("执行了 C")
        }
      ```
5. **GCD队列有哪几种**
    - 并行队列，可以让多个任务并发执行
    - 串行队列，多个任务依次执行
    >可使用dispatch_get_global_queue 获取全局并发队列，也可使用dispatch_queue_create创建新的队列，创建队列可指定队列类型，
    DISPATCH_QUEUE_SERIAL：串行队列，
    DISPATCH_QUEUE_CONCURRENT：并行队列

    >**提示** 
    串行与并行指的是任务的执行方式。
6. **GCD常用的函数**
    GCD有两个用来执行任务的函数：
    - **dispatch_sync**   执行同步任务, 不具备开启线程的能力。
    - **dispatch_async**  执行异步任务, 具备开启子线程的能力。
    
    >**提示**
    同步与异步指的是是否开启线程的能力

## RunLoop

1. **什么是 RunLoop? 和线程的关系?**
    - RunLoop 就是运行循环，内部就是使用了一个 `do-while` 循环；RunLoop 和线程一一对应的关系，一个线程对应着一个 RunLoop，在其内部是使用的一个字典存储 RunLoop 和线程，线程作为键，RunLoop 作为值。
2. **RunLoop 的应用场景**
    线程保持存活、卡顿检测(Observer)、NSTimer 等一些操作
3. **为什么需要线程保持存活？**
    - 在一些场景中，可能需要频繁的创建线程执行任务，就会造成系统不必要的性能开销，这样可以使用 RunLoop 保持一个线程存活。

    **追问:** 如何线程保持存活？
    - 在一个子线程的任务中，开启运行循环(RunLoop)，具体做法如下

      ```swift
      RunLoop.current.add(NSMachPort(), forMode: .default)
      RunLoop.current.run()
      ```
      运行 runloop 的方法有三个:
      ```swift
      run()
      run(until limitDate: Date)
      run(mode: RunLoop.Mode, before limitDate: Date)
      ```

      其中前两个方法都是无法停止的，其内部就是一个 `while` 循环调用 `run(mode: .default, before: date)` 这个方法，最后一个方法，只循环一次就会结束。 

4. **NSRunLoop 和 CFRunLoopRef 的区别**
    - NSRunLoop 是面向对象的，是基于 CFRunLoopRef 的封装，使用起来更简单方便
    - CFRunLoopRef 是 C 语言编写的，属于面向过程，API 更加底层，使用稍微复杂
5. **CFRunLoopRef 的数据结构**
    RunLoop对象的底层就是一个CFRunLoopRef结构体，它里面存储着：
      1. _pthread：RunLoop与线程是一一对应关系
      2. _commonModes：存储着 NSString 对象的集合（Mode 的名称）
      3. _commonModeItems：存储着被标记为通用模式的Source0/Source1/Timer/Observer
      4. _currentMode：RunLoop当前的运行模式
      5. _modes：存储着RunLoop所有的 Mode（CFRunLoopModeRef）模式

      ```c
      // CFRunLoop.h
      typedef struct __CFRunLoop * CFRunLoopRef;
      // CFRunLoop.c
      struct __CFRunLoop {
          pthread_t _pthread;  // 与线程一一对应
          CFMutableSetRef _commonModes;
          CFMutableSetRef _commonModeItems;
          CFRunLoopModeRef _currentMode;
          CFMutableSetRef _modes;
          ...
      };

      ```

6. **CFRunLoopModeRef 的数据结构**
    - CFRunLoopModeRef代表RunLoop的运行模式；
    - 一个RunLoop包含若干个 Mode，每个 Mode 又包含若干个Source0/Source1/Timer/Observer；
    - RunLoop启动时只能选择其中一个 Mode，作为 currentMode；
    - 如果需要切换 Mode，只能退出当前 Loop，再重新选择一个 Mode 进入，切换模式不会导致程序退出；
    - 不同 Mode 中的Source0/Source1/Timer/Observer能分隔开来，互不影响；
    - 如果 Mode 里没有任何Source0/Source1/Timer/Observer，RunLoop会立马退出。

    ```c
    // CFRunLoop.h
    typedef struct __CFRunLoopMode *CFRunLoopModeRef;
    // CFRunLoop.c
    struct __CFRunLoopMode {
        CFStringRef _name;             // mode 类型，如：NSDefaultRunLoopMode
        CFMutableSetRef _sources0;     // CFRunLoopSourceRef
        CFMutableSetRef _sources1;     // CFRunLoopSourceRef
        CFMutableArrayRef _observers;  // CFRunLoopObserverRef
        CFMutableArrayRef _timers;     // CFRunLoopTimerRef
        ...
    };

    ```
7. **Source有两种类型：source0 和 source1**
    - `source1` 基于 mach_port, 来自系统内核或者其他进程或线程的事件，可以主动唤醒休眠中的 RunLoop。mach_port可以理解成进程间相互发送消息的一种机制就好, 比如屏幕点击, 网络数据的传输都会触发sourse1。
    - `source0` 非基于 port 的处理事件，什么叫非基于 port 的呢？就是说你这个消息不是其他进程或者内核直接发送给你的。一般是APP内部的事件, 比如hitTest:withEvent的处理, performSelectors的事件。

    - 举个简单例子：一个APP在前台静止着，此时，用户用手指点击了一下APP界面，那么过程就是下面这样的：
    我们触摸屏幕,先摸到硬件(屏幕)，屏幕表面的事件会被IOKit先包装成Event,通过mach_Port传给正在活跃的APP , Event先告诉source1（mach_port）,source1唤醒RunLoop, 然后将事件Event分发给source0,然后由source0来处理。如果没有事件,也没有timer,则runloop就会睡眠, 如果有,则runloop就会被唤醒,然后跑一圈。

8. **CFRunLoopAddObserver**
    - 监控RunLoop的状态。RunLoop的状态如下
      ```c
      typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
        kCFRunLoopEntry = (1UL << 0),         // 即将进入 RunLoop
        kCFRunLoopBeforeTimers = (1UL << 1),  // 即将处理 Timer
        kCFRunLoopBeforeSources = (1UL << 2), // 即将处理 source
        kCFRunLoopBeforeWaiting = (1UL << 5), // 即将进入休眠
        kCFRunLoopAfterWaiting = (1UL << 6),  // 刚从休眠中唤醒，但是还没完全处理完事件
        kCFRunLoopExit = (1UL << 7),          // 即将退出Loop
      }
      ```
      >**温馨提示** 
      可以使用 CFRunLoopAddObserver 监听主线程的卡顿


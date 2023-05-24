# iOS 技术面试问题总结

总结一些在求职经历中的遇到的一些面试问题

## Swift 和 Objc

1. **什么是逃逸闭包**
    - 当闭包作为⼀个实际参数传递给⼀个函数的时候，并且是在函数返回之后调⽤，我们就说这个闭包逃逸了。当我们声明⼀个接受闭 包作为形式参数的函数时，你可以在形式参数前写 `@escaping` 来明确闭包是允许逃逸的。

## 多线程

1. **NSNotificationCenter 发送通知和接受通知在一个线程吗？**
    - 在一个线程中，如果在子线程发送消息，那么接受消息也是在子线程中。
    
    **追问：** 为什么在同一个线程中？

    - 实际上发送通知都是同步的，不存在异步操作。而所谓 的异步发送，也就是延迟发送，在合适的实际发送。

    **追问:** 如何实现异步发送 

    - 让通知的执行方法异步执行即可, 通过NSNotificationQueue，将通知添加到队列当中，立即将控制权返回给调用者，在合适的时机发送通知，从而不会阻塞当前的调用。
2. **iOS 中多线程有哪些**
    - GCD
    - NSOpration
    - NSThread
    - pthread
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


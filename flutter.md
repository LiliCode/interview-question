# Flutter 的一些知识点

## Dart 语言知识点

 

## Flutter 知识点

1. **渲染原理**

  - 三棵树简介
    - Widget Tree 部件树
      存放渲染内容、它只是一个配置数据结构，创建是非常轻量的，在页面刷新的过程中随时会重建

    - Element Tree 元素树
      是分离 WidgetTree 和真正的渲染对象的中间层， WidgetTree 用来描述对应的Element 属性,同时持有Widget和RenderObject，存放上下文信息，通过它来遍历视图树，支撑UI结构。

    - RenderObject 渲染树
      用于应用界面的布局和绘制，负责真正的渲染，保存了元素的大小，布局等信息，实例化一个 RenderObject 是非常耗能的

    - 流程: 当应用启动时 Flutter 会遍历并创建所有的 Widget 形成 Widget Tree，通过调用 Widget 上的 `createElement()` 方法创建每个 `Element` 对象，形成 Element Tree。最后调用 Element 的 `createRenderObject()` 方法创建每个渲染对象，形成一个 Render Tree。

  - flutter为什么要设计成这样呢？为什么要弄成复杂的三层结构？
    - 答案是性能优化。如果每一点细微的操作就去完全重绘一遍UI，将带来极大的性能开销。flutter 的三棵树型模式设计可以有效地带来性能提升。

    - widget 的重建开销非常小，所以可以随意的重建，因为它不一会导致页面重绘，并且它也不一定会常常变化。 而renderObject如果频繁创建和销毁成本就很高了，对性能的影响比较大，因此它会缓存所有页面元素，只是当这些元素有变化时才去重绘页面。

    - 而判断页面有无变化就依靠element了，每次 widget 变化时 element 会比较前后两个 widget，只有当某一个位置的 Widget 和新 Widget 不一致，才会重新创建 Element 和 widget；其他时候则只会修改 RenderObject 的配置而不会进行耗费性能的 RenderObject 的实例化工作了。

    >**参考资料**
    https://blog.csdn.net/jdsjlzx/article/details/125397397

2. **StatefulWidget 的生命周期**
  - 如下图所示
    ![](./assets/stateful-widget-life-cycle.PNG)

3. **Flutter App 的生命周期**
  - 使用 `WidgetsBindingObserver` mixin 监听生命周期，重写 `didChangeAppLifecycleState()` 方法来获取。通过该接口可以获取是生命周期状态在 `AppLifecycleState` 枚举中。
  - AppLifecycleState就是App的生命周期，包含四个
    1. `inactive`: App 处于并不活跃的状态，无法处理用户响应
    2. `resumed`: App 可见并能响应用户的输入
    3. `paused`: App 不可见并不能响应用户的输入，但是在后台继续活动中
    4. `detached`: App 进程已经被杀死

    ```dart
    class MyHomePageState extends State<TestPage> with WidgetsBindingObserver {
      @override
      Widget build(BuildContext context) {
    
      }
    
      @override
      void initState(){
        super.initState();

        // 添加监听
        WidgetsBinding.instance.addObserver(this);
      }
    
      @override
      void didChangeAppLifecycleState(AppLifecycleState state) {
        // App 的生命周期回调
        switch (state) {
          case AppLifecycleState.inactive:
            print('app -> inactive');
            break;
          case AppLifecycleState.resumed:
            print('app -> resumed 恢复');
            break;
          case AppLifecycleState.paused:
            print('app -> paused 挂起');
            break;
          case AppLifecycleState.detached:
            print('app -> detached 已经退出(进程已杀死)');
            break;
        }
      }
    }
    ```

    参考资料: http://www.ppmy.cn/news/543.html?action=onClick

3. **如何监听路由跳转**

  1. 让页面混入 `RouteAware` 这么一个 mixin
  2. 在 `initState()` 或者 `didChangeDependencies()` 中添加订阅监听的代码；在 `dispose()` 中添加移除监听的代码

      ```dart
      // 订阅路由事件
      Routes.routeObserver.subscribe(this, ModalRoute.of(context) as PageRoute);

      // 取消订阅
      Routes.routeObserver.unsubscribe(this);
      ```

  3. 重写 `RouteAware` 中的方法
  
      ```dart
      @override
      void didPopNext() {
        super.didPopNext();
        print('didPopNext');
      }

      @override
      void didPushNext() {
        super.didPushNext();
        print('didPushNext');
      }
      ```

4. **监听 FlutterView 的尺寸变化**
  - 这个监听常用作监听键盘弹出或隐藏
  - 如何使用
    1. 当前 Widget 混入 一个 mixin `WidgetsBindingObserver`
    2. initState() 中添加监听，dispose() 中移除监听

        ```dart
        WidgetsBinding.instance.addObserver(this);
        WidgetsBinding.instance.removeObserver(this);
        ```
    3. 在 `didChangeMetrics()` 方法中添加 frame 变化的监听
        
        ```dart
        @override
        void didChangeMetrics() {
          super.didChangeMetrics();

          // 监听 FlutterView 尺寸变化
          WidgetsBinding.instance.addPostFrameCallback((t) {
            final viewInsets = MediaQuery.of(context).viewInsets;
            if (viewInsets.bottom > 0) {
              // 键盘弹出
              print('键盘弹出来了');
            }
          });
        }
        ```

5. **Isolate**
  - 所有的 Dart 代码都运行在 Isolate 中。Isolate 有自己私有的内存空间和一个运行事件循环的独立线程。
  - 大部分 Dart 应用在一个 Isolate 中运行全部代码，也可以根据需要创建更多 Isolate。如果某个操作计算量如此之大以至于它在主 Isolate 运行中会导致掉帧，可以使用 Isolate.spawn() 或Flutter’s compute() function 方法。这些方法都会创建独立的 Isolate 来做密集计算，让主 Isolate 专注重建和渲染 Widget 树。
  - 新创建的 Isolate 有自己的事件循环和内存，原先的 Isolate (即创建新 Isolate 的那个 Isolate) 不能访问这些内存。这种机制正是 Isolate 名字的来源：内存块之间彼此隔离。
  - 事实上，Isolate 之间能协作的唯一方式是消息传递。一个 Isolate 可以向另一个 Isolate 发送消息，接收方在其事件循环处理收到的消息。
  - Isolate 中内存分配和垃圾回收不需要锁定。Isolate 中只有一个线程，如果它不是很忙的话，内存并不会快速变化，所以不必锁定。这非常适合 Flutter 应用，它时常要迅速地构建和销毁 Widget 树。

  参考资料：https://cloud.tencent.com/developer/article/1860255

6. **Isolate 和鲜线程的区别**
  - 线程与线程之间是共享内存的，而 isolate 和 isolate 之间是不共享的，所以叫 isolate (隔离)。
  
    >**提示**
      因为 Dart 没有共享内存的并发，没有竞争的可能性所以不需要锁，也就不用担心死锁的问题

7. **Isolate 之间如何传递消息（通信）**
  - 由于 isolate 之间没有共享内存，所以他们之间的通信唯一方式只能是通过 `Port` 进行，而且 Dart 中的消息传递总是异步的。

  - 通信使用: SendPort、ReceivePort

8. 
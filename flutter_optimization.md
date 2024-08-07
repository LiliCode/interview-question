# Flutter 应用的一些优化方法

总结一些 Flutter 应用的一些常用的优化方式，天天搬砖，随时更新。

## 图片资源的优化

  1. 图片资源的延迟加载并缓存，降低网络流量，下次快速展示图片相同的图片。
  2. 在列表上面使用缩略图进行展示，缩略图尺寸小所占用的内存空间很小，不会出现内存暴增的现象。
  3. 如果网络图片所占的内存空间很大，可以使用图片裁切，例如 CachedNetworkImage 的 memCacheWidth 和 memCacheHeight 属性、ExtendedImage 的 cacheWidth 和 cacheHeight 属性，使用图片剪裁的功能进行展示图片，不会造成内存暴增。
  4. 想要减少图片下载的流量可以使用 webp 图片格式，资源服务器存放 webp 格式图片，但是 webp 图片解码会更耗时，会造成性能的影响。
  5. 尽量减少使用带有透明度的图片或者图标，会导致图层混合，CPU 会计算图片下面的组件的颜色和图案，会存在额外的性能开销

## 页面的重绘优化

  1. 降低页面刷新的频率，减少 setState() 方法刷新的范围，控制 setState 的刷新粒度，做局部刷新，不能一个页面一小部分更改就刷新整个页面。
  2. 大量的计算业务计算驱动页面刷新，控制刷新的粒度，使用状态管理器，拆分更精细的 viewModel，减少 build() 函数调用的次数。也可以使用 StreamBuilder 去做局部刷新，也可以使用 GetX 中的可观察对象进行进步监听刷新。
  3. 在使用 AnimatedBuilder 展示动画的时候，将不需要依赖动画的部分更新放入 child 属性中，将需要根据动画变化的部分放入 builder 回调中进行构建。在动画执行中不要使用剪裁，如果需要用到图片剪裁可以在动画执行之前进行剪裁。
  4. 不要在 build() 函数中执行耗时操作，否则会很卡
  5. build() 函数中堆叠了大量 Widget，会导致三个问题：代码可读性差(难以维护)、难以复用、影响性能

      1. 代码可读性差：会导致后期难以维护，会造成无法预料的 bug
      2. 难以复用：所有的代码都在这一个函数中，无法将公共的布局用在其他地方
      3. 影响性能：如果一大堆 Widget 堆叠，使用 setState() 函数进行刷新的时候，所有的 Widget 都需要进行重建

## 耗时操作的优化

  1. 一般的耗时操作，可以使用异步操作的 API 进行，比如 Future、Stream。
  2. 需要处理大量数据这种耗时操作，可以放入子线程（Dart 中是 Isolate），使用 `compute` 函数创建一个新的 Isolate 进行处理，例如：

    - 网络请求之后的大量数据
    - 读取大文件

  3. 执行一些耗时操作，可以在 UI 层面上展示一个加载动画，给用户一个正在加载的反馈
  4. 如果列表一个超复杂的 Widget 需要展示，在一帧时间内无法完成绘制，可以使用分帧上屏的方法进行异步展示

## 列表懒加载

  如果列表数据存在很多或者不可预知，可以使用 ListView.builder()、GridView.builder()、PageView.builder() 等一些构建方式进行懒加载。对于使用 Row、Column 组件一次性布局，如果有很多相同布局的组件就会导致每一个都会重复去创建，如果使用懒加载列表的方式进行展示，可以避免很多重复创建的组件，从而降低内存开销。

## 使用 const 组件或者常量

  const 在 dart 语法中是定义常量，而且是在编译时就会确定的常量，而 final 则是在程序运行期间的一次性赋值之后就不能被改变。对于 const 声明的组件，在编译期间就已经确定，不会存在运行时创建和状态改变的问题，这样大大提高了性能。

## 数据预加载

  1. 比如很多地方需要用到同一组数据，可以在第一次加载之后缓存到内存中，这样其他页面调用的时候就可以快速的读取缓存数据
  2. 如果两个页面存在关联性，前一个页面的数据可以在后一个页面中使用，可以在前一个页面获取数据之后，将后一个页面使用的数据处理成数据模型的方式传递给下一个页面。
  
## 应用启动避免过多的耗时操作

  1. 启动的时候不要过多的网络请求，过多的请求会造成服务器压力，对手机性能也有一定的影响；可以将一些优先级没那么高的请求剔除，放在后续业务逻辑中进行获取。
  2. 避免过多的耗时操作，否则影响启动速度

## 页面保活

  如果在 TabBarView 中的一些页面，使用 keepAlive 的方式进行状态保持，避免了重复的渲染和网络请求


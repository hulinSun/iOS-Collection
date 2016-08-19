#博客总结
#####git fork 流

```
git remote add upstream git@123.57.152.76:teamfrontend/dreammentor.git

git fetch upstream

git merge upstream/master
fork 关联元地址  
```

##### 异步渲染

```
YYAsyncLayer：继承自CALayer，绘制、创建绘制线程的部分都在这个类。
YYTransaction：用于创建RunloopObserver监听MainRunloop的空闲时间，并将YYTranaction对象存放到集合中。
YYSentinel：提供获取当前值的value（只读）属性，以及- (int32_t)increase自增加的方法返回一个新的value值，用于判断异步绘制任务是否被取消的工具

kCFRunLoopBeforeWaiting：Runloop将要进入休眠。
kCFRunLoopExit：即将退出本次Runloop。
在监听这个mainrunloop 的回调里面执行刷新方法 [transaction.target performSelector:transaction.selector];

对外接口YYAsyncLayerDelegate代理中提供- (YYAsyncLayerDisplayTask *)newAsyncDisplayTask方法用于回调绘制的代码，以及是否异步绘制的BOOl类型属性displaysAsynchronously，同时重写CALayer的display 方法来调用绘制的方法- (void)_displayAsync:(BOOL)async。
```
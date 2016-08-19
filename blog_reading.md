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

#####runtime
* 替换系统自带的方法
* 字典转模型
* 一键归档，解档
* 万能控制器跳转
* 分类中设置属性


```
/ 这个规则肯定事先跟服务端沟通好，跳转对应的界面需要对应的参数
NSDictionary *userInfo = @{
                           @"class":@"HSFeedsViewController",                         @"property": @{@"ID": @"123",                          @"type":@"12"                          }
};



- (void)push:(NSDictionary *)params
{
    // 类名
    NSString *class =[NSString stringWithFormat:@"%@", params[@"class"]];
    const char *className = [class cStringUsingEncoding:NSASCIIStringEncoding];
    // 从一个字串返回一个类
    Class newClass = objc_getClass(className);
    if (!newClass)
    {
        // 创建一个类
        Class superClass = [NSObject class];
        newClass = objc_allocateClassPair(superClass, className, 0);
        // 注册你创建的这个类
        objc_registerClassPair(newClass);
    }
    // 创建对象
    id instance = [[newClass alloc] init];
    // 对该对象赋值属性
    NSDictionary * propertys = params[@"property"];
    [propertys enumerateKeysAndObjectsUsingBlock:^(id key, id obj, BOOL *stop) {
        // 检测这个对象是否存在该属性
        if ([self checkIsExistPropertyWithInstance:instance verifyPropertyName:key]) {
            // 利用kvc赋值
            [instance setValue:obj forKey:key];
        }
    }];
    // 获取导航控制器
    UITabBarController *tabVC = (UITabBarController *)self.window.rootViewController;
    UINavigationController *pushClassStance = (UINavigationController *)tabVC.viewControllers[tabVC.selectedIndex];
    // 跳转到对应的控制器
    [pushClassStance pushViewController:instance animated:YES];
}

// 检测对象是否存在该属性
- (BOOL)checkIsExistPropertyWithInstance:(id)instance verifyPropertyName:(NSString *)verifyPropertyName
{
    unsigned int outCount, i;
    // 获取对象里的属性列表
    objc_property_t * properties = class_copyPropertyList([instance
                                                           class], &outCount);
    for (i = 0; i < outCount; i++) {
        objc_property_t property =properties[i];
        //  属性名转成字符串
        NSString *propertyName = [[NSString alloc] initWithCString:property_getName(property) encoding:NSUTF8StringEncoding];
        // 判断该属性是否存在
        if ([propertyName isEqualToString:verifyPropertyName]) {
            free(properties);
            return YES;
        }
    }
    free(properties);
    return NO;
}
```




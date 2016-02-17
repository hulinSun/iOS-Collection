# iOS-Collection
个人对于iOS的总结

```
1. 第一响应者:叫出键盘的叫做第一响应者
* 辞去第一响应者(退出键盘)：[self.textView resignFirstResponder]
* 成为第一响应者(交出键盘):  [self.textView becomeFirstResponder]
* 叫出键盘的另一种方式: - (BOOL)endEditing:(BOOL)force; // 让view 或者view的子控件辞去第一响应者的职务
* 注意点:在iPad中退出键盘-->如果为formsheet 的话，那么直接退出键盘的话，是达不到效果的
[searchBar endEditing:YES];
- (BOOL)disablesAutomaticKeyboardDismissal ->return NO; //需要实现这个方法
Default implementation affects UIModalPresentationFormSheet visibility.
```

```
2. UIView 的动画
* 首尾式动画:注意，应该在开启动画之后立即设置动画的一些属性(时长，重复次数，代理放动画内部设置能够监听到动画的结束)
[UIView beginAnimations:(NSString *) context:(void *)]  -> 中间放执行动画的内容 void *(指向任何数据类型) --> id
[UIView setAnimationDelegate:self]; // 可以监听到动画的开始和动画的结束
[UIView commitAnimations]];
* block式动画：比较方便，可以延迟动画执行，监听到动画执行完之后做事情
[UIView animateWithDuration: animations:^ { code}];
[UIView animateWithDuration: animations:<#^(void)animations#> completion:<#^(BOOL finished)completion#>];
```

```
3. 如果多处代码大部分时相同的，只有在不同情况下那么少数代码不同，考虑抽取出方法出来，相同的代码放在方法中，不同的值当做参数。根据不同的情况需求(枚举类型)来做不同的处理
```

```
4.UIView 的属性 frame bounds center Transfrom
* @property(nonatomic) CGRect frame;
* 控件所在矩形框在父控件中的位置和尺寸(以父控件的左上角为坐标原点)  可以定义控件的位置(oarigin)和⼤小(size)
* @property(nonatomic) CGRect bounds;
* 控件所在矩形框的位置和尺寸(以⾃⼰左上角为坐标原点,所以bounds的x、y⼀一般为0)  可以定义控件的大小(size)
* @property(nonatomic) CGPoint center;
* 控件中点的位置(以父控件的左上角为坐标原点)  可以定义控件的位置(center)
```

```objc
Transfrom: 可修改控件的位移，缩放比例，旋转

self.imgView.transform = CGAffineTransformMakeRotation(M_PI_4); // angle ->弧度制
self.imgView.transform = CGAffineTransformRotate(self.imgView.transform, M_PI_4);
self.imgView.transform = CGAffineTransformMakeScale(0.7, 0.7);
self.imgView.transform = CGAffineTransformScale(self.imgView.transform, 0.7, 0.7);
self.imgView.transform = CGAffineTransformMakeTranslation(-50, 100);
self.imgView.transform = CGAffineTransformTranslate(self.imgView.transform, -50, 100);
* 清空transform ->赋值为常量 CGAffineTransformIdentity
* 使用心得:传入transform的值时，那么就会在上次transform的基础上做一些操作。在这种情况下时可以叠加的，如果不传入transform这个参数时，不能叠加。 make:只执行一次.在原来的基础上执行后，就不执行了

```

```
5. 标准的枚举写法
* 枚举类型本质上就是整数，定义的时候，如果只指定了第一个数值，后续的数值会依次递增
typedef enum {
    kMovingDirTop = 10,
    kMovingDirBottom,  // 11
    kMovingDirLeft,
    kMovingDirRight,
} kMovingDir;
```

```
图片浏览器Demo
* 看好plist，plist的根节点是什么就创建什么。
* KVC字典转模型 [self setValuesForKeysWithDictionary:dict];
* 懒加载(延迟加载)的好处
重写getter方法可以完成懒加载模式。代码的可读性强.注意循环引用的问题
用到时再加载,节省了不必要的内存资源,不需要关心什么时候去创建
只保证创建一次。内存中只有这一个对象.(如果经常用到UI控件，并且都是一样的，不需要重复创建，可以采取懒加载(特例UI控件用strong))
懒加载要注意一个清空问题，如果你移除了UI控件，没有清空，但还要重新用重新创建，不清空有可能过不去创建的代码
```

```
帧动画(针对imgView)
1.最重要的一点是清除缓存
[self.imgView performSelector:@selector(setAnimationImages:) withObject:nil afterDelay:self.imgView.animationDuration + 1];
[UIImage imageWithContentsOfFile:] ->用图片在沙盒的路径加载，没有缓存(缓存会经过一些操作后清楚)
[UIImage imageNamed:] -> 图片所占用的内存会一直在程序中，有缓存
2.%02d --> 不够两位补零
```

```
Xcode 路径问题(iOS8开始 沙盒和bundle位置安装路径改变 :bundle.path打印一下就ok)
1. Xcode6 bundle 路径：/Users/apple/Library/Developer/CoreSimulator/Devices/45A2D2FE-E2F1-40B2-B0C6-679EB21E93CB/data/Containers/Bundle/Application/-----> Bundle
2. Xcode6 沙盒路径：Users/apple/Library/Developer/CoreSimulator/Devices/45A2D2FE-E2F1-40B2-B0C6-679EB21E93CB/data/Containers/Data/Application -----> Data/Application

3. Xcode文档安装路径 /Applications/Xcode.app/Contents/Developer/Documentation/DocSets
4. Xcode模拟器安装路径 /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs

5. Xcode 插件路径/Users/sunhulin/Library/Application Support/Developer/Shared/Xcode/Plug-ins 如果插件失效，删除掉重新装，记住重启之后一定要选择loadBundles
6.xcode插件失效 解决命令行：获取当前环境下的UUID，添加到插件的plist文件中
find ~/Library/Application\ Support/Developer/Shared/Xcode/Plug-ins -name Info.plist -maxdepth 3 | xargs -I{} defaults write {} DVTPlugInCompatibilityUUIDs -array-add defaults read /Applications/Xcode.app/Contents/Info.plist DVTPlugInCompatibilityUUID

7.Xcode自带头文件的路径
/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator7.1.sdk/System/Library/Frameworks/UIKit.framework/Headers
* 修改了系统自带头文件后,Xcode会报错
解决方案:删掉下面文件夹的缓存即可(aplle是电脑的用户名)
/Users/aplle/资源库/Developer/Xcode/DerivedData
/Users/aplle/Library/Developer/Xcode/DerivedData

```

```
@property 作用
生成getter方法  生成setter方法
生成带下划线的成员变量（记录属性内容）
readonly的属性不会生成带下划线的成员变量！要想访问需要这样写{int _age} 这样访问

```

```
@synthesize合成指令，主动指定属性使用的成员变量名称 ->@synthesize image = _image;
```

```
超级猜图Demo
* 相框效果用按钮：设置contentInset-> background(背景)是白色 -> img 设置图片
* [self.view bringSubviewToFront:self.imgView]; 把指定的View 放在上面，可见->遮盖的时候用
* GCD dispatch_after 和 [self performSelector: withObject: afterDelay:];有延迟做一些事情的效果
* view的封装，需要什么就传什么，但是考虑到代码的先后调用顺序
* 宏的规范写法 #define kJNOptionBtnCount 21
* 让数组中每个元素都执行某个方法,不需要遍历，快捷方法
> [self.answerView.subviews makeObjectsPerformSelector:@selector(removeFromSuperview)]
* 控件的懒加载，如果每次都要用，每次都要创建，考虑用懒加载。打印内存地址
* [textView sizeThatFits:CGSizeMake(MAXFLOAT, FLT_MAX)] // 牛逼的方法
* [self.tableView beginUpdates]; // 动态算高，textView有输入
* [self.tableView endUpdates];

```

```objc
排序-> 方法与返回值，返回排过序的数组
arr = [arr sortedArrayUsingComparator:^NSComparisonResult(id obj1, id obj2) {
    return [obj1 compare:obj2]; // 返回的是blcok 的返回值 升序
}];
```

```
UIScrollViewDemo
* bounces:弹簧效果 scrollEnabled:能否滚动 show-:是否显示横竖滚动条
* scrollView的方法很简单，只需要添加scrollView，并且设置contentSize
* contentOffset:表示UIScrollView滚动的位置, 内容左上角和scrollView左上角的间距
* contentInset: 表示能够在UIScrollView的4周增加额外的滚动区域
* 如果在Storyboard中添加了ScrollView的子控件,要想scrollView滚动,必须取消autolayout)
* scrollView实现缩放要设置图片缩放的最大最小尺寸，并且在viewForZoomingInScrollView 方法中返回需要缩放的View，知道缩放那个view才能缩放
* 自带的方法 滚到某个位置 -(void)setContentOffset:(CGPoint)contentOffset animated:(BOOL)animated;

```

```
UITableView 多组汽车Demo
*-(NSArray *) sectionIndexTitlesForTableView ->提示标题
* 嵌套模型数组 :NSDictionary *di in dict[@"cars"] -->dict[@"cars"] 不能用self.cars
* 滑动删除代理方法: commitEditingStyle 编辑样式
* cell 重用原理
当滚动列表时,部分UITableViewCell会移出窗口,UITableView会将窗口外的UITableViewCell放⼊一个对象池中,等待重用。当UITableView要求dataSource返回UITableViewCell时,dataSource会先查看这个对象池,如果池中有未使用的UITableViewCell,dataSource会用新的数据配置这个UITableViewCell,然后返回给 UITableView,重新显示到窗口中,从⽽避免创建新对象
* cell 重用解决方案
UITableViewCell有个NSString *reuseIdentifier属性,可以在初始化UITableViewCell的时候传入⼀个特定的字符串标识来设置reuseIdentifier(一般 用UITableViewCell的类名)。当UITableView要求dataSource返回UITableViewCell时,先通过一个字符串标识到对象池中查找对应类型的UITableViewCell对象,如果有,就重用,如果没有,就传入这个字符串标识来初始化⼀个UITableViewCell对象

* 为了避免为单元格重复分配内存空间，UITableView提供了循环引用机制
dequeueReusableCellWithIdentifier可以使用指定的标示符在缓冲池中查找可重用Cell
如果没有找到cell再实例化cell
使用static修饰符可以保证字符串变量只被分配一次内存空间，在此可以与宏的方式进行一下对比
```

```
UIAlertView
* alertView.alertViewStyle 弹出样式
* [alertView textFieldAtIndex:0] 获取文本框样式下的文本框
* alertView.tag = indexPath.row;在Alert代理方法里用,用alertView.tag记录对应行号的模型,把上面的行数传递给下面
* 内部有很多私有属性->通过KVC获取
```

```
数据刷新 :永恒的两步走
* 修改模型数据
* 刷新表格-->重新加载数据
全局刷新：[self.tableView reloadData]
局部刷新[self.tableView reloadRowsAtIndexPaths: withRowAnimation:]

```

```
表格处理
*  NSIndexPath *indexPath = [self.tableView indexPathForSelectedRow]; // 获取选中哪一行的cell
* 每个tableview都有自己的编辑样式,tableView.editing = YES,就会显示表格编辑界面
* 实现(void)tableView:(UITableView *)tableView commitEditingStyle:(UITableViewCellEditingStyle)editingStyle forRowAtIndexPath:(NSIndexPath *)indexPath;
> 如果没有实现-(UITableViewCellEditingStyle)tableView:(UITableView *)tableView editingStyleForRowAtIndexPath:(NSIndexPath *)indexPath 这个代理方法，每个cell 默认的编辑样式都是删除样式。意味着可以滑动删除
> 判断 editingStyle 是哪一种编辑样式，然后做对应的事情
*> 添加 :添加数据，刷新表格 insertRowsAtIndexPaths
*> 删除 :删除数据，刷新表格 deleteRowsAtIndexPaths

* cell 交换位置代理方法 moveRowAtIndexPath
> 表格交换位置 ， 数据交换位置
*>将源从数组中取出           id source = self.dataList[sourceIndexPath.row];
*>将源从数组中删除           [self.dataList removeObjectAtIndex:sourceIndexPath.row];
*>将源插入到数组中的目标位置   [self.dataList insertObject:source atIndex:destinationIndexPath.row];

```

```
自定义cell
* [self.contentView  addSubview:newView]; 如果想自定义cell，把要添加的view放在contentView中
* 代码自定义在initWithStyle里面做一些初始化创建，属性设置的操作。尺寸统一在layoutSubviews里面
* xib自定义在awakeFromNib里面设置
* 动态高度
> 返回cell高度的代理方法会在cell初始化之前调用.
*> 解决办法：
a. 设置两个模型，JNStatus(数据模型) 和JNStatusFrame(尺寸)模型。
b. 在JNStatusFrame 的set status方法里面做拦截，计算所有子控件的尺寸。
c. 在控制器中懒加载的时候 statusF.status = status; 尺寸模型就知道了一切。就早早的知道了高度.
> 优化：如果摸个控件一直时一成不变的，那么考虑懒加载。节省内存
> 数据和模型在一起。在模型里面加载数据.提供返回数据(加载plist)的方法。控制器值负责掉方法.大管家。不应该关心数据 MVC

* 用于在自定义Cell时，Cell被选中后的处理 : -(void)setSelected:(BOOL)selected animated:(BOOL)animated

* 问题
1> JNStatusFrame中的控件尺寸是根据内容计算的，外部不允许修改，应该设置为readonly
2> JNStatusFrame模型中已经包含了HMStatus模型数据，完全可以用来给Cell赋值，而无需再次创建
3> JNStatusFrame模型中已经计算好了每一个控件的尺寸，在Cell中应该可以直接使用，不需要再次计算

解决方法：
1> 将所有frame的属性设置为readonly
提示：
(1) 如果不重写getter方法，则无需使用@synthesize指令，此时_的成员变量是存在的
(2) 一旦重写了getter方法，带_的成员变量则会被覆盖

2> 添加类方法，从plist加载直接创建statusFrames的数据数组  +(NSArray *)statusFrames; // 不带参数的类工厂方法
```

```
代理为什么用weak?
假设自定义控件CustomView有一个代理属性delegate, 他的代理是控制器，意味着控制器对CustomView 强引用。 如果代理时strong，那么会发生循环引用，CustomView 代理属性强引用控制器,控制器强引用CustomView ，循环引用，造成内存泄露，没法释放控制器和CustomView。
```

```
QQ好友列表
* 模型中 的数组里又存放模型
> 代码
-(instancetype)initWithDict:(NSDictionary *)dict; // 好友组:(name online friends(里面装的时好友))
{
        if (self == [super init]) {
            self.name = dict[@"name"];
            self.online = dict[@"online"];
            NSMutableArray *array = [NSMutableArray array];
            // 遍历好友数组
            for (NSDictionary *di in dict[@"friends"]) {
            JNFriend *friend = [JNFriend friendWithDict:di];
            [array addObject:friend];
        }
        self.friends = array; // 面向模型开发
    }
    return self;
}

* BOOL的运用，标记一些状态
> 两种状态取反 self.group.open = !self.group.isOpen;
> 心得：
1> 一般用在控制器中拥有它，在某处置NO，那么在相反的情况出置YES,保证具体情况的准确性
2> 存在模型中，记录特殊情况下的状态，根据YES\NO 来做对应的操作

*block 的引用 view --> 控制器
1> typedef void(^headVewBlock)(id); // 参数传id 类型
2> 在某个view中拥有属性 @property(nonatomic,copy)headVewBlock block;
3> 在view里面 block 传值，通知做事情
if (self.block) {
    self.block(@"block 传值");  // 在内部的blcok里面传值，外面做block里事情
}
4> 在外面。控制器
view.block= ^(id sender){
    /** 要做的事情，在控制器里面*/
    NSLog(@"---%@",sender);
};

5> 内存泄漏问题
__weak JNViewController *weakSelf = self; // 注意这里需要*
__weak typeof(self) weakSelf = self; //这里不需要*

* UITableViewHeaderFooterView 也是封装好的，继承就好了
> headerView循环使用,循环使用的机制和cell 类似

* 小三角形转动的问题
self.headBtn.imageView.contentMode = UIViewContentModeCenter;
self.headBtn.imageView.clipsToBounds = NO;   超出控件边框范围的内容都剪掉

* UIView 方法
/**系统自动调用(留给子类去实现)**/
- (void)didAddSubview:(UIView *)subview;
- (void)willRemoveSubview:(UIView *)subview;
- (void)willMoveToSuperview:(UIView *)newSuperview;

// 当一个控件添加到父控件中系统自动调用。（例子：QQ箭头）
- (void)didMoveToSuperview;
- (void)willMoveToWindow:(UIWindow *)newWindow;
- (void)didMoveToWindow;
/**系统自动调用**/
```

```
runloop
* 每个线程都有一个runloop对象.
* 作用：程序一启动，在主线程中就开启(runloop)消息循环，保证程序的运行
* runloop 有一个输入源(input source)处理异步消息的,里面有个port的线程接口，用来处理其他线程的消息(performSelector: on thread:)
* 还有一个Timersource(用来定时接受处理一些主线程的ui事件)处理同步消息的;
```

```
copy
*堆：由编译器自动分配并释放，一般存放函数的参数值，局部变量等，代码块一过，会自动释放
*栈：由程序员负责管理,如果程序员不释放，操作系统会自动释放
* 内存管理
> 内存管理范围：只有oc对象才进行内存管理，基本数据类型不需要。
> 原因：oc对象存放在堆中，内存管理由程序员来管理，而基本数据类型存放在栈中，栈内存系统会自动回收.
*> 指针变量也存放在栈内存中.但指针变量指向的对象，存放在堆中[[Car alloc]init];alloc ->分配内存空间 init -> 值得一些初始化
*> 根据引用计数器(有多少人引用这个对象)来判断对象是否应该被回收，每一个oc对象内部都会分配4个字节的存储空间来存放引用计数器
*> block的copy策略
* 1.block 只能用copy 策略，相当于strong
* 2. 栈内存: 自动释放
* 3.堆内存 ：释放交给程序员管理
* 4.如果没有对block进行copy操作，block默认存储于栈空间
* 5.如果对block进行copy操作，block就存储于堆空间
* 6.如果block存储于栈空间，不会对block内部所用到的对象产生强引用
* 7.如果block存储于堆空间，就会对block内部所用到的对象产生强引用
* 8.栈时轻量级的，没有copy就在栈，不会发生强应用
* clang -rewrite-objc main.m 可以看出block 的底层实现

* 深复制：复制的是内容，即复制出来的是新对象
* 浅复制: 复制的是指针。并没有复制出新对象，只是复制的指针引用了这个对象
* copy 和strong 的区别
> 区别在于你是否想别人改变你的属性，如果是copy的话，那么再属性的setter 方法里面拷贝的副本出来，意味着，你只是修改拷贝出来的副本的属性，而不是自身的属性，而strong是可以修改自身的属性，因为修改的不是副本而是本身，是否用copy 取决于需求

* UI控件为什么用weak？
> 因为控制器有一个强指针引用这自己的view (self.View) View 有一个属性强引用着(self.subview)子控件数组。所以，当UI控件加入到self.view 的时候，子控件数组就有一个强指针引用这UI控件，所以，控制器没有必要对UI控件强引用，因为有self.subview
```

```
QQTalkDemo
tableview 小细节
* self.tableView.separatorStyle =UITableViewCellSeparatorStyleSingleLineEtched;
* self.tableView.keyboardDismissMode = UIScrollViewKeyboardDismissModeOnDrag; 拖拽tableView就会隐藏键盘
* self.tableView.allowsSelection = NO;  //不让选中
* 手势注意点：如果在一个可以滚动的view中，比如只能上下滚动的scrollView中添加上下轻扫的手势监听，这个事件会被scrollview
* 键盘处理：－>UIKeyboardWillShowNotification 监听键盘的通知
* 自定义键盘：self.inputField.inputView :(可以时datePick，等pickView)
* 键盘的工具条：inputField.inputAccessryView (ToolBar一般放在键盘里面的，只能加BarButtonItem)
> 换掉系统自带的键盘之后一定要退出键盘之后在弹出键盘
> CGRect frame = [note.userInfo[UIKeyboardFrameEndUserInfoKey] CGRectValue];
> UITextField 的leftView 可以设置自定义的view，要想显示，必须要设置leftViewMode
```

```
pickerViewDemo
* - (void)selectRow:(NSInteger)row inComponent:(NSInteger)component animated:(BOOL)animated
* numberOfComponentsInPickerView  // 有多少组:
* numberOfRowsInComponent // 每组有多少航
* didSelectRow // 选中了某一行
* titleForRow // 为某一行设置标题
* viewForRow // 自定义View
```

```
autolayout
iPhone4S(3.5英寸)       320 * 480
iPhone5 5S(4英寸)       320 * 568
iPhone6(4.7英寸)        375 * 667
iPhone6plus(5.5英寸)    414 * 736

*  autolayout 和autoresizing 在storyboard中只能选中其一
*  autoresizing 的不足：autoresizing只满足于父子关系的view的情况，但是满足不了相同层级view的约束(iPad开发用到)
*   满足不了两个view之间的间距是固定的需求
UIViewAutoresizingNone                 没有约束
UIViewAutoresizingFlexibleLeftMargin   左边可以伸缩，右边是固定的
UIViewAutoresizingFlexibleRightMargin  右边可以伸缩，左边是固定的
UIViewAutoresizingFlexibleTopMargin    顶部可以伸缩，底部是固定的
UIViewAutoresizingFlexibleBottomMargin 底部可以伸缩，顶部是固定的
UIViewAutoresizingFlexibleWidth      会随着父控件的尺寸改变而做相应的宽度改变
UIViewAutoresizingFlexibleHeight     会随着父控件的尺寸改变而做相应的高度改变

* autolayout(最好给view起别称，方便参照)
* 取消lauanch screen 的xib -> 项目配置-> 清楚launch screen file->launch images source(use asset catalog)->migarte
* autolayout细节
> 用了autolayout，那么不需要再设置frame，但是用了autolayout必须要设置：宽 高 x y 对应的约束
> 黄色警告： 代表控件当前的约束和实际的位置不匹配-> 一般是update frame
> 红色错误: 缺少某个frame 的尺寸约束 或者是两个约束的冲突，这里约束宽度100 和那里约束宽度110 之间的约束冲突
> top layout guide: 代表上面20的状态栏
> constrians to margins ->iOS8开始 系统从内部调节，为了视图美观，让控制器的view留一点间距。
> 不要装逼，一下子拖四五个view来搞。约束多，线太多很复杂！两个两个慢慢搞

*全屏图片适配技巧：首先图片的上下部分空出一些不重要的部分，设置图片水平垂直居中，然后设置图片左右间距为0,这样就能保证不管大小的屏幕都能让图片居中显示，头尾两边不重要的部分按情况显示
* 如果一个控件，默认有内容，autolayout会默认算出尺寸
* autolayout 动画：约束 连线，然后改变约束的constant 值，通过动画显示，注意：修改约束的值在动画内动画外写都是可以的，但要注意[self.view layoutIfNeed]self.view的拥有的约束。
* 代码实现autolayout
公式：view1.attr1 = view2.attr2 * multiplier + contant
button.left = self.view.left*0 + 20
button.right = self.view.width*1 + (-20)
注意：translateAutoresizingToConstraints这个属性默认是YES，这个属性代表：将视图默认自带的Auto resizing特性是否自动转换为对应的约束。既然使用代码来创建约束，那么就不要让系统自带的转换过来的约束影响添加的自定义约束，所以该属性要设置为NO，为了保证效果，可以将视图以及视图的父视图的该属性都设置为NO
```

```
Xocde6新特性
* 在新建项目的时候，少了empty空项目的选项
* 项目里面没有了pch文件，需要自己编写导入(build Setting 里面设置 profiex header->X6Test/profiex header.pch)
*Xcode6 取消类前缀，因为swift语言有命名空间这一说，类似于java的包。所以取消了类前缀，配置类前缀的方法(工程 - targets --class prefix)
* 工程新增了一个xib文件：LaunchScreen.xib，顾名思义，当程序启动的时候会调用到，和之前的Launch Image类似，这个xib用起来会更加的灵活.LaunchScreen.xib可以拖控件表示显示启动画面，不能设置file‘s owner 不能处理点击事件
* 在Images.xcassets新增了iPhone6和iPhone6+的启动图片和iPhone6+的横屏图片,Xocde6 直接支持jpg图片格式，并且能够像设置sb一样，设置图片的宽高(与sizeClass 类似)能够调整不同状态下显示不同图片 *表示任意的 - 表示压缩 + 表示正常
* Xocde6的控制器变成豆腐干形状，范围很大，是为了方便同时适配iPad 和iPhone，提供预览功能
* sizeClass 的引入。控制不同状态下，显示不同空间，有着类似revel的效果
```

```
iOS8新特性 --->最大的特色是为了统一，安全(隐私的授权)
UIAlertController
* UIPopoverPresentationController ->一定要先设置second.modalPresentationStyle = UIModalPresentationPopover;在设置sourceView
* UIPresentationController: 管理所有通过modal 出来的控制器
> 管理所有调用了-(void)presentViewController:(UIViewController *)viewControllerToPresent animated: (BOOL)flag completion:(void (^)(void))completion 方法的控制器
> UIPresentationController 的属性 p.presentedViewController  p.presentingViewController
* stroyBoard的seague 有所改变，show(push)

* UIPresentationController
1> 管理所有Modal出来的控制器
2> 管理所有通过- (void)presentViewController:(UIViewController *)viewControllerToPresent animated: (BOOL)flag completion:(void (^)(void))completion方法显示出来的控制器
3> 管理\监听切换控制器的过程
4> presentingViewController:后面的控制器
5> presentedViewController:前面的控制器
6> presentedView:前面的控制器的view

屏幕适配
1> 发展历程
代码计算frame -> autoreszing(父控件和子控件的关系) -> autolayout(任何控件都可以产生关系) -> sizeclass

2> sizeclass
* 仅仅是对屏幕进行了分类, 真正排布UI元素还得使用autolayout
* 不再有横竖屏的概念, 只有屏幕尺寸的概念
* 不再有具体尺寸的概念, 只有抽象尺寸的概念
* 把宽度和高度各分为3种情况
1) Compact : 紧凑(小)
2) Any : 任意
3) Regular : 宽松(大)
4) 符号代表
- : Compact
* : Any
+ : Regular
5) 继承性
* * : 其它8种情况都会继承
* - : 会被- - \ + -继承
+ * : 会被+ - \ + +继承
6) sizeclass和autolayout的作用
sizeclass:仅仅是对屏幕进行了分类
autolayout:对屏幕中各种元素进行约束(位置\尺寸)

```

```
程序启动原理

1.执行main函数
2.执行UIApplicationMain 函数
* 创建UIApplication对象
* 创建UIApplication的delegate对象
3.delegate对象开始处理(监听)系统事件 ->(没有storyboard情况   )
* 程序启动完毕的时候, 就会调用代理的application:didFinishLaunchingWithOptions:方法
* 在application:didFinishLaunchingWithOptions:中创建UIWindow
* 创建和设置UIWindow的rootViewController
* 显示窗口

3.(没有storyBoard)根据Info.plist获得最主要storyboard的文件名,加载最主要的storyboard
* 创建UIWindow
* 创建和设置UIWindow的rootViewController
* 显示窗口
applicationDelegate
>  applicationWillResignActive 程序失去焦点
>  applicationDidEnterBackground 程序进入后台
>  applicationWillEnterForeground 程序即将进入前台
>  applicationDidBecomeActive 程序获得焦点
>  applicationWillTerminate 程序即将退出
*  4. 结束程序 程序正常退出，UIApplicationMain 函数才会返回

```

```
控制器的生命周期
* loadView -> viewWillAppear -> viewDidLoad -> viewDidAppear -> viewWillDisappear ->viewDidDisappear
-->(收到内存警告) ->didReceiveMemoryWarning ->viewWillUnload ->(销毁view)->viewDidUnload--(如果需要再次显示这个view的时候)-->loadView

* loadView方法只会调用一次，换掉系统的view能用到
* 控制器的view创建采用懒加载(延迟加载)方式，用到了我才创建
* viewDidUnload 里面做了什么事情
* 一般用来内存管理，因为view已经不再了，那么里面的数据都应该清空
-(void)viewDidUnload{
    [super viewDidUnload];
    self.apps = nil; // 清空指针。控制器不会报错
    self.persons = nil;
//    [self.apps release]; // 内存管理
//    [self.persons release];
}

* [super didReceiveMemoryWarning]做了什么事情
> didReceiveMemoryWarning --> 系统会查看是否有一个view不用了(Is there a view?)
> YES -> 这个view能不能被销毁(can it be released?)
> YES ->viewWillUnload-->release the view (view== nil ,(outlet)subview == nil)
> viewDidUnload
```

```
私人通讯录Demo
* 监听文本框TextField文字改变的三种方法：代理，通知 [textField addTarget:](UIControlEventEditingChanged)继承UIControl
* 自动型segue ->自动跳转
* 手动型segue -> 要判断是否符合条件,然后执行segue (手动型segue必须要设置segue 的标示)
> [self performSegueWithIdentifier:@"login2contact" sender:@"testSeague"];  两个sender是一样的
> -(void)prepareForSegue: sender:这个方法必须在来源控制器里执行 执行跳转的一些准备工作，传值什么的
> segue的三个属性: 目标控制器 ，来源控制器，id标示
* 导航控制器pop的话，应该从栈中取出来(self.navigationController.viewControllers) 在pop，而不是创建alloc init
* 修改导航栏 (需要在info.plist配置 View controller-based status bar appearance == NO)
* 自定义分割线： 要想自定义分割线必须要自定义cell
* 设置UIView背景色为黑色，设置透明度。
* 在layoutSubviews里面设置尺寸(在awakeFromNib方法里设置不准确)
```

**item**
```objc
self.navigationController.navigationBar.barStyle = UIStatusBarStyleDefault;
// 修改了UIBarButtonItem 文字的颜色 箭头颜色
[self.navigationController.navigationBar setTintColor:[UIColor redColor]];
UIBarButtonItem *backItem = [[UIBarButtonItem alloc]init];
self.navigationItem.backBarButtonItem = backItem;

```

```
数据存储:plist 偏好设置 NSCoding归档
* plist 文件存储：能存储一些普通对象(字典，数组，NSString，NSNumber等，不能存储自定义对象，比如person)
>  存 [dict writeToFile:path atomically:YES]; (atomically:原子性操作YES，比较安全，先创建临时文件，直到完全写入才导入)
>  取  NSDictionary *dict = [NSDictionary dictionaryWithContentsOfFile:path];
* 偏好设置(preference)
> 存：NSUserDefaults *dfts = [NSUserDefaults standardUserDefaults];
[dfts setObject:@"江南" forKey:@"name"]; // 用一个key来存储对象，int，double，bool等
[dfts synchronize];  // 必须要发出同步命令，否则取数据的时候不准确
> 取：NSUserDefaults *dfts = [NSUserDefaults standardUserDefaults];
NSString *name = [dfts objectForKey:@"name"];

* NSCoding:能够存自定义对象，存取方法暴力。(注意一定要调用父类的方法)
> 存取的对象要 遵守NSCoding 协议、以JNStudent为例(内部存取实现)
// 每次从文件中恢复（解码）对象时，都会调用这个方法。制定如何解码文件中的数据为对象的实例变量
- (id)initWithCoder:(NSCoder *)decoder
{
    if ([super initWithCoder:decoder]) { // 有继承的话，要调用父类的方法，那么就能存下来
        self.no = [decoder decodeIntForKey:@"no"];
        self.school = [decoder decodeObjectForKey:@"school"];
        self.score = [decoder decodeIntForKey:@"score"];
    }
    return self;
}
// 每次归档对象时都会调用这个方法，制定如何归档对象中的实例变量，
- (void)encodeWithCoder:(NSCoder *)encoder
{
    [super encodeWithCoder:encoder]; // 有继承，用到父类的属性的话，一定要调用父类的方法，这样才能存取到父类的属性
    [encoder encodeObject:self.school forKey:@"school"];
    [encoder encodeInt:self.no forKey:@"no"];
    [encoder encodeInt:self.score forKey:@"score"];
}

>再外面存：
/*
+ (NSData *)archivedDataWithRootObject:(id)rootObject;
+ (BOOL)archiveRootObject:(id)rootObject toFile:(NSString *)path;
*/
JNStudent *stu = [JNStudent studentWithNo:10 score:100 school:@"北京大学"];
[NSKeyedArchiver archiveRootObject:stu toFile:JNStuPath];JNStuPath一般写成宏，这样存取方便

>再外面取：存是什么对象，取就是什么对象
+ (id)unarchiveObjectWithData:(NSData *)data;
+ (id)unarchiveObjectWithFile:(NSString *)path;
JNStudent *stu = [NSKeyedUnarchiver unarchiveObjectWithFile:JNStuPath];
* 数据存储的注意点：只要数据修改(添加，修改，删除)，就要保存数据。 一般配合工具类来一起使用
```

```
Quartz2D
* 要用Quartz2D绘图，需要自定义View，在View 的-drawRect方法里面做一些画图操作，drawRect方法系统会默认只调用一次。但是，如果这个自定义View没有尺寸Frame的话,这个方法就不会被调用
* [self setNeedsDisplay] 重绘
* 底层基于c语言的api. [UIBezierPath bezierPathWith..] 利用贝塞尔路径封装了一些基本图形的api
* API
> CGContextRef ctx = UIGraphicsGetCurrentContext(); 获得图形上下文
> CGContextMoveToPoint(ctx, 10, 10); 拼接路径（下面代码是搞一条线段）
> CGContextAddLineToPoint(ctx, 100, 100);

> CGContextAddRect(CGContextRef c, CGRect rect) 添加一个矩形>
> CGContextAddEllipseInRect(CGContextRef context, CGRect rect) 添加一个椭圆
(x,y)原点 radius：半径 startAngle：开始起点角度，注意要传弧度值  clockwise：0顺时针 1逆时针
> CGContextAddArc(CGContextRef c, CGFloat x, CGFloat y,CGFloat radius, CGFloat startAngle, CGFloat endAngle, int clockwise)添加一个圆弧
> CGContextStrokePath(ctx); // CGContextFillPath(ctx);绘制路径

> CGContextSaveGState(CGContextRef c) 将当前的上下文copy一份,保存到栈顶(那个栈叫做”图形上下文栈”)
> CGContextRestoreGState(CGContextRef c) 将栈顶的上下文出栈,替换掉当前的上下文

* 矩阵操作
> CGContextScaleCTM(CGContextRef c, CGFloat sx, CGFloat sy) 缩放
> CGContextRotateCTM(CGContextRef c, CGFloat angle) 旋转
> CGContextTranslateCTM(CGContextRef c, CGFloat tx, CGFloat ty)平移

* 图片水印
> UIGraphicsBeginImageContextWithOptions(CGSize size, BOOL opaque, CGFloat scale) 开启一个基于位图的图形上下文
> UIGraphicsGetImageFromCurrentImageContext(); 从上下文中取得图片（UIImage）
> UIGraphicsEndImageContext(); 结束基于位图的图形上下文
> NSData *data = UIImagePNGRepresentation(newImg);(图片不能写入文件，转换成二进制数据写入文件)

* 图片裁剪
> CGContextClip(CGContextRef c) 将当前上下所绘制的路径裁剪出来（超出这个裁剪区域的都不能显示）
> 思路: 先画大圆 -> 再画小圆 -> clip -> 画图片
> CGContextClip的意思是：前面画完的没有裁剪，clip代码一过，对后面的有裁剪效果，这就是为什么 小圆裁剪之后为什么还有外边框

CGContextAddArc(ctx, centre.x, centre.y, bigRadius, 0, M_PI * 2, 0); // 画大圆
[[UIColor redColor]set];
CGContextFillPath(ctx); // 大圆画完之后马上就要要渲染出来
CGContextAddArc(ctx, centre.x, centre.y, bigRadius - border, 0, M_PI * 2, 0); // 画小圆
CGContextClip(ctx); // 因为之前的都已经渲染画出去了，所以没有被剪掉，裁剪只针对后面
[oldImg drawInRect:CGRectMake(border, border, oldImgW, oldImgW)]; // 画图片
UIImage *newImg = UIGraphicsGetImageFromCurrentImageContext();

*截图-> 开启位图上下文，获取图片(注意针对图层操作)
> - (void)renderInContext:(CGContextRef)ctx; 调用某个view的layer的renderInContext:方法即可
```

```
涂鸦Demo
* 事件处理知识
UIView是UIResponder的子类，可以覆盖下列4个方法处理不同的触摸事件
一根或者多根手指开始触摸view，系统会自动调用view的下面方法
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event

一根或者多根手指在view上移动，系统会自动调用view的下面方法（随着手指的移动，会持续调用该方法）
- (void)touchesMoved:(NSSet *)touches withEvent:(UIEvent *)event

一根或者多根手指离开view，系统会自动调用view的下面方法
- (void)touchesEnded:(NSSet *)touches withEvent:(UIEvent *)event

触摸结束前，某个系统事件(例如电话呼入)会打断触摸过程，系统会自动调用view的下面方法
- (void)touchesCancelled:(NSSet *)touches withEvent:(UIEvent *)event

提示：touches中存放的都是UITouch对象 一根手指对应一个UITouch对象
当用户用一根手指触摸屏幕时，会创建一个与手指相关联的UITouch对象

UITouch的作用
保存着跟手指相关的信息，比如触摸的位置、时间、阶段
当手指移动时，系统会更新同一个UITouch对象，使之能够一直保存该手指在的触摸位置
当手指离开屏幕时，系统会销毁相应的UITouch对象


触摸产生时所处的窗口@property(nonatomic,readonly,retain) UIWindow    *window;

触摸产生时所处的视图@property(nonatomic,readonly,retain) UIView      *view;

短时间内点按屏幕的次数，可以根据tapCount判断单击、双击或更多的点击
@property(nonatomic,readonly) NSUInteger          tapCount;

记录了触摸事件产生或变化时的时间，单位是秒
@property(nonatomic,readonly) NSTimeInterval      timestamp;

当前触摸事件所处的状态
@property(nonatomic,readonly) UITouchPhase        phase;

返回值表示触摸在view上的位置 locationInView
这里返回的位置是针对view的坐标系的（以view的左上角为原点(0, 0)）
调用时传入的view参数为nil的话，返回的是触摸点在UIWindow的位置

该方法记录了前一个触摸点的位置 previousLocationInView:(UIView *)view;

UIEvent：称为事件对象，记录事件产生的时刻和类型，每产生一个事件，就会产生一个UIEvent对象

事件类型
@property(nonatomic,readonly) UIEventType     type;// 转场动画类型
@property(nonatomic,readonly) UIEventSubtype  subtype; // 方向

事件产生的时间
@property(nonatomic,readonly) NSTimeInterval  timestamp;

UIEvent还提供了相应的方法可以获得在某个view上面的触摸对象（UITouch）
4个触摸事件处理方法中，都有NSSet *touches和UIEvent *event两个参数
一次完整的触摸过程中，只会产生一个事件对象，4个触摸方法都是同一个event参数
如果两根手指同时触摸一个view，那么view只会调用一次touchesBegan:withEvent:方法，touches参数中装着2个UITouch对象
如果这两根手指一前一后分开触摸同一个view，那么view会分别调用2次touchesBegan:withEvent:方法，并且每次调用时的touches参数中只包含一个UITouch对象
根据touches中UITouch的个数可以判断出是单点触摸还是多点触摸

* 如果用到多条线的话，那么考虑多用贝塞尔曲线。一个贝塞尔曲线就相当于一条线里面点的数组，避免了一层数组的操作
* 如果贝塞尔曲线有各自自己的属性，那么考虑继承，创建一个类方法，创建的时候就有了属性
* UIImageWriteToSavedPhotosAlbum -> 写入图片(监听的代理方法用系统推荐的方法)
* 不要一味的封装，要想自定义控件变得牛逼灵活，要设计一些灵活好用的接口给别人调用. 面向对象的设计

* 事件的产生和传递
• 发生触摸事件后，系统会将该事件加入到一个由UIApplication管理的事件队列中
• UIApplication会从事件队列中取出最前面的事件，并将事件分发下去以便处理，通常，先发送事件给应用程序的主窗口（keyWindow）
• 主窗口会在视图层次结构中找到一个最合适的视图来处理触摸事件，这也是整个事件处理过程的第一步
• 找到合适的视图控件后，就会调用视图控件的touches方法来作具体的事件处理
> 注意点： 如果父控件不能接收到触摸事件，那么，子控件就不可能接收到事件
> 还有一种情况，如果你想父控件接收事件，注意不要让这个子控件是触摸事件吞掉(UIView里面的按钮，设置按钮不可交互)

* 不接收事件的三种情况
不接收用户交互 userInteractionEnabled = NO
隐藏 hidden = YES
透明 alpha = 0.0 ~ 0.01
提示：UIImageView的userInteractionEnabled默认就是NO，因此UIImageView以及它的子控件默认是不能接收触摸事件的

* 事件的完整处理过程
1. 先将事件对象由上往下传递(由父控件传递给子控件),找到最合适的控件来处理这个事件
2.调用最合适控件的touches...方法
3. 如果调用了[super touches..],就会将事件顺着相应这链条往上传递，传递给上一个响应者
4.接着就会调用上一个响应者的 touches..方法

* 响应者链条
1.响应者链条是由多个响应者对象连接起来的链条(什么是响应者对象：能够处理事件的对象)
2.利用响应者链条，能让多个控件处理同一个触摸事件
3.谁是上一个响应者
> 如果当前这个view是控制器的view，那么控制器就是上一个响应者
> 如果当前这个view不是控制器的view，那么父控件是上一个响应者

* 响应者链的事件传递过程
1.如果view的控制器存在，就传递给控制器；如果控制器不存在，则将其传递给它的父视图
2.在视图层次结构的最顶级视图，如果也不能处理收到的事件或消息，则其将事件或消息传递给window对象进行处理
3.如果window对象也不处理，则其将事件或消息传递给UIApplication对象
4.如果UIApplication也不能处理该事件或消息，则将其丢弃

事件处理底层的核心方法

-(BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event
-(UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event
* 方法调用过程
* 1. 当有了一个touchesBegan事件，那么，系统会遍历所有显示在屏幕上的子控件（从window开始一层一层遍历）,调用pointInside方法，询问，那个触摸点是不是你的。直到遍历到了控件返回了YES，停止遍历，找到了发生触摸事件的控件
* 2.然后调用hitTest方法，询问你，这事件是你的，你准备交给谁处理。返回的view就是处理事件的view
* 总之要想来到一个控件的hitTest方法，那么这个控件的pointInside一定是返回YES。因为pointInside是第一步
* 如果没有实现这两个方法，那么系统更具响应者链条来处理事件

```

```
CALayer
> CALayer负责视图中显示的内容和动画
> UIView负责监听和响应事件
> 属性:圆角、边框、阴影及3D形变
注意：UIImageView中不仅一个子图层，因此设置圆角时需要使用setMasksToBounds:YES，让所有子图层跟随边框，不过设置该属性后，无法使用阴影效果
解决办法：可以在底层附加一个UIView实现阴影效果

@property CGRect bounds; 宽度和高度
@property CGPoint position; 位置(默认指中点，具体由anchorPoint决定)
@property CGPoint anchorPoint; 锚点(x,y的范围都是0-1)layer最重要的属性:决定layer身上的哪一个点放在pos那个点上,默认(0.5,0.5)
@property CGColorRef backgroundColor; 背景颜色(CGColorRef类型)
@property CATransform3D transform; 形变属性
@property CGColorRef borderColor; 边框颜色(CGColorRef类型)
@property CGFloat borderWidth;  边框宽度
@property CGFloat cornerRadius; 圆角半径
@property(retain) id contents;  内容(比如设置为图片CGImageRef)
imgLayer.contents = (__bridge id)[UIImage imageNamed:@"阿狸头像"].CGImage; // 桥接转换
@property CGColorRef shadowColor;
@property float shadowOpacity;  阴影不透明(0.0 ~ 1.0)
@property CGSize shadowOffset;  阴影偏移位置

> 隐式动画
每一个UIView内部都默认关联着一个CALayer，称这个Layer为Root Layer。所有的非Root Layer都存在着隐式动画，隐式动画的默认时长为1/4秒。
当修改非Root Layer的部分属性时，相应的修改会自动产生动画效果，能执行隐式动画的属性被称为“可动画属性”，诸如:
bounds: 缩放动画
position: 平移动画
opacity: 淡入淡出动画（改变透明度）
在文档中搜素animatable可以找到所有可动画属性

如果要关闭默认的动画效果，可以通过动画事务方法实现：
[CATransaction begin]; // 取消系统默认的效果
[CATransaction setDisableActions:YES];
// ...
[CATransaction commit];

> 要在CALayer上绘图，有两种方法:
1.创建一个CALayer的子类，然后覆盖drawInContext:方法，可以使用Quartz2D API在其中进行绘图
2.设置CALayer的delegate，然后让delegate实现drawLayer:inContext:方法进行绘图

注意：不能再将UIView设置为这个CALayer的delegate，因为UIView对象已经是内部层的delegate，再次设置会出问题
无论使用哪种方法，都必须向层发送setNeedsDisplay消息，以触发相应绘图方法的调用
```

```
核心动画
* 开发步骤:
1.初始化一个动画对象(CAAnimation)并设置一些动画相关属性
2.CALayer中很多属性都可以通过CAAnimation实现动画效果，包括：opacity、position、transform、bounds、contents等（可以在API文档中搜索：CALayer Animatable Properties）
3.添加动画对象到层（CALayer）中，开始执行动画
4.通过调用CALayer的addAnimation:forKey增加动画到层（CALayer）中，这样就能触发动画了。通过调用removeAnimationForKey可以停止层中的动画
5.Core Animation的动画执行过程都是在后台操作的，不会阻塞主线程 (刷新UI界面再主线程中)

* CAAnimation
是所有动画对象的父类，负责控制动画的持续时间和速度，是个抽象类，不能直接使用，应该使用它具体的子类
属性说明：(所有子类的动画都有这些属性)
duration：动画的持续时间
repeatCount：重复次数，无限循环可以设置HUGE_VALF或者MAXFLOAT
repeatDuration：重复时间
removedOnCompletion：默认为YES，代表动画执行完毕后就从图层上移除，图形会恢复到动画执行前的状态。如果想让图层保持显示动画执行后的状态，那就设置为NO，不过还要设置fillMode为kCAFillModeForwards
fillMode：决定当前对象在非active时间段的行为。比如动画开始之前或者动画结束之后
beginTime：可以用来设置动画延迟执行时间，若想延迟2s，就设置为CACurrentMediaTime()+2，CACurrentMediaTime()为图层的当前时间
timingFunction：速度控制函数，控制动画运行的节奏
delegate：动画代理

*CABasicAnimation (功能不强大,只能从一个数值（fromValue）变到另一个数值（toValue）)
basic.keyPath = @"position";
NSValue *toValue = [NSValue valueWithCGPoint:CGPointMake(200, 200)]; basic.toValue = toValue;
basic.removedOnCompletion = NO; // 默认一执行完动画就移除动画，取消系统自带的效果
basic.fillMode = kCAFillModeForwards;

# key的作用
1.keyAnimate.keyPath = @"transform.translation.x";可以当做keypath来用，但是keypath功能比key强
2.用来 当做移除动画时的标记key
[self.layer removeAnimationForKey:@"shake"];
[self.layer addAnimation:basic forKey:nil];

* CAKeyframeAnimation: 关键帧动画
values：NSArray对象。里面的元素称为“关键帧”(keyframe)。动画对象会在指定的时间（duration）内，依次显示values数组中的每一个关键帧
path：可以设置一个CGPathRef、CGMutablePathRef，让图层按照路径轨迹移动。path只对CALayer的anchorPoint和position起作用。如果设置了path，那么values将被忽略
keyTimes：可以为对应的关键帧指定对应的时间点，其取值范围为0到1.0，keyTimes中的每一个时间值都对应values中的每一帧。如果没有设置keyTimes，各个关键帧的时间是平分的
* CABasicAnimation可看做是只有2个关键帧的CAKeyframeAnimation

* CAAnimationGroup——动画组(多个动画组合起来,可以做出很炫的效果)
animations：用来保存一组动画对象的NSArray
默认情况下，一组动画对象是同时运行的，也可以通过设置动画对象的beginTime属性来更改动画的开始时间

*CATransition:转场动画，能够为层提供移出屏幕和移入屏幕的动画效果。转场动画执行过程中无法交互
UINavigationController就是通过CATransition实现了将控制器的视图推入屏幕的动画效果
动画属性:
type：动画过渡类型
subtype：动画过渡方向(最好设置出你想要的效果)
startProgress：动画起点(在整体动画的百分比)
endProgress：动画终点(在整体动画的百分比)

#warning 注意点 ：核心动画都是假象，不能改变layer的真实属性的值。展示的位置和实际的位置不同。实际位置永远在最开始位置
UIView封装的转场动画 方法
> 单视图
+ (void)transitionWithView:(UIView *)view duration:(NSTimeInterval)duration options:(UIViewAnimationOptions)options animations:(void (^)(void))animations completion:(void (^)(BOOL finished))completion;
duration：动画的持续时间
view：需要进行转场动画的视图
options：转场动画的类型
animations：将改变视图属性的代码放在这个block中
completion：动画结束后，会自动调用这个block
```





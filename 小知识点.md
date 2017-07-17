# iOS-Collection
个人对于iOS的总结

**1. 第一响应者:叫出键盘的叫做第一响应者**

```
* 辞去第一响应者(退出键盘)：[self.textView resignFirstResponder]
* 成为第一响应者(交出键盘):  [self.textView becomeFirstResponder]
* 叫出键盘的另一种方式: - (BOOL)endEditing:(BOOL)force; // 让view 或者view的子控件辞去第一响应者的职务
* 注意点:在iPad中退出键盘-->如果为formsheet 的话，那么直接退出键盘的话，是达不到效果的
[searchBar endEditing:YES];
- (BOOL)disablesAutomaticKeyboardDismissal ->return NO; //需要实现这个方法
Default implementation affects UIModalPresentationFormSheet visibility.
```

**2.UIView 的动画**
```
* 首尾式动画:注意，应该在开启动画之后立即设置动画的一些属性(时长，重复次数，代理放动画内部设置能够监听到动画的结束)
[UIView beginAnimations:(NSString *) context:(void *)]  -> 中间放执行动画的内容 void *(指向任何数据类型) --> id
[UIView setAnimationDelegate:self]; // 可以监听到动画的开始和动画的结束
[UIView commitAnimations]];
* block式动画：比较方便，可以延迟动画执行，监听到动画执行完之后做事情
[UIView animateWithDuration: animations:^ { code}];
[UIView animateWithDuration: animations:<#^(void)animations#> completion:<#^(BOOL finished)completion#>];
```

**3.枚举**
```
如果多处代码大部分时相同的，只有在不同情况下那么少数代码不同，考虑抽取出方法出来，相同的代码放在方法中，不同的值当做参数。根据不同的情况需求(枚举类型)来做不同的处理
```

**4.UIView 的属性**
```
 frame bounds center Transfrom
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

**5. 标准的枚举写法**
```
* 枚举类型本质上就是整数，定义的时候，如果只指定了第一个数值，后续的数值会依次递增
typedef enum {
    kMovingDirTop = 10,
    kMovingDirBottom,  // 11
    kMovingDirLeft,
    kMovingDirRight,
} kMovingDir;
```

**6.图片浏览器Demo**
```
* 看好plist，plist的根节点是什么就创建什么。
* KVC字典转模型 [self setValuesForKeysWithDictionary:dict];
* 懒加载(延迟加载)的好处
重写getter方法可以完成懒加载模式。代码的可读性强.注意循环引用的问题
用到时再加载,节省了不必要的内存资源,不需要关心什么时候去创建
只保证创建一次。内存中只有这一个对象.(如果经常用到UI控件，并且都是一样的，不需要重复创建，可以采取懒加载(特例UI控件用strong))
懒加载要注意一个清空问题，如果你移除了UI控件，没有清空，但还要重新用重新创建，不清空有可能过不去创建的代码
```

**7.帧动画(针对imgView)**
```
1.最重要的一点是清除缓存
[self.imgView performSelector:@selector(setAnimationImages:) withObject:nil afterDelay:self.imgView.animationDuration + 1];
[UIImage imageWithContentsOfFile:] ->用图片在沙盒的路径加载，没有缓存(缓存会经过一些操作后清楚)
[UIImage imageNamed:] -> 图片所占用的内存会一直在程序中，有缓存
2.%02d --> 不够两位补零
```

**8.Xcode 路径问题**
```
(iOS8开始 沙盒和bundle位置安装路径改变 :bundle.path打印一下就ok)
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

**9.@property 作用**
```
生成getter方法  生成setter方法
生成带下划线的成员变量（记录属性内容）
readonly的属性不会生成带下划线的成员变量！要想访问需要这样写{int _age} 这样访问

```

**10.@synthesize**
```
@synthesize合成指令，主动指定属性使用的成员变量名称 ->@synthesize image = _image;
```

**11.超级猜图Demo**
```
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

**12.UIScrollViewDemo**
```
* bounces:弹簧效果 scrollEnabled:能否滚动 show-:是否显示横竖滚动条
* scrollView的方法很简单，只需要添加scrollView，并且设置contentSize
* contentOffset:表示UIScrollView滚动的位置, 内容左上角和scrollView左上角的间距
* contentInset: 表示能够在UIScrollView的4周增加额外的滚动区域
* 如果在Storyboard中添加了ScrollView的子控件,要想scrollView滚动,必须取消autolayout)
* scrollView实现缩放要设置图片缩放的最大最小尺寸，并且在viewForZoomingInScrollView 方法中返回需要缩放的View，知道缩放那个view才能缩放
* 自带的方法 滚到某个位置 -(void)setContentOffset:(CGPoint)contentOffset animated:(BOOL)animated;

```

**13.UITableView 多组汽车Demo**
```
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

**14.UIAlertView**
```
* alertView.alertViewStyle 弹出样式
* [alertView textFieldAtIndex:0] 获取文本框样式下的文本框
* alertView.tag = indexPath.row;在Alert代理方法里用,用alertView.tag记录对应行号的模型,把上面的行数传递给下面
* 内部有很多私有属性->通过KVC获取
```

**15.数据刷新**
```
数据刷新 :永恒的两步走
* 修改模型数据
* 刷新表格-->重新加载数据
全局刷新：[self.tableView reloadData]
局部刷新[self.tableView reloadRowsAtIndexPaths: withRowAnimation:]

```

**16.表格处理**
```
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

**17.自定义cell**
```
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

**18.代理为什么用weak?**
```
假设自定义控件CustomView有一个代理属性delegate, 他的代理是控制器，意味着控制器对CustomView 强引用。 如果代理时strong，那么会发生循环引用，CustomView 代理属性强引用控制器,控制器强引用CustomView ，循环引用，造成内存泄露，没法释放控制器和CustomView。
```

**19.QQ好友列表**
```
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

**20.runloop**
```
* 每个线程都有一个runloop对象.
* 作用：程序一启动，在主线程中就开启(runloop)消息循环，保证程序的运行
* runloop 有一个输入源(input source)处理异步消息的,里面有个port的线程接口，用来处理其他线程的消息(performSelector: on thread:)
* 还有一个Timersource(用来定时接受处理一些主线程的ui事件)处理同步消息的;
```

**21.copy**
```
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

**21.QQTalkDemo**
```
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

**22.pickerViewDemo**
```
* - (void)selectRow:(NSInteger)row inComponent:(NSInteger)component animated:(BOOL)animated
* numberOfComponentsInPickerView  // 有多少组:
* numberOfRowsInComponent // 每组有多少航
* didSelectRow // 选中了某一行
* titleForRow // 为某一行设置标题
* viewForRow // 自定义View
```

**23.autolayout**
```
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

**24.Xocde6新特性**
```
* 在新建项目的时候，少了empty空项目的选项
* 项目里面没有了pch文件，需要自己编写导入(build Setting 里面设置 profiex header->X6Test/profiex header.pch)
*Xcode6 取消类前缀，因为swift语言有命名空间这一说，类似于java的包。所以取消了类前缀，配置类前缀的方法(工程 - targets --class prefix)
* 工程新增了一个xib文件：LaunchScreen.xib，顾名思义，当程序启动的时候会调用到，和之前的Launch Image类似，这个xib用起来会更加的灵活.LaunchScreen.xib可以拖控件表示显示启动画面，不能设置file‘s owner 不能处理点击事件
* 在Images.xcassets新增了iPhone6和iPhone6+的启动图片和iPhone6+的横屏图片,Xocde6 直接支持jpg图片格式，并且能够像设置sb一样，设置图片的宽高(与sizeClass 类似)能够调整不同状态下显示不同图片 *表示任意的 - 表示压缩 + 表示正常
* Xocde6的控制器变成豆腐干形状，范围很大，是为了方便同时适配iPad 和iPhone，提供预览功能
* sizeClass 的引入。控制不同状态下，显示不同空间，有着类似revel的效果
```

**25.iOS8新特性**
```
最大的特色是为了统一，安全(隐私的授权)
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

**25.程序启动原理**
```
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

**26.控制器的生命周期**
```
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

**27.私人通讯录Demo**
```
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

***item***
```objc
self.navigationController.navigationBar.barStyle = UIStatusBarStyleDefault;
// 修改了UIBarButtonItem 文字的颜色 箭头颜色
[self.navigationController.navigationBar setTintColor:[UIColor redColor]];
UIBarButtonItem *backItem = [[UIBarButtonItem alloc]init];
self.navigationItem.backBarButtonItem = backItem;

```

**28.数据存储**
```
plist 偏好设置 NSCoding归档
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

**29.Quartz2D**
```
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


**30.事件处理**
```
* 
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

**31.CALayer**
```
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

**32.核心动画**
```
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


**33.网易彩票Demo**
```
1. 类似于QQ那种主流框架，项目的起点都是tabbar控制器，导航控制器，自定义tabbar开始(继承UIView)
2. 再系统自带的tabbar里面添加自定义的Mytabbar,大师特别注意的是 要在viewWillAppear 方法里面，遍历系统自带的tabbar。 删掉UITabbarButton (继承自UIControl)
3. 控制器父类的抽取，self继承注意点，子类的各有特色
4. 宏的引用
#define JNUserDefaults [NSUserDefaults standardUserDefaults]
// 加载json 对象
#define ILJson(name) [NSJSONSerialization JSONObjectWithData:[NSData dataWithContentsOfURL:[[NSBundle mainBundle] URLForResource:@#name withExtension:nil]] options:NSJSONReadingAllowFragments error:nil]
5. 抽取存储的工具类，内部封装存取的两个方法. 注意self.title 作为key的妙用
6. 应用的操作(打电话，发短信，打开iTunes 等)
7. CollectionViewController
* 必须要有流水布局，重写init方法，从内部拦截设置流水布局
- (id)init
{
    UICollectionViewFlowLayout *layout = [[UICollectionViewFlowLayout alloc]init];
    if (self =[super initWithCollectionViewLayout:layout]) {
        layout.itemSize = CGSizeMake(80, 80);// 拿到布局很重要，可以设置很多属性
        layout.minimumInteritemSpacing = 10; // 水平方向的间距
        layout.minimumLineSpacing = 10;  // 垂直方向的间距
        layout.sectionInset = UIEdgeInsetsMake(20, 0, 0, 0);
    }
    return self;
}
* 注册代码cell [self.collectionView registerClass:[JNProductCell class] forCellWithReuseIdentifier:reuseIdentifier]
* 注册xib加载的cell [self.collectionView registerNib:nib forCellWithReuseIdentifier:JNProductCellID];
* 小坑：collectionView默认是cell 个数超出了范围才会滚动，但是如果不够，下拉刷新的情况下，设置self.collectionView.alwaysBounceHorizontal = YES 就可以了
```

**34.Core Fundation **
```
Core Fundation : 基于c语言的一套框架
> Core Fundation 中的数据类型一般和Fundation 都是可以互相转换的，这里用到桥接转换(不同框架之间架起一个桥梁,类似于强制类型转换，只不过要加上__bridge 表示不同框架的类型转换)
NSString *str = @"haha";
CFStringRef str2 = (__bridge CFStringRef)str;
NSString *str3 = (__bridge NSString*)str2;
> Core Fundation 中的数据类型创建(含有creat,copy,new)一般都要自己手动释放CFRelease();
/** 桥接总结：
* Foundation 和 Core  Foundation 相互转换. 桥接
* 以后在使用C语言的函数时, 只要函数名称包含creat/copy/retain 就必须自己手动释放CFRelease，因为c语言的东西不归OC的ARC管理
* 桥接转换再MRC情况下： 直接强制转换
* CFStringRef strC = (CFStringRef)strOC;
* NSString *strOC = ( NSString *)strC;
*
* 桥接转换再MRC情况下： 桥接转换  __bridge / __bridge_retained / __bridge_transfer
* __bridge : CFStringRef strC = (__bridge CFStringRef)strOC
> 不会转移对象的所有权，意味着strOC 释放不能用了，strC就不能用
* __bridge_retained: CFStringRef strC = (__bridge_retained CFStringRef)strOC;
> 会转移对象的所有权，意味着strOC 释放了，strC 还能用，
> 但是注意，虽然能用，strC 必须要手动释放，c语言的东西不归arc 管理，需要自己释放
> CFStringRef strC = CFBridgingRetain(strOC);// retain 给你了，表明strC持有拥有权，内存归自己管理要自己手动释放
* __bridge_transfer: NSString *strOC = (__bridge_transfer NSString *)strC;
> C-->OC:它会将对象的所有权转移给strOC, 也就是说, 即便strC被释放了, strOC也可以使用
> 他会自动释放strC, 也就是以后我们不用手动释放strC
> NSString *strOC = CFBridgingRelease(strC); // 转换给你，我会自动释放
*
*/
```

**35.static 关键字作用**
```
*  static 修饰全局变量
>  会让这个全局变量只在当前的这个文件中能使用，其他文件不能使用。
>  举例： 单例static ，只在类的.m 文件中声明，别的文件中不能使用修改我的全局变量,如果不加static，那么别人可以拿到我全局变量修改，这样的话就失去了单例的意义
*  static 修饰局部变量
>  修饰局部变量 :
>  局部变量的生命周期 跟 全局变量 类似,但是不能改变作用域
>  能保证局部变量永远只初始化1次，在程序运行过程中，永远只有1分内存
>  举例： tableview cell 重用中，修饰了局部的ID 变量，这样保证这个ID 只初始化一次，只有一次内存
```

**36.extern关键字**
```
> extern 是 C/C++语言中表明函数和全局变量作用范围（可见性）的关键字，该关键字告诉编译器，
其声明的函数和变量可以在本模块或 其它模块中使用。
```

**37.图片的加载**
```
[UIImage imageNamed:@"home"];  加载png图片

一、非retina屏幕
1、3.5 inch（320 x 480）
* home.png

二、retina屏幕
1、3.5 inch（640 x 960）
* home@2x.png

2、4.0 inch（640 x 1136）
* home-568h@2x.png（如果home是程序的启动图片，才支持自动加载）

三、举例（以下情况都是系统自动加载）
1、home是启动图片
* iPhone 1\3G\3GS -- 3.5 inch 非retina ：home.png
* iPhone 4\4S -- 3.5 inch retina ：home@2x.png
* iPhone 5\5S\5C -- 4.0 inch retina ：home-568h@2x.png

2、home不是启动图片
* iPhone 1\3G\3GS -- 3.5 inch 非retina ：home.png
* iPhone 4\4S -- 3.5 inch retina ：home@2x.png
* iPhone 5\5S\5C -- 4.0 inch retina ：home@2x.png

3、总结
* home.png ：3.5 inch 非retina
* home@2x.png ：retina
* home-568h@2x.png ：4.0 inch retina + 启动图片

```


**38.创建了一个控件，就是看不见**
```
1.当前控件没有添加到父控件中
2.当前控件的hidden = YES
3.当前控件的alpha <= 0.01
4.没有设置尺寸（frame.size、bounds.size）
5.位置不对（当前控件显示到窗口以外的区域）
6.背景色是clearColor
7.当前控件被其他可见的控件挡住了
8.当前控件是个显示图片的控件（没有设置图片\图片不存在，比如UIImageView）
9.当前控件是个显示文字的控件（没有设置文字\文字颜色跟后面的背景色一样，比如UILabel、UIButton）
10.检查父控件的前9种情况

一个控件能看见，但是点击后没有任何反应：
1.当前控件的userInteractionEnabled = NO
2.当前控件的enabled = NO
3.当前控件不在父控件的边框范围内
4.当前控件被一个背景色是clearColor的控件挡住了
5.检查父控件的前4种情况
6.。。。。。。
文本输入框没有在主窗口上：文本输入框的文字无法输入
```

**39.父子控制器的问题 **
```
* 如果发现：控制器的view还在，但是view上面的数据不显示，极大可能是因为：控制器被提前销毁了
* 一个控制器的view是可以随意调整尺寸和位置的
* 一个控制器的view是可以随意添加到其他view中
* 如果将一个控制器的view，添加到其他view中显示，那么要想办法保证控制器不被销毁
* 原则：只要view在，view所在的控制器必须得在，这样才能保证view内部的数据和业务逻辑正常
* 方案： self.childViewControllers(属性强引用着，并且并且，添加的控制器成为了self 的子控制器) [self addChildViewController:]
* 规范： 如果两个控制器的互为父子关系，那么，控制器的view 一定要是父子关系，不然父子控制器之间的交互会受很大的影响(监听屏幕旋转等(如果子控制器的view 显示在了父控制器的view上面，那么子控制器就能监听到父控制器的屏幕旋转,没有显示子控制器view的不会通知))
```

**40.const**
```
const的用法: 看const 修饰的是什么 
* const int age = 10; // 表示age只读并且不能修改
* int const age = 10; // 和上面的一样
* const NSString *name = @"tom"; *name是只读，不能修改,，但name指针可以修改
* NSString *const name = @"tom"; name指针只读，不能修改，一般用这一个来修饰一些常量，指针指向的数是可以修改的，但指针是不可修改的
* 用法 再JNConst.h 声明, .m文件实现
> UIKIT_EXTERN const CGFloat JNHeight
> const CGFloat JNHeight = 1.65;
> 在一个函数声明中，const 可以修饰形参，表明它是一个输入参数，在函数内部不能改变其值；
> 对于类的成员函数，若指定其为 const 类型，则表明其是一个常函数，不能修改类的成员变量；
```

**41.lazy init**
```
-(UILabel *)msgLabel
{
    if (!_msgLabel) {
    _msgLabel = ({
        UILabel *msgLabel = [UILabel new];
        msgLabel.textAlignment = NSTextAlignmentCenter;
        msgLabel;
        });
    }
    return _msgLabel;
}

```

**42.小技巧**
```
1.自定义了leftBarbuttonItem左滑返回手势失效了怎么办?
self.navigationItem.leftBarButtonItem = [[UIBarButtonItem alloc]initWithImage:img style:UIBarButtonItemStylePlain target:self action:@selector(onBack:)];
self.navigationController.interactivePopGestureRecognizer.delegate = (id<UIGestureRecognizerDelegate>

2.TableView不显示没内容的Cell怎么办?
self.tableView.tableFooterView = [[UIView alloc] init];

3.ScrollView莫名其妙不能在viewController划到顶怎么办?
self.automaticallyAdjustsScrollViewInsets = NO;

4.键盘事件写的好烦躁,使用IQKeyboardManager(github上可搜索)
5.app滑动流畅 KMCGeigerCounter

6怎么在不新建一个Cell的情况下调整separaLine的位置?
_myTableView.separatorInset = UIEdgeInsetsMake(0, 100, 0, 0);

7.导航条返回键带的title太讨厌了,怎么让它消失!

[[UIBarButtonItem appearance] setBackButtonTitlePositionAdjustment:UIOffsetMake(0, -60) forBarMetrics:UIBarMetricsDefault];

8.CoreData--> MagicalRecord

9.CollectionView 怎么实现tableview那种悬停的header?
CSStickyHeaderFlowLayout

注：参考叶孤城简书tip
```

**43.新浪微博项目总结**
```
1.授权
> 第一步（利用webView 加载 授权页面）
*  URL：https://api.weibo.com/oauth2/authorize
请求参数：
* client_id：申请应用时分配的AppKey
* redirect_uri：授权成功后的回调地址
> 第二步（webView 代理方法中，拦截回调页面的加载，截取毁掉地址后面的code码。那其实就是requestToken(code)）
> 第三步：发送请求，获取AccessToken
* URL：https://api.weibo.com/oauth2/access_token
* 请求参数：
* client_id：申请应用时分配的AppKey
* client_secret：申请应用时分配的AppSecret
* grant_type： 填写 authorization_code
* redirect_uri：授权成功后的回调地址
* code：授权成功后返回的requestToken（截串的code）

2. 工具类的使用
> 加载表情的一些操作不应该再控制器中完成，要避免给予控制器的压力，这个时候应该抽取工具类来专门的加载表情
> 表情info。plist的加载，注意，当文件托在项目中，Xcode默认生成的是虚拟文件夹，如果遇到相同的文件名，再不同的文件夹中，应该creat foler reference,这个时候文件再bundle 中就是以文件夹的目录情况显示
>   因为这是一个类方法，不是面向对象的方法。所以不能够通过self.recent 拿到这个数组,保证内存中只有一个数组，因为没必要每次都创建或者访问

static NSMutableArray *_recent; // 因为这是一个类，不是实力变量，不能通过self.recent 来获取成员变量，面向的是类方法，而不是对象方法。这个时候考虑到这样写法。同时也保证了内存中只有这一个对象，每次修改我都保存了
+(void)initialize  //  当使用到这个类的时候就会调用一次,再这里进行懒加载，牛逼
{
    _recent = [NSKeyedUnarchiver unarchiveObjectWithFile:JNEmotionFilePath];
    if (_recent == nil) {
        _recent = [NSMutableArray array];
    }
}

* 数组遍历删除问题
* 最好不要再遍历数组的的同时删除数组的元素，因为，遍历数组时刻依赖着数组的个数，而个数影响着次数，所以即使要遍历删除，应该要时刻监听着数组个个数，for 循环应该写arr.count(保证每次都能拿到准确的数组格式)
* removeObject 内部会调用isEquall 的方法，来判断你要删除的数组元素和我数组里面的对象是否是同一个对象，是同一个对象的话我才给你删除，但是有时候不一定要删除的是同一个对象，你数组元素和我数组元素数组相同(emotion情况)的特殊需求我也要删除，这个时候考虑到重写对象的isEquall 的方法

-(BOOL)isEqual:(JNEmotion *)e
{
    [super isEqual:e];
// 改变系统自带的判断方法 isEqualToString 判断的是内容， == 判断的是否是同一个对象
    return [self.chs isEqualToString:e.chs] || [self.code isEqualToString:e.code];
}

> 存储数据一般都使用工具类封装起来,再内部实现存和取的方法，统一管理。
* MJCodingImplementation 内部实现了存取的code 方法.内部的原理是利用runtime 来获取ivarlist 列表，然后遍历做一些归档解档的操作

> 网络请求的工具类，为了避免项目对框架的依赖，自己再套一层外套 (注意block 做回调参数的写法)
+ (void)get:(NSString *)url parames:(NSDictionary *)parame  success:(void(^)(id responseObj))success failure:(void(^)(NSError *error))failure;
> 但是大多数情况下，如果在控制器中利用HTTPTool工具类发送网络请求，控制器还是知道了太多了，而且也应该面向模型开发。这个时候，应该在抽取一个业务类。业务类专门做自己的业务。项目的层次结构性强、

3.良好的见哦按交互体验应该在viewDidAppear 方法里面弹出键盘 viewWillDisappear 退出键盘，如果在viewDidload 里面，或造成一些卡的情况，因为在加载键盘造成卡顿
* 键盘处理，监听键盘的通知
* self.textView.inputAccessoryView  // inputAccessoryView是工具条.
* self.textView.inputView  是键盘 // 自定义键盘
* 键盘的切换。 根据self.textView.inputView == nil 来判断键盘是系统自带的还是自定义键盘.

if (self.textView.inputView == nil) { // 用的是系统自带的键盘
    self.toolbar.showKeyboard = YES;
    self.textView.inputView = self.emotionKeyboard;
}else{ // 已经是表情键盘了
    self.textView.inputView = nil; // 清空，方便下次判断
    self.toolbar.showKeyboard = NO;
}
[self.textView endEditing:YES];
#warning 更换了系统自带的键盘，那么必须要退出键盘之后再重新叫出键盘,不然键盘不显示
// 延迟0.1s 左动画
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(0.1 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
    [self.textView becomeFirstResponder]; // 键盘的弹出动画。
});

*  BOOL 的巧妙应用可是实现加锁的情况，再特殊的情况代码前后置 YES 和 NO, 再判断的地方做YES 或者NO 的return 操作.
* [self.textView deleteBackward]; // 系统自带的删除功能，会在光标后面删除
* [self insertText:string]; 会自动再光标后面凭借字符(emoji本质是字符串)

*  调用系统自带的 相册，相机，因为代码很相似，可以考虑将sourceType抽取出来，打开不同的情况

```

```objc
-(void)openCamera
{
    // 判断是否支持照相功能（只有真机才能打开相机，不然会crash）
    if (![UIImagePickerController isSourceTypeAvailable:UIImagePickerControllerSourceTypeCamera])  return; // 不支持那么直接返回
    #warning 如果觉得每次选图片都只能选一张，可以用collectionView来实现多选，通过assetsFramework 框架能拿到系统的图片url
    JNPhotoController *photoVC = [[JNPhotoController alloc]init];
    photoVC.sourceType = UIImagePickerControllerSourceTypeCamera;
    photoVC.allowsEditing = YES;
    photoVC.delegate = self;
    self.modalPresentationStyle = UIModalPresentationOverCurrentContext;
    [self presentViewController:photoVC animated:YES completion:nil];
}

```

**44.弹出菜单的动画**
```objc
-(void)makeOpen
{
    // 每隔按钮隔0.1秒做动画
    [self.menubtns enumerateObjectsUsingBlock:^(JNComposeMenuButton *btn, NSUInteger idx, BOOL *stop) {
            [UIView animateWithDuration:0.5 delay:0.05 * idx usingSpringWithDamping:0.7 initialSpringVelocity:0 options:UIViewAnimationOptionCurveLinear animations:^{
            btn.y -= 300;
        } completion:^(BOOL finished) {
        }];
    }];
}

```

**版本新特性的判断 : 拿到上次的版本号于当前的版本号相比较，不同就显示新特性**
```objc
+(void)chooseController
{
    // 版本号的key
    NSString *versionKey = (__bridge NSString*)kCFBundleVersionKey;

    // 取出上次的版本号
    NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
    NSString *lastVersion = [defaults objectForKey:versionKey];

    // 取出当前版本号
    NSString *currentVersion = [NSBundle mainBundle].infoDictionary[versionKey];

    // 取出window
    UIWindow *window = [UIApplication sharedApplication].keyWindow;

    // 判断 。版本号不相同就显示新版本
    if (![lastVersion isEqualToString:currentVersion]) { // 不相同
    window.rootViewController = [[JNNewFeatureController alloc]init];
    }else{ // 相同,微博界面
    window.rootViewController = [[JNTabbarController alloc]init];
}
    // 存储版本号
    [defaults setObject:currentVersion forKey:versionKey];
    [defaults synchronize]; // 同步
}

```

**45.坐标系转换**
```
坐标系转换： 最重要的是找准坐标系的坐标原点 bounds 是以自己的左上角为原点，自身就是一个坐标系。不用bounds 那么就要有superView
```

**46.拦截代理的setter方法，有代理才做事情**
```objc
- (void)setDelegate:(id<JNEmotionBarDelegate>)delegate
{
    _delegate = delegate;
    #warning 刚创建时候self 的代理为为空，所以再setdelegate 里面拦截，有代理我才选中
    [self btnClick:self.defaultButton];
}

```

**47.系统的自动渲染**
```objc
UIImage *img = [UIImage imageNamed:emotion.png];
img = [img imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal] // 系统会自动渲染成蓝色的，所以不需要渲染，总是保持原生的

```

```
* 避免控件之间的循环引用, 不用的隐藏，用的显示，有YES一定要有NO
#warning 避免循环引用的出现还有一个比较好的方案是告诉模型，是否显示或者隐藏，然后刷新数据（MVC 通过模型来修改界面）
self.check.hidden = !self.deal.isChecking(再setdeal 方法里->内部掉刷新cell 的界面)

self.deal.checking = !self.deal.isChecking
self.check.hidden = !self.check.hidden

* 如果你自定义系统自带的控件，例如titleView,即便你设置了xy,依然会被系统自带的覆盖，但是你一定要设置宽高，不然没有尺寸、显示不出来
* get方法与set方法拦截的选择。 如果重写了get方法，意味着在外面每次获取都会来到get方法。这样保证了数据的实时更新，但是，每次都来都get方法，会怎加一些不必要的开支，而如果进行set方法拦截的话，优势在于不必平凡的调用，只计算一次，但是不能保证实时的一些更新操作
```

**48.日期**
```
> 设置日期格式（声明字符串里面每个数字和单词的含义）
> E:星期几  M:月份 d:几号(这个月的第几天)  H:24小时制的小时  m:分钟  s:秒  y:年
> NSLocale *locale = [[NSLocale alloc]initWithLocaleIdentifier:@"en_US"];
fmt.locale = locale; // 如果是真机调试，转换这种欧美时间，需要设置locale
> 日历对象
NSCalendar *calendar = [NSCalendar currentCalendar]; // 一定要是current。获取当前的日历对象
// 表明要获取那个时间段的差值
NSCalendarUnit unit = NSCalendarUnitYear | NSCalendarUnitMonth | NSCalendarUnitDay |NSCalendarUnitHour | NSCalendarUnitMinute | NSCalendarUnitSecond;
NSDateComponents *cmps =  [calendar components:unit fromDate:creatDate toDate:now options:0]; // from 与to 两个位置随便写。 option 写0

```

**49. 通知 后台**
```objc
iOS8开始,对用户的隐私的加强，需要注册配置信息
if ([[UIDevice currentDevice].systemVersion floatValue] > 8.0 ) { // iOS8
    UIUserNotificationSettings *setting = [UIUserNotificationSettings settingsForTypes:UIUserNotificationTypeBadge categories:nil];
    [application registerUserNotificationSettings:setting];
}

* 后台处理
// 开启后台任务
__block UIBackgroundTaskIdentifier taskID = [application beginBackgroundTaskWithExpirationHandler:^{ 
    // 过期就停止
    [application endBackgroundTask:taskID];
}];

app的状态
1.死亡状态：没有打开app
2.前台运行状态
3.后台暂停状态：停止一切动画、定时器、多媒体、联网操作，很难再作其他操作
4.后台运行状态

* 后台任务总结
1.在代理的applicationDidEnterBackground 方法中写开启后台任务，返回任务ID
2.配置plist文件添加Required background modes，
3.string里面添加 App plays audio or streams audio/video using AirPlay
在Info.plst中设置后台模式：Required background modes == App plays audio or streams audio/video using AirPlay 搞一个0kb的MP3文件，没有声音
循环播放
以前的后台模式只有3种:1.保持网络连接 2.多媒体应用 3.VOIP:网络电话
```

```
bool 类型和set方法联合使用
再外面设置情况，重写set方法，再里面setter方法拦截，因为这时候在里面bool已经有值，根据不同的bool 类型的值。有对应的情况
```

```
 遍历scrollView 内部子控件可以发现，自带了两个view，这两个view 是系统自带的水平垂直滚动条。计算尺寸不方便，所以当对应的属性设置为no 的时候，那么scrollView 的子控件就没有这两个多余的滚动条了
```

**50.分页小算法**
```
NSUInteger count = (emotions.count + JNPageEmotionCount - 1) / JNPageEmotionCount; // JNPageEmotionCount == 20
```

**51.touch 事件**
```
有的时候，我们会利用touch 来监听点击，但是这种情况下，点击事件会被子控件吞掉。所以不是太可靠。可以通过长按手势来做到只监听自己的触摸，而不监听子控件的触摸事件
```

```
取消emoji 显示时系统自带动画，但是这是系统的动画，一旦取消了。整个西东的动画行为就会取消，所以应该立马设置回去
```

```objc
UIView setAnimationsEnabled:NO];
// DO WHAT U WANNA DO
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(0.1 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
    [UIView setAnimationsEnabled:YES]; // 代码一过掉，马上恢复动画
});
* 注意，dispatch_after 是延迟提交，并不是延迟运行
Enqueue，就是入队，指的就是将一个Block在特定的延时以后，加入到指定的队列中，不是在特定的时间后立即运行，dispatch_after只是延时提交block，并不是延时后立即执行。所以想用dispatch_after精确控制运行状态的朋友可要注意了
```

```
设置attach 的属性
CGFloat fontWH = self.font.lineHeight; // 字体的行高
NSAttributedString ,NSTextAttachment 附件对象attach的bounds的xy可以设置。字体的行高 font.lineHeight
```


**52.性能优化**
```
1. 用ARC管理内存
2. 在正确的地方使用reuseIdentifier
3. 尽可能使Views不透明
4. 避免庞大的XIB
5. 不要block主线程
6. 在Image Views中调整图片大小
7. 选择正确的Collection
8. 打开gzip压缩
中级（这些是你可能在一些相对复杂情况下可能用到的）
9. 重用和延迟加载Views
10. Cache, Cache, 还是Cache！
11. 权衡渲染方法
12. 处理内存警告
13. 重用大开销的对象
14. 使用Sprite Sheets
15. 避免反复处理数据
16. 选择正确的数据格式
17. 正确地设定Background Images
18. 减少使用Web特性
19. 设定Shadow Path
20. 优化你的Table View
21. 选择正确的数据存储选项
进阶级（这些建议只应该在你确信他们可以解决问题和得心应手的情况下采用）
22. 加速启动时间
23. 使用Autorelease Pool
24. 选择是否缓存图片
25. 尽量避免日期格式转换
```

**53.静态库的制作**
```
->static libary ->写一些核心代码-->找到.a 文件，真机模拟器分别build一次->showinFinder -->打包资源(bundle)
如果没有暴露头文件的解放方法：build pharse ->copy files

* 真机文件夹下得静态库只能用于真机上, 模拟器文件夹下得静态库只能用于模拟器下

* 可以借助 lipo -info 静态库文件地址 指令查看当前静态库支持的平台
* 可以借助 lipo -create libdev/lib08-staticDemo.a  libPro/lib08-staticDemo.a  -output HMTool.a 指令将模拟器和真机的静态库合并为一个静态库
* lipo -create 需要合并的静态库1 需要合并的静态库2 -output 合并之后的文件名称
* Xcode6之后自动导入框架，但是如果用.a静态库的话，那么必须要导入
注意: 虽然将真机和模拟器的静态库合并在一起之后, 以后我们就不用关心当前是允许在模拟器还是真机了, 但是如果在程序发布时还是建议大家使用真机的静态库. 小
lipo -create libsim/libJNTool.a libdev/libJNTool.a -output JNXIXI.a

```

**54.正则表达式**
```
*  正则表达式使用步骤
*  1.创建正则表达式对象：定义规则（正则表达式）(贪婪匹配)
*  2.利用正则表达式匹配

* 常见字符含义
.	匹配除换行符以外的任意字符
\w	匹配字母或数字或下划线或汉字
\s	匹配任意的空白符
\d	匹配数字
\b	匹配单词的开始或结束
^	匹配字符串的开始
$	匹配字符串的结束
*	重复零次或更多次
+	重复一次或更多次
?	重复零次或一次
{n}	重复n次
{n,}	重复n次或更多次
{n,m}	重复n到m次
```


**55.缓存逻辑**
```
* 首先从本地缓存加载
* 本地没有数据再去进入下拉刷新状态   加载服务器端数据
* 否则不加载服务器数据  只让 用户主动加载数据
```

**56.image Mode**
```
> UIViewContentModeScaleToFill : 图片拉伸至填充整个UIImageView（图片可能会变形）
> UIViewContentModeScaleAspectFit : 图片拉伸至完全显示在UIImageView里面为止（图片不会变形）

> UIViewContentModeScaleAspectFill:图片拉伸至 图片的宽度等于UIImageView的宽度 或者 图片的高度等于UIImageView的高度 为止
> UIViewContentModeRedraw : 调用了setNeedsDisplay方法时，就会将图片重新渲染
1.凡是带有Scale单词的，图片都会拉伸
2.凡是带有Aspect单词的，图片都会保持原来的宽高比，图片不会变形
```

**57.性能测试(性能分析，内存分析)**
```
1.静态分析(Anaylize)
* 检测代码是否有潜在的内存泄露
* 编译器觉得不太合适的代码(32位，64位的适配问题)

2.动态分析(profile-instrument)
* 检测程序再运行过程中的内存变化
* allocations：能看清楚app的内存分配情况
* leaks ：能看清楚app再何时产生内存泄露(有红色的代表有内存泄露，注意官方webView就有内存泄露)
* timeprofile 表示程序的运行时间分析（耗时间的操作，算法的优化）
内存泄露：改释放的对象没有被释放
内存溢出：内存爆了，不够用

注意点：APNs限制了每个notification的payload最大长度是256字节，超长的消息是不能发送的

预编译指令
c 提供的预处理功能主要有以下三种： 1 ）宏定义　 2 ）文件包含　 3 ）条件编译
```

**58.Autoreleasepool的工作原理**
```
Autorelease的对象是在什么时候被release的？
> autorelease实际上只是把对release的调用延迟了，对于每一个Autorelease，系统只是把该Object放入了当前的Autorelease pool中，当该pool被释放时，该pool中的所有Object会被调用Release。对于每一个Runloop， 系统会隐式创建一个Autorelease pool，这样所有的release pool会构成一个象CallStack一样的一个栈式结构，在每一个Runloop结束时，当前栈顶的Autorelease pool会被销毁，这样这个pool里的每个Object（就是autorelease的对象）会被release。那什么是一个Runloop呢？ 一个UI事件，Timer call， delegate call， 都会是一个新的Runloop。那什么是一个Runloop呢？ 一个UI事件，Timer call， delegate call， 都会是一个新的Runloop。

> 在线程方法内不创建线程池，会在控制台有警告。系统认为分线程是独立运行，需要有一个独立的 NSAutoreleasePool
> 为了对pool内部的一些局部变量的释放，避免引起内存泄露
```

**59.分类的作用 **
```
1 可以使本来需要在.h中声明的方法放到.m文件中声明，使方法变成私有。
2 可以扩展或覆盖一个类的功能，包括系统类，维护了代码原本的结构不受影响。
3 可以分散代码到不同的文件之中，比如系统类库里有一个NSObject的类别，并没有写在NSObject类里，而写到另外一个类里，主要是因为这个类别扩展的功能跟那个类相关，便于将来查看。(利于团队协作，分模块开发)
```

**60.copy **
```
* 需要遵守NSCopying的协议，调用copy方法，内部会调用copywithZone 方法
- (id)copyWithZone:(NSZone *)zone {
    MyObject *copy = [[[self class] allocWithZone: zone] init];
    copy.username = [self.username copyWithZone:zone];
    return copy;
}
示例：NSString *str1 = @"111",NSString *str2 = @"111"
> 这种创建对象方式会造成内存地址共享，s1和s2完全指向同一个地址空间，也就是同一个对象了，改变或释放这个地址空间的内容就意味着s1和s2都变了。为了避免修改一方影响到另外一方，字符串赋值时常copy一个新的对象比较安全。
```

**61.支付宝**
```
1. 取支付宝官网申请“开通支付宝视同权限”
* 填写个人信息\公司信息(绝对真实，可靠，不是随便给你用的)
* 签约
* 等待审核

2.审核通过
* seller id
* partner id
* 后面加密用到的文件(公钥\私钥)

3.去支付宝官网下载支付宝sdk（网页版\无线版）
1> 生成订单信息
2> 签名信息
3> 利用订单信息，签名信息，签名类型生成一个订单字符串
4> 打开客户端支付
```

```objc
- (IBAction)buy {
    // 1.生成订单信息
    // 订单信息 == order == [order description]
    AlixPayOrder *order = [[AlixPayOrder alloc] init];
    order.productName = self.deal.title;
    order.productDescription = self.deal.desc;
    order.partner = PartnerID;
    order.seller = SellerID;
    order.amount = [self.deal.current_price description];
    // 2.签名加密
    id<DataSigner> signer = CreateRSADataSigner(PartnerPrivKey);
    // 签名信息 == signedString
    NSString *signedString = [signer signString:[order description]];

    // 3.利用订单信息、签名信息、签名类型生成一个订单字符串
    NSString *orderString = [NSString stringWithFormat:@"%@&sign=\"%@\"&sign_type=\"%@\"",
    [order description], signedString, @"RSA"];

    // 4.打开客户端,进行支付(商品名称,商品价格,商户信息)(有seletor，target 是用来处理网页支付的回调)
    [AlixLibService payOrder:orderString AndScheme:@"tuangou" seletor:@selector(getResult:) target:self];
}

// 网页处理结果回调
- (void)getResult:(NSString *)result{}

// 客户端处理回调
/** 当从其他应用跳转到当前应用时,就会调用这个方法 */
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation
{
    AlixPayResult * result = nil;
    if (url != nil && [[url host] compare:@"safepay"] == 0) {
    NSString * query = [[url query] stringByReplacingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
    #if ! __has_feature(objc_arc)
        result = [[[AlixPayResult alloc] initWithString:query] autorelease];
    #else
        result = [[AlixPayResult alloc] initWithString:query];
    #endif
    }
    if (result.statusCode == 9000) { // 表面上交易成功，但是为了安全，判断是否结果正确
// 用公钥验证签名 严格验证请使用result.resultString与result.signString验签
        id<DataVerifier> verifier = CreateRSADataVerifier(AlipayPubKey);
        if ([verifier verifyString:result.resultString withSign:result.signString]) {
                //验证签名成功，交易结果无篡改
                //交易成功
            } else {
            // 失败
            }
        } else {
        // 失败
    }
    return  YES;
}
/*打开Demo 的错误:依赖系统库libcrypto.a libssl.a CG SystemconFiguaration**/

```

**63.美团地图总结**
```
1. 首先，，iOS8开始 需要用户的授权[self.mgr requestAlwaysAuthorization];并在plist中配置NSLocationAlwaysUsageDescription允许在后台获取GPS的描述

2. 要设置追踪模式，customMapView.userTrackingMode =  MKUserTrackingModeFollow

3. 再mapView 的更新到用户位置的地方，利用反地理编码获取城市名，注意地标placemark的lociaty 和addressdict 中 省级市，和市的取值.获取到城市名，再区域改变的时候哪一个代理方法里面，发送网络请求，获取附近的团购

4. 因为用户刚进入地图的时候也需要知道周边的团购，那么就应该再刚更新到用户位置的代理方法里面主动掉一下代理方法、

5. 获取团购数据插大头针。但是要注意，因为地图每滚动一次就会发一次请求，运行的性能太差，应该用一个bool 常量过滤，如果我正在处理网络请求，那么就不要再发送请求了。

6. 每次滚动都插大头针。如果，我插的大头症的位置存在于现在地图上的大头针，[contains objc]，所以要重写isEquall 方法，告诉系统，只要你的位置和我的大头针样，那么我就认为，这两个大头针相同，我没有必要再在这个地方插大头针了。所以continue ，不插已经有的大头针

7. 如果想改变大头针的图片，只能用父类annotationView。并且一般情况下大头针模型都需要绑定一个团购模型
```


**64.推送**
```
注意点：APNs限制了每个notification的payload最大长度是256字节，超长的消息是不能发送的
```

**65.GCD**
```
> dispatch_once_t必须是全局或static变量
非全局或非static的dispatch_once_t变量在使用时会导致非常不好排查的bug,正确写法如下
//静态变量，保证只有一份实例，才能确保只执行一次
static dispatch_once_t onceToken;
dispatch_once(&onceToken, ^{
//单例代码 其实就是保证dispatch_once_t只有一份实例
});

> 注意，dispatch_after 是延迟提交，并不是延迟运行
Enqueue，就是入队，指的就是将一个Block在特定的延时以后，加入到指定的队列中，不是在特定的时间后立即运行，dispatch_after只是延时提交block，并不是延时后立即执行。所以想用dispatch_after精确控制运行状态的朋友可要注意了
> Enqueue，就是入队，指的就是将一个Block在特定的延时以后，加入到指定的队列中，不是在特定的时间后立即运行！
> 在main线程使用“同步”方法提交Block，必定会死锁。
```

```
SEL:其实是对方法的一种包装，将方法包装成一个SEL类型的数据，去找对应的方法地址。找到方法地址就可以调用方法
方法的存储位置
> 每个类的方法列表都存储在类对象中
> 每个方法都有一个与之对应的SEL类型的对象
> 根据一个SEL对象就可以找到方法的地址，进而调用方法
> SEL类型的定义
typedef struct objc_selector *SEL;

```

**66.runtime**
```
Objective-C 类也是对象，runtime 通过创建 Meta Classes 来处理这些。当你发送一个消息像这样 [NSObject alloc] 你正在向类对象发送一个消息，这个类对象需要是 MetaClass 的实例，MetaClass 也是 root meta class 的实例。当你说继承自 NSObject 时，你的类指向 NSObject 作为自己的 superclass。然而，所有的 meta class 指向 root metaclass 作为自己的 superclass。所有的 meta class 只是简单的有一个自己响应的方法列表。所以当你向一个类对象发送消息如 [NSObject alloc]，然后实际上 objc_msgSend() 会检查 meta class 看看它是否响应这个方法，如果他找到了一个方法，就在这个 Class 对象上执行（译注：class 是一个实例对象的类型，Class 是一个类（class）的类型。对于完全的 OO 来说，类也是个对象，类是类类型(MetaClass)的实例，所以类的类型描述就是 meta class）

* 日志输出
> __FILE__ ：源代码文件名
> __LINE__ ：NSLog代码在第几行
> __func__ ：当前的方法
> _cmd ：代表着当前方法的SEL
> - (void)test { //下面的代码会引发死循环
    [self performSelector:_cmd]
}

```

**67. 通知的安全性用法**
``` 
> 收到通知的对象被称为观察者,而且必须添加到NSNotificationCenter。除非你有很强的理由不去添加到NSNotificationCenter,这样他就总是defaultCenter。在init方法 viewDidLoad方法 viewWillAppear方法里面添加观察者是比较好的选择，你应该尽可能晚地添加观察者和尽快地删除它来提高性能并避免一些不希望出现的bug
> 如果想让一个控制器的view显示出来的时候才监听，不需要后台监听的时候，那么最严谨的方法是在viewWillAppear 添加通知，viewWillDisAppear 移除通知
```


**68.MVVM 黑魔法**
```
不要在viewDidLoad里面初始化你的view然后再add，这样代码就很难看。在viewDidload里面只做addSubview的事情，然后在viewWillAppear里面做布局的事情，最后在viewDidAppear里面做Notification的监听之类的事情。至于属性的初始化，则交给getter去做

ViewController基本上是大部分业务的载体
先是life cycle，然后是Delegate方法实现，然后是event response，然后才是getters and setters
数据管理者，数据加工者，数据展示者 中Model就是作为数据管理者，View作为数据展示者，Controller作为数据加工者
M应该做的事：
给ViewController提供数据
给ViewController存储数据提供接口
提供经过抽象的业务基本组件，供Controller调度

C应该做的事：
管理View Container的生命周期
负责生成所有的View实例，并放入View Container
监听来自View与业务有关的事件，通过与Model的合作，来完成对应事件的业务。
V应该做的事：
响应与业务无关的事件，并因此引发动画效果，点击反馈（如果合适的话，尽量还是放在View去做）等。
界面元素表达

MVCS
苹果自身就采用的是这种架构思路，从名字也能看出，也是基于MVC衍生出来的一套架构。从概念上来说，它拆分的部分是Model部分，拆出来一个Store。这个Store专门负责数据存取。但从实际操作的角度上讲，它拆开的是Controller。

这算是瘦Model的一种方案，瘦Model只是专门用于表达数据，然后存储、数据处理都交给外面的来做。MVCS使用的前提是，它假设了你是瘦 Model，同时数据的存储和处理都在Controller去做。所以对应到MVCS，它在一开始就是拆分的Controller。因为 Controller做了数据存储的事情，就会变得非常庞大，那么就把Controller专门负责存取数据的那部分抽离出来，交给另一个对象去做，这个 对象就是Store。这么调整之后，整个结构也就变成了真正意义上的MVCS。

胖Model包含了部分弱业务逻辑
胖Model要达到的目的是，Controller从胖Model这里拿到数据之后，不用额外做操作或者只要做非常少的操作，就能够将数据直接应用在View上
胖Model相对比较难移植，虽然只是包含弱业务，但好歹也是业务
瘦Model只负责业务数据的表达，所有业务无论强弱一律扔到Controller。瘦Model要达到的目的是，尽一切可能去编写细粒度Model，然后配套各种helper类或方法来对弱业务做抽象，强业务依旧交给Controller

MVVM着重想要解决的问题是尽可能地减少Controller的任务。不管MVVM也好，MVCS也好，他们的共识都是Controller会随着软件的成长，变很大很难维护很难测试

，MVCS是认为Controller做了一部分Model的事情，要把它拆出来变成Store，MVVM是认为Controller做了太多数据加工的事情，所以MVVM把数据加工的任务从Controller中解放了出来，使得Controller只需要专注于数据调配的工作，ViewModel则去负责数据加工并通过通知机制让View响应ViewModel的改变。

MVVM是基于胖Model的架构思路建立的，然后在胖Model中拆出两部分：Model和ViewModel。关于这个观点我要做一个额外解释：胖Model做的事情是先为Controller减负，然后由于Model变胖，再在此基础上拆出ViewModel，跟业界普遍认知的MVVM本质上是为Controller减负这个说法并不矛盾，因为胖Model做的事情也是为Controller减负。

另外，我前面说MVVM把数据加工的任务从Controller中解放出来，跟MVVM拆分的是胖Model也不矛盾。要做到解放Controller，首先你得有个胖Model，然后再把这个胖Model拆成Model和ViewModel。

AOP（Aspect Oriented Programming），面向切片编程，这也是面向XX编程系列术语之一哈，但它跟我们熟知的面向对象编程没什么关系。

什么是切片？(runtime Swizzling 就是通过这个来实现面向切片编程的)

程序要完成一件事情，一定会有一些步骤，1，2，3，4这样。这里分解出来的每一个步骤我们可以认为是一个切片。
什么是面向切片编程？
你针对每一个切片的间隙，塞一些代码进去，在程序正常进行1，2，3，4步的间隙可以跑到你塞进去的代码，那么你写这些代码就是面向切片编程。

AOP一般都是需要有一个拦截器，然后在每一个切片运行之前和运行之后（或者任何你希望的地方），通过调用拦截器的方法来把这个jointpoint扔到外面，在外面获得这个jointpoint的时候，执行相应的代码。

在iOS开发领域，objective-C的runtime有提供了一系列的方法，能够让我们拦截到某个方法的调用，来实现拦截器的功能，这种手段我们称为Method Swizzling。Aspects通过这个手段实现了针对某个类和某个实例中方法的拦截
```

**69.美团总结**
```
> 元数据工具类
1、如果项目中一些数据是一成不变的并且比较多，最好的是应该抽取元数据的工具类，你需要什么数据我给你，并且，最好的情况下我只加载一次，没有必要加载多次，最好的办法是static 加载一次
2. 工具类就应该有工具类的责任，把一些复杂的操作屏蔽再工具类中，比如，你给我一个名字，那么我就返回这个名字的城市模型给你，屏蔽业务细节
> 数据库工具类(initialize方法里面创建数据库)
1.将数据库的一些CURD操作屏蔽再内部，不要外界关心
2.分页小算法。加载哪一页的数据
int size = 20;
int pos = (page - 1) * size;
> 网络请求工具类
* 本来这里可以用类方法请求数据的，但是，DPRequest 需要有一个代理对象来监听网络请求的过程，返回数据给你，如果用了类方法，那么意味着我不能发请求，因为代理只能由对象来充当，而不能由类担任，所以这地方用对象方法，因为是对象发放，那么最好是一个单例
* 面向模型开发

> 有的时候，如果再xib中我用autolayout来约束控件，但是这个控件没有尺寸，那么有很大的可能性是系统自带的autoresizing 搞得鬼，这个时候，可以把这个控件的autoresizingMask值none 试一试,防止系统内部的一下拉升操作
> UICollectionView的一些小坑
* iOS8的新方法，监听viewWillTransitionToSize 横竖屏的滚动.
* iPad中cell的间距一般是通过横竖屏的滚动来给以不同的分布，结合布局的flow.sectionInset\minimumLineSpacing等一些属性来调整好看的样子
* WebView小坑
> 如果加载的不是你想加载的页面，那么就隐藏webView。这是一个小技巧
> [webView stringByEvaluatingJavaScriptFromString:js]; // oc与jsp的交互
> NSString *html = [webView stringByEvaluatingJavaScriptFromString:@"document.getElementsByTagName('html')[0].outerHTML;"]; // 先获取源码，再网页上德源码进行jsp操作更快

> supportedInterfaceOrientations,返回控制器支持的方向,返回的枚举类型要包含mask字眼
> searchBar做titleView 的问题。默认情况下，searchbar会拉的很长，一直到导航栏的宽度差不多。这个时候外面包装了UIView,以后发现系统自带的不好改变尺寸，可以尝试一下这么搞.

```

**70.自定义输出日志**
```
#ifdef DEBUG // 处于开发阶段
#define JNLog(...) NSLog(__VA_ARGS__)
#else // 处于发布阶段
#define JNLog(...)
#endif

fmt输出日志 #define NSString(...) [NSString stringWithFormat:__VA_ARGS__]

```

**开发常用宏**
```
/**判断是真机还是模拟器 */
#if TARGET_OS_IPHONE
//iPhone Device
#endif

#if TARGET_IPHONE_SIMULATOR
//iPhone Simulator
#endif

/**使用ARC和不使用ARC */
#if __has_feature(objc_arc)
//compiling with ARC
#else
// compiling without ARC
#endif

// 方正黑体简体字体定义
#define FONT(F) [UIFont fontWithName:@"FZHTJW--GB1-0" size:F]
//定义一个API
#define APIURL @"http://xxxxx/" //登陆API #define APILogin [APIURL stringByAppendingString:@"Login"]
//设置View的tag属性
#define VIEWWITHTAG(_OBJECT, _TAG) [_OBJECT viewWithTag : _TAG]
//程序的本地化,引用国际化的文件
#define MyLocal(x, ...) NSLocalizedString(x, nil)
//G－C－D
#define BACK(block) dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), block)
#define MAIN(block) dispatch_async(dispatch_get_main_queue(),block) 
//NSUserDefaults 实例化
#define USER_DEFAULT [NSUserDefaults standardUserDefaults] 
//由角度获取弧度 有弧度获取角度
#define degreesToRadian(x) (M_PI * (x) / 180.0)
#define radianToDegrees(radian) (radian*180.0)/(M_PI)
//读取本地图片
#define LOADIMAGE(file,ext) [UIImage imageWithContentsOfFile:[[NSBundle mainBundle]pathForResource:file ofType:ext]] 
//定义UIImage对象
#define IMAGE(A) [UIImage imageWithContentsOfFile:[[NSBundle mainBundle] pathForResource:A ofType:nil]]
```

**71.Tableview 性能优化**

```

提前计算并缓存好高度（布局），因为heightForRowAtIndexPath:是调用最频繁的方法；
异步绘制，遇到复杂界面，遇到性能瓶颈时，可能就是突破口；
滑动时按需加载，这个在大量图片展示，网络加载的时候很管用！（SDWebImage已经实现异步加载，配合这条性能杠杠的）。
除了上面最主要的三个方面外，还有很多几乎大伙都很熟知的优化点：

正确使用reuseIdentifier来重用Cells
尽量使所有的view opaque，包括Cell自身
尽量少用或不用透明图层
如果Cell内现实的内容来自web，使用异步加载，缓存请求结果
减少subviews的数量
在heightForRowAtIndexPath:中尽量不使用cellForRowAtIndexPath:，如果你需要用到它，只用一次然后缓存结果
尽量少用addView给Cell动态添加View，可以初始化时就添加，然后通过hide来控制是否显示
```




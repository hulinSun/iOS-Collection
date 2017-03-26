#博客总结
#####git fork 流

```
git remote add upstream git@123.57.152.76:teamfrontend/dreammentor.git

git fetch upstream

git merge upstream/master
fork 关联元地址  
```

需求：B要加入A的项目，不论是作为B的初始项目进行二次开发还是成为A项目的一员加入一起开发，步骤如下：

1.B首先要fork一个。'
B首先到A的github上，也就是此项目的位置:https://github.com/A/durit，然后单击fork，然后你（B）的github上就出现了一个fork，位置是：https://github.com/B/durit

2.B把自己的fork克隆到本地。
$ git clone https://github.com/B/durit
(当你clone到本地，会有一个默认的远程名叫"origin"，它指向了fork on github，也就是B上的fork，而不是指向A)

3.现在你是主人，为了保持与A的durit的联系，你需要给A的durit起个名，供你来驱使。
$ cd durit
$ git remote add upstream https://github.com/A/durit
(现在改名为upstream，这名随意，现在你（B）管A的durit叫upstream，以后B就用upstream来和A的durit联系了)

4.获取A上的更新(但不会修改你的文件)。
$ git fetch upstream
（这不，现在B就用upstream来联系A了）

5.合并拉取的数据
$ git merge upstream/master
（又联系了一次，upstream/master，前者是你要合并的数据，后者是你要合并到的数据（在这里就是B本地的durit了））

6.在B修改了本地部分内容后，把本地的更改推送到B的远程github上。
$ git add 修改过的文件
$ git commit -m "注释"
$ git push origin master
（目前为止，B上的github就跟新了）

7.然后B还想让修改过的内容也推送到A上，这就要发起pull request了。
 打开B的github,也就是https://github.com/B/durit
 点击Pull Requests
 单击new pull request
 单击create pull request
 输入title和你更改的内容
 然后单击send pull request
 这样B就完成了工作，然后就等着主人A来操作了。

8.在B想要更新A的github上到内容时，结果冲突，因为B和A同时修改了文件，比如说是README.ME，该这样做：
$ git status(查看冲突文件)
找到冲突文件(README.MD)后，打开并修改，解决冲突后
$ git add README.MD
$ git commit -m "解决了冲突文件README.MD"
现在冲突解决了，可以更新A的内容了，也就是上面第4步和第5步

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

NSDictionary *userInfo = @{@"class":@"HSFeedsViewController",@"property":@{@"ID": @"123",  @"type":@"12"}};

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

#####NSString copy strong

**不可变字符串 strong copy 之后都是指向同一份内存地址（做了浅拷贝），但是可变字符串，做的是深拷贝。这时候改变string  strong修饰的字符串也会跟着改变，而深拷贝的对象是新的对象，不会改变，变不变取决于需求，但是一般情况下都不希望别人改变，所以用copy**

```
 NSString *string = [NSString stringWithFormat:@"abc"];
    self.strongString = string;
    self.copyedString = string;

    NSLog(@"origin string: %p, %p", string, &string);
    NSLog(@"strong string: %p, %p", strongString, &strongString);
    NSLog(@"copy string: %p, %p", copyedString, &copyedString);
    
    origin string: 0x7fe441592e20, 0x7fff57519a48
strong string: 0x7fe441592e20, 0x7fe44159e1f8
copy string: 0x7fe441592e20, 0x7fe44159e200

//若果将string 改成mutablestring
NSMutableString *string = [NSMutableString stringWithFormat:@"abc"];

origin string: 0x7ff5f2e33c90, 0x7fff59937a48
strong string: 0x7ff5f2e33c90, 0x7ff5f2e2aec8
copy string: 0x7ff5f2e2aee0, 0x7ff5f2e2aed0
```

#####GPUImage

```
// Base classes
#import "GPUImageOpenGLESContext.h"
#import "GPUImageOutput.h"
#import "GPUImageView.h"
#import "GPUImageVideoCamera.h"
#import "GPUImageStillCamera.h"
#import "GPUImageMovie.h"
#import "GPUImagePicture.h"
#import "GPUImageRawDataInput.h"
#import "GPUImageRawDataOutput.h"
#import "GPUImageMovieWriter.h"
#import "GPUImageFilterPipeline.h"
#import "GPUImageTextureOutput.h"
#import "GPUImageFilterGroup.h"
#import "GPUImageTextureInput.h"
#import "GPUImageUIElement.h"
#import "GPUImageBuffer.h"
 
// Filters
#import "GPUImageFilter.h"
#import "GPUImageTwoInputFilter.h"
 
 
#pragma mark - 调整颜色 Handle Color
 
#import "GPUImageBrightnessFilter.h"                //亮度
#import "GPUImageExposureFilter.h"                  //曝光
#import "GPUImageContrastFilter.h"                  //对比度
#import "GPUImageSaturationFilter.h"                //饱和度
#import "GPUImageGammaFilter.h"                     //伽马线
#import "GPUImageColorInvertFilter.h"               //反色
#import "GPUImageSepiaFilter.h"                     //褐色（怀旧）
#import "GPUImageLevelsFilter.h"                    //色阶
#import "GPUImageGrayscaleFilter.h"                 //灰度
#import "GPUImageHistogramFilter.h"                 //色彩直方图，显示在图片上
#import "GPUImageHistogramGenerator.h"              //色彩直方图
#import "GPUImageRGBFilter.h"                       //RGB
#import "GPUImageToneCurveFilter.h"                 //色调曲线
#import "GPUImageMonochromeFilter.h"                //单色
#import "GPUImageOpacityFilter.h"                   //不透明度
#import "GPUImageHighlightShadowFilter.h"           //提亮阴影
#import "GPUImageFalseColorFilter.h"                //色彩替换（替换亮部和暗部色彩）
#import "GPUImageHueFilter.h"                       //色度
#import "GPUImageChromaKeyFilter.h"                 //色度键
#import "GPUImageWhiteBalanceFilter.h"              //白平横
#import "GPUImageAverageColor.h"                    //像素平均色值
#import "GPUImageSolidColorGenerator.h"             //纯色
#import "GPUImageLuminosity.h"                      //亮度平均
#import "GPUImageAverageLuminanceThresholdFilter.h" //像素色值亮度平均，图像黑白（有类似漫画效果）
 
#import "GPUImageLookupFilter.h"                    //lookup 色彩调整
#import "GPUImageAmatorkaFilter.h"                  //Amatorka lookup
#import "GPUImageMissEtikateFilter.h"               //MissEtikate lookup
#import "GPUImageSoftEleganceFilter.h"              //SoftElegance lookup
 
 
 
 
#pragma mark - 图像处理 Handle Image
 
#import "GPUImageCrosshairGenerator.h"              //十字
#import "GPUImageLineGenerator.h"                   //线条
 
#import "GPUImageTransformFilter.h"                 //形状变化
#import "GPUImageCropFilter.h"                      //剪裁
#import "GPUImageSharpenFilter.h"                   //锐化
#import "GPUImageUnsharpMaskFilter.h"               //反遮罩锐化
 
#import "GPUImageFastBlurFilter.h"                  //模糊
#import "GPUImageGaussianBlurFilter.h"              //高斯模糊
#import "GPUImageGaussianSelectiveBlurFilter.h"     //高斯模糊，选择部分清晰
#import "GPUImageBoxBlurFilter.h"                   //盒状模糊
#import "GPUImageTiltShiftFilter.h"                 //条纹模糊，中间清晰，上下两端模糊
#import "GPUImageMedianFilter.h"                    //中间值，有种稍微模糊边缘的效果
#import "GPUImageBilateralFilter.h"                 //双边模糊
#import "GPUImageErosionFilter.h"                   //侵蚀边缘模糊，变黑白
#import "GPUImageRGBErosionFilter.h"                //RGB侵蚀边缘模糊，有色彩
#import "GPUImageDilationFilter.h"                  //扩展边缘模糊，变黑白
#import "GPUImageRGBDilationFilter.h"               //RGB扩展边缘模糊，有色彩
#import "GPUImageOpeningFilter.h"                   //黑白色调模糊
#import "GPUImageRGBOpeningFilter.h"                //彩色模糊
#import "GPUImageClosingFilter.h"                   //黑白色调模糊，暗色会被提亮
#import "GPUImageRGBClosingFilter.h"                //彩色模糊，暗色会被提亮
#import "GPUImageLanczosResamplingFilter.h"         //Lanczos重取样，模糊效果
#import "GPUImageNonMaximumSuppressionFilter.h"     //非最大抑制，只显示亮度最高的像素，其他为黑
#import "GPUImageThresholdedNonMaximumSuppressionFilter.h" //与上相比，像素丢失更多
 
#import "GPUImageSobelEdgeDetectionFilter.h"        //Sobel边缘检测算法(白边，黑内容，有点漫画的反色效果)
#import "GPUImageCannyEdgeDetectionFilter.h"        //Canny边缘检测算法（比上更强烈的黑白对比度）
#import "GPUImageThresholdEdgeDetectionFilter.h"    //阈值边缘检测（效果与上差别不大）
#import "GPUImagePrewittEdgeDetectionFilter.h"      //普瑞维特(Prewitt)边缘检测(效果与Sobel差不多，貌似更平滑)
#import "GPUImageXYDerivativeFilter.h"              //XYDerivative边缘检测，画面以蓝色为主，绿色为边缘，带彩色
#import "GPUImageHarrisCornerDetectionFilter.h"     //Harris角点检测，会有绿色小十字显示在图片角点处
#import "GPUImageNobleCornerDetectionFilter.h"      //Noble角点检测，检测点更多
#import "GPUImageShiTomasiFeatureDetectionFilter.h" //ShiTomasi角点检测，与上差别不大
#import "GPUImageMotionDetector.h"                  //动作检测
#import "GPUImageHoughTransformLineDetector.h"      //线条检测
#import "GPUImageParallelCoordinateLineTransformFilter.h" //平行线检测
 
#import "GPUImageLocalBinaryPatternFilter.h"        //图像黑白化，并有大量噪点
 
#import "GPUImageLowPassFilter.h"                   //用于图像加亮
#import "GPUImageHighPassFilter.h"                  //图像低于某值时显示为黑
 
 
#pragma mark - 视觉效果 Visual Effect
 
#import "GPUImageSketchFilter.h"                    //素描
#import "GPUImageThresholdSketchFilter.h"           //阀值素描，形成有噪点的素描
#import "GPUImageToonFilter.h"                      //卡通效果（黑色粗线描边）
#import "GPUImageSmoothToonFilter.h"                //相比上面的效果更细腻，上面是粗旷的画风
#import "GPUImageKuwaharaFilter.h"                  //桑原(Kuwahara)滤波,水粉画的模糊效果；处理时间比较长，慎用
 
#import "GPUImageMosaicFilter.h"                    //黑白马赛克
#import "GPUImagePixellateFilter.h"                 //像素化
#import "GPUImagePolarPixellateFilter.h"            //同心圆像素化
#import "GPUImageCrosshatchFilter.h"                //交叉线阴影，形成黑白网状画面
#import "GPUImageColorPackingFilter.h"              //色彩丢失，模糊（类似监控摄像效果）
 
#import "GPUImageVignetteFilter.h"                  //晕影，形成黑色圆形边缘，突出中间图像的效果
#import "GPUImageSwirlFilter.h"                     //漩涡，中间形成卷曲的画面
#import "GPUImageBulgeDistortionFilter.h"           //凸起失真，鱼眼效果
#import "GPUImagePinchDistortionFilter.h"           //收缩失真，凹面镜
#import "GPUImageStretchDistortionFilter.h"         //伸展失真，哈哈镜
#import "GPUImageGlassSphereFilter.h"               //水晶球效果
#import "GPUImageSphereRefractionFilter.h"          //球形折射，图形倒立
 
#import "GPUImagePosterizeFilter.h"                 //色调分离，形成噪点效果
#import "GPUImageCGAColorspaceFilter.h"             //CGA色彩滤镜，形成黑、浅蓝、紫色块的画面
#import "GPUImagePerlinNoiseFilter.h"               //柏林噪点，花边噪点
#import "GPUImage3x3ConvolutionFilter.h"            //3x3卷积，高亮大色块变黑，加亮边缘、线条等
#import "GPUImageEmbossFilter.h"                    //浮雕效果，带有点3d的感觉
#import "GPUImagePolkaDotFilter.h"                  //像素圆点花样
#import "GPUImageHalftoneFilter.h"                  //点染,图像黑白化，由黑点构成原图的大致图形
 
 
#pragma mark - 混合模式 Blend
 
#import "GPUImageMultiplyBlendFilter.h"             //通常用于创建阴影和深度效果
#import "GPUImageNormalBlendFilter.h"               //正常
#import "GPUImageAlphaBlendFilter.h"                //透明混合,通常用于在背景上应用前景的透明度
#import "GPUImageDissolveBlendFilter.h"             //溶解
#import "GPUImageOverlayBlendFilter.h"              //叠加,通常用于创建阴影效果
#import "GPUImageDarkenBlendFilter.h"               //加深混合,通常用于重叠类型
#import "GPUImageLightenBlendFilter.h"              //减淡混合,通常用于重叠类型
#import "GPUImageSourceOverBlendFilter.h"           //源混合
#import "GPUImageColorBurnBlendFilter.h"            //色彩加深混合
#import "GPUImageColorDodgeBlendFilter.h"           //色彩减淡混合
#import "GPUImageScreenBlendFilter.h"               //屏幕包裹,通常用于创建亮点和镜头眩光
#import "GPUImageExclusionBlendFilter.h"            //排除混合
#import "GPUImageDifferenceBlendFilter.h"           //差异混合,通常用于创建更多变动的颜色
#import "GPUImageSubtractBlendFilter.h"             //差值混合,通常用于创建两个图像之间的动画变暗模糊效果
#import "GPUImageHardLightBlendFilter.h"            //强光混合,通常用于创建阴影效果
#import "GPUImageSoftLightBlendFilter.h"            //柔光混合
#import "GPUImageChromaKeyBlendFilter.h"            //色度键混合
#import "GPUImageMaskFilter.h"                      //遮罩混合
#import "GPUImageHazeFilter.h"                      //朦胧加暗
#import "GPUImageLuminanceThresholdFilter.h"        //亮度阈
#import "GPUImageAdaptiveThresholdFilter.h"         //自适应阈值
#import "GPUImageAddBlendFilter.h"                  //通常用于创建两个图像之间的动画变亮模糊效果
#import "GPUImageDivideBlendFilter.h"               //通常用于创建两个图像之间的动画变暗模糊效果
 
 
#pragma mark - 尚不清楚
#import "GPUImageJFAVoroniFilter.h"
#import "GPUImageVoroniConsumerFilter.h"
```

#####JSPatch 

**只需要在项目里引入极小的引擎文件，就可以使用 JavaScript 调用任何 Objective-C 的原生接口，替换任意 Objective-C 原生方法。目前主要用于下发 JS 脚本替换原生 Objective-C 代码，实时修复线上 bug。**


**JSPatch 执行顺序问题?**

**JSPatch所有动态替换的函数，都必须在JS执行完了之后，第二次再执行，才会全面以新替换的js代码进行工作。**

JSPatch 会在当前项目的 bundle 里寻找 main.js 文件执行，效果与最终线上用户下载脚本执行一样，测试完后就可以准备上线这个脚本。

注意 +testScriptInBundle 不能与 +startWithAppKey: 一起调用，+testScriptInBundle 只用于本地测试，测试完毕后需要去除。

**HOT FIX**
上传新的js脚本可以直接全量下发，也可以选择 开发预览 或 灰度或条件下发，也可以使用自定义 RSA key 对脚本进行加密签名。

上传完成后，对应版本的 APP 会请求下载这个脚本保存在本地，以后每次启动都会执行这个脚本。至此线上 bug 修复完成。

**一般的补丁信息**

```

@interface YFPatchModel : MTLModel<YFPatchModel>
@property (copy, nonatomic) NSString * patchId; //!< 补丁id.用于唯一标记一个补丁.因为补丁,后期可能需要更新,删除,添加等操作.
@property (copy, nonatomic) NSString * md5; //!< 文件的md5值,用于校验.
@property (copy, nonatomic) NSString * url; //!< 文件的URL路径.
@property (copy, nonatomic) NSString * ver; //!< 补丁对应的APP版本.不需要服务器返回,但需要本地存储此值.这个值在涉及到多个版本的补丁共存时,在应用升级时会很有价值.
@property (assign, nonatomic) YFPatchModelStatus status; //!< 补丁状态.此状态值由本地管理和维护.

@end
```

*时间顺序*

```
application:didFinishLaunchingWithOptions:
JSPatch发起网络请求拉patch
app的rootViewController触发ViewDidload运行完毕，依然是未修正的错误界面
JSPatch网络请求拉取回来，执行JS
JS已经执行成功ViewDidLoad已经被替换，但是界面已经生成，新的正确的ViewDidLoad并不会再次执行
效果：我的viewDidLoad为啥不能修改啊？
```


比喻：

viewDidload的函数代码就好比建筑设计图
运行起来后的界面就好比建好的建筑
时间顺序：

viewDidLoad有bug需要改（建筑设计图图纸错了）
旧viewDidLoad先执行，并且创建好了界面（工人已经按着错图纸把建筑建好了）
JSPatch执行了hotfix（设计师修改设计图纸）
JSPatch看起来没效果(就算你改好了建筑图纸，已经建好的建筑是不会有任何改变的)

解决办法：2个

* 在建造建筑之前，把图纸改好
JSPatch在使用的时候，第一次下载网络请求是要时间的，所以才会发生修改图纸，在建筑建好之后。但是补丁已经下载完成，第二次运行app，新的图纸已经存在本地，是可以在创建rootViewController之前，就先把patch运行，让新图纸生效的。

* 不要修改图纸了，直接去修改建筑
当你网络请求在JSPatch下载完Patch之后，通过callback，进行完全自定义的处理，窗户坏了，直接改窗户，门坏了修门，你也可以自定义把房子推倒了重建,
如果你使用的是JSPatchSDK，那么头文件有一个callback的API，JSPatchSDK提供了JS下载完成的这个时机，具体怎么修，纯看使用者自己。 如果你是用的是Github源码，那么自己按着这个思路自行处理


**实现原理 ： JS传递字符串给OC，OC通过 Runtime 接口调用和替换OC方法。**
 
 * Objective-C是动态语言，具有运行时特性，该特性可通过类名称和方法名的字符串获取该类和该方法，并实例化和调用
 
 ```
 Class class = NSClassFromString(“UIViewController");
id viewController = [[class alloc] init];  
SEL selector = NSSelectorFromString(“viewDidLoad");
[viewController performSelector:selector];
 ```
 
* 也可以替换某个类的方法为新的实现

```
static void newViewDidLoad(id slf, SEL sel) {}
class_replaceMethod(class, selector, newViewDidLoad, @"");
```

* 互传消息

```
JS和OC是通过JavaScriptCore互传消息的。OC端在启动JSPatch引擎时会创建一个 JSContext 实例，JSContext 是JS代码的执行环境，可以给 JSContext 添加方法。JS通过调用 JSContext 定义的方法把数据传给OC，OC通过返回值传会给JS。调用这种方法，它的参数/返回值 JavaScriptCore 都会自动转换，OC里的 NSArray, NSDictionary, NSString, NSNumber, NSBlock 会分别转为JS端的数组/对象/字符串/数字/函数类型。
对于一个自定义id对象，JavaScriptCore 会把这个自定义对象的指针传给JS，这个对象在JS无法使用，但在回传给OC时OC可以找到这个对象。对于这个对象生命周期的管理，如果JS有变量引用时，这个OC对象引用计数就加1 ，JS变量的引用释放了就减1，如果OC上没别的持有者，这个OC对象的生命周期就跟着JS走了，会在JS进行垃圾回收时释放
```

**方法替换**

<http://blog.cnbang.net/tech/2808/>

**Method 保存了一个方法的全部信息，包括SEL方法名，type各参数和返回值类型，IMP该方法具体实现的函数指针。
**

**把 UIViewController 的 -viewDidLoad 方法给替换成我们自定义的方法，APP里调用 UIViewController 的 viewDidLoad 方法都会去到上述 viewDidLoadIMP 函数里，在这个新的IMP函数里调用JS传进来的方法，就实现了替换 -viewDidLoad 方法为JS代码里的实现，同时为 UIViewController 新增了个方法 -ORIGViewDidLoad 指向原来 viewDidLoad 的IMP，JS可以通过这个方法调用到原来的实现**

**当调用一个 NSObject 对象不存在的方法时，并不会马上抛出异常，而是会经过多层转发，层层调用对象的 -resolveInstanceMethod:, -forwardingTargetForSelector:, -methodSignatureForSelector:, -forwardInvocation: 等方法**

**最后 -forwardInvocation: 是会有一个 NSInvocation 对象，这个 NSInvocation 对象保存了这个方法调用的所有信息，包括 Selector 名，参数和返回值类型，最重要的是有所有参数值，可以从这个 NSInvocation 对象里拿到调用的所有参数值。我们可以想办法让每个需要被JS替换的方法调用最后都调到 -forwardInvocation:，就可以解决无法拿到参数值的问题了**


**以替换 UIViewController 的 -viewWillAppear: 方法为例：**

* 把UIViewController的 -viewWillAppear: 方法通过 class_replaceMethod() 接口指向一个不存在的IMP: class_getMethodImplementation(cls, @selector(__JPNONImplementSelector))，这样调用这个方法时就会走到 -forwardInvocation:。

* 为 UIViewController 添加 -ORIGviewWillAppear: 和 -_JPviewWillAppear: 两个方法，前者指向原来的IMP实现，后者是新的实现，稍后会在这个实现里回调JS函数。

* 改写 UIViewController 的 -forwardInvocation: 方法为自定义实现。一旦OC里调用 UIViewController 的 -viewWillAppear: 方法，经过上面的处理会把这个调用转发到 -forwardInvocation: ，这时已经组装好了一个 NSInvocation，包含了这个调用的参数。在这里把参数从 NSInvocation 反解出来，带着参数调用上述新增加的方法 -JPviewWillAppear: ，在这个新方法里取到参数传给JS，调用JS的实现函数。整个调用过程就结束了


![示意图](/Users/mac/Documents/githup/iOS-Collection/屏幕快照 2016-08-24 下午5.42.00.png)


**HOTFIX**

```

- (void)awakeFromObjection
{
    /* 基本思路: 安装本地所有补丁 --> 联网更新补丁信息,并安装有更新或新增加的补丁. */
    /* DEBUG模式,总会执行测试函数,以测试某个JS. */
    [self mcDebug];
    
    /* 安装本地已有补丁. */
    [self mcInstallLocalPatchs];
    
    /* 联网获取最新补丁. */
    [[[[self mcFetchLatestPatchs] flattenMap:^RACStream *(NSArray<YFPatchModel> * models) {
        return [self mcUpdateLocalPatchs: models];
    }] then:^RACSignal *{/* 更新本地补丁文件. */
        return [self mcUpdateAllLocalPatchFiles];
    }] subscribeNext:^(id<YFPatchModel> patch) { /* 安装新增或有更新的补丁. */
        [self mcInstallPatch: patch];
    }];
}

```

#####GCD 信号量

*信号量就是一个资源计数器，对信号量有两个操作来达到互斥，分别是P和V操作。 一般情况是这样进行临界访问或互斥访问的： 设信号量值为1， 当一个进程1运行时，使用资源，进行P操作，即对信号量值减1，也就是资源数少了1个。这是信号量值为0。系统中规定当信号量值为0是，必须等待，直到信号量值不为零才能继续操作。 这时如果进程2想要运行，那么也必须进行P操作，但是此时信号量为0，所以无法减1，即不能P操作，那么久导致了阻塞。这样就达到了进程1排他访问。 当进程1运行结束后，释放资源，进行V操作。资源数重新加1，这是信号量的值变为1. 这时进程2发现资源数不为0，信号量能进行P操作了，立即执行P操作。信号量值又变为0.次数进程2咱有资源，排他访问资源。 这就是信号量来控制互斥的原理*

**简单来讲 信号量为0则阻塞线程，大于0则不会阻塞。则我们通过改变信号量的值，来控制是否阻塞线程，从而达到线程同步**

```
dispatch_semaphore_create 创建一个semaphore　
dispatch_semaphore_signal 发送一个信号　
dispatch_semaphore_wait 等待信号
```
**第一个函数有一个整形的参数，我们可以理解为信号的总量，dispatch_semaphore_signal是发送一个信号，自然会让信号总量加1，dispatch_semaphore_wait等待信号，当信号总量少于0的时候就会一直等待，否则就可以正常的执行，并让信号总量-1，根据这样的原理，我们便可以快速的创建一个并发控制来同步任务和有限资源访问控制。**


```
// 创建队列组
    dispatch_group_t group = dispatch_group_create();   
// 创建信号量，并且设置值为10
    dispatch_semaphore_t semaphore = dispatch_semaphore_create(10);   
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);   
    for (int i = 0; i < 100; i++)   
    {   // 由于是异步执行的，所以每次循环Block里面的dispatch_semaphore_signal根本还没有执行就会执行dispatch_semaphore_wait，从而semaphore-1.当循环10此后，semaphore等于0，则会阻塞线程，直到执行了Block的dispatch_semaphore_signal 才会继续执行
        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);   
        dispatch_group_async(group, queue, ^{   
            NSLog(@"%i",i);   
            sleep(2);   
// 每次发送信号则semaphore会+1，
            dispatch_semaphore_signal(semaphore);   
        });   
    }
```

#####Aspects AOP 面相切片编程

**解决复杂的统计问题**

#####iOS 常用小技巧

* 打印View所有子视图

```
po [[self view]recursiveDescription]
```

* Cell 的分割线的15像素

```
首先在viewDidLoad方法加入以下代码：
 if ([self.tableView respondsToSelector:@selector(setSeparatorInset:)]) {
        [self.tableView setSeparatorInset:UIEdgeInsetsZero];    
}   
 if ([self.tableView respondsToSelector:@selector(setLayoutMargins:)]) {        
        [self.tableView setLayoutMargins:UIEdgeInsetsZero];
}
然后在重写willDisplayCell方法
- (void)tableView:(UITableView *)tableView willDisplayCell:(UITableViewCell *)cell 
forRowAtIndexPath:(NSIndexPath *)indexPath{   
    if ([cell respondsToSelector:@selector(setSeparatorInset:)]) {       
             [cell setSeparatorInset:UIEdgeInsetsZero];    
    }    
    if ([cell respondsToSelector:@selector(setLayoutMargins:)]) {        
             [cell setLayoutMargins:UIEdgeInsetsZero];    
    }
}
```

* 方法调用时间

```
// 获取时间间隔
#define TICK   CFAbsoluteTime start = CFAbsoluteTimeGetCurrent();
#define TOCK   NSLog(@"Time: %f", CFAbsoluteTimeGetCurrent() - start)
```

* KVC 的数组操作

```
NSArray *array = [NSArray arrayWithObjects:@"2.0", @"2.3", @"3.0", @"4.0", @"10", nil];
CGFloat sum = [[array valueForKeyPath:@"@sum.floatValue"] floatValue];
CGFloat avg = [[array valueForKeyPath:@"@avg.floatValue"] floatValue];
CGFloat max =[[array valueForKeyPath:@"@max.floatValue"] floatValue];
CGFloat min =[[array valueForKeyPath:@"@min.floatValue"] floatValue];
NSLog(@"%fn%fn%fn%f",sum,avg,max,min);
```

* 比特操作

```
 NSArray *products = @[product1,product2,product3];
    NSLog(@"avg:%@",[products valueForKeyPath:@"@avg.price"]);
    NSLog(@"count:%@",[products valueForKeyPath:@"@count.price"]);
    NSLog(@"max:%@",[products valueForKeyPath:@"@max.price"]);
    NSLog(@"min:%@",[products valueForKeyPath:@"@min.price"]);
    NSLog(@"sum:%@",[products valueForKeyPath:@"@sum.price"]);
```

* Nil Null nil NSNull 的区别

```
* nil是OC的，空对象，地址指向空（0）的对象。对象的字面零值
* Nil是Objective-C类的字面零值
* NULL是C的，空地址，地址的数值是0，是个长整数
* NSNull用于解决向NSArray和NSDictionary等集合中添加空值的问题
```
* backItem 返回按钮

```
[[UIBarButtonItem appearance] setBackButtonTitlePositionAdjustment:UIOffsetMake(0, -60) forBarMetrics:UIBarMetricsDefault];
```

* KVC 修改UITextField 中的placeholder 文字颜色

```
[text setValue:[UIColor redColor] forKeyPath:@"_placeholderLabel.textColor"];
```

* 让Xcode的控制台支持LLDB类型的打印

```
打开终端输入三条命令:
    touch ~/.lldbinit
    echo display @import UIKit >> ~/.lldbinit
    echo target stop-hook add -o "target stop-hook disable" >> ~/.lldbinit
```

* block 判空处理

```
#define BLOCK_EXEC(block, ...) if (block) { block(__VA_ARGS__); };   
// 宏定义之前的用法
 if (completionBlock)   {   
    completionBlock(arg1, arg2); 
  }    
// 宏定义之后的用法
 BLOCK_EXEC(completionBlock, arg1, arg2);
```

##### NSOperation 

**串行vs并发**

*串行和并发的主要区别在于允许同时执行的任务数量。串行，指的是一次只能执行一个任务，必须等一个任务执行完成后才能执行下一个任务；并发，则指的是允许多个任务同时执行。*

**同步vs异步**

*同步和异步操作的主要区别在于是否等待操作执行完成，亦即是否阻塞当前线程。同步操作会等待操作执行完成后再继续执行接下来的代码，而异步操作则恰好相反，它会在调用后立即返回，不会等待操作的执行结果* 

**并发vs非并发Operation**

我们都是通过将 operation 添加到一个 operation queue 的方式来执行 operation 的，然而这并不是必须的。我们也可以直接通过调用 start 方法来执行一个 operation ，但是这种方式并不能保证 operation 是异步执行的。NSOperation 类的 isConcurrent 方法的返回值标识了一个 operation 相对于调用它的 start 方法的线程来说是否是异步执行的。在默认情况下，isConcurrent 方法的返回值是 NO ，也就是说会阻塞调用它的 start 方法的线程。

如果我们想要自定义一个并发执行的 operation ，那么我们就必须要编写一些额外的代码来让这个 operation 异步执行。比如，为这个 operation 创建新的线程、调用系统的异步方法或者其他任何方式来确保 start 方法在开始执行任务后立即返回。

在绝大多数情况下，我们都不需要去实现一个并发的 operation 。如果我们一直是通过将 operation 添加到 operation queue 的方式来执行 operation 的话，我们就完全没有必要去实现一个并发的 operation 。因为，当我们将一个非并发的 operation 添加到 operation queue 后，operation queue 会自动为这个 operation 创建一个线程。因此，只有当我们需要手动地执行一个 operation ，又想让它异步执行时，我们才有必要去实现一个并发的 operation 。

对于一个非并发的 operation ，我们需要做的就只是执行 main 方法中的任务以及能够正常响应取消事件就可以了，其它的复杂工作比如依赖配置、KVO 通知等 NSOperation 类都已经帮我们处理好了。而对于一个并发的 operation ，我们还需要重写 NSOperation 类中的一些现有方法。接下来，我们将会介绍如何自定义这两种不同类型的 NSOperation 子类。


**配置并发执行的 Operation**

* start ：必须的，所有并发执行的 operation 都必须要重写这个方法，替换掉 NSOperation 类中的默认实现。start 方法是一个 operation 的起点，我们可以在这里配置任务执行的线程或者一些其它的执行环境。另外，需要特别注意的是，在我们重写的 start 方法中一定不要调用父类的实现；

* main ：可选的，通常这个方法就是专门用来实现与该 operation 相关联的任务的。尽管我们可以直接在 start 方法中执行我们的任务，但是用 main 方法来实现我们的任务可以使设置代码和任务代码得到分离，从而使 operation 的结构更清晰；

* isExecuting 和 isFinished ：必须的，并发执行的 operation 需要负责配置它们的执行环境，并且向外界客户报告执行环境的状态。因此，一个并发执行的 operation 必须要维护一些状态信息，用来记录它的任务是否正在执行，是否已经完成执行等。此外，当这两个方法所代表的值发生变化时，我们需要生成相应的 KVO 通知，以便外界能够观察到这些状态的变化；

* isConcurrent ：必须的，这个方法的返回值用来标识一个 operation 是否是并发的 operation ，我们需要重写这个方法并返回 YES 。

executing 和 finished 属性都被声明成了只读的 readonly 。所以我们在 NSOperation 子类中就没有办法直接通过 setter 方法来自动触发 KVO 通知，这也是为什么我们需要在接下来的代码中手动触发 KVO 通知的原因。

```
- (void)start {
    if (self.isCancelled) {
        [self willChangeValueForKey:@"isFinished"];
        _finished = YES;
        [self didChangeValueForKey:@"isFinished"];
        return;
    }
    
    [self willChangeValueForKey:@"isExecuting"];
    [NSThread detachNewThreadSelector:@selector(main) toTarget:self withObject:nil];
    _executing = YES;
    [self didChangeValueForKey:@"isExecuting"];
}
```


#####Dispatch_block

```
// 用法和一般block 一样，可以用来做没有参数的block 回调
@property (nonatomic, copy) dispatch_block_t leftBlock;
```

##### 后台下载

```
- (NSURLSession *)backgroundSession {
    static NSURLSession *session = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        //这个sessionConfiguration 很重要， com.zyprosoft.xxx  这里，这个com.company.这个一定要和 bundle identifier 里面的一致，否则ApplicationDelegate 不会调用handleEventsForBackgroundURLSession代理方法
        NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration backgroundSessionConfigurationWithIdentifier:@"com.zyprosoft.backgroundsession"];
        session = [NSURLSession sessionWithConfiguration:configuration delegate:self delegateQueue:nil];
    });
    return session;
}

//在切换到后台后调用
- (void)application:(UIApplication *)application handleEventsForBackgroundURLSession:(NSString *)identifier
  completionHandler:(void (^)())completionHandler {
    self.backgroundSessionCompletionHandler = completionHandler;
    //添加本地通知
    [self presentNotification];
}


#pragma mark - NSURLSessionDelegate
//一个session结束之后，会在后台调用
- (void)URLSessionDidFinishEventsForBackgroundURLSession:(NSURLSession *)session
{
    AppDelegate *appDelegate = (AppDelegate *)[[UIApplication sharedApplication] delegate];
    if (appDelegate.backgroundSessionCompletionHandler) {
        void (^completionHandler)() = appDelegate.backgroundSessionCompletionHandler;
        appDelegate.backgroundSessionCompletionHandler = nil;
        completionHandler();
    }
    NSLog(@"所有任务已完成!");
}

```

#####全屏返回手势

```
// 将系统自带的手势禁用掉，将系统自带的手势添加到view中，这样就实现了全屏返回手势。 关键在于找准target action 
- (void)viewDidLoad {
    [super viewDidLoad];
    NSLog(@"interactivePopGestureRecognizer = %@ \n\n",self.interactivePopGestureRecognizer);
    NSLog(@"delegate = %@",self.interactivePopGestureRecognizer.delegate);
    //获取系统自带滑动手势的target对象
    id target = self.interactivePopGestureRecognizer.delegate;
    NSLog(@"target = %@", target);
    // 创建全屏滑动手势，调用系统自带滑动手势的target的action方法
    UIPanGestureRecognizer *pan = [[UIPanGestureRecognizer alloc]initWithTarget:target action:@selector(handleNavigationTransition:)];
    //设置代理手势，拦截手势触发
    pan.delegate = self;
    //给导航控制器的view添加全屏滑动手势
    [self.view addGestureRecognizer:pan];
    // 禁止使用系统自带的滑动手势
    self.interactivePopGestureRecognizer.enabled = NO;
}
```


##### 读取二维码

```
#pragma mark  读取二维码
-(void)readQRCoder
{
    //1.实例化摄像头设备
    AVCaptureDevice *device = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];
    
    //2.设置输入,把摄像头作为输入设备
    //因为模拟器是没有摄像头的，因此在此最好做个判断
    NSError *error = nil;
    AVCaptureDeviceInput *input = [AVCaptureDeviceInput deviceInputWithDevice:device error:&error];
    if (error) {
        NSLog(@"没有摄像头%@", error.localizedDescription);
        return;
    }
    //3.设置输出(Metadata元数据)
    AVCaptureMetadataOutput *outPut = [[AVCaptureMetadataOutput alloc] init];
    //3.1 设置输出的代理
    //使用主线程队列，相应比较同步，使用其他队列，相应不同步，容易让用户产生不好的体验。
    [outPut setMetadataObjectsDelegate:self queue:dispatch_get_main_queue()];
    
    //4.拍摄会话
    AVCaptureSession *session = [[AVCaptureSession alloc]init];
    //添加session的输入和输出
    [session addInput:input];
    [session addOutput:outPut];
    //4.1设置输出的格式
    [outPut setMetadataObjectTypes:@[AVMetadataObjectTypeQRCode]];
    
    //5.设置预览图层(用来让用户能够看到扫描情况)
    AVCaptureVideoPreviewLayer *preview = [AVCaptureVideoPreviewLayer layerWithSession:session];
    //5.1设置preview图层的属性
    [preview setVideoGravity:AVLayerVideoGravityResizeAspectFill];
    //5.2设置preview图层的大小
    [preview setFrame:self.view.bounds];
    //5.3将图层添加到视图的图层
    [self.view.layer insertSublayer:preview atIndex:0];
    self.previewLayer = preview;
    //6.启动会话
    [session startRunning];
    self.session = session;
}

#pragma mark 输出代理方法
//此方法是在识别到QRCode并且完成转换，如果QRCode的内容越大，转换需要的时间就越长。
-(void)captureOutput:(AVCaptureOutput *)captureOutput didOutputMetadataObjects:(NSArray *)metadataObjects fromConnection:(AVCaptureConnection *)connection
{
    //会频繁的扫描，调用代理方法
    //1如果扫描完成，停止会话
    [self.session stopRunning];
    //2删除预览图层
    [self.previewLayer removeFromSuperlayer];
    NSLog(@"%@",metadataObjects);
    //设置界面显示扫描结果
    if (metadataObjects.count > 0) {
        AVMetadataMachineReadableCodeObject *obj = metadataObjects[0];
        //提示：如果需要对url或者名片等信息上扫描，再次扩展就好了。
        _label.text = obj.stringValue;
    }
}
```

#####隐藏导航栏的细线
```
// 方式1.tabbar 也可以这么做
self.navigationController?.navigationBar.shadowImage = UIImage()

// 方式2:巧妙的写法
self.navigationController?.navigationBar.clipsToBounds = true;

// 方式三：比较正规
 private func findHairlineImageViewUnderView(view: UIView) -> UIImageView? {
        if view is UIImageView && view.bounds.height <= 1.0 {
            return view as? UIImageView
        }
        for subView in view.subviews {
            guard let i = findHairlineImageViewUnderView(subView) else{ return nil }
            return i
        }
        return nil
   }
   
// Swift 中升级的属性
navigationController?.hidesBarsOnSwipe = true 
    
```

##### 关联
```
-(NSString *)associatedObject_retain{
    return objc_getAssociatedObject(self, _cmd);
}
- (void)setAssociatedObject_retain:(NSString *)associatedObject_retain{
    objc_setAssociatedObject(self, @selector(associatedObject_retain), associatedObject_retain, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
```

#####keychain
**可以用NSUserDefaults存储数据信息，但是对于一些私密信息，比如账号、密码等等，就需要使用更为安全的keychain了。而Keychain的信息是存在于每个应用（app）的沙盒之外的，所以keychain里保存的信息不会因App被删除而丢失，在用户重新安装App后依然有效，数据还在。比如优步下载app之后不需要登录，自动登录了**
[参考链接](http://blog.sina.com.cn/s/blog_5971cdd00102vqgy.html)

#####KVO 注册依赖键
```

//重写getter方法
- (NSString *)accountForBank {
    return [NSString stringWithFormat:@"account for %@ is %d", self.bankCodeEn, self.accountBalance];
}
//定义了这种依赖关系后，我们就需要以某种方式告诉KVO，当我们的被依赖属性修改时，会发送accountForBank属性被修改的通知。
//当两个属性之中的一个改变，就会激发kvo
+ (NSSet *)keyPathsForValuesAffectingAccountForBank {
    return [NSSet setWithObjects:@"accountBalance", @"bankCodeEn", nil];
}
```


#####视频压缩

```
使用系统自带的AVAssetExportSession，代码和网上说的差不多
AVURLAsset *avAsset = [AVURLAsset URLAssetWithURL:sourceUrl options:nil];
    NSArray *compatiblePresets = [AVAssetExportSession exportPresetsCompatibleWithAsset:avAsset];
    if ([compatiblePresets containsObject:AVAssetExportPresetHighestQuality]){
        AVAssetExportSession *exportSession = [[AVAssetExportSession alloc] initWithAsset:avAsset presetName:AVAssetExportPresetMediumQuality];
        NSDateFormatter *formater = [[NSDateFormatter alloc] init];//用时间给文件全名，以免重复，在测试的时候其实可以判断文件是否存在若存在，则删除，重新生成文件即可
        [formater setDateFormat:@"yyyy-MM-dd-HH:mm:ss"];
        NSString * resultPath = [NSHomeDirectory() stringByAppendingFormat:@"/Documents/output-%@.mp4", [formater stringFromDate:[NSDate date]]];
        exportSession.outputURL = [NSURL fileURLWithPath:resultPath];
        exportSession.outputFileType = AVFileTypeMPEG4;
        exportSession.shouldOptimizeForNetworkUse = YES;
        [exportSession exportAsynchronouslyWithCompletionHandler:^(void){
             switch (exportSession.status) {
                 case AVAssetExportSessionStatusUnknown:
                     NSLog(@"AVAssetExportSessionStatusUnknown");
                     break;
                 case AVAssetExportSessionStatusWaiting:
                     NSLog(@"AVAssetExportSessionStatusWaiting");
                     break;
                 case AVAssetExportSessionStatusExporting:
                     NSLog(@"AVAssetExportSessionStatusExporting");
                     break;
                 case AVAssetExportSessionStatusCompleted:
                     NSLog(@"AVAssetExportSessionStatusCompleted");
                     break;
                 case AVAssetExportSessionStatusFailed:
                     NSLog(@"AVAssetExportSessionStatusFailed");
                     break;
             }
         }];
    }
```

##### 控制动画执行时间

```
keyFrameAnimation.beginTime = CACurrentMediaTime() + 0.5
```

#### 屏幕旋转

* 物理方向：[UIDevice currentDevice].orientation (UIDeviceOrientation类型)

```
if (![UIDevice currentDevice].generatesDeviceOrientationNotifications) {
        [[UIDevice currentDevice] beginGeneratingDeviceOrientationNotifications];
 }
NSLog(@"%d",[UIDevice currentDevice].orientation);
[[UIDevice currentDevice] endGeneratingDeviceOrientationNotifications];
```

* 界面显示方向： viewController.interfaceOrientation
* [UIApplication sharedApplication].statusBarOrientation

```
- (BOOL)shouldAutorotate NS_AVAILABLE_IOS(6_0);
- (NSUInteger)supportedInterfaceOrientations 
- (UIInterfaceOrientation)preferredInterfaceOrientationForPresentation NS_AVAILABLE_IOS(6_0);
```

####AVPLay

```
- (void)createTimer{
    __weak typeof(self) weakSelf = self;
    self.timeObserve = [self.player addPeriodicTimeObserverForInterval:CMTimeMakeWithSeconds(1, 1) queue:nil usingBlock:^(CMTime time){
        
        AVPlayerItem *currentItem = weakSelf.playerItem;
        NSArray *loadedRanges = currentItem.seekableTimeRanges;
        if (loadedRanges.count > 0 && currentItem.duration.timescale != 0) {
            // 当前播放时间
            NSInteger currentTime = (NSInteger)CMTimeGetSeconds([currentItem currentTime]);
            // 视频总时长
            CGFloat totalTime     = (CGFloat)currentItem.duration.value / currentItem.duration.timescale;
            // 播放进度
            CGFloat value         = CMTimeGetSeconds([currentItem currentTime]) / totalTime;
            [weakSelf.controlView zf_playerCurrentTime:currentTime totalTime:totalTime sliderValue:value];
        }
    }];
}
```

```

- (NSTimeInterval)availableDuration {
    NSArray *loadedTimeRanges = [[_player currentItem] loadedTimeRanges];
    CMTimeRange timeRange     = [loadedTimeRanges.firstObject CMTimeRangeValue];// 获取缓冲区域
    float startSeconds        = CMTimeGetSeconds(timeRange.start);
    float durationSeconds     = CMTimeGetSeconds(timeRange.duration);
    NSTimeInterval result     = startSeconds + durationSeconds;// 计算缓冲总进度
    return result;
}
```

```

- (void)seekToTime:(NSInteger)dragedSeconds completionHandler:(void (^)(BOOL finished))completionHandler
{
    if (self.player.currentItem.status == AVPlayerItemStatusReadyToPlay) {
        // seekTime:completionHandler:不能精确定位
        // 如果需要精确定位，可以使用seekToTime:toleranceBefore:toleranceAfter:completionHandler:
        // 转换成CMTime才能给player来控制播放进度
        [self.controlView zf_playerActivity:YES];
        [self.player pause];
        CMTime dragedCMTime = CMTimeMake(dragedSeconds, 1); //kCMTimeZero
        __weak typeof(self) weakSelf = self;
        [self.player seekToTime:dragedCMTime toleranceBefore:CMTimeMake(1,1) toleranceAfter:CMTimeMake(1,1) completionHandler:^(BOOL finished) {
            [weakSelf.controlView zf_playerActivity:NO];
            // 视频跳转回调
            if (completionHandler) { completionHandler(finished); }
            [weakSelf.player play];
            weakSelf.seekTime = 0;
            weakSelf.isDragged = NO;
            // 结束滑动
            [weakSelf.controlView zf_playerDraggedEnd];
            if (!weakSelf.playerItem.isPlaybackLikelyToKeepUp && !weakSelf.isLocalVideo) { weakSelf.state = ZFPlayerStateBuffering; }
            
        }];
    }
}

```

```

    if (playerItem) {
        [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(moviePlayDidEnd:) name:AVPlayerItemDidPlayToEndTimeNotification object:playerItem];
        [playerItem addObserver:self forKeyPath:@"status" options:NSKeyValueObservingOptionNew context:nil];
        [playerItem addObserver:self forKeyPath:@"loadedTimeRanges" options:NSKeyValueObservingOptionNew context:nil];
        // 缓冲区空了，需要等待数据
        [playerItem addObserver:self forKeyPath:@"playbackBufferEmpty" options:NSKeyValueObservingOptionNew context:nil];
        // 缓冲区有足够数据可以播放了
        [playerItem addObserver:self forKeyPath:@"playbackLikelyToKeepUp" options:NSKeyValueObservingOptionNew context:nil];
    }
    
```

```
       [self.imageGenerator generateCGImagesAsynchronouslyForTimes:[NSArray arrayWithObject:[NSValue valueWithCMTime:dragedCMTime]] completionHandler:handler];
```

通常情况下可以通过判断播放器的播放速度来获得播放状态。如果rate为0说明是停止状态，1是则是正常播放状态。

当然AVPlayerItem是有通知的，但是对于获得播放状态和加载状态有用的通知只有一个：播放完成通知AVPlayerItemDidPlayToEndTimeNotification。在播放视频时，特别是播放网络视频往往需要知道视频加载情况、缓冲情况、播放情况，这些信息可以通过KVO监控AVPlayerItem的status、loadedTimeRanges属性来获得。当AVPlayerItem的status属性为AVPlayerStatusReadyToPlay是说明正在播放，只有处于这个状态时才能获得视频时长等信息；当loadedTimeRanges的改变时（每缓冲一部分数据就会更新此属性）可以获得本次缓冲加载的视频范围（包含起始时间、本次加载时长），这样一来就可以实时获得缓冲情况。然后就是依靠AVPlayer的- (id)addPeriodicTimeObserverForInterval:(CMTime)interval queue:(dispatch_queue_t)queue usingBlock:(void (^)(CMTime time))block方法获得播放进度，这个方法会在设定的时间间隔内定时更新播放进度，通过time参数通知客户端

##### oc 结构体玩法

```

struct JMRadius {
    CGFloat topLeftRadius;
    CGFloat topRightRadius;
    CGFloat bottomLeftRadius;
    CGFloat bottomRightRadius;
};
typedef struct JMRadius JMRadius;

static inline JMRadius JMRadiusMake(CGFloat topLeftRadius, CGFloat topRightRadius, CGFloat bottomLeftRadius, CGFloat bottomRightRadius) {
    JMRadius radius;
    radius.topLeftRadius = topLeftRadius;
    radius.topRightRadius = topRightRadius;
    radius.bottomLeftRadius = bottomLeftRadius;
    radius.bottomRightRadius = bottomRightRadius;
    return radius;
}

static inline NSString * NSStringFromJMRadius(JMRadius radius) {
    return [NSString stringWithFormat:@"{%.2f, %.2f, %.2f, %.2f}", radius.topLeftRadius, radius.topRightRadius, radius.bottomLeftRadius, radius.bottomRightRadius];
}
```



#### 运行时判断类型
 在Swift中，如果想在运行时判断一个对象的类型是不是可选类型，则可以使用Mirror，所有的可选类型都将返回optional。

Mirror的displayStyle属性的类型是Mirror.DisplayStyle，这是一个枚举类型，用于为Mirror的主体对象提供一种建议性的解析。枚举值范围之外的类型，其Mirror的displayStyle属性将返回nil

```
let value: Int? = 3
Mirror(reflecting: value as Any).displayStyle
/** 如果不是`struct` `class` `enum` tuple optional
 collection dictionary set 这些类型,那么返回nil */
```

#### 自定义操作符
在自定义操作符时，可以以dot(.)开头，这种情况下，操作符后面还可以包含其它的dot(.)

但如果操作符不是以dot开头，则后面不能再包含dot，如&quot;operator +.+&quot;这个声明会被看成是&quot;+&quot;操作符后面跟了个&quot;.+&quot;操作符。编译器会给出错误提示。

*首先在全局使用operator关键字来声明操作符，同时用prefix、infix或postfix来声明操作符的位置；然后在所需要的类/结构体中实现操作符。*

#### selector 玩法

```

fileprivate extension Selector{
    /// 这里给统一的注释也ok
    static let loginButtonTaped = #selector(ViewController.buttonTap)
}

class ViewController: UIViewController{
    func buttonTap(){ }
    
    func setup()  {
        let btn = UIButton()
        btn.addTarget(self, action: .loginButtonTaped, for: .touchUpInside)
    }
}
```


#### 指针

可以使用withUnsafePointer(to:_:)函数来获取一个变量的指针

withUnsafePointer(to:_:)将第一个参数转换为指针，然后将这个指针作为参数去调用第二个参数指定的闭包。如果闭包有返回值，它将作为函数的返回值。

需要注意的是，生成的指针的生命周期限定于闭包内部，不能将其指定给外部的变量。

```

var age = 5
withUnsafePointer(to: &age) {p in print(p)}

func printPoint<T>(ptr: UnsafePointer<T>){
    print(ptr)
}
printPoint(ptr: &age)
```

#### HTTPS
适配HTTPS做双向验证时，如何放置证书？有两种方式，AFNetworking支持拖拽文件，自动检测并绑定，这个很好用，所以这种方式大家用的多。还有另外一种方式，可以将证书文件转成base64，再用NSURLProtocol缓存起来。原生的NSURLSession支持该方式，当然，AFNetworking也支持该方式。所有如果你看到一个项目，发现项目中没有cer文件，也不一定说
明该项目不支持双向认证

 HTTPS 请求做 SSL Pinning，即在客户端本地钉一个服务端的证书，在 SSL 握手阶段除了校验证书的有效性和合法性之外，还对服务端返回证书内容与本地的进行比对，只有相同才通过校验，这样可以阻止中间人攻击，以及避免数据被抓包监听。
 
####defer

关键字 defer 允许我们推迟到函数返回之前（或任意位置执行 return 语句之后）一刻才执行某个语句或函数（为什么要在返回之后才执行这些语句？因为 return 语句同样可以包含一些操作，而不是单纯地返回某个值）。当有多个 defer 行为被注册时，它们会以逆序执行（类似栈，即后进先出
关键字 defer 的用法类似于面向对象编程语言 Java 和 C# 的 finally 语句块，它一般用于释放某些已分配的资源

####static class
static和class这两个关键字都可以修饰类的方法，以表明这个方法是一个类方法。不过这两者稍微有一些区别：class修饰的类方法可以被子类重写，而static修饰的类方法则不能

#### contains
Swift中如果想判断一个Array中是否包含某个元素，我们可以使用contains方法

不过这个方法要求数组中的元素类型实现了Equatable协议，否则无法使用，还好Swift为我们提供了另一个contains方法，可以自定义谓词条件作为判断依据

这个方法会查看数组是否包含满足给定的谓词条件的元素。可以看到这个方法是一个高阶函数，其参数是一个尾随闭包，在闭包内我们可以根据实际需要来实现我们自己的判断。所以上面的判断可以如图3实现。

当然，对于元素类型实现了Equatable协议的数组，也可以使用这个方法。可以自定义谓词条件，查看数组是否有满足此条件的元素
 
 ```
struct Person: Equatable{
    let age = 20
    var name: String
    
    public static func ==(lhs: Person, rhs: Person) -> Bool{
        return lhs.name == rhs.name
    }
}

let arr = [1,2,3,4,5,6]
arr.contains(3)

let p1 = Person(name: "jack")
let p2 = Person(name: "rose")
let arr1 = [p1,p2]
arr1.contains { (p) -> Bool in
    return p.name == "jack"
}
arr1.contains(p1)
p1 == p2 // 实现了equalable 协议，就能使用== 判等
 ```
 
#### 截屏通知
 UIApplicationUserDidTakeScreenshot通知来告诉App用户做了截屏操作，如图1代码所示(Swift+Xcode 8)。我们只需要监听这个通知，并执行想要的操作即可
 
#### static
在Swift中，如果我们希望懒加载一个对象的属性，则可以在类/结构体中给属性加上lazy修饰符。

而全局变量或常量是自带lazy特性的，它们总是在用到的时候才去初始化，而这种情况下不需要用lazy来修饰。另外存储型类属性也是在初次使用时才初始化，并确保只会初始化一次，不需要用lazy来修饰。如果用lazy去修饰它，则编译器会报错

#### 倒序遍历

```
let nsarr: NSArray = [1,2,3,4,5,6,7,8]
nsarr.enumerateObjects(options: .reverse) { (item, idx, stop) in
    print(item)
    if idx == 2{ stop.pointee = true }
}
```

#### oc 类属性
```
@interface JNDataTool ()
@property (class,nonatomic, strong) NSMutableArray *arrs;
@end

@implementation JNDataTool

static NSMutableArray *_arr = nil;

+(NSMutableArray *)arrs{
    if (_arr == nil) {
        _arr = [NSMutableArray array];
    }
    return _arr;
}

+(void)setArrs:(NSMutableArray *)arrs{
    if (arrs != _arr) {
        _arr = arrs;
    }
}
@end

void test(){
    NSLog(@"%@--",JNDataTool.arrs);
}

```

####xcode 6.3 兼容swift

**苹果为了减轻我们的工作量，专门提供了两个宏：NS_ASSUME_NONNULL_BEGIN和NS_ASSUME_NONNULL_END。在这两个宏之间的代码，所有简单指针对象都被假定为nonnull，因此我们只需要去指定那些nullable的指针。**

#### swift options 
Swift不支持C语言中枚举值的整型掩码操作的技巧。在Swift中，一个枚举可以表示一组有效选项的集合，但却没有办法支持这些选项的组合操作(“&”、”|”等)。理论上，一个枚举可以定义选项值的任意组合值，但对于n > 3这种操作，却无法有效的支持。

为了支持类NS_OPTIONS的枚举，Swift 2.0中定义了OptionSetType协议
OptionSetType是选项集合类型，它定义了一些基本操作，包括集合操作(union, intersect, exclusiveOr)、成员管理(contains, insert, remove)、位操作(unionInPlace, intersectInPlace, exclusiveOrInPlace)以及其它的一些基本操作

```
struct Directions: OptionSetType {
    var rawValue:Int
    init(rawValue: Int) {
        self.rawValue = rawValue
    }
    static let Up: Directions = Directions(rawValue: 1 << 0)
    static let Down: Directions = Directions(rawValue: 1 << 1)
    static let Left: Directions = Directions(rawValue: 1 << 2)
    static let Right: Directions = Directions(rawValue: 1 << 3)
}
let leftUp: Directions = [Directions.Left, Directions.Up]
if leftUp.contains(Directions.Left) && leftUp.contains(Directions.Up) {
    // ...
}
```

#### selector

Selector是Objective-C的产物，它用于在运行时作为一个键值去找到对应方法的实现
这就要求selector引用的方法必须对ObjC运行时是可见的。而Swift是静态语言，虽然继承自NSObject的类默认对ObjC运行时是可见的，但如果方法是由private关键字修饰的，则方法默认情况下对ObjC运行时并不是可见的，所以就导致了以上的异常：运行时并没找到SwipeCardView类的beginDragged:方法。
我们必须将private修饰的方法暴露给运行时。正确的做法是在 private 前面加上 @objc 关键字，
另外需要注意的是，如果我们的类是纯Swift类，而不是继承自NSObject，则不管方法是private还是internal或public，如果要用在Selector中，都需要加上@objc修饰符。

#### 加锁
加锁解锁是一件非常好性能的事情，尤其是用sync 方式加的锁，如果很多地方加锁都引用这self 的话，那么锁等待的时间会非常长，所以性能方面不是很好，但是可以通过GCD 的信号量来实现加锁的效果，
线程等待 ，排他远离 pv操作 互斥原理.yycache 就是用了着了操作对做了优化.

#### 并发与并行

1. 并发是单个处理器执行多个任务，这些任务在重叠的时间段内交叉执行，但在任何一个时间点都只有一个任务在执行。这些任务在逻辑上是同时执行，而实际上是交叉执行的。
2. 并行是多个处理器或多核下执行多个任务，这些任务可以同时执行，即同一时间点可以有多个不同的任务在执行。这些任务在物理上是同时执行的。


#### 闭包捕获

```

var thing = "cars"
//let closure = { [thing] in // 注意，这里是捕获列表
//    print("I love \(thing)") // cars
//}
let closure = {
    print(thing) // aieplanes
}
thing = "airplanes"
closure()

```
当声明闭包的时候，捕获列表会创建一份thing的copy，所以被捕获到的值是不会改变的，即使你改变thing的值。
如果你去掉闭包中的捕获列表，编译器会使用引用代替copy。在这种情况下，当闭包被调用时，变量的值是可以改变的。

#### IBDesign 
IB_DESIGNABLE的宏的功能就是让XCode动态渲染出该类图形化界面。UIView 或 NSView使用IB_DESIGNABLE宏声明时候，就是让Interface Builder知道它应该在UIStoryboard或者Xib中画布上直接渲染视图，不需要等到编译运行后就能预先展示出来效果 。
IBInspectable修饰属性，可以是用户自定义的运行时属性，让支持KVC的属性能够在Attribute Inspector中配置。

```
IB_DESIGNABLE
@interface IBDesignableImageView : UIImageView
@property(nonatomic,assign) IBInspectable CGFloat cornerRadius;

```


#### SDWebimage

• UIImageView+WebCache:  setImageWithURL:placeholderImage:options: 先显示 placeholderImage ，同时由SDWebImageManager 根据 URL 来在本地查找图片。

• SDWebImageManager: downloadWithURL:delegate:options:userInfo: SDWebImageManager是将UIImageView+WebCache同SDImageCache链接起来的类， SDImageCache：queryDiskCacheForKey:delegate:userInfo:用来从缓存根据CacheKey查找图片是否已经在缓存中

• 如果内存中已经有图片缓存， SDWebImageManager会回调SDImageCacheDelegate : imageCache:didFindImage:forKey:userInfo:

• 而 UIImageView+WebCache 则回调SDWebImageManagerDelegate:  webImageManager:didFinishWithImage:来显示图片。

• 如果内存中没有图片缓存，那么生成 NSInvocationOperation 添加到队列，从硬盘查找图片是否已被下载缓存。

• 根据 URLKey 在硬盘缓存目录下尝试读取图片文件。这一步是在 NSOperation 进行的操作，所以回主线程进行结果回调 notifyDelegate:。

• 如果上一操作从硬盘读取到了图片，将图片添加到内存缓存中（如果空闲内存过小，会先清空内存缓存）。SDImageCacheDelegate 回调 imageCache:didFindImage:forKey:userInfo:。进而回调展示图片。

• 如果从硬盘缓存目录读取不到图片，说明所有缓存都不存在该图片，需要下载图片，回调 imageCache:didNotFindImageForKey:userInfo:。

• 共享或重新生成一个下载器 SDWebImageDownloader 开始下载图片。

• 图片下载由 NSURLConnection 来做，实现相关 delegate 来判断图片下载中、下载完成和下载失败。

• connection:didReceiveData: 中利用 ImageIO 做了按图片下载进度加载效果。

• connectionDidFinishLoading: 数据下载完成后交给 SDWebImageDecoder 做图片解码处理。

• 图片解码处理在一个 NSOperationQueue 完成，不会拖慢主线程 UI。如果有需要对下载的图片进行二次处理，最好也在这里完成，效率会好很多。

• 在主线程 notifyDelegateOnMainThreadWithInfo: 宣告解码完成，imageDecoder:didFinishDecodingImage:userInfo: 回调给 SDWebImageDownloader。

• imageDownloader:didFinishWithImage: 回调给 SDWebImageManager 告知图片下载完成。

• 通知所有的 downloadDelegates 下载完成，回调给需要的地方展示图片。

• 将图片保存到 SDImageCache 中，内存缓存和硬盘缓存同时保存。

• 写文件到硬盘在单独 NSInvocationOperation 中完成，避免拖慢主线程。

•  如果是在iOS上运行，SDImageCache 在初始化的时候会注册notification 到 UIApplicationDidReceiveMemoryWarningNotification 以及  UIApplicationWillTerminateNotification,在内存警告的时候清理内存图片缓存，应用结束的时候清理过期图片。

• SDWebImagePrefetcher 可以预先下载图片，方便后续使用。

### 消息转发三个步骤

动态方法解析

```

void functionForMethod1(id self, SEL _cmd) {
   NSLog(@"%@, %p", self, _cmd);
}
+ (BOOL)resolveInstanceMethod:(SEL)sel {
    NSString *selectorString = NSStringFromSelector(sel);
    if ([selectorString isEqualToString:@"method1"]) {
        class_addMethod(self.class, @selector(method1), (IMP)functionForMethod1, "@:");
    }
    return [super resolveInstanceMethod:sel];
}
```

备用接收者
```
- (id)forwardingTargetForSelector:(SEL)aSelector {
    NSLog(@"forwardingTargetForSelector");
    NSString *selectorString = NSStringFromSelector(aSelector);
    // 将消息转发给_helper来处理
    if ([selectorString isEqualToString:@"method2"]) {
        return _helper;
    }
    return [super forwardingTargetForSelector:aSelector];
}
```


完整转发

```
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector {
    NSMethodSignature *signature = [super methodSignatureForSelector:aSelector];
    if (!signature) {
        if ([SUTRuntimeMethodHelper instancesRespondToSelector:aSelector]) {
            signature = [SUTRuntimeMethodHelper instanceMethodSignatureForSelector:aSelector];
        }
    }
    return signature;
}
- (void)forwardInvocation:(NSInvocation *)anInvocation {
    if ([SUTRuntimeMethodHelper instancesRespondToSelector:anInvocation.selector]) {
        [anInvocation invokeWithTarget:_helper];
    }
}
```

#### Swizzle
+load会在类初始加载时调用，+initialize会在第一次调用类的类方法或实例方法之前被调用。这两个方法是可选的，且只有在实现了它们时才会被调用。由于method swizzling会影响到类的全局状态，因此要尽量避免在并发处理中出现竞争的情况。+load能保证在类的初始化过程中被加载，并保证这种改变应用级别的行为的一致性。相比之下，+initialize在其执行时不提供这种保证–事实上，如果在应用中没为给这个类发送消息，则它可能永远不会被调用。 ----> 单利，饿汉式 --> java 的单利 static

因为swizzling会改变全局状态，所以我们需要在运行时采取一些预防措施。原子性就是这样一种措施，它确保代码只被执行一次，不管有多少个线程。GCD的dispatch_once可以确保这种行为，我们应该将其作为method swizzling的最佳实践。

```
Selector(typedef struct objc_selector *SEL)：用于在运行时中表示一个方法的名称。一个方法选择器是一个C字符串，它是在Objective-C运行时被注册的。选择器由编译器生成，并且在类被加载时由运行时自动做映射操作。
Method(typedef struct objc_method *Method)：在类定义中表示方法的类型
Implementation(typedef id (*IMP)(id, SEL, ...))：这是一个指针类型，指向方法实现函数的开始位置。这个函数使用为当前CPU架构实现的标准C调用规范。每一个参数是指向对象自身的指针(self)，第二个参数是方法选择器。然后是方法的实际参数。
理解这几个术语之间的关系最好的方式是：一个类维护一个运行时可接收的消息分发表；分发表中的每个入口是一个方法(Method)，其中key是一个特定名称，即选择器(SEL)，其对应一个实现(IMP)，即指向底层C函数的指针。
```


#### HTTPS

#####客户端发起HTTPS请求

这个没什么好说的，就是用户在浏览器里输入一个https网址，然后连接到server的443端口。

#####服务端的配置

采用HTTPS协议的服务器必须要有一套数字证书，可以自己制作，也可以向组织申请。区别就是自己颁发的证书需要客户端验证通过，才可以继续访问，而使用受信任的公司申请的证书则不会弹出提示页面(startssl就是个不错的选择，有1年的免费服务)。这套证书其实就是一对公钥和私钥。如果对公钥和私钥不太理解，可以想象成一把钥匙和一个锁头，只是全世界只有你一个人有这把钥匙，你可以把锁头给别人，别人可以用这个锁把重要的东西锁起来，然后发给你，因为只有你一个人有这把钥匙，所以只有你才能看到被这把锁锁起来的东西。

#####传送证书

这个证书其实就是公钥，只是包含了很多信息，如证书的颁发机构，过期时间等等。

#####客户端解析证书

这部分工作是有客户端的TLS来完成的，首先会验证公钥是否有效，比如颁发机构，过期时间等等，如果发现异常，则会弹出一个警告框，提示证书存在问题。如果证书没有问题，那么就生成一个随即值。然后用证书对该随机值进行加密。就好像上面说的，把随机值用锁头锁起来，这样除非有钥匙，不然看不到被锁住的内容。

#####传送加密信息

这部分传送的是用证书加密后的随机值，目的就是让服务端得到这个随机值，以后客户端和服务端的通信就可以通过这个随机值来进行加密解密了。

#####服务端解密信息

服务端用私钥解密后，得到了客户端传过来的随机值(私钥)，然后把内容通过该值进行对称加密。所谓对称加密就是，将信息和私钥通过某种算法混合在一起，这样除非知道私钥，不然无法获取内容，而正好客户端和服务端都知道这个私钥，所以只要加密算法够彪悍，私钥够复杂，数据就够安全。

#####传输加密后的信息

这部分信息是服务段用私钥加密后的信息，可以在客户端被还原

#####客户端解密信息

客户端用之前生成的私钥解密服务段传过来的信息，于是获取了解密后的内容。整个过程第三方即使监听到了数据，也束手无策。


### ASPECT hook方法 面相切片编程

解决冲突


#### runloop 的作用
常驻线程。要想runloop一直不退出，那么给他事情做就可以。可以给timer source 那么runloop就会一直工作。所以这就是常驻线程的背后原理 [runloop addport] 本质上就给添加source、那么线程就可以一直在了。达到了场主线程的目的


### super 
特别注意，super 不是一个指针。self 是一个指针，super 只是编译指示器，就是给编译器看的，只要编译器看到super 这个标志，就会让当前对象调用父类的方法。 本质上还是self 调用的方法， 所以[super class] 返回的还是当前对象的类型





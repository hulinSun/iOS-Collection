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

#####NSString copy strong

**不可变字符串 strong copy 之后都是指向同一份内存地址（做了浅拷贝），但是可变字符串，做的是深拷贝。这时候改变string  strong修饰的字符串也会跟着改变，而深拷贝的对象是新的对象，不会改变，变不变取决于需求，但是一般情况下都不希望别人改变，所以用copy **

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
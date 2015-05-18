CLImageEditor
===

CLImageEditor为iPhone应用程序提供了基本的图像编辑功能。这个ViewController使用起来非常简单，并且还能够很容易地作为UIImagePickerController的一部分来使用。

![sample](Demo/CLImageEditorDemo/CLImageEditorDemo/sample.jpg)


安装
---

方法一：直接复制

使用CLImageEditor最简单的方法是将CLImageEditor组（或者目录）中的所有文件复制到您的应用中。并且添加以下几个框架(Build Phases > Link Binary With Libraries): Accelerate, CoreGraphics, CoreImage。

可选工具包在OptionalImageTools中，您可以根据需要来添加。

##### 方法二：使用git的submodule

另外，你应该能够建立一个Git子模块，并引用您的Xcode项目的文件。

`git submodule add https://github.com/yackle/CLImageEditor.git`

##### 方法三：使用CocoaPods

[CocoaPods](http://cocoapods.org/) CocoaPods是一个Objective-C项目依赖的包管理工具.

`pod 'CLImageEditor'`

或者

`pod 'CLImageEditor/AllTools'`

通过指定AllTools subspec，所有的图像工具，包括可选工具安装。

#### 可选工具包

下面是可选工具包：

`pod 'CLImageEditor/ResizeTool'`

`pod 'CLImageEditor/StickerTool'`

`pod 'CLImageEditor/TextTool'`

`pod 'CLImageEditor/SplashTool'`



用法
---
入门CLImageEditor是非常的简单。只需要使用UIImage初始化它，并为其设置一个委托。然后你可以使用它作为一个平常的ViewController了。


```  objc

#import "CLImageEditor.h"

@interface ViewController()
<CLImageEditorDelegate>
@end

- (void)presentImageEditorWithImage:(UIImage*)image
{
    CLImageEditor *editor = [[CLImageEditor alloc] initWithImage:image];
    editor.delegate = self;
	
    [self presentViewController:editor animated:YES completion:nil];
}

```

当用于UIImagePickerController时，CLImageEditor可以在picker调用`pushViewController:animated:` 方法时作为一个参数来使用。
```  objc

#pragma mark- UIImageController delegate

- (void)imagePickerController:(UIImagePickerController *)picker didFinishPickingMediaWithInfo:(NSDictionary *)info
{
    UIImage *image = [info objectForKey:UIImagePickerControllerOriginalImage];
    
    CLImageEditor *editor = [[CLImageEditor alloc] initWithImage:image];
    editor.delegate = self;
    
    [picker pushViewController:editor animated:YES];
}

```

当图像被编辑之后，editor将调用委托的`imageEditor:didFinishEdittingWithImage:` 方法。委托的方法需要接受编辑过的图像作为参数。

```  objc


#pragma mark- CLImageEditor delegate

- (void)imageEditor:(CLImageEditor *)editor didFinishEdittingWithImage:(UIImage *)image
{
    _imageView.image = image;
    [editor dismissViewControllerAnimated:YES completion:nil];
}

```

此外，当你想捕获了cancel callback时，可以调用可选的delegate imageEditorDidCancel：方法。

更多详细信息，请参阅 `CLImageEditorDemo`。


自定义
---

图标放在CLImageEditor.bundle里面。您可以通过改写图标来改变外观。

其他关于主题设置功能还没有实现。


##### 菜单定制
图片工具可以使用`CLImageToolInfo`来自定义。

CLImageEditor的`toolInfo`属性可以通过函数来访问每个工具的信息。例如，`subToolInfoWithToolName:recursive:` 方法用于获取一个特定名称的工具信息。

```  objc
CLImageEditor *editor = [[CLImageEditor alloc] initWithImage:_imageView.image];
editor.delegate = self;

CLImageToolInfo *tool = [editor.toolInfo subToolInfoWithToolName:@"CLToneCurveTool" recursive:NO];
```

当获得工具的信息后，通过改变其属性，您可以自定义菜单上的图像工具了。

```  objc
CLImageToolInfo *tool = [editor.toolInfo subToolInfoWithToolName:@"CLToneCurveTool" recursive:NO];
tool.title = @"TestTitle";
tool.available = NO;     // if available is set to NO, it is removed from the menu view.
tool.dockedNumber = -1;  // Bring to top
//tool.iconImagePath = @"test.png";
```

* `dockedNumber` 用来作为菜单项的一个键来进行简单的排序。从而确定每个菜单项的顺序。

工具列表可以用下面的代码来获得。

```  objc
NSLog(@"%@", editor.toolInfo);
NSLog(@"%@", editor.toolInfo.toolTreeDescription);
```

目前，为ios7提供了如下工具：
```
CLFilterTool
	CLDefaultEmptyFilter
	CLDefaultLinearFilter
	CLDefaultVignetteFilter
	CLDefaultInstantFilter
	CLDefaultProcessFilter
	CLDefaultTransferFilter
	CLDefaultSepiaFilter
	CLDefaultChromeFilter
	CLDefaultFadeFilter
	CLDefaultCurveFilter
	CLDefaultTonalFilter
	CLDefaultNoirFilter
	CLDefaultMonoFilter
	CLDefaultInvertFilter    
CLAdjustmentTool
CLEffectTool
	CLEffectBase
	CLSpotEffect
	CLHueEffect
	CLHighlightShadowEffect
	CLBloomEffect
	CLGloomEffect
	CLPosterizeEffect
	CLPixellateEffect
CLBlurTool
CLRotateTool
CLClippingTool
CLResizeTool
CLToneCurveTool
CLStickerTool
CLTextTool
```

一些工具还有`optionalInfo`属性，它可以用来定制更详细的信息。

###### 裁剪工具（Clipping tool）

裁剪工具允许您设置预设比例及纵向/横向按钮的是否可见。

``` objc
NSArray *ratios = @[
                    @{@"value1":@0, @"value2":@0,       @"titleFormat":@"Custom"}, // if either value is zero, free form is set.
                    @{@"value1":@1, @"value2":@1,       @"titleFormat":@"%.1f : %.1f"},
                    @{@"value1":@1, @"value2":@1.618,   @"titleFormat":@"%g : %g"},
                    @{@"value1":@2, @"value2":@3},
                    @{@"value1":@3, @"value2":@2},
                    ];

CLImageToolInfo *tool = [editor.toolInfo subToolInfoWithToolName:@"CLClippingTool" recursive:NO];
tool.optionalInfo[@"ratios"] = ratios;
tool.optionalInfo[@"swapButtonHidden"] = @YES;
```

###### 缩放工具（Resize tool）


你可以调整预设尺寸和最大尺寸。

``` objc
CLImageToolInfo *tool = [editor.toolInfo subToolInfoWithToolName:@"CLResizeTool" recursive:NO];
tool.optionalInfo[@"presetSizes"] = @[@240, @320, @480, @640, @800, @960, @1024, @2048];
tool.optionalInfo[@"limitSize"] = @3200;
```

###### 贴纸工具（Sticker tool）

您可以重新设置一个路径来存放贴纸图片文件的bundle。

``` objc
CLImageToolInfo *tool = [editor.toolInfo subToolInfoWithToolName:@"CLStickerTool" recursive:NO];
tool.optionalInfo[@"stickerPath"] = @"yourStickerPath";
```

License
---
CLImageEditor 遵循 MIT License, 详见 [LICENSE](LICENSE).

---
title: iOS的那些第三方库
date: 2016-06-24 14:47:12
tags:
---
最近比较空，就整理了一下之前项目中用过的并且比较实用的第三方库。还对同类型的第三方库做了实用性的比较。话不多说=，=进入正题

### SDWebImage & YYWebImage & PINRemoteImage

对于图片缓存、显示网络图片这一块，我一直用的都是[SDWebImage](https://github.com/rs/SDWebImage)，而且也没出过什么大的问题，蛮稳定的。只不过之前看到一篇关于文章中介绍说[图片如何像浏览器那样边下载变现实](http://blog.ibireme.com/2015/11/02/ios_image_tips/),当时觉得如果能够这样优雅的显示图片，确实在用户体验上能够提升不少。恰巧SDWebImage这个库是在图片完全下载完之后才绘制到界面上的，所以每次图片加载都是有一种突然间跳动变化的感觉，相对来说体验上会有稍许的差别。而其他两个[YYWebImage](https://github.com/ibireme/YYWebImage)和[PINRemoteImage](https://github.com/pinterest/PINRemoteImage)在体验要优于SDWebImage不少。
注意只有第一种逐行扫描才是兼容PNG和JPEG格式的。但是用户体验优越的是第二种和第三种。但是相对来说出现PNG和JPEG的图片
而这三者中，相对来说PINRemoteImage上手比较慢，成本比较大。
而对于另外SDWebImage和YYWebImage来说，其实各有千秋。两者对外的接口基本上差不多，所以基本没有额外的使用成本。而两者的优势呢，我个人觉得SDWebImage相对来说稳定，因为毕竟近千次的comment以及100多个issue摆在那里。而YYWebImage在图片的呈现方式上做的比较优雅，用户体验比较好，可定制的内容也比较多。
最后总结下：SDWebImage和YYWebImage这两个都有各自的可取之处，如果你求稳定，那么SDWebImage肯定是首选，如果你注重用户体验，或者说加载的图片都是比较大的，那么我觉得还是YYWebImage比较合适。

### JSONModel & YYModel

对于数据的封装，网络数据的映射。最一开始的时候我是自己手动解析的，当时就感觉要累趴下了。之后用了[JSONModel](https://github.com/JSONModel/JSONModel)这个库，感觉不错。类似解析映射的库有很多，如:FastEasyMapping,Mantle,MJextension等。但是但是为什么我选择了这两个呢。原因请看下面这张图：
![截图](http://7xrcp9.com1.z0.glb.clouddn.com/blogblog222.png?imageView2/2/w/600/q/75)
这张图是摘自[iOS JSON 模型转换库评测](http://blog.ibireme.com/2015/10/23/ios_model_framework_benchmark/)
从图中可以看出[YYModel](https://github.com/ibireme/YYModel)的效率比较高。因此当时考虑着，在以后是不是要弃用JSONModel，转战YYModel了。
但是看了YYModel之后，我发现它虽然效率高，但是并没有像JSONModel那么“人性化”。这也是在实际的运用中的出来的，在JSONModel中一个model里的属性可以设置它是否是Optional的。

```` objectivec

@interface UserModel : JSONModel

@property(nonatomic,strong)NSString *userId;
@property(nonatomic,strong)NSString <Optional>*userName;
@property(nonatomic,strong)NSString <Optional>*avatar;
@property(nonatomic,strong)NSNumber <Optional>*gender;
@property(nonatomic,strong)NSString <Optional>*accessToken;
@property(nonatomic,strong)NSNumber <Optional>*createdAt;
@property(nonatomic,strong)NSNumber <Optional>*score;
@end

````

如上面的`UserModel`其中的有不少属性是Optional的，也就是说这个model只有`userId`是必定要有的，其他的可以没有。这个在实际的运用中还是满常见的。比如说有些用户有名字，有些没有名字。而用户id作为标示用户的唯一标准，肯定是存在的。
而YYModel却没有。
所以相比较之下，虽然在效率上低了点，但是对于这个的感知，其实在一定程度上可以有多忽略。主要还是在考虑实用性上面。所以我觉得对于数据的封装来说，JSONModel还是一个不错的库。

### SVProgressHUD & MBProgressHUD

对于这两个库功能都很简单，就是作为指示器的功能。相对来说[SVProgressHUD](https://github.com/SVProgressHUD/SVProgressHUD)使用起来比较方便,功能比较单一。而[MBProgressHUD](https://github.com/jdg/MBProgressHUD)可定制化的成分比较多，但是相对的使用起来比较的麻烦。
如果同样需要显示一个操作完成的指示器，那么MBProgessHUD实现如下：
```` objectivec
    MBProgressHUD *hud = [MBProgressHUD showHUDAddedTo:self.view animated:YES];
    hud.mode = MBProgressHUDModeCustomView;
    UIImage *image = [[UIImage imageNamed:@"Checkmark"] imageWithRenderingMode:UIImageRenderingModeAlwaysTemplate];
    hud.customView = [[UIImageView alloc] initWithImage:image];
    hud.square = YES;
    hud.label.text = @"Done";
    [hud hideAnimated:YES afterDelay:3.f];
````

而SVProgressHUD只需要一行：
```` objectivec
    [SVProgressHUD showSuccessWithStatus:@"Done"];
````

可想而知，直接使用起来SVProgressHUD有多大的优势。但是看了两者在设备上的呈现，我还发现MBProgressHUD的指示器相对来说比较精细，而SVProgressHUD的有点过大，导致略显粗糙。所以追求美感的同学我建议使用MBProgressHUD。虽然说MBProgressHUD使用起来有点复杂，但是完全可以在它的基础之上在封装一层就可以达到使用起来和SVProgressHUD一样方便~

### 暂时over

恩，以上这些第三方库，就是这次整理出来的。也许你们会差异，神马！才这么几个，其实肯定不止。只是在之前的使用过程中存在疑虑，这几个(SDWebImage,JSONModel,SVProgressHUD)，这三个是否有其他更好的库替换。最后经过调研，尝试之后。在我之后的开发中也许SDWebImage会被YYWebImage替换，而JSONModel坚守住了，SVProgressHUD的话果断换成MBProgressHUD，(因为我是一个无比追求美感和体验的程序员)，但是肯定会在MBProgressHUD上面再封装一层的。不过其实对YYWebImage这个库还是略有保留。因为对于图片的显示这一块在现在的app中占的比重太大了。所以也许会考虑SDWebImage和YYWebImage两者都存在，进行无缝切换。
好了，就这么多~



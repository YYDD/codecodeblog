---
title: 利用runtime来修改轮子满足自己
date: 2017-08-18 11:29:44

---

对于开源世界来说，用别人造过的轮子是一件很普遍的事情。而且很多时候在短时间内要写出一个完美的库也并不是一件容易的事情。但是因为业务需求的不同，多多少少会出现需要在别人的库上做一点点小小的修改（当然也可能是很多小小的修改）。对于如何做修改也是有一定讲究的。
那么我就以过来人的身份来讲一些我的几点看法吧。<!--more-->
### 千万不要随意改动库的源码
这一点，是我一直抵触的！我也会对同事说不要随意改动库的代码。一来是尊重原创者，毕竟作者开源出来，也是经过一番精雕细琢，要尊重他人的“血汗”。二来，也是最主要的一点，如果直接改动原库的代码，那么下次作者更新的时候，你想要更新库就会“捉瞎”，完全不知道当时自己改动了哪些地方，无法再更新后的库中保持和原有一致的逻辑。也许也有人会觉得，现在的功能已经可以满足我了，并不需要再更新。但是你不要忘记，开源的世界，大家都在使用，会有很多人提一些优秀的意见和建议，来帮助一起优化、改进。当你看到有`修改bug`、`优化性能`这样的`commit`的时候，难道不心动去更新么。所以切记不要随意动作者的源码。在ios中`cocoapod`这个第三方托管，很好的避免了这一点，当你想要修改源码的时候都会有提醒代码已经被锁定。

### category（扩展类）
<blockquote>
You use categories to define additional methods of an existing class—even one whose source code is unavailable to you—without subclassing. You typically use a category to add methods to an existing class, such as one defined in the Cocoa frameworks. The added methods are inherited by subclasses and are indistinguishable at runtime from the original methods of the class. 
</blockquote>
以上这段话是苹果官方文档对于category的解释。

其实对于想要修改别人的源码来满足需求的时候，就应该如上述所说把源码认为是不可变。那么如何使用category来满足需求呢，下面举个例子来了解下。

就拿iOS中的图片缓存库`SDWebImage`来说，

```` objectivec

/**
 * Set the imageView `image` with an `url` and a placeholder.
 *
 * The download is asynchronous and cached.
 *
 * @param url         The url for the image.
 * @param placeholder The image to be set initially, until the image request finishes.
 * @see sd_setImageWithURL:placeholderImage:options:
 */
- (void)sd_setImageWithURL:(NSURL *)url placeholderImage:(UIImage *)placeholder;

````

上面这个方法是用来加载网络图片的，其中`placeholder`是默认占位图，现在有一个需求对于全局的图片都使用同一张的占位图。那么这个时候如果每次都使用上面的方法来实现真是一个大大的体力活。这个时候可以用category来额外多加一个方法，直接在方法的实现中再调用上面的加载图片的方法。这样做即节省了“体力”，还方便做默认图片的修改。而且别人阅读代码也是一目了然。

### Method Swizzle
method swizzle是一种黑魔法，方法替换，主要是hook原有的方法。既然可以hook原有的方法，那就会有很多事情可以做。一个类只要暴露了方法，就可以hook。

具体例子还是拿`SDWebImage`来说。

```` objectivec

/**
 * Set the imageView `image` with an `url` and a placeholder.
 *
 * The download is asynchronous and cached.
 *
 * @param url         The url for the image.
 * @param placeholder The image to be set initially, until the image request finishes.
 * @see sd_setImageWithURL:placeholderImage:options:
 */
- (void)sd_setImageWithURL:(NSURL *)url placeholderImage:(UIImage *)placeholder;



/**
 * Set the imageView `image` with an `url`, placeholder and custom options.
 *
 * The download is asynchronous and cached.
 *
 * @param url            The url for the image.
 * @param placeholder    The image to be set initially, until the image request finishes.
 * @param options        The options to use when downloading the image. @see SDWebImageOptions for the possible values.
 * @param completedBlock A block called when operation has been completed. This block has no return value
 *                       and takes the requested UIImage as first parameter. In case of error the image parameter
 *                       is nil and the second parameter may contain an NSError. The third parameter is a Boolean
 *                       indicating if the image was retrieved from the local cache or from the network.
 *                       The fourth parameter is the original image url.
 */
- (void)sd_setImageWithURL:(NSURL *)url placeholderImage:(UIImage *)placeholder options:(SDWebImageOptions)options completed:(SDWebImageCompletionBlock)completedBlock;

````

看上面两个方法，第一个方法是比较常见的加载网络图片的用法，通过`SDWebImage`的实现可以知道第一个方法最终会传递实现第二个方法，而options则是一个图片的加载规则，默认的是下载一次失败之后就不再下载，如果说我们想在项目中全局修改这个属性，想下载失败了之后还是要继续下载，直到显示出来，怎么办呢。当然最简单的方法就是改源代码。这个之前也已经提过了，是最最最不可取的方法。其实这个时候可以用method swizzle就可以达到效果。其中hook的是`- (void)sd_setImageWithURL:(NSURL *)url placeholderImage:(UIImage *)placeholder options:(SDWebImageOptions)options completed:(SDWebImageCompletionBlock)completedBlock;`方法。

具体实现如下面代码。
特别要注意的是在交换的方法内，想要在执行原有的方法，继续调用本方法。可以细细品味下因为方法替换了~

```` objectivec

+(void)load
{
	//交换方法
    SEL originalSelector = @selector(sd_setImageWithURL:placeholderImage:options:progress:completed:);
    SEL swizzleSelector = @selector(trick_setImageWithURL:placeholderImage:options:progress:completed:);
    
    Method originalMethod = class_getInstanceMethod([self class], originalSelector);
    Method swizzleMethod = class_getInstanceMethod([self class], swizzleSelector);
    
    method_exchangeImplementations(originalMethod, swizzleMethod);
}


- (void)trick_setImageWithURL:(NSURL *)url placeholderImage:(UIImage *)placeholder options:(SDWebImageOptions)options progress:(SDWebImageDownloaderProgressBlock)progressBlock completed:(SDWebImageCompletionBlock)completedBlock
{
    
    if (options == 0) {
        options = SDWebImageRetryFailed | SDWebImageLowPriority;
    }
    
    //执行原有方法
    [self trick_setImageWithURL:url placeholderImage:placeholder options:options progress:progressBlock completed:^(UIImage *image, NSError *error, SDImageCacheType cacheType, NSURL *imageURL) {
        
        if (completedBlock) {
            completedBlock(image,error,cacheType,imageURL);
        }
    
    }];

}

````

### KVO
KVO全称:(Key-value observing）键值观察。相比较而言我比较喜欢叫监听。但是有一点必须了解的是:使用KVO的要求是对象必须能支持kvc机制——所有NSObject的子类都支持这个机制。
所谓监听，就是某个属性或者值改变的时候给反馈。最常接触的就是`wkwebview`的进度条，通过监听`estimatedProgress `的值来改变进度条状态。

在之前的项目中，就有这样一个需求，使用了某个第三方库，还是个静态库(.a)，但是对于UI的定制化又比较高，对于库给出的配置方法、暴露的属性已经远远不能满足了。因为是静态库，源码看不到，连修改源码这条路都走不通。

然而这个时候可通过监听来做一些调整。

有这么一个case，库中有的viewA在一定条件下触发，然后通过动画显示出来的，但是最终定格的位置和我们实际的位置有所偏差，这个时候可以用kvo，监听viewA的frame变化，获取新值的时候做修改来符合要求。但是要注意的是在修改时候还会触发kvo，所以在修改的方法内要做一点trick，如果已经和预期的一致了，则不再修改。不然则无限调用了。通过类似的方法，在项目中完全实现了UI定制化。


以上主要介绍了三种方法来帮助更好的使用别人的轮子，其实远远不止这些。也许有人会说这样做有点杀鸡用牛刀的感觉，但是虽然可能在一开始改动的时候确实麻烦了一些，但是这个是为了方便以后的更替，其实这么做还是为了保持了“千万不要随意去改动库的源码”的原则。






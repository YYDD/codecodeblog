---
title: iOS的category&swizzle
date: 2016-07-08 15:07:36
tags:
---
category(类的扩展)，我真心觉得是一个奇妙的东西。用法有很多。比如说可以把一个类更加细分化、可以以切面的形式做一些坏坏的事情等等。
之前我突然间脑中蹦出了这样一个想法，如果一个类有两个扩展类，那么这两个扩展类加载的先后顺序是怎样的呢？还有一个想法和前面这个类似，就是在扩展类中用swizzle（这个的用法，具体就不介绍了~相信大家都很腻害的~）做方法替换，那么在不同的扩展类中，我对于同一个方法做了多次的替换，那么这时候具体的实行顺序又是怎样的呢？<!--more-->
我觉得多说无益，还是直接写个demo测试下，来得实在。

<center>![截图](http://7xrcp9.com1.z0.glb.clouddn.com/blogQQ20160708-0%402x.png?imageView2/2/w/500/q/50)</center>

如上图中，在demo中我创建了一个`Test`的类，以及两个关于`Test`的扩展类`Test+t1`和`Test+t2`，然后分别在实现文件中的`+(void)load`方法中输出日志，运行后得出加载的顺序依次是：`Test`,`Test+t1`,`Test+t2`。当然对于第一个是`Test`是没有疑问的，而对于后面的两者的顺序，其实我是有疑惑的，为什么是这样的顺序而不是反一下呢。后来发现其实加载顺序和下图中的build Phases里面的compile Sources里面的前后顺序有关。经过测试确实是如果我把这两者的顺序对换下，加载时候顺序就会有变化。

<center>![截图](http://7xrcp9.com1.z0.glb.clouddn.com/blogQQ20160708-1%402x.png?imageView2/2/w/500/q/50)</center>

ok~那这个问题，我觉得认为是解决了。其实对于哪个扩展类先加载，哪个后加载，照理来说是没有关系的，因为正常的情况下，逻辑处理上不应该或者说得严格点应该是不可能在不同的扩展类之间有先后或者说相互依赖的关系。只不过我比较转牛角尖而已。

那么还有一个问题是关于swizzle的问题。
先来看下，我的demo是这样的。
在`Test`中有一个公开的方法

```` objectivec

	-(void)doLog
	{
	    NSLog(@"test----doLog");
	}

````

而在`Test+t1`中将`doLog`方法和`swizzle_doLog_t1`方法做替换，之后再调用原方法

```` objectivec

	+(void)load
	{
	    [Test doSwizzle];
	}
	
	+(void)doSwizzle
	{
	    Method originalMethod = class_getInstanceMethod(self, @selector(doLog));
	    Method swizzleMethod = class_getInstanceMethod(self, @selector(swizzle_doLog_t1));
	    
	    method_exchangeImplementations(originalMethod, swizzleMethod);
	}
	
	
	-(void)swizzle_doLog_t1
	{
	    NSLog(@"t1----doLog");
	    [self swizzle_doLog_t1];

	}

````

在`Test+t2`中用同样形式对`doLog`方法做替换

```` objectivec

	+(void)load
	{
	    NSLog(@"test+t2 --load");
	    
	    [Test doSwizzle1];
	}
	
	+(void)doSwizzle1
	{
	    Method originalMethod = class_getInstanceMethod(self, @selector(doLog));
	    Method swizzleMethod = class_getInstanceMethod(self, @selector(swizzle_doLog_t2));
	    
	    
	//    SEL selector = method_getName(originalMethod);
	//    NSLog(@"%c",sel_getName(selector));
	    
	    method_exchangeImplementations(originalMethod, swizzleMethod);
	}
	
	
	-(void)swizzle_doLog_t2
	{
	    NSLog(@"t2----doLog");
	    [self swizzle_doLog_t2];
	}
	
````

好了，demo算是写完了。那么如果加载的时候顺序是`Test`,`Test+t1`,`Test+t2`。这时候调用`Test`的`doLog`方法，具体会依次执行哪些方法呢？
下面是输入的日志的顺序：
2016-07-08 15:56:52.961 Test[2205:172308] t2----doLog
2016-07-08 15:56:52.961 Test[2205:172308] t1----doLog
2016-07-08 15:56:52.961 Test[2205:172308] test----doLog
恩，也就是说方法调用中，依次的顺序是：`swizzle_doLog_t2`,`swizzle_doLog_t1`,`doLog`。
为什么会是这样的？这时候有必要理解下swizzle具体做的事情了，或者说它的实现的原理是什么。
过多的不说了，我觉得和上面的顺序有关的，只要理解下面这句话就可以了:
<strong>swizzle做替换其实替换的是selector的IMP</strong>
好好的感受下这句话的意思~~~~~
对于上面的，我们来一步步分开来的看吧。对于`Test+t1`的替换，其实只是把`doLog`这个方法(selector)的IMP由原来的换成了`swizzle_doLog_t1`这个方法原先的IMP。所以说在`Test+t2`的替换中`doLog`此时的IMP已经是`swizzle_doLog_t1`的IMP，所以...才会出现上面日志中输出的那种情况。具体如下图所示：

<center>![截图](http://7xrcp9.com1.z0.glb.clouddn.com/blogpppp.png?imageView2/2/w/700/q/50)</center>

文章开始提到的两个问题，就这样也算全部解决了。我觉得会有这两个问题的产生，第一个我归结于对于ios的编译的过程不了解，以及头脑发热。而第二个问题我觉得是主要对于swizzle的实现机制、原理并没有了解的很透彻~~




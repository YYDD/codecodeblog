---
title: NSTimer和GCD
date: 2016-07-26 14:40:06
tags:
---
在ios的实际运用中，延迟执行方法，一般有`GCD`、`NSTimer`和`performSelector`方法。这里我主要要讲的是`GCD`和`NSTimer`这两大类。

### NSTimer
NSTimer的创建很简单，基本上如果是直接用`@selector`的话，基本上就下面两种方法：
```` objectivec
    + (NSTimer *)timerWithTimeInterval:(NSTimeInterval)ti target:(id)aTarget selector:(SEL)aSelector userInfo:(nullable id)userInfo repeats:(BOOL)yesOrNo;
    + (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)ti target:(id)aTarget selector:(SEL)aSelector userInfo:(nullable id)userInfo repeats:(BOOL)yesOrNo;
````

对于这两个方法虽然乍一眼看过去没差，参数什么的都一样的。但是其实这里还是存在一点点小坑的。不过在讲这个小坑之前需要讲一下RunLoop。<!--more-->
**NSRunLoop**是消息机制的处理模式。NSRunLoop的作用在于有事情做的时候使得当前的NSRunLoop的线程工作，没有事情做的时候让其线程休眠。NSTimer默认是添加到当前NSRunLoop中，也可以手动指定添加到自己新建的NSRunLoop中。NSRunLoop就是一直在循环检测，从线程的开始到线程的结束，检测事件。
而NSRunLoop大概有以下几种模式：

 - default模式：几乎包括所有输入源（除NSConnection）。即NSDefaultRunLoopModel模式
 - model模式：处理modal panels
 - connection模式：处理NSConnection事件，属于系统内部，用户基本不用
 - event tracking模式：如组件拖动输入源，但是不处理定时事件。即UITrackingRunLoopModel
 - common model模式：这是一组可配置的通用模式。将input sources与该模式关联则同时也将input sources与该组中的其他模式进行了关联。即NSRunLoopCommonModel
 
 好了对于NSRunLoop的科普，就暂时先到这里~
 继续讲上面两种创建NSTimer的方法。而这两个方法的区别也正是坑的所在。第一个方法也就是`timerWithTimeInterval`这个方法只是创建了`NSTimer`，并不会将他添加到任何一个RunLoop中。所以这个的意思就是说该方法创建出来的`NSTimer`并不能直接跑定时器，因为他不在任何一个RunLoop钟。所以还需要将他add到一个RunLoop中。而第二个方法`scheduledTimerWithTimeInterval`创建的时候就已经将`NSTimer`自动添加到默认的RunLoop中了。那么可以说第二个方法相对来说是比较好用的，因为他并不需要再多做一步将其加入到RunLoop中。恩~可以这么说，但是这里我还要小小的科普一个知识。也是关于RunLoop的。
如果当前的模式不是NSTimer所在的模式，也就是RunLoop进入了休眠状态，那么这时候NSTimer肯定是不会持续响应的。那么具体哪几种情况会这样呢，就简单的举个例子，也是最常见的。比如通过`scheduledTimerWithTimeInterval`创建的NSTimer是在默认的RunLoop中，也就是NSDefaultRunLoopModel中的，但是如果这时候界面中有列表，滑动列表，这个操作当前的RunLoop会切换到UITrackingRunLoopModel，这个时候NSTimer就不会响应。

基于以上这一点，大家是不是觉得之前的有些定时器的实现好像会有点小问题呢。那么如何做到在任何情况下都可以保证NSTimer响应呢？其实很简单，主要把NSTimer加入到NSRunLoopCommonModel模式下即可。这一点对于用`timerWithTimeInterval`创建的完全没问题，因为他一开始就没有在任何一个RunLoop中，而用`scheduledTimerWithTimeInterval`创建的因为一开始在默认的RunLoop中，重新将其add到另一个model中不知道会不会有影响，然而我直接测试看效果是没有影响的，但是具体的后台的操作是怎样的，我就不清楚了。所以在不知情的情况下，我认为还是用`timerWithTimeInterval`创建，比较靠谱。


### GCD
对于GCD，我只能说好用！好用！真好用！！！=，=。
GCD都是以block的形式，所以用起来相对来说比较方便，而且代码看起来也比较清爽。最主要的一点是GCD所执行的可以跨进程响应，这一点是NSTimer没办法做到的。GCD之前一直说有一点不足的是执行的逻辑没办法取消。不过如果用GCD来创建timer的话，完全没有这个顾虑，这个兼职是结合了GCD和NSTimer的优点。具体的创建如下：

```` objectivec
    self.timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0));
    dispatch_source_set_timer(self.timer, DISPATCH_TIME_NOW, 1.0 * NSEC_PER_SEC, 0.5 * NSEC_PER_SEC);
    dispatch_source_set_event_handler(self.timer, ^{
        NSLog(@"-----test....dispatch_timer");
        dispatch_cancel(self.timer);    //执行一次就取消
    });
    dispatch_resume(self.timer);
````

看上面的代码，是不是有种帅爆了的感觉。但是这里也同样有个小坑，就是所创建的`dispatch_source_t`类型的`timer`，需要对他持有，所以是全局变量，不然的话在arc的情况下，会马上被回收掉。

### 总结
对于这两者，其实各有千秋吧。
但是单单就我自己而言，我是比较喜欢GCD的这种形式的，主要原因还是GCD的这种形式的代码结构比较清晰。然而因为GCD用的是block的形式，那么就肯定会有block的循环引用的问题。而NSTimer如果不在特定的时候停止，也会一直强持有，也会造成内存泄露的问题。
所以说各有千秋，看大家的喜好咯~



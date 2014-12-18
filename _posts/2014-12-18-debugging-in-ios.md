---
layout: post
title: iOS开发中常用的调试方法
description: "Great debugging in iOS"
categories: Foundation
tags: [debug]
imagefeature: "/blog/debugging_in_ios_cover.jpg"
comments: true
mathjax: null
featured: true
published: true
---


Bugs，理论上是永远存在的，很多同学比较憎恶。何为bug？——小强是也，百打不死，它还和颜悦色，只时不时的作弄你一下。我倒觉得既然早晚得面对，与其憎恶还不如喜欢，**“与之斗，其乐无穷”**。<br>
然而，日前帮“某生”调试了几个Bugs，了解到为什么其憎恶bug到极点，因为他不知道怎么跟bugs斗。因此，根据个人（有限的）经验整理一些常用的调试方法，聊做入门之参考。


---
>“工欲善其事，必先利其器。”<small><cite title="Plato">《论语·卫灵公》</cite></small>

    Nori是在XCode环境下做iOS开发的，以下内容仅供XCode用户参考，如熟练掌握XCode调试的请走后门，不送……

XCode本身提供了大量的调试功能和工具，Nori比较懒只看了XCode文档的一部分，如欲成大神，自己回去把XCode的官方文档从头到尾看一遍。废话结束，开始正题。

---
<section>
  <header>
    <h2 >XCode常用的Bugs调试手段如下：</h2>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section>

---

## <del>用眼睛从头到尾看</del>

这是Nori在09年刚入行时候的“调试”方法，此法本不提也罢，列出来是为了提醒大家该摒弃此法，效率太低。

---

## 强大的NSLog

NSLog()基本上是iOS开发中最常用的方法，新手第一个学习的函数估计就是他了。其定义如下：
`void NSLog ( NSString *format, ... )`<br/>
最简单莫过于`NSLog(@"Hello World");` 此项本不必多说，只说说其*format和一些特殊的用法。

### NSLog()的format

format格式众多，不一一列举在此只罗列常用几种：

<table class="table table-bordered table-hover">
    <thead>
        <tr>
            <th>format</th><th>含义</th>
        </tr>
    </thead>
    <tbody>
        <tr><td>%@    </td><td>对象</td></tr>
        <tr><td>%p    </td><td><b>指针</b></td></tr>
        <tr><td>%d, %i</td><td>整型</td></tr>
        <tr><td>%u    </td><td>无符整型</td></tr>
        <tr><td>%x, %X</td><td>二进制整型</td></tr>
        <tr><td>%o    </td><td>八进制整型</td></tr>
        <tr><td>%lld  </td><td>64位长整型（long long）</td></tr>
        <tr><td>%f    </td><td>浮点型</td></tr>
        <tr><td>%lf    </td><td>64位双字浮点型</td></tr>
        <tr><td>%e    </td><td>浮点/双字 （科学计数法）</td></tr>
        <tr><td>%s    </td><td>C字符串</td></tr>
        <tr><td>%c    </td><td>字符</td></tr>
        <tr><td>%C    </td><td>unichar</td></tr>
    </tbody>
</table>

### NSLog日志在控制台着色

关于日志的框架很多，只介绍Nori常用的一个简单着色的XCode插件——[XcodeColors](https://github.com/robbiehanson/XcodeColors)。<br/>
安装之后，在添加以下宏定义为XCodecolorHelper.h，然后全局import，[更多用法自查](https://github.com/robbiehanson/XcodeColors#how-to-use-xcodecolors)。
{% highlight objective-c%}
#define XCODE_COLORS_ESCAPE  @"\033["

#define XCODE_COLORS_RESET_FG  XCODE_COLORS_ESCAPE @"fg;" // Clear any foreground color
#define XCODE_COLORS_RESET_BG  XCODE_COLORS_ESCAPE @"bg;" // Clear any background color
#define XCODE_COLORS_RESET     XCODE_COLORS_ESCAPE @";"   // Clear any foreground or background color

// log蓝色内容
#define LogBlue(frmt, ...) NSLog((XCODE_COLORS_ESCAPE @"fg0,0,255;" frmt XCODE_COLORS_RESET), ##__VA_ARGS__)
// log红色内容
#define LogRed(frmt, ...) NSLog((XCODE_COLORS_ESCAPE @"fg255,0,0;" frmt XCODE_COLORS_RESET), ##__VA_ARGS__)
// log绿色内容
#define LogGreen(frmt, ...) NSLog((XCODE_COLORS_ESCAPE @"fg0,255,0;" frmt XCODE_COLORS_RESET), ##__VA_ARGS__)
// log制定色内容
// color格式，注意分号
//    fgrgb;，前景文字色，例如@"bg0,255,0;"
//    bgrgb;，背景色，例如@"bg255,0,0;"
// 输出绝对是“红配绿赛狗屁”
#define NXLogColored(color, frmt, ...) NSLog((XCODE_COLORS_ESCAPE color frmt XCODE_COLORS_RESET), ##__VA_ARGS__)
{% endhighlight %}
效果如图：<br>
<figure>
    <a href="{{ site.url }}/images/blog/debugging_in_ios_1.png"><img src="{{ site.url }}/images/blog/debugging_in_ios_1.png"></a>
</figure>

### 利用私有API输出对象的更多信息

在一些时候，我们需要打印更多关于对象的信息，通常我们会自己重写-description或者-debugDescription方法来解决。自从知道了iOS7之后NSObject基类有了几个私有方法之后，一切都改变了。
{% highlight objective-c%}
@interface NSObject (Private)
// 返回格式化的字符串，内容为该对象的成员变量以及类型和值（并且明确标出继承自父类的成员变量）
-(id)_ivarDescription;
// 返回格式化的字符串，内容为该对象的成员方法和属性列表，包括getter和setter，省略掉父类中继承来的方法等信息
-(id)_shortMethodDescription;
// 返回格式化的字符串，内容为该对象的成员方法和属性列表，包括getter和setter，包含完整父类中继承来的方法等信息
-(id)_methodDescription;
@end
{% endhighlight %}
调试时，将以上内容添加到工程。然后调用即可。
{% highlight objective-c %}
NSLog(@"%@",[obj _ivarDescription]);
NSLog(@"%@",[obj _shortMethodDescription]);
NSLog(@"%@",[obj _methodDescription]);
{% endhighlight %}
例如：
{% highlight objective-c %}
#import "ViewController.h"
@interface NSObject (Private)
-(id)_ivarDescription;
-(id)_shortMethodDescription;
-(id)_methodDescription;
@end

// 定义一个简单的类
@interface SampleClass : NSObject
@property (nonatomic, strong) NSString *aProperty;
@end

@implementation SampleClass{
    NSString *aVar;
}

- (instancetype)initWithString:(NSString *)string {
    self = [super init];
    if(self) {
        aVar = string;
        self.aProperty = string;
    }
    return self;
}
@end

@interface ViewController ()
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    id obj = [[SampleClass alloc] initWithString:@"Hello"];
    NSLog(@"%@",[obj _ivarDescription]);
    NSLog(@"%@",[obj _shortMethodDescription]);
    NSLog(@"%@",[obj _methodDescription]);
}

@end
{% endhighlight %}
运行结果太长不全贴了，只贴_ivarDescription和_shortMethodDescription结果。

<figure>
    <a href="{{ site.url }}/images/blog/debugging_in_ios_2.png"><img src="{{ site.url }}/images/blog/debugging_in_ios_2.png"></a>
</figure>

如何，有没有看到全裸美女的感觉呢？

---

## 更强的断点（Breakpoint）

断点（Breakpoint）是调试工程最常用，也是最传统的方法了，其实XCode断点调试这方面Nori不太敢写，因为了解不够全面，没能耐着性子把文档看完，但是本着敝帚也得拿出来显摆的原则，写写自己知道的或许能帮助一下哪些还不知道的。

### 在指定行添加/删除断点

![]({{ site.url }}/images/blog/debugging_in_ios_3.gif)

### 调试工具栏

![]({{ site.url }}/images/blog/debugging_in_ios_4.png)

1. 断点调试开关，添加断点时自动打开（图为打开），如果关闭XCode将disable所有断点；
2. 运行时暂停，进入调试状态；
3. 单步执行，点击一次执行一行，其他组合键<kbd>ctrl</kbd>+click、<kbd>ctrl</kbd>+<kbd>shift</kbd>+click，分别对应`指令`和`线程`。
4. 进入执行，如果执行到方法调用，点击即可进入该方法内部，配合5进行调试，其他组合键<kbd>ctrl</kbd>+click、<kbd>ctrl</kbd>+<kbd>shift</kbd>+click，同样分别对应`指令`和`线程`。
5. 当执行到方法内部，点击直接完成当前方法执行。

### 条件断点/约束断点（Conditional Breakpoints）

<p class="bg-info">条件断点/约束断点（Conditional Breakpoints），即在满足指定条件下触发的断点。</p>

1. 添加断点；
2. 在断点上右键，Edit Breakpoint...；
3. 在Condition框中添加条件。

[![]({{ site.url }}/images/blog/debugging_in_ios_5.gif)]({{ site.url }}/images/blog/debugging_in_ios_5.gif)

### 异常断点（Exceptions Breakpoint）

异常断点（Exceptions Breakpoint），当运行时有**异常抛出时**自动break，进入调试状态。因为是抛异常，所以貌似没法用来查找过度释放所导致的Crash。<br/>

1. 在XCode左侧导航里切换到断点导航，<kbd>⌘</kbd>+<kbd>7</kbd>;
2. 单机“+”, Add Exceptions Breakpoint;<br/>![]({{ site.url }}/images/blog/debugging_in_ios_6.png)
3. 然后运行工程，如果运行中又异常被抛出，即自动break进入调试。

<pre>此外，也可以通过前面提到的编辑菜单，进行修改所监听的异常类型、以及触发的时机（throw/catch）阶段。</pre>

### 符号断点（Symbolic Breakpoint）

符号断点（Symbolic Breakpoint），根据设定符号（可以是方法名、函数名、只要是通过目标-消息结构实现的一切符号）监听消息发送（方法、函数的执行）。<br/>

1. 在XCode左侧导航里切换到断点导航，<kbd>⌘</kbd>+<kbd>7</kbd>;
2. 单机“+”, Add Symbolic Breakpoint;<br/>![]({{ site.url }}/images/blog/debugging_in_ios_7.png)
3. 在出现的编辑Popover里的Symbol框里添加要加breakpoint的符号，例如：<br/>`viewDidLoad`：给所有viewDidLoad方法调用处添加breakpoint;<br/>`[ViewController viewDidLoad]`：只给ViewController类的viewDidLoad方法调用处添加breakpoint。

### 断点追踪(Watch point)

断点追踪(Watch Breakpoints)，顾名思义就是在断点处追踪某个变量值得变化，这是个很实用的功能。<br/>例，在SampleClass里添加一个int型属性total；
{% highlight objective-c %}
@interface SampleClass : NSObject
@property (nonatomic, assign) int total;
@end

@implementation SampleClass

- (instancetype)init{
    self = [super init];
    if(self) {
        _total = 0; // add breakpoint here
    }
    return self;
}
@end

@interface ViewController ()
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    SampleClass *obj = [[SampleClass alloc] init];
    [self doSomething];
    obj.total += arc4random() % 10;
    [self doSomething];
    obj.total += arc4random() % 10;
    [self doSomething];
    obj.total += arc4random() % 10;
    [self doSomething];
    obj.total += arc4random() % 10;
    [self doSomething];
    obj.total += arc4random() % 10;
    [self doSomething];
    obj.total += arc4random() % 10;
}

- (void)doSomething {
    NSLog(@"I'm do something...");
}
{% endhighlight %}

1. 在需要开始监听的位置加一个breakpoint，本例加载初始化从开始就监听total属性的变化；<br/>[![]({{ site.url }}/images/blog/debugging_in_ios_8.png)]({{ site.url }}/images/blog/debugging_in_ios_8.png)
2. 运行，断点触发之后，在下发左侧的调试窗口中找到要监听变化的变量，本例为`_total`（因为total是用@propty构造器生成的，构造器在生成getter和setter的时候，同时还自动检测/生成一个带_前缀的成员变量）;
3. 在_total上右键，watch "_total";
4. 点击“继续运行”按钮（原暂停按钮），继续运行，每当_total的数值发生变化的时候，都会暂停并且在右侧控制台输出变化前后的值。

[![]({{ site.url }}/images/blog/debugging_in_ios_9.gif)]({{ site.url }}/images/blog/debugging_in_ios_9.gif)


### 断点动作（Breakpoint Action）

在前面编辑断点的时候，细心的读者应该已经发现在弹出的Popover了有个Action选项，可以添加6种Action: `AppleScript`、 `Capture GPU Frame`、 `Debugger Command`、 `Log Message`、 `Shell Command`、 `Sound`。Nori比较常用的是Log Message & Debugger Command。

#### Log Message

Log Message下分两种方式，一种是控制台输出，一种是给哥读出来。还以前边的SampleClass为例。

1. 修改ViewController如下，并在38行添加断点：<br/>[![]({{ site.url }}/images/blog/debugging_in_ios_10.png)]({{ site.url }}/images/blog/debugging_in_ios_10.png)
2. 编辑断点，为了验证在条件/约束框（Condition）里添加条件“`number > 1`”，即在参数number>1时才触发断点；添加Action，并选择Log Message, 输入**“`The current number is @number@, and the current total is @obj.total@.`”**至于是输出到控制台（Log message to console）还是让它蹩脚的读出来（Speak message）自己决定吧。<br/>[![]({{ site.url }}/images/blog/debugging_in_ios_11.png)]({{ site.url }}/images/blog/debugging_in_ios_11.png)
3. 运行，结果为“The current number is 2, and the current total is 1.”，如下。<br/>[![]({{ site.url }}/images/blog/debugging_in_ios_12.png)]({{ site.url }}/images/blog/debugging_in_ios_12.png)

注：**...total is <font color="red">1</font>.**, 再次提醒大家，breakpoint触发的时，其所在的行（以及所有暂停状态游标所在的行），都尚未执行。本例中即obj.total += 2，如果将本例中的断点拖动到39行，那么结果就是“The current number is 2, and the current total is **<font color="red">3</font>**.”。

#### Debugger Command

操作基本同上，不复述，上图：<br/>[![]({{ site.url }}/images/blog/debugging_in_ios_15.png)]({{ site.url }}/images/blog/debugging_in_ios_15.png)<br/>

另外说明，`po` 之后可以跟变量名，也可跟方法和函数的调用，例如，`po obj`，`po [self doSomeAction]`（doSomeAction是自己写的方法）。因此，在处理异常和其他断点的时候，我们可以通过`po <方法调用>`来触发相应的处理方法，举个极端的例子：

1. 添加一个方法-dealNumber:，目的是凡是number超过1（breakpoint条件决定），就给他砍去1刀（dealNumber决定）。<br/>[![]({{ site.url }}/images/blog/debugging_in_ios_13.png)]({{ site.url }}/images/blog/debugging_in_ios_13.png)
2. 修改断点：`po number = [self dealNumber:number]`
3. 运行结果：<br/>[![]({{ site.url }}/images/blog/debugging_in_ios_14.png)]({{ site.url }}/images/blog/debugging_in_ios_14.png)<br/>其中1是断点触发时`po`后方法调用的返回值，2是`NSLog(@"%d", obj.total);`。可见经过此折腾，原本应该是3的结果变成2了。

#### Sound

操作基本同上，为什么拿出来说呢？因为很实用，Nori以前做过不少的数据容错处理，为了不耽误时间把机器的声音开大，然后，run，再然后……健个身，和妹子聊回天啥的，哈哈。

其他的就不说了，提醒一点，一个breakpoint里Action可以加多种，多个。Actions组合使用可以大大提高调试的效率。

---

## Quick Look

这应该比较常见，有的时候不小心就会出来，5.1以后大家应该都比较熟知。所以不多做介绍，附个[简单教程](http://www.tuicool.com/articles/zemYJ3b)。

---

## 调试仪表盘（Debug Gauges）& Instruments

这个太高大上了，我就不叨叨啥了。……呵呵，有点烂尾了，以后有空再叨叨吧……

---


以上是**[@Nori](http://weibo.com/pimacun1024)**比较常用的调试手段，比较粗陋，学习更多Debug方法请走传送门:[Debugging in Xcode](https://developer.apple.com/videos/wwdc/2014/#413)、[Apple Developer Support - Debugging](https://developer.apple.com/support/technical/debugging/)。如有谬误欢迎斧正，如果还有什么你知道的我没提到的，欢迎分享。


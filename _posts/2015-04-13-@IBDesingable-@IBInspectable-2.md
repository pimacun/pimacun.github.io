---
layout: post
title: 如何在Xcode 6+中创建可设计的视图控件（二）
description: "@IBDesingable & @IBInspectable - 2"
categories: Foundation
tags: [design, xcode]
imagefeature: "/blog/@IBDesingable_@IBInspectable_cover.jpg"
comments: true
mathjax: null
featured: true
published: true
---


本篇是对上一篇[如何在Xcode 6+中创建可设计的视图控件](http://pimacun.github.io/foundation/@IBDesingable-@IBInspectable/)的补充。摒弃了令人头疼的`drawRect:`，通过`-(void)layoutSubviews`和iOS8+之后引入的`-(void)prepareForInterfaceBuilder`来完成InterfaceBuilder内的重绘和初始化工作。

---
>首先，先说下`-(void)prepareForInterfaceBuilder`，这是iOS8之后Simulator中对NSObject添加的一个方法，主要用来初始化当前类在InterfaceBuilder中内容。然后，为@IBInspectable变量赋值时，通过`-(void)layoutSubviews`方法进行界面重绘和布局。

---
<section>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section>

---

## 目标

本次依然是一个简单的控件CircleStepper，效果如下。<br/>
<figure>
    <a href="{{ site.url }}/images/blog/@IBDesingable_@IBInspectable-2-all.gif"><img src="{{ site.url }}/images/blog/@IBDesingable_@IBInspectable-2-all.gif"></a>
</figure>
CircleStepper有5个@IBInspectable属性：`bgColor`、`fgColor`、`fontSize`、`stepCount`、`currentStep`。分别对应弧形进度条底色、弧形进度条前景色、中间Label的字体大小、进度上限和当前进度。

## 结构

关于`@IBDesignable`和`@IBInspectable`的使用，详见[前文](http://pimacun.github.io/foundation/@IBDesingable-@IBInspectable/)，不在复述。<br/>
直接看本次Demo的结构。
首先在`-(void)prepareForInterfaceBuilder`和`-(void)awakeFromNib`方法中对界面元素完成初始；
然后在`-(void)layoutSubviews`方法中，通过属性值对界面元素进行重绘和布局，OK。

## 实施

### 定义成员变量（属性）

1.新建CircleStepper类，定义5个@IBInspectable变量如下：
{% highlight objective-c %}
IB_DESIGNABLE
@interface CircleStepper : UIView

@property(nonatomic, strong) IBInspectable UIColor *bgColor;
@property(nonatomic, strong) IBInspectable UIColor *fgColor;
@property(nonatomic, assign) IBInspectable CGFloat fontSize;
@property(nonatomic, assign) IBInspectable CGFloat stepCount;
@property(nonatomic, assign) IBInspectable CGFloat currentStep;

@end
{% endhighlight %}
<br/>
2.为了定义变量`stepperLabel`、`bgLayer`、`fgLayer`分辨对应3个界面元素，这里几个私有变量。
{% highlight objective-c %}
@interface CircleStepper ()

@property (nonatomic, strong) UILabel *stepperLabel;
@property (nonatomic, strong) CAShapeLayer *bgLayer;
@property (nonatomic, strong) CAShapeLayer *fgLayer;

@end
{% endhighlight %}

### 初始化

 定义初始化方法`-(void)setup`，并且在`prepareForInterfaceBuilder`和`awakeFromNib`中分别调用(如果需要添加代码创建，那么重载`initWithFrame:`方法，调用`setup`)。在`setup`方法中对两个弧形的进度条（CAShapeLayer）和中间Label进行初始和基础设置。代码如下：
{% highlight objective-c %}
- (void)prepareForInterfaceBuilder {
    [super prepareForInterfaceBuilder];
    [self setup];
}

- (void)awakeFromNib {
    [super awakeFromNib];
    [self setup];
}
- (void)setup {
    // Default
    if (!self.backgroundColor) {
        self.backgroundColor = [UIColor clearColor];
    }
    if (!self.bgColor) {
        self.bgColor = [UIColor blackColor];
    }
    if (!self.fgColor) {
        self.fgColor = [UIColor colorWithWhite:1.000 alpha:0.750];
    }
    if (self.stepCount == 0) {
        self.stepCount = 10;
        self.currentStep = 5;
    }
    if (self.fontSize == 0) {
        self.fontSize = 26;
    }
    
    // Setup bg
    _bgLayer = [CAShapeLayer layer];
    _bgLayer.fillColor = [UIColor clearColor].CGColor;
    _bgLayer.strokeColor = _bgColor.CGColor;
    _bgLayer.strokeEnd = 1;
    [self.layer addSublayer:_bgLayer];

    // Setup fg
    _fgLayer = [CAShapeLayer layer];
    _fgLayer.fillColor = [UIColor clearColor].CGColor;
    _fgLayer.strokeColor = _fgColor.CGColor;
    _fgLayer.strokeEnd = 0;
    [self.layer addSublayer:_fgLayer];
    
    
    // Setup stepper label
    _stepperLabel = [[UILabel alloc] init];
    _stepperLabel.font = CIRLE_STEPPER_LABLE_FONT;
    _stepperLabel.textColor = [UIColor whiteColor];
    _stepperLabel.backgroundColor = [UIColor clearColor];
    _stepperLabel.textAlignment = NSTextAlignmentCenter;
    _stepperLabel.text = @"0/0";
    [_stepperLabel setTranslatesAutoresizingMaskIntoConstraints:NO];
    [self addSubview:_stepperLabel];
}
{% endhighlight %}
<br/>

### 界面刷新

重载`- (void)layoutSubviews`方法，刷新界面元素并且添加动画。代码如下：
{% highlight objective-c %}
- (void)layoutSubviews {NSLog(@"%@", @"LayoutSubviews");
    [super layoutSubviews];
    // 刷新
    [self setupShapeLayer:_bgLayer];
    [self setupShapeLayer:_fgLayer];
    [_stepperLabel setFrame:self.bounds];
    // 动画
    [self animate];
}
{% endhighlight %}
因`bgLayer`和`fgLayer`基本相同，都是简单的弧形，所以定义`-(void)setupShapeLayer:`方法，对其进行界面刷新。代码如下：
{% highlight objective-c %}
- (void)setupShapeLayer:(CAShapeLayer *)shapeLayer {
    shapeLayer.frame = self.bounds;
    shapeLayer.lineWidth = CIRLE_STEPPER_LINE_WIDTH;
    CGPoint center = CGPointMake(CGRectGetWidth(self.bounds) / 2.0, CGRectGetHeight(self.bounds) / 2.0);
    CGFloat radius = CGRectGetWidth(self.bounds) * 0.4;
    CGFloat startAngle = DegreesToRadians(135.0);
    CGFloat endAngle = DegreesToRadians(45.0);
    UIBezierPath *layerPath = [UIBezierPath bezierPathWithArcCenter:center radius:radius startAngle:startAngle endAngle:endAngle clockwise:true];
    shapeLayer.path = layerPath.CGPath;
}
{% endhighlight %}
利用基本的CABasicAnimation让数值变化看起来更生动一些。代码如下：
{% highlight objective-c %}
- (void)animate {
    _stepperLabel.text = [NSString stringWithFormat:@"%.0f/%.0f", _currentStep, _stepCount];
    CGFloat fromValue = _fgLayer.strokeEnd;
    CGFloat toValue = _currentStep / _stepCount;
    if (_fgLayer.presentationLayer != nil && [_fgLayer.presentationLayer isKindOfClass:[CAShapeLayer class]]) {
        fromValue = [(CAShapeLayer *)_fgLayer.presentationLayer strokeEnd];
    }
    int pctChange = fabs(fromValue - toValue);
    CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"strokeEnd"];
    animation.fromValue = [NSNumber numberWithFloat:fromValue];
    animation.toValue = [NSNumber numberWithFloat:toValue];
    animation.duration = pctChange * 4;
    [_fgLayer removeAnimationForKey:@"stroke"];
    [_fgLayer addAnimation:animation forKey:@"stroke"];
    [CATransaction begin];
    [CATransaction setDisableActions:YES];
    _fgLayer.strokeEnd = toValue;
    [CATransaction commit];
}
{% endhighlight %}
然后，就没有然后。

## 总结 & Demo下载

本来想给Label添加个UIFont的@IBInspectable属性，但是失败了不知道怎么加，所以只添加了fontSize属性，有知道的可以私信我，我再修改。<br/>
<a href="http://pan.baidu.com/s/1hqIlBTy" target="_blank">![](/images/download.png)</a><br/>
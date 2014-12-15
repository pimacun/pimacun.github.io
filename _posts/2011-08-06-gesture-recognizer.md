---
layout: post
title: "基本手势"
description: "sample gesture recognizer"
category: Foundation
tags: [gesture]
imagefeature: codeplace.jpg
comments: false
mathjax: null
featured: false
published: true
---

长久以来一直都依靠touchesXx方法来定制手势，所以忽略了许多程序自定义的手势。今天回头重来，重新关注下，iOS SDK自带的好东西。

## 轻扫
先尝试swipe手势，实现如下：
{% highlight objective-c %}
UISwipeGestureRecognizer *swipe = [[UISwipeGestureRecognizer alloc] initWithTarget:self action:@selector(handleSwipe:)]; [swipe setDelegate:self]; swipe.direction = UISwipeGestureRecognizerDirectionRight | UISwipeGestureRecognizerDirectionLeft; [self.view addGestureRecognizer:swipe]; [swipe release];
{% endhighlight %}
很简单，Demo就不上了，以后完善了各种手势，再上吧。

---

## 用手势区分单双击：
{% highlight objective-c %}
- (void)viewDidLoad {
    
    UITapGestureRecognizer *singleTapOne = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(handleSingleTap:)];
singleTapOne.numberOfTouchesRequired = 1; singleTapOne.numberOfTapsRequired = 1; singleTapOne.delegate = self;
    
    UITapGestureRecognizer *singleTapTwo = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(handleSingleTap:)];
singleTapTwo.numberOfTouchesRequired = 2; singleTapTwo.numberOfTapsRequired = 1; singleTapTwo.delegate = self;
    
UITapGestureRecognizer *doubleTapOne = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(handleDoubleTap:)];
doubleTapOne.numberOfTouchesRequired = 1; doubleTapOne.numberOfTapsRequired = 2; doubleTapOne.delegate = self;
    
UITapGestureRecognizer *doubleTapTwo = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(handleDoubleTap:)];
doubleTapTwo.numberOfTouchesRequired = 2; doubleTapTwo.numberOfTapsRequired = 2; doubleTapTwo.delegate = self;
    
[singleTapOne requireGestureRecognizerToFail:doubleTapOne]; // Single tap requires double tap to fail
    [singleTapTwo requireGestureRecognizerToFail:doubleTapTwo]; 
    
[self.view addGestureRecognizer:singleTapOne]; [singleTapOne release];
    [self.view addGestureRecognizer:singleTapTwo]; [singleTapTwo release];
[self.view addGestureRecognizer:doubleTapOne]; [doubleTapOne release];
[self.view addGestureRecognizer:doubleTapTwo]; [doubleTapTwo release];
    
    [super viewDidLoad];
}
- (void)handleSingleTap:(UITapGestureRecognizer *)sender {
    if (sender.numberOfTouchesRequired == 1) {
        NSLog(@"Single Tap with a finger.");
    }
    else if (sender.numberOfTouchesRequired == 2) {
        NSLog(@"Single Tap with two finger.");
    }
}
- (void)handleDoubleTap:(UITapGestureRecognizer *)sender {
    if (sender.numberOfTouchesRequired == 1) {
        NSLog(@"Double Tap with a finger.");
    }
    else if (sender.numberOfTouchesRequired == 2) {
        NSLog(@"Double Tap with two finger.");
    }
}
{% endhighlight %}

## 其他
其他手势以后补上……
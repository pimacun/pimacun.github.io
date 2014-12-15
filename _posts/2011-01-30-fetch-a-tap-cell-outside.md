---
layout: post
title: "接收cell外tap事件"
description: "fetch a tap event out of cell on the table"
category: Snippet
tags: [tableview, uikit]
imagefeature: codeplace.jpg
comments: false
mathjax: null
featured: false
published: true
---

需求：检测并处理TableView上cell外的touch事件

基本思路：<br>
UITableView是UiView的子类，所以继承了 `- (UIView*)hitTest:(CGPoint) withEvent:(UIEvent*)`方法，因此我们可以通过重载TableView的`hitTest:`逻辑来实现修改touch事件执行逻辑，实现如下:
{% highlight objective-c %}
@interface NXTableView : NXTableView {
}
@end

@implementation NXTableView
- (UIView*)hitTest:(CGPoint)point withEvent:(UIEvent*)event {
    // check if a row was hit
    NSIndexPath *indexPath = [self indexPathForRowAtPoint:point];
    if(!indexPath) {
        NSLog(@"outside Cell");
    }
    return [super hitTest:pointwithEvent:event];
}
@end
{% endhighlight %}

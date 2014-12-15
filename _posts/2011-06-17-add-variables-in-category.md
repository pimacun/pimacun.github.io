---
layout: post
title: "利用类别(Category)为对象添加“实例变量”"
description: "Add a instance variable to object with category"
category: Foundation
tags: [category]
imagefeature: codeplace.jpg
comments: false
mathjax: null
featured: false
published: true
---

类别`(Category)`是Objective-C比较常用的一个特性。常常用来扩展一功能类、将不同的功能分属不同的文件、以及为现场的类添加新的方法和功能，甚至有的时候我们也会用以重装某些方法来实现多态或满足其他需求（不提倡，原因暂不予讨论）。但是，我们只能添加方法，却不能通过类别给已经存在的类添加新的成员变量。因此，每有变量增需，一般都是采用继承。<br/>
但是如果确有添加有限个成员变量却不想再产生一个子类，通过类别倒是也能实现，这需要利用ObjectiveC的C API中函数:
{% highlight objective-c %}
void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)
id objc_getAssociatedObject(id object, const void *key)
{% endhighlight %}
详细参见官方的 **[ObjectiveC Runtime Reference](http://developer.apple.com/library/mac/#documentation/Cocoa/Reference/ObjCRuntimeRef/Reference/reference.html)**。<br/>
废话到此结束，现在写怎么做(以给UIView添加一个string成员作为名字为例)：

### i. 在定义类别之前, 首先引入`<objc/runtime.h>`.
{% highlight objective-c %}
#import <objc/runtime.h>
{% endhighlight %}
### ii. 定义&实现类别(以UIView为例)<br/>
定义:
{% highlight objective-c %}
@interface UIView(AddVariables)
@property (nonatomic, retain) NSString *viewName;
@end
{% endhighlight %}
实现:
{% highlight objective-c %}
// 定义存取的Key
static const char *viewNameKey = "viewNameKey";

@implementation UIView(AddVariables)
// get方法
- (NSString *)viewName {
    return (NSString *)objc_getAssociatedObject(self, viewNameKey);
}
// set方法
- (void)setViewName:(NSString *)newViewNameKey {
    objc_setAssociatedObject(self, viewNameKey, newViewNameKey, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
@end
{% endhighlight %}

### iii. 调用测试:

{% highlight objective-c %}
- (void)viewDidLoad {

    UIView *testView = [[UIView alloc] initWithFrame:self.view.bounds];
    [self.view addSubview:testView];
    [testView release];

    [testView setViewName:@"Nori's View"];
    NSLog(@"[testView's newPropery]:%@", testView.viewName);

    [super viewDidLoad];
}
{% endhighlight %}

输出结果如下:
<pre>
testView's newPropery:Nori's View
</pre>

<a href="http://pan.baidu.com/s/1jGIbEHg" target="_blank" title="工程源码">![附件1](/images/download.png)</a>

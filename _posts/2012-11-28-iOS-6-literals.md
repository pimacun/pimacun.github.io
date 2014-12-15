---
layout: post
title: "iOS 6 新语法"
description: "iOS 6 @literals"
category: Language
tags: [language]
imagefeature: codeplace.jpg
comments: false
mathjax: null
featured: false
published: true
---


iOS6放出来有一段时间，添加了许多特性，其中最引人注意的是literals（简写），今天抽空整理了一下，可能不完善，聊做备忘速查只用。

- ### 字符串
{% highlight Objective-C %}
NSNumber *theLetterZ = @'Z';
// equivalent to [NSNumber numberWithChar:'Z'];
{% endhighlight %}

- ### 整型
{% highlight Objective-C %}
NSNumber *fortyTwo = @42;
// equivalent to [NSNumber numberWithInt:42];

NSNumber *fortyTwoUnsigned = @42U;
// equivalent to [NSNumber numberWithUnsignedInt:42U];

NSNumber *fortyTwoLong = @42L;
// equivalent to [NSNumber numberWithLong:42L];

NSNumber *fortyTwoLongLong = @42LL;
// equivalent to [NSNumber numberWithLongLong:42LL];
{% endhighlight %}

- ### 浮点型
{% highlight Objective-C %}
NSNumber *piFloat = @3.141592654F;
// equivalent to [NSNumber numberWithFloat:3.141592654F];

NSNumber *piDouble = @3.1415926535;
// equivalent to [NSNumber numberWithDouble:3.1415926535];
{% endhighlight %}

- ### 布尔型
{% highlight Objective-C %}
NSNumber *yesNumber = @YES;
// equivalent to [NSNumber numberWithBool:YES];

NSNumber *noNumber = @NO;
// equivalent to [NSNumber numberWithBool:NO];
{% endhighlight %}
##### Objective-C++ 也同样支持`@true`和`@false`
{% highlight Objective-C %}
#ifdef __cplusplus
  NSNumber *trueNumber = @true;
  // equivalent to [NSNumber numberWithBool:(BOOL)true];

  NSNumber *falseNumber = @false;
  // equivalent to [NSNumber numberWithBool:(BOOL)false];
#endif
{% endhighlight %}

- ### 运算 
  `@( expression )`


  - 数值运算
    <pre>
  NSNumber *smallestInt = @(-INT_MAX - 1);
  // [NSNumber numberWithInt:(-INT_MAX - 1)];
  NSNumber *piOverTwo = @(M_PI / 2);
  // [NSNumber numberWithDouble:(M_PI / 2)];

  - 枚举
    <pre>
typedef enum { Red, Green, Blue } Color;
NSNumber *favoriteColor = @(Green);
// [NSNumber numberWithInt:((int)Green)];

  - 字符串
    <pre>
NSString *path = @(getenv("PATH"));
// [NSString stringWithUTF8String:(getenv("PATH"))];
NSArray *pathComponents = [path componentsSeparatedByString:@":"];

  - C字符串
    <pre>
NSMutableArray *args = [NSMutableArray new];
NSMutableDictionary *options = [NSMutableDictionary new];
while (--argc) {
      const char *arg = *++argv;
      if (strncmp(arg, "--", 2) == 0) {
          options[@(arg + 2)] = @(*++argv);  // --key value
      } else {
          [args addObject:@(arg)];            // positional argument
      }
}

- ### 容器（集合）类

  - NSDictionary
    <pre>
    NSDictionary *dictionary = @{
    @"name" : NSUserName(),
    @"date" : [NSDate date],
    @"processInfo" : [NSProcessInfo processInfo]
};
NSString *str = dictionary[@"name"];

  - NSArray
    <pre>
NSArray *array = @[ @"Hello", NSApp, [NSNumber numberWithInt:42] ];
NSString *str = array[0];

  - NSMutableArray
    <pre>
NSArray *array = [ @[ @"Ocarina", @"Flute", @"Harp" ] mutableCopy];
NSString *str = array[0];
array[0] = @“change”;

- ### 操作
{% highlight Objective-C %}
NSMutableArray *array = ...;
NSUInteger idx = ...;
id newObject = ...;
id oldObject = array[idx];
array[idx] = newObject;           // 替换对象

NSMutableDictionary *dictionary = ...;
NSString *key = ...;
oldObject = dictionary[key];
dictionary[key] = newObject;      // 替换对象
{% endhighlight %}

### 注意事项：
---
1. 生成的对象都是`autorelease`;
2. 兼容6一下设备需进行`可用性检测`，以防崩溃。

---
layout: post
title: "关于块对象"
description: "About the Blocks"
category: Foundation
tags: [block]
imagefeature: codeplace.jpg
comments: false
mathjax: null
featured: false
published: true
---

关于块对象（block），以前大都用来代替Delegate来完成一些事情。所以，一直没怎么注意。
但是最近发现确实很有用，所以写个Demo，发上来，大家共鉴。

使用方法分好多种，常用的示例大体如下：

- ### 定义block
{% highlight Objective-C %} 
NSComparisonResult (^theBlock)(NSString *value) = ^NSComparisonResult (NSString *value) {
    if([value isEqualToString:@"Nori"]) {
        return NSOrderedSame;
    }
    return NSOrderedAscending;
};
{% endhighlight %}

- ### 定义以block为参数的函数
{% highlight Objective-C %} 
void useCodeBlock(NSComparisonResult (^theBlock)(NSString *value), NSString *value) {
    if(theBlock(value) == NSOrderedSame) {
        NSLog(@"Equal: What a clever boy %@ is.", value);
    }
    else {
        NSLog(@"unEqual: %@ is a Fool.", value);
    }
}
{% endhighlight %}

- ### 定义以block为参数得方法
{% highlight Objective-C %} 
- (NSMutableArray *)filterArray:(NSArray *)inArray withBlock:(NSComparisonResult (^)(NSString *value))block {
    NSMutableArray *result = [NSMutableArray array];
    for(NSString *value in inArray) {
        if(block(value) != NSOrderedSame) {
            [result addObject:value];
        }
    }
    return result;
}
{% endhighlight %}

- ### 测试
{% highlight Objective-C %} 
- (void)viewDidLoad {
    [super viewDidLoad];

    // test 1
    useCodeBlock(theBlock, @"Leo");
    useCodeBlock(theBlock, @"Nori");

    // test 2
    NSArray *array = [[NSArray alloc] initWithObjects:@"Leo", @"Amber", @"Nori", @"Wesly", @"Steven", @"Rain", @"Alice", @"Tracy", @"Nori", nil];
    NSArray *filtedArray = [[self filterArray:array withBlock:theBlock] retain];
    [array release];
    NSLog(@"%@", filtedArray);
    [filtedArray release];

}
{% endhighlight %}
<b>测试结果输出：</b>
{% highlight Objective-C %} 
2011-09-07 11:45:55.534 BlocksDemo[3234:207] unEqual: Leo is a Fool.
2011-09-07 11:45:55.535 BlocksDemo[3234:207] Equal: What a clever boy Nori is.
2011-09-07 11:45:55.535 BlocksDemo[3234:207] (
    Leo,
    Amber,
    Wesly,
    Steven,
    Rain,
    Alice,
    Tracy
)
{% endhighlight %}
此外，稍做说明。**block本质是代码块（对象）的指针，所以需要进行内存管理的，这个demo没有处理**。其管理方式和普通的对象内存管理基本相同。例如，做属性得时候，注意dealloc中得release。

<a href="http://pan.baidu.com/s/1eQ5Y6Wi" target="_blank">![附件1](/images/download.png)</a>

Block做筛选器
---

---

关于Block做筛选器，文档上说是并行的，速度会快。写了个demo，确实速度有所提高。
一下是一个Block做筛选器的Demo，求2-20000间素数。相同得算法，block会快很多。
{% highlight Objective-C %} 
@interface PrimeFinder : NSObject {
    NSInteger maxNumber;
    NSDate *startedDate;
    NSDate *endedDate;
    NSMutableArray *primes;

    NSMutableArray *candidates;
}
@property (retain, nonatomic) NSMutableArray * candidates;

@property (retain, nonatomic) NSMutableArray * primes;
@property (retain, nonatomic) NSDate * startedDate;
@property (retain, nonatomic) NSDate * endedDate;

- (id)initWithMaxNumber:(NSInteger)inMaxNumber;
- (void)startWithNormal;
-(void)startWithBlock;
- (NSTimeInterval)elapsedTime;

@end

@interface PrimeFinder ()

-(BOOL)isPrime:(NSInteger)number;
-(NSMutableArray *)filterArray:(NSArray *)inArray withBlock:(BOOL (^)(id))block;

@end

@implementation PrimeFinder

@synthesize primes, startedDate, endedDate, candidates;

- (void)dealloc {
    self.candidates = nil;
    self.primes = nil;
    self.startedDate = nil;
    self.endedDate = nil;
    [super dealloc];
}

- (id)initWithMaxNumber:(NSInteger)inMaxNumber {
    if ((self = [super init])) {
        maxNumber = inMaxNumber;
        primes = [[NSMutableArray alloc] init];

        candidates = [NSMutableArray new];
        for (int n = 2; n <= inMaxNumber; ++ n) {
            [candidates addObject:[NSNumber numberWithInteger:n]];
        }
    }
    return self;
}
- (void)startWithNormal {
    [self setStartedDate:[NSDate date]];
    for(NSInteger n = 2; n <= maxNumber; ++n) {
        if([self isPrime:n]) {
            [primes addObject:[NSNumber numberWithInteger:n]];
        }
    }
    [self setEndedDate:[NSDate date]];
}

- (NSMutableArray *)filterArray:(NSArray *)inArray withBlock:(BOOL (^)(id))block {
    NSMutableArray *result = [NSMutableArray array]; for(id item in inArray) {
        if(block(item)) [result addObject:item];
    } return result;
}
-(void)startWithBlock {
    [self setStartedDate:[NSDate date]];
    NSMutableArray *result = [NSMutableArray array];
    typedef void (^dispatch_block_t)(void);//此语句是系统的，本不改有，为了方便大家理解，所以写上了。
    dispatch_queue_t globalQueue = dispatch_get_global_queue(0, 0);
    dispatch_group_t group = dispatch_group_create();

    for(NSInteger number = 2; number <= maxNumber; ++number) {
        dispatch_block_t isPrime = ^{
            for (NSInteger n = 2; n < number; ++n) {
                if ((number % n) == 0) {
                    return;
                }
            }
            @synchronized(result) {
                [result addObject:[NSNumber numberWithInt:number]];
            }
        };
        dispatch_group_async(group, globalQueue, isPrime);
    }
    dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
    [self setEndedDate:[NSDate date]];
    [self setPrimes:result];
}

-(BOOL)isPrime:(NSInteger)number {
    for(NSInteger n = 2; n < number; ++n) {
        if((number % n) == 0) {
        return NO;
        }
    }
    return YES;
}

- (NSTimeInterval)elapsedTime {
    return [endedDate timeIntervalSinceDate:startedDate];
}

@end
{% endhighlight %}
测试代码：
{% highlight Objective-C %} 
PrimeFinder *finder = [[PrimeFinder alloc] initWithMaxNumber:200000];
[finder startWithNormal];
NSLog(@"Found all the primes in %fs with Normal", [finder elapsedTime]);
[finder startWithBlock];
NSLog(@"Found all the primes in %fs with Block", [finder elapsedTime]);
[finder release];
{% endhighlight %}
结果如下：
{% highlight Objective-C %} 
Found all the primes in 7.195403s with Normal
Found all the primes in 6.285822s with Block
{% endhighlight %}

<a href="http://pan.baidu.com/s/1jGl7DfK" target="_blank">![附件2](/images/download.png)</a>

注，因为for中各block是`并行`的，所以就要考虑到`互斥锁`的问题。所以，应当将对result得操作放在block内，加锁`@synchronized`（result）｛xxoo｝。

还有，内存管理没做，如果要使用block得时候，请注意这些细节。

---

##（续）今天补上系统一点的使用方法：

- ### 声明和定义分离
a. 声明类型（typedef），那么可以有多种实现<br>
{% highlight Objective-C %}
// 声明类型：
typedef <return type> (^<block name>) (type arg1, ...);

// 完成定义：
<block name> <instance name> = ^<return type> (type arg1, ...) {
    ...
    return ...;
};
// 使用：
<instance name> (a, ...);

// 例：
typedef void (^PrintBlock) (NSObject *obj);
typedef int (^PlusBlocks) (int arg1, int arg2);

- (void)test {
 PrintBlock print = ^(NSObject *obj) {
        NSLog(@"%@", obj);
    };
    PrintBlock printPtr = ^(NSObject *obj) {
        NSLog(@"ptr: %@", obj);
    };
    printPtr (@"Hello Blocks.");
    print(@"Hello Blocks.");
    
    PlusBlocks add = ^int (int a, int b) {
        return a + b;
    };
    int sum = add(10,2);
    NSLog(@"%i", sum);
}
{% endhighlight %}
b. 声明Block（不声名类型），那么只能有一个定义
{% highlight Objective-C %}
// 声明:
<return type> (^<block name>) (type arg1, ...);
// 定义：
<block name> = ^<return type> (type arg1, ...) {
 ...
 return ...;
};
// 使用：
<block name> (a, ...);
// 例：
int (^PlusBlocks) (int arg1, int arg2);
- (void)test {
    PlusBlocks = ^int (int a, int b) {
        return a + b;
    };
    int sum = PlusBlocks(10,2);
    NSLog(@"%i", sum);
}
{% endhighlight %}

- ### 声明并定义
{% highlight Objective-C %}
// 声明并定义:
<return type> (^<block name>) (type arg1, ...) = ^<return type>(type arg1, ...) { do something... return ... };
// 使用：
<block name>(a, ...);

// 例:
- (void)test {
    int (^CusBlocks)(int a, int b) = ^(int a, int b){
        return a + b;
    };
    NSLog(@"%i", CusBlocks(12,12));
}
{% endhighlight %}

- ### 与方法混用（使用方法设置并生成block）
注意内存的使用，方法返回的block要`copy`；
外部使用完毕要`release`。
{% highlight Objective-C %}
例：
- ( int(^)(int) )blockRaisedToPower:(int)y {
   int (^block) (int) = ^ int (int x) {
       return pow(x, y); // Close the value of "y"
   };
   return Block_copy(block); 
   // 注:如果你想在其他地方使用该block，那么在return时记得copy
}
// 测试代码
- (void)test {
   int (^square) (int) = [self blockRaisedToPower:2];// 平方
   int (^cube) (int) = [self blockRaisedToPower:3];// 3次方
   int (^zenzizenzizenzic) (int) = [self blockRaisedToPower:8];// 8次方
 
   NSLog(@"%d", square(3));// 9
   NSLog(@"%d", cube(3)); // 27
   NSLog(@"%d", zenzizenzizenzic(3));// 6561
 
   // 用完记得release掉，copy的block
   Block_release(square);
   Block_release(cube);
   Block_release(zenzizenzizenzic);
}
{% endhighlight %}

- ### 与方法混用（做Delegate）
{% highlight Objective-C %}
- (void)xxx... usingBlock:(<block name>)<arg name>;
{% endhighlight %}
方法执行的时候传入，并保持（直接等于）；
在合适的时候调用，用完记得`release`（例，在dealloc里）。

- ### 注意事项
    - **void可省**；
    - **声明时，形参可省**；
    - **内存管理**：
blcok默认是创建在`栈`中而不是堆中，所以要通过`copy`移动到堆中以保持，用完之后记得`release`。
block会默认保持引用的外部变量（比如self），为了防止循环引用而死锁，引用而不保持的变量应该加`__block`标识，例如：
{% highlight Objective-C %}
// MyClass.h 的成员变量
WhateverObject *anObject;
BOOL aBoolean;
 
// MyClass.m
anObject = [[WhateverObject alloc] initWithBlock: ^ {
   NSLog(@"%d", aBoolean); // Possible retain cycle
}];
{% endhighlight %}
title: iOS开发ARC内存管理
date: 2015-09-19 21:43:52
tags: 
- 内存 
- 基础
---

`ARC`  `内存管理`   

在了解ARC之前，先了解OC内存管理里面的`对象生命周期`、`引用计数`、`对象所有权` ，（自行查阅相关资料）

本文简单介绍iOS开发中ARC(Automatic Reference Counting，`自动引用计数`)内存管理的一些要点，所以不会讲的很全面。详细的关于ARC的信息请参见苹果的！[官方文档](https://developer.apple.com/library/ios/releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html)与网上的其他教程） <!-- more -->

### 本文的主要内容： 
* ARC的本质 
* \__strong, __weak
* ARC与Toll-Free Bridging

### ARC的本质 
先引用官方的关于ARC的一句描述： 
 
Automatic Reference Counting (ARC) is a compiler-level feature that simplifies the process of managing object lifetimes (memory management) in Cocoa applications. 
 
ARC是编译器（时）特性，而不是运行时特性（关于运行时`runtime`是深入学习iOS的重要内容，有兴趣的同学可以自己深入学习） 
 
ARC只是相对于MRC的改进，本质上依然是通过管理对象的引用计数。 
 
如果你启用了ARC，只管像平常那样按需分配并使用对象，编译器会帮你插入retain和release语句。

### \__strong, __weak

**__strong**

表示引用为强引用。对应在定义property时的"strong"。所有对象只有当没有任何一个强引用指向时，才会被释放。

注意：如果在声明引用时不加修饰符，那么引用将默认是强引用。当需要释放强引用指向的对象时，需要将强引用置nil。

**__weak**

表示引用为弱引用。对应在定义property时用的"weak"。弱引用不会影响对象的释放，即只要对象没有任何强引用指向，该对象依然会被释放。不过好在，对象在被释放的同时，指向它的弱引用会自动被置nil，这个技术叫zeroing weak pointer。这样有效得防止无效指针、野指针的产生。__weak一般用在block、delegate关系中防止循环引用或者用来修饰指向由Interface Builder编辑与生成的UI控件。

**注意**

@property (nonatomic,strong) NSString *myString;

NSString __strong *myString;

__strong关键字和strong特性不可同时使用，内存管理的关键字和特性是不能一起使用的，两者相互排斥。<br>

苹果的文档中明确地写道：

You should decorate variables correctly. When using qualifiers in an object variable declaration,

the correct format is:

ClassName * qualifier variableName;

按照这个说明，要定义一个weak型的NSString引用，它的写法应该是：

	NSString * __weak str = @"hehe"; // 正确！

而不应该是：

	__weak NSString *str = @"hehe";  // 不推荐！

我相信很多人都和我一样，从开始用ARC就一直用上面那种错误的写法。

那这里就有疑问了，既然文档说是错误的，为啥编译器不报错呢？文档又解释道：

Other variants are technically incorrect but are “forgiven” by the compiler. To understand the issue, seehttp://cdecl.org/.

好吧，看来是考虑到很多人会用错，所以在编译器这边贴心地帮我们忽略并处理掉了这个错误。

### ARC与Toll-Free Bridging

ARC只对可保留的对象指针有效。例如：

* 代码块指针
* Objective-C指针

例如CoreFoundation指针，CF指针由人工管理，手动的CFRetain和CFRelease来管理，注，CF中没有autorelease。

CocoaFoundation指针与CoreFoundation指针转换，需要考虑的是所指向对象所有权的归属。ARC提供了3个修饰符来管理。

1. __bridge，什么也不做，仅仅是转换。此种情况下：

      *  从Cocoa转换到Core，需要人工CFRetain，否则，Cocoa指针释放后， 传出去的指针则无效。
      
      * 从Core转换到Cocoa，需要人工CFRelease，否则，Cocoa指针释放后，对象引用计数仍为1，不会被销毁。

2. __bridge_retained，转换后自动调用CFRetain，即帮助自动解决上述i的情形。

3. __bridge_transfer，转换后自动调用CFRelease，即帮助自动解决上述ii的情形。
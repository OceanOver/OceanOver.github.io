title: NSString内存相关特性strong、copy
date: 2015-09-20 21:55:29
tags: 
- NSString 
- 内存
---

在声明一个NSString属性时，对于其内存相关特性，通常有两种选择( **基于ARC环境** )：strong与copy。大部分的时候NSString的属性都是copy，那copy与strong的情况下到底有什么区别呢？<!-- more -->

`示例 `

我们定义一个类，并为其声明两个字符串属性，如下所示：

	@property (nonatomic, strong) NSString *strongString;
	@property (nonatomic, copy) NSString *copyedString;

	
上面的代码声明了两个字符串属性，其中一个内存特性是strong，一个是copy。下面我们来看看它们的区别。


首先，我们用一个不可变字符串来为这两个属性赋值，

	- (void)stringTest {
    	NSString *string = [NSString stringWithFormat:@"test"];
    	self.strongString = string;
    	self.copyedString = string;
    	NSLog(@"原字符串:%p %p %@",string,&string,string);
    	NSLog(@"strongString:%p %p %@",_strongString,&_strongString,_strongString);
    	NSLog(@"copyedString:%p %p %@",_copyedString,&_copyedString,_copyedString);
    	//改变string
    	string = @"123";
    	NSLog(@"原字符串:%p %p %@",string,&string,string);
    	NSLog(@"strongString:%p %p %@",_strongString,&_strongString,_strongString);
    	NSLog(@"copyedString:%p %p %@",_copyedString,&_copyedString,_copyedString);
    }
	
其输出结果是：

	原字符串:0x7473657445 0x7fff5fbff7d8 test
	strongString:0x7473657445 0x100206d78 test
	copyedString:0x7473657445 0x100206d80 test
	
	原字符串:0x100005260 0x7fff5fbff7d8 123
	strongString:0x7473657445 0x100206d78 test
	copyedString:0x7473657445 0x100206d80 test						
我们要以看到，这种情况下，不管是strong还是copy属性的对象，其指向的地址都是同一个，即为string指向的地址。但此时改变string值，实际改变了string指向的地址，strongString、copyedString任指向原地址，显示源地址内容“test”。

接下来，我们把string由不可变改为可变对象，看看会是什么结果。即将下面这一句

	- (void)mutableStringTest {
    	NSMutableString *string = [NSMutableString stringWithFormat:@"test"];
    	self.strongString = string;
    	self.copyedString = string;
    	NSLog(@"原字符串:%p %p %@",string,&string,string);
    	NSLog(@"strongString:%p %p %@",_strongString,&_strongString,_strongString);
    	NSLog(@"copyedString:%p %p %@",_copyedString,&_copyedString,_copyedString);
    	//改变string
    	[string appendString:@"123"];
    	NSLog(@"原字符串:%p %p %@",string,&string,string);
    	NSLog(@"strongString:%p %p %@",_strongString,&_strongString,_strongString);
    	NSLog(@"copyedString:%p %p %@",_copyedString,&_copyedString,_copyedString);
	}

其输出结果是：

	原字符串:0x100114cd0 0x7fff5fbff7d8 test
	strongString:0x100114cd0 0x100106ef8 test
	copyedString:0x7473657445 0x100106f00 test
	
	原字符串:0x100114cd0 0x7fff5fbff7d8 test123
	strongString:0x100114cd0 0x100106ef8 test123
	copyedString:0x7473657445 0x100106f00 test

此时，我们如果去修改string字符串的话，因为_strongString与string是指向同一对象，所以_strongString的值也会跟随着改变(`需要注意的是，此时_strongString的类型实际上是NSMutableString，而不是NSString`)；可以看到：而_copyedString是指向另一个对象的，所以并不会改变。


<br>
<font size=4 color=green>结论</font>

而上面的例子可以看出，当源字符串是NSString时，由于字符串是不可变的，所以，不管是strong还是copy属性的对象，都是指向源对象，copy操作只是做了次浅拷贝。

当源字符串是NSMutableString时，strong属性只是增加了源字符串的引用计数，而copy属性则是对源字符串做了次深拷贝，产生一个新的对象，且copy属性对象指向这个新的对象。另外需要注意的是，这个copy属性对象的类型始终是NSString，而不是NSMutableString，因此其是不可变的。

所以，在声明NSString属性时，到底是选择strong还是copy，可以根据实际情况来定。不过，一般我们将对象声明为NSString时，都不希望它改变，所以大多数情况下，我们建议用copy，以免因可变字符串的修改导致的一些非预期问题。

比如声明的一个NSString \*str变量，然后把一个NSMutableString \*mStr变量的赋值给它了，如果要求str跟着mStr变化，那么就用retain;如果str不能跟着mStr一起变化，那就用copy。而对于要把NSString类型的字符串赋值给str，那两都没啥区别。不会影响安全性，内存管理也一样。

关于字符串的内存管理，还有些有意思的东西，可以参考 [NSString特性分析学习](http://blog.cnbluebox.com/blog/2014/04/16/nsstringte-xing-fen-xi-xue-xi/)。

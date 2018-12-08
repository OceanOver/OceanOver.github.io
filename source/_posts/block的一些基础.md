title: block的一些基础
date: 2015-09-29 19:56:56
tags: 
- block 
- 基础
---
在异步编程时我们经常进行函数回调，由于函数调用是异步执行的，我们如果想让一个操作执行完之后执行另一个函数，则无法按照正常代码书写顺序进行编程，因为我们无法获知前一个方法什么时候执行结束，此时我们经常会用到委托或者代码块（Block）。Block就是一个函数体（匿名函数），它是ObjC对于闭包的实现，在块状中我们可以持有或引用局部变量，同时利用Block你可以将一个操作作为一个参数进行传递（是不是想起了C语言中的函数指针）。<!-- more -->


### block定义
1. Block类型定义：<font color=#EF4700>**返回值类型(^ 变量名)(参数列表)**</font>（注意Block也是一种类型）；
2. Block的typedef定义：<font color=#EF4700>**返回值类型(^类型名称)(参数列表)**</font>；
3. Block的实现：<font color=#EF4700>**^(参数列表){操作主体}**</font>；


### block与存取变量
* 可以读取和Block pointer同一個范围的变量值

``` obj-c
    int a = 5;
    int (^myBlock) (int) = ^(int b) {
        return a + b;
    };
    a = 8;
    int result = myBlock(4);
    NSLog(@"%d",result);//结果是9
```
实际上，myBlock在实现部分用到a這個变量值的时候是把a的值copy下來。所以之后a即使改变了对于myBlock里copy的值是沒有影响的

``` obj-c
    NSMutableArray * mutableArray = [NSMutableArray arrayWithObjects:@"one",@"two",@"three",nil];
    void (^myBlock1)(NSMutableArray *) = ^(NSMutableArray *array) {
        [array removeLastObject];
    };
    myBlock1(mutableArray);
    NSLog(@"test array %@", mutableArray);
```
如果這個变量的值是个pointer的話，它指到的值是可以在block里被改变。

* 直接存取 static 的变量

``` obj-c
    static int a2 = 5;
    int (^myBlock2) (int) = ^(int b) {
        return a2 + b;
    };
    a2 = 8;
    int result2 = myBlock2(4);
    NSLog(@"result2:%d",result2);//结果是 12
```

* Block 变量

``` obj-c
    __block int a3 = 5;
    int (^myBlock3) (int) = ^(int b) {
        return a3 + b;
    };
    a3 = 8;
    int result3 = myBlock3(4);
    NSLog(@"result3:%d",result3);//结果是 12
```
在某個变量前面如果加上修饰字 \__block 的話(注意block前有兩個下划线)，這個变量又叫做block variable（变量）。那在block里就可以任意修改此变量值，变量值的变化也可以知道。

`补充：` \__block 和 \__weak修饰符的区别

1. \__block不管是ARC还是MRC模式下都可以使用，可以修饰对象，还可以修饰基本数据类型。
2. \__weak只能在ARC模式下使用，也只能修饰对象（NSString），不能修饰基本数据类型(int)。
3. \__block对象可以在block中被重新赋值，\__weak不可以。

### block与引用循环

retain cycle问题的根源在于Block和obj可能会互相强引用，互相retain对方，这样就导致了retain cycle，最后这个Block和obj就变成了孤岛，谁也释放不了谁。

``` obj-c
typedef void(^CustomBlock)();
@interface ViewController : UIViewController
@property(nonatomic,copy) CustomBlock myBlock;
@end
```

``` obj-c
    _myBlock = ^() {
        NSLog(@"%@",self);//警告self
    };
```
直接循环引用。

ViewController --强引用------>_myBlock(堆block,会retain 内部引用对象变量)

_myBlock------retain--------->self(ViewController)
<br>

``` obj-c
typedef void(^CustomBlock)();
@interface BlockTester : NSObject
@property(nonatomic,assign) NSInteger statusCode;
@property(nonatomic,copy) CustomBlock myBlock;
- (void)useBlock:(CustomBlock)block;
@end
```

``` obj-c
@interface ViewController () {
    BlockTester *_blockTester;
}
@end
```

``` obj-c
    _blockTester.myBlock = ^() {
        NSLog(@"%@",self);//警告self
    };
```
间接循环引用。

ViewController -----retain--> _blockTester

_blockTester -------强引用--->myBlock(堆block,会retain 内部引用对象变量)

myBlock  ---retain--------->self(ViewController)
<br>

``` obj-c
    dispatch_async(dispatch_get_main_queue(), ^{
        NSLog(@"1:%@",self);//没有警告self
    });
```
self 并没有直接或间接的去强引用 gcd block。 可以想象 gcd block 会统一被管理在调度中心来管理，反正不是self。
<br>

``` obj-c
    [_blockTester useBlock:^{
        if (_blockTester.statusCode == 200) {
            NSLog(@"2:%@",self);//没有警告self
        }
    }];
```
栈block 并不会对内部引用对象变量 retain.而是直接引用对象变量地址。

这里 self 和  _blockTester 并没有对 方法参数block 做任何引用操作，block 一直存在于栈上，直到执行完毕被释放掉。
     
当然block 也没有对self  做任何操作，直接引用了地址。

`喜欢block的同学，推荐了解` [BlocksKit](https://github.com/zwaldowski/BlocksKit)    

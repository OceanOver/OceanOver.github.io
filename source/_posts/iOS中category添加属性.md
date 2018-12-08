title: iOS中category添加属性
date: 2015-10-07 16:15:39
tags:
- runtime
- 基础+
---

我们知道，使用Category可以很方便地为现有的类增加方法，但却无法直接增加实例变量。不过从Mac OS X v10.6开始，系统提供了Associative References，这个问题就很容易解决了。这种方法也就是所谓的关联(association)，我们可以在runtime期间动态地添加任意多的属性，并且随时读取。<!--more-->所用到的两个重要runtime API是：

``` obj-c
OBJC_EXPORT void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)
OBJC_EXPORT id objc_getAssociatedObject(id object, const void *key)
__OSX_AVAILABLE_STARTING(__MAC_10_6, __IPHONE_3_1);
```

`例：`我们现在打算利用category对UILabel进行属性补充。一般我们有个原则：`能用category扩展就不用继承，因为随着继承深度的增加，代码的可维护性也会增加很多。`使用category可以这么做：

``` obj-c
#import <UIKit/UIKit.h>
#import <objc/runtime.h>
@interface UILabel (Associate)
@property(nonatomic,copy) NSString *labelId;
@end
```

``` obj-c
@implementation UILabel (Associate)
@dynamic labelId;
static char labelIdKey;
- (void)setLabelId:(NSString *)labelId {
    objc_setAssociatedObject(self, &labelIdKey, labelId, OBJC_ASSOCIATION_COPY_NONATOMIC);
}
- (NSString *)labelId {
    return objc_getAssociatedObject(self, &labelIdKey);
}
@end
```

1. key：我们注意到在函数签名中key的类型const void *，这表示key仅仅是一个地址。关键字是一个void类型的指针。每一个关联的关键字必须是唯一的。通常都是会采用静态变量来作为关键字。
2. policy：这里的policy跟属性声明中的retain、assign、copy是一样的，不再赘述。关联策略表明了相关的对象是通过赋值，保留引用还是复制的方式进行关联的；还有这种关联是原子的还是非原子的。这里的关联策略和声明属性时的很类似。这种关联策略是通过使用预先定义好的常量来表示的。
3. 在implement开始处的@dynamic声明。@dynamic则告诉编译器，编译阶段不用为我生成setter与getter，在runtime我已经自己实现了setter与getter。


---

### 延伸阅读
### 关联对象相关的BlocksKit

BlocksKit是对Cocoa Touch Block编程更进一步的支持，它简化了Block编程，发挥Block的相关优势，让更多UIKit类支持Block式编程。

详细了解blockskit参见[BlocksKit源码分析](http://blog.csdn.net/cshun1990/article/details/45462031)

关联对象相关的BlocksKit是对objc_setAssociatedObject、objc_getAssociatedObject、objc_removeAssociatedObjects这几个原生关联对象函数的封装。主要是封装其其内存管理语义。

部分函数声明如下：

``` obj-c
//@interface NSObject (BKAssociatedObjects)
//以OBJC_ASSOCIATION_RETAIN_NONATOMIC方式绑定关联对象
- (void)bk_associateValue:(id)value withKey:(const void *)key;
//以OBJC_ASSOCIATION_COPY_NONATOMIC方式绑定关联对象
- (void)bk_associateCopyOfValue:(id)value withKey:(const void *)key;
//以OBJC_ASSOCIATION_RETAIN方式绑定关联对象
- (void)bk_atomicallyAssociateValue:(id)value withKey:(const void *)key;
//以OBJC_ASSOCIATION_COPY方式绑定关联对象
- (void)bk_atomicallyAssociateCopyOfValue:(id)value withKey:(const void *)key;
//弱绑定
- (void)bk_weaklyAssociateValue:(__autoreleasing id)value withKey:(const void *)key;
//删除所有绑定的关联对象
- (void)bk_removeAllAssociatedObjects;
```

除了弱绑定以外，其它BlocksKit函数都是简单封装。弱绑定有点扩展关联对象原生语义的感觉。

``` obj-c
@interface _BKWeakAssociatedObject : NSObject
@property (nonatomic, weak) id value;
@end
```

``` obj-c
- (void)bk_weaklyAssociateValue:(__autoreleasing id)value withKey:(const void *)key
{
    _BKWeakAssociatedObject *assoc = objc_getAssociatedObject(self, key);
    if (!assoc) {
        assoc = [_BKWeakAssociatedObject new];
        //做一个_BKWeakAssociatedObject对象中间层
        //以非原子持有的方式绑定一个_BKWeakAssociatedObject对象
        //_BKWeakAssociatedObject该对象又有真实对象的弱引用
        [self bk_associateValue:assoc withKey:key];
    }
    //_BKWeakAssociatedObject的weak property设置为真正应该关联的对象
    assoc.value = value;
}
```
从以上实现代码可以看出，所谓弱绑定实际上是在关联对象之间做了一个中间层，让本对象以OBJC_ASSOCIATION_RETAIN_NONATOMIC的形式去关联中间层（_BKWeakAssociatedObject），而中间层又以weak property的形式去存储真正关联对象的指针。

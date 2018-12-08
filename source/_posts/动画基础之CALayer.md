title: 动画基础之CALayer
date: 2015-09-28 19:33:11
tags: 
- 动画 
- 基础
---
之所以介绍CALayer，是因为CALayer是深入了解iOS动画开发的重要基础内容。CALayer包含在QuartzCore框架中，这是一个跨平台的框架，既可以用在iOS中又可以用在Mac OS X中。在使用Core Animation开发动画的本质就是将CALayer中的内容转化为位图从而供硬件操作，所以要熟练掌握动画操作必须先来熟悉CALayer。<!-- more -->

在UIView中有一个layer属性作为根图层，根图层上可以放其他子图层，在UIView中所有能够看到的内容都包含在layer中。

在iOS中CALayer的设计主要是了为了内容展示和动画操作，CALayer本身并不包含在UIKit中，它不能响应事件。<br>	由于CALayer在设计之初就考虑它的动画操作功能，	CALayer很多属性在修改时都能形成动画效果，这种属性称为“隐式动画属性”。<br> 但是对于UIView的根图层而言属性的修改并不形成动画效果，因为很多情况下根图层更多的充当容器的做用，如果它的属性变动形成动画效果会直接影响子图层。另外，UIView的根图层创建工作完全由iOS负责完成，无法重新创建，但是可以往根图层中添加子图层或移除子图层。

属性	|说明	|是否支持隐式动画
----|-------|:-----------:
anchorPoint|和中心点position重合的一个点，称为“锚点”，锚点的描述是相对于x、y位置比例而言的默认在图像中心点(0.5,0.5)的位置|是
backgroundColor|	图层背景颜色|	是
borderColor	|边框颜色	|是
borderWidth	|边框宽度	|是
bounds	|图层大小	|是
contents|	图层显示内容，例如可以将图片作为图层内容显示	|是
contentsRect|	图层显示内容的大小和位置|是
cornerRadius|	圆角半径|	是
`doubleSided`|图层背面是否显示，默认为YES|	`否`
`frame`|	图层大小和位置，不支持隐式动画，所以CALayer中很少使用frame，通常使用bounds和position代替	|`否`
hidden|	是否隐藏|	是
mask|图层蒙版|	是
maskToBounds|	子图层是否剪切图层边界，默认为NO|	是
opacity|	透明度 ，类似于UIView的alpha	|是
position|	图层中心点位置，类似于UIView的center|	是
shadowColor|	阴影颜色|	是
shadowOffset|	阴影偏移量|	是
shadowOpacity|	阴影透明度，注意默认为0，如果设置阴影必须设置此属性|	是
shadowPath|	阴影的形状|	是
shadowRadius|	阴影模糊半径|	是
sublayers|	子图层|	是
sublayerTransform|	子图层形变|	是
transform|	图层形变|	是

* 隐式属性动画的本质是这些属性的变动默认隐含了CABasicAnimation动画实现。
* 在CALayer中很少使用frame属性，因为frame本身不支持动画效果，通常使用bounds和position代替。
* CALayer中透明度使用opacity表示而不是alpha；中心点使用position表示而不是center。
* anchorPoint属性是图层的锚点，范围在（0~1,0~1）表示在x、y轴的比例，这个点永远可以同position（中心点）重合，当图层position固定后，调整anchorPoint即可达到调整图层显示位置的作用（因为它永远和position重合）

``` obj-c
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    [self drawCustomLayer];
}

// 绘制图层
- (void)drawCustomLayer {
    CGSize size = [UIScreen mainScreen].bounds.size;
    //创建图层
    _layer = [[CALayer alloc] init];
    //设置背景颜色,由于QuartzCore是跨平台框架，无法直接使用UIColor
    _layer.backgroundColor = [UIColor colorWithRed:0.657 green:1.000 blue:0.712 alpha:1.000].CGColor;
    //设置中心点(中心点使用position而不是center)
    _layer.position=CGPointMake(size.width/2, size.height/2);
    _layer.bounds = CGRectMake(0, 0, Width, Width);
    _layer.cornerRadius = Width / 2;
    //设置阴影
    _layer.shadowColor = [UIColor colorWithRed:0.765 green:0.284 blue:0.289 alpha:1.000].CGColor;
    _layer.shadowOffset = CGSizeMake(2, 2);
    _layer.shadowOpacity = 0.8;
    //设置锚点(永远和position重合),CGPointZero equivalent to CGPointMake(0, 0)
    _layer.anchorPoint = CGPointMake(0.5, 0.5);//CGPointZero;
    
    //加到根图层
    [self.view.layer addSublayer:_layer];
}

//点击旋转（放大，缩小，平移... 通过修改position和bounds均可产生动画）
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event {
    static int index = 0;
    if (index == 0) {
        _layer.transform = CATransform3DMakeScale(1, -1, 1);
        index = 1;
    }
    else {
        _layer.transform = CATransform3DMakeScale(1, 1, 1);
        index = 0;
    }
}
``` 
当点击屏幕时会有旋转动画，在点击屏幕的时候修改图层的属性，这些属性的变动默认隐含了CABasicAnimation动画实现。
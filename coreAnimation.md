# iOS Core Animation
## iOS Core Animation Advanced Techniques
### 图层树
* CALayer没有暴露给UIView的特性
	* 阴影，圆角边框，边框颜色
	* 3D变换和位置变换
	* 不规则边界
	* 内容的alpha掩码
	* 多阶段非线性动画
* 图层树是UIKit的基础，表示了你在iOS应用中在屏幕上看到的所有内容

### 寄宿图
* ` layer.content = (__bridge id)image.CGImage `
* CALayer的**contentsScale**属性是支持高清屏机制的一部分。它决定了由layer自动创建图片的尺寸和显示图片内容的缩放。该值默认是1.0，表示采用每个像素一个点。如果是2.0表示每个像素两个点（高清屏），像素时物理单位，点是虚拟单位。对于CGImage，其内部没有缩放的概念。UIImage是有这个概念的。当我们采用*kCAGravityResizeAspect*时，图片会缩放适配layer，因此这个概念不起作用。` layer.contentScale = [UIScreen mainScreen].scale `
* **masksToBounds**用来去掉超出layer的内容部分
* **contentsRect**属性可以用来描述图片在layer区域内显示的子矩形区域。即可以将图层的内容裁剪显示。
* contentsCenter属性是一个用来定义在layer内缩放区域以及边界不变区域的矩形。默认是{0, 0, 1, 1}，即随着layer做拉伸。**通常用于小图拉大图**。
* UIView没有提供默认的**-drawRect:**实现，如果检测到有提供**-drawRect:**，会申请一个视图大小的区域乘以*contentsScale*空间。对于通过contents设置图片或给背景设置一个固定的颜色是不需要一个可定制的寄宿图的，因此也就不需要实现**-drawRect:**。如果没有必要，只会浪费内存和CPU时间。
* 在视图第一次显示在屏幕上时，*drawRect*会被调用，在这个函数中，可以使用Core Graphics在寄宿图上绘制，结果会被缓存起来直到需要被更新（如调用*setNeedDisplay*）
* CALayer有一个**delegate**属性，用来向layer提供内容信息。
* **- (void)displayLayer:(CALayerCALayer *)layer;**当需要重绘时，可以通过这个接口来设置它的*contents*，然后就不会调别的接口了。如果没有实现这个接口会调用**- (void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx;**在调用这个接口前，CALayer会创建一个空的合适大小的图用于在上面绘制
* 对于使用了代理的layer，需要用户手动调用**display**来更新；另外，*masksToBounds*属性默认会enable，因为绘制的上下文限定了绘制区域
* 当使用视图的layer时，没必要实现*-displayLayer:*或*-drawLayer:inContext:*在图层的寄宿图上绘制；只需实现*-drawRect:* ，UIView会处理好这一切，包括在需要的时候调用-display

### 图层几何学
* **frame**表示layer在外部的坐标，即它在父layer中占的空间；bounds表示内部坐标，｛0，0｝表示的是左上角，position表示的是与父layer相关的一个锚点
* 当操作view的*frame*，*bounds*和*center*时，实际是在操作layer的对应的属性
* **frame**是个虚拟属性，它是通过*bounds*，*position*和*transform*计算而成，对这些属性的改变会影响到*frame*，同理，修改*frame*也会影响到这些属性
* **anchorPoint**即锚点，是用来移动layer的句柄。它控制layer的*frame*位置，默认下，锚点位于layer的中心。锚点的作用主要是用在对layer做变换时使用，起到支点的作用，如以这点为支点进行旋转点，缩放点，平移点等
* **position**是layer中*anchorPosition*点在父layer中的位置坐标，因此*position*点是相对父layer的，*anchorPoint*是相对layer的，两者是相对不同的坐标空间的一个重合点。*position*是根据*anchorPoint*来确定的。
* **anchorPoint**的默认值是（0.5，0.5），也是*anchorPoint*默认在layer的中心点。使用*addSublayer*函数添加layer时，如果已知layer的*frame*值，*position*值可以计算如下：

```
position.x = frame.origin.x + 0.5 * bounds.size.width;
position.y = frame.origin.y + 0.5 * bounds.size.height;
```

这个中间的0.5是因为*anchorPoint*取默认值，更通用的公式是：

```
position.x = frame.origin.x + anchorPoint.x * bounds.size.width;
position.y = frame.origin.y + anchorPoint.y * bounds.size.height;
```

如果单方面修改layer的*position*位置不会对*anchorPosition*产生影响，同理，如果修改*anchorPosition*也不会影响*position*；受影响的只是*frame.origin*，即layer相对与父layer的坐标原点

```
frame.origin.x = position.x - anchorPosition.x * bounds.size.width;
frame.origin.y = position.y - chanorPosition.y * bounds.size.height;
```

这就是为什么修改*anchorPoint*会移动layer，因为*position*不受影响，只能是*frame.origin*做相应的改变，因而会移动layer

* 更多关于frame，anchorPoint和postion的参考如下：
> [anchorPoint, frame, positon](http://wonderffee.github.io/blog/2013/10/13/understand-anchorpoint-and-position/)

* CALayer提供了在不同layer之间转换坐标系的接口

```
- (CGPoint)convertPoint:(CGPoint)point fromLayer:(CALayer *)layer; 
- (CGPoint)convertPoint:(CGPoint)point toLayer:(CALayer *)layer; 
- (CGRect)convertRect:(CGRect)rect fromLayer:(CALayer *)layer;
- (CGRect)convertRect:(CGRect)rect toLayer:(CALayer *)layer;
```

* 坐标系翻转，对于iOS而言，设置某个layer的*geometryFlipped*为YES意味着它的父layer的坐标系将会垂直翻转，这个翻转会一直影响到它的所有子layer，直到某个*geometryFlipped*的设为YES
* UIView是二维的，CALayer是三维的。用*zPosition*可以改变layer的显示顺序
* 通过*-containsPoint:* and *-hitTest:*来判断某个点是否在在指定的layer中，**注意在layer的顺序**
* CALayer没有自动布局，对于需要动态调整layer大小的情况，可以通过*CALayerDelegate*的*- (void)layoutSublayersOfLayer:(CALayer *)layer;*接口实现，当layer的*bounds*改变或者*-setNeedsLayout*被调用的时候，这个接口会被调用，让你有机会重新布局或改变所有相关layer的大小

### 可视效果
* 圆角**cornerRadius**，默认情况下，这个圆角只会影响layer的背景色，不会影响laye的图片和子layer，如果*masksToBounds*为YES的话，所有layer里面的内容都会收到影响
* layer的边缘线**borderWidth**和**borderColor**，该边缘线绘制在边界内，该内容层最上层，高于子layer。layer的边缘线不会考虑layer的内容或者子图层
* 向下的阴影，用来表示视图的深度，*shadowOpacity*，*shadowColor*，*shadowOffset*，*shadowRadius*，*shadowRadius*
* layer的阴影考虑的图形的内容
* **mask**属性，定义了他的父图层的可见区域；*mask*的颜色被忽略了，它只关心轮廓
* 缩放过滤，当一幅图片需要显示不同的尺寸时，一个算法（缩放过滤）会应用到图片上，实际像素映射到显示的像素上
* **minificationFilter**表示压缩显示，**magnificationFilter**表示放大显示，默认的算法都是*kCAFilterLinear*，对于没有斜线显示的小图，*kCAFilterNearest*效果最好
* 对于透明度，用*opacity*，这个透明度会影响自己layer和所有子layer，如果想让所有的子图层有一致的显示效果，可以使用**shouldRasterize**，它让layer和子layer在应用opaity前融合，另外需要调整**rasterizationScale**

### 变换
* 仿射变换，是二维变换，仿射的含义是对于平行的边在变换后仍然保持平行
* 创建一个仿射矩阵

```
CGAffineTransformMakeRotation(CGFloat angle)
CGAffineTransformMakeScale(CGFloat sx, CGFloat sy) 
CGAffineTransformMakeTranslation(CGFloat tx, CGFloat ty)
```

例如：

```
//rotate the layer 45 degrees
CGAffineTransform transform = CGAffineTransformMakeRotation(M_PI_4);
self.layerView.layer.affineTransform = transform;
```

* 在一个已有的变换上继续变换

```
CGAffineTransformRotate(CGAffineTransform t, CGFloat angle) 
CGAffineTransformScale(CGAffineTransform t, CGFloat sx, CGFloat sy) 
CGAffineTransformTranslate(CGAffineTransform t, CGFloat tx, CGFloat ty)
```

* 常量矩阵 CGAffineTransformIdentity
* 将两个矩阵连接成一个新的矩阵
`CGAffineTransformConcat(CGAffineTransform t1, CGAffineTransform t2);`
* **对于连续的变换，后面的矩阵受到前面变换矩阵的影响**，如先缩放50%，再旋转30度，再平移200，实际上该layer会跑到它的右下角去。因为200的平移是先缩小50%，再旋转了30度的结果
* 斜切变换

```
CGAffineTransform CGAffineTransformMakeShear(CGFloat x, CGFloat y) {
	CGAffineTransform transform = CGAffineTransformIdentity; transform.c = -x;
	transform.b = y;
	return transform;
}
- (void)viewDidLoad {
	[super viewDidLoad];
	//shear the layer at a 45-degree angle
	self.layerView.layer.affineTransform = CGAffineTransformMakeShear(1, 0); 
}
```

#### 3D变换
* layer的**transform**属性是**CATransform3D**类型
* 创建3D变换矩阵

```
CATransform3DMakeRotation(CGFloat angle, CGFloat x, CGFloat y, CGFloat z) 
CATransform3DMakeScale(CGFloat sx, CGFloat sy, CGFloat sz) 
CATransform3DMakeTranslation(Gloat tx, CGFloat ty, CGFloat tz)
```

*旋转满足左手定理
![1.png](resource/coreAnimation/1.png)
* 投射视角，为了让我们感觉到远的东西比进的东西小一些，我们需要调整视角变换，即z变换，调整的参数如图m34
![2.png](resource/coreAnimation/2.png) 
默认条件下，m34为0，我们可以设置该值为-1.0/d，这个d表示的是想象的照相机到屏幕的距离，通常可以设置再500到1000之间
* **sublayerTransform**，对于所有的子图层，可以通过设置这个值来统一所有子layer的视角
* **doubleSided**用来控制是否显示双面

### 特殊的图层
#### CAShapeLayer
* 使用矢量图形而不是位图，通过颜色和线宽属性来定义一个CGPath，CAShapeLayer会自动渲染它
* 特点
	* 快速，使用硬件加速来绘制
	* 内存空间省
	* 可以再边界外面绘制
	* 没有像素化
* 通过它可以实现可定制圆角
//define path parameters
CGRect rect = CGRectMake(50, 50, 100, 100); 
CGSize radii = CGSizeMake(20, 20); 
UIRectCorner corners = UIRectCornerTopRight |
		    UIRectCornerBottomRight | 
		     UIRectCornerBottomLeft;
//create path
UIBezierPath *path = [UIBezierPath bezierPathWithRoundedRect:rect byRoundingCorners:corners cornerRadii:radii];

#### CATextLayer
* CATextLayer渲染比UILabel快
* rich text

#### CATransformLayer

#### CGGradientLayer
* 用于生产平滑过度的颜色渐变

#### CAReplicatorLayer
* 用于高效生产一组相似的layer

#### CAScrollLayer

#### CATiledLayer
* 逐块显示

#### CAEmitterLayer
* 显示离子或火焰等效果

#### AVPlayerLayer

### 瘾式动画
* 所有屏幕上的元素都可能有动画，如果你不想要动画，需要显式关掉动画
* 动画的时间有当前的transaction设置，动画类型有layer的actions控制
* transaction是一种机制，核心动画用它来封装一组属性动画，任何可以动画的属性值改变时，一个给定的transaction不会立刻改变，而是只有当这个transaction被提交时才开始动画到新的值
* transaction由CATransaction类管理，该类管理一个堆栈的transactions，不能直接创建transaction，只能通过begin压栈一个transaction和commit出栈一个transaction
* 核心动画自动在每个run loop中开始一个新的transaction，任何动画属性值改变时，都会分组到这个transaction中，并且会动画0.25s
* 当属性变化时，CALayer自动应用的动画叫做actions，当一个layer的属性被修改，它会调用自己的-actionForKey:方法，这个方法在返回action时的搜索路径为：
	* 首先检查他的delegate是否视线了-actionForLayer:forKey方法，如果实现，则调用后返回；
	* 如果没有代理或没有实现-actionForLayer:forKey, layer会检查actions字典，这个字典中包含了一个属性名映射的actions
	* 如果actions没有，则继续搜索style字典
	* 如果style还是没有，会调用-defaultActionForKey:方法，在这个里面定义了标准的属性actions
* 如果-actionForKey:返回nil，就没有动画，否则会通过这个action进行动画
* 对于直接修改UIView对应的layer没有动画的原因就是UIView实现了-actionForLayer:forKey方法，当不在动画block中时返回nil，否则返回非nil
* UIView的layer主动禁止了瘾式动画，要在这上面动画，需要通过UIVIew动画方法或者子类UIVIew后然后改写-actionForLayer:forKey:来创建一个瘾式动画
* 对于其它layer，可以通过实现-actionForLayer:forKey:或这提供actions字典
* 当设置layer的属性时，你只是定义了一个model，属性改变时，model立刻改变，但是它的view是一个动画，不会立刻改变，而是有一个过程，即核心动画代表了controller，控制动画何时开始和结束
* 在iOS中，屏幕每秒60次重绘。如果动画时长超过1/60s，则需要有个地方来存储中间过程的layer，这个中间过程的layer是presention layer
* 对于presention lay需要在两种情况下处理：
	* 实现基于时间的动画，用它来定位layer在屏幕上的位置，便于在动画中与其它元素对齐
	* 动画中的layer响应用户输入

### 显式动画
* 显式动画可以让你针对某个属性自定义动画或创建非线性动画
* 第一种显式动画为属性动画，有两种形式，基本的和关键帧
* 一个动画就是一个改变持续了一段时间，最简单的改变就是由一个值变为另一个值，这就是CABasicAnimation的涉及初衷
* CABasicAnimation扩展了CAPropertyAnimation，`id fromValue, id toValue, id byValue`, 这三个值只需指定两个即可

|Type Object Type | Code Example|
|:---------------:|-------------|
|CGFloat NSNumber | id obj = @(float);|
|CGPoint NSValue  |id obj = [NSValue valueWithCGPoint:point); |
|CGSize NSValue   |id obj = [NSValue valueWithCGSize:size);|
|CGRect NSValue   |id obj = [NSValue valueWithCGRect:rect);|
|CATransform3D NSValue |id obj = [NSValue valueWithCATransform3D:transform);|
|CGImageRef id    |id obj = (__bridge id)imageRef;|
|CGColorRef id    |id obj = (__bridge id)colorRef;|

* 动画不会修改layer的model，只是修改它的presentation

```
- (IBAction)changeColor {
	CGFloat red = arc4random() / (CGFloat)INT_MAX; 
	CGFloat green = arc4random() / (CGFloat)INT_MAX; 
	CGFloat blue = arc4random() / (CGFloat)INT_MAX; 
	UIColor *color = [UIColor colorWithRed:red green:green blue:blue alpha:1.0];
	//create a basic animation
	CABasicAnimation *animation = [CABasicAnimation animation]; 
	animation.keyPath = @"backgroundColor";
	animation.toValue = (__bridge id)color.CGColor;
	//apply animation to layer
	[self.colorLayer addAnimation:animation forKey:nil];
}
```

* 如果将我们的动画做成一个layer的action，那么它可以通过改变属性来触发动画，这是最简单的一种做法。并且可以保持属性值和动画状态一致。如果我们不能做到这点的画，我们需要及时更新属性值，通常有两种情况，一是动画前，一是动画后

```
CALayer *layer = self.colorLayer.presentationLayer ?: self.colorLayer; 
animation.fromValue = (__bridge id)layer.backgroundColor; 
[CATransaction begin];
[CATransaction setDisableActions:YES]; 
self.colorLayer.backgroundColor = color.CGColor;
[CATransaction commit];
```

* 一条较好的可扩展的添加一个动画的方法

```
- (void)applyBasicAnimation:(CABasicAnimation *)animation toLayer:(CALayer *)layer
￼{	
	//set the from value (using presentation layer if available)
	animation.fromValue = [layer.presentationLayer ?: layer valueForKeyPath:animation.keyPath];
	//update the property in advance
	//note: this approach will only work if toValue != nil [CATransaction begin];
	[CATransaction setDisableActions:YES];
	[layer setValue:animation.toValue forKeyPath:animation.keyPath]; 
	[CATransaction commit];
	//apply animation to layer
	[layer addAnimation:animation forKey:nil]; 
}
```

在显式动画中，如何判断一个动画是否已经做完，可以通过*CAAnimationDelegate*的`-animationDidStop:finished:`来做到。并且如果我们给输入参数`CABasicAnimation`添加key值的话，我们将可以通过该值来访问我们的值。

* 关键帧动画
有*CAKeyframeAnimation*定义，属性动画的一种，它不像*CABasicAnimation*那样只限定了开始和结束，而是可以给出任意多个序列。在关键帧动画中，主动画只负责绘制一些关键的帧，一些不是很关键的帧由不是高水平的画家来绘制。同理，程序员只提供一些关键帧，核心动画通过插值来补充其间的空隙
* **CAKeyframeAnimation**有一个属性**rotationMode**，设置它为*kCAAnimationRotateAuto*，动画的方向就会沿着切线运动
* **transform.ratation**不能直接设置这个属性，因为transform是一个结构体，没有kvc，它是虚拟属性，专门用于动画的。当你通过这些虚拟属性来动画时，核心动画会更新transfrom属性的实际值，通过一个类*CAValueFunction*。
* **CAValueFunction**用于将一个浮点值转成*CATransform3D*中的实际值。可以通过改变*CAPropertyAnimation*中的*valueFunction*来改变默认值。

* 动画组
* 将动画拼成一组，CAAnimationGroup，通过给它的属性animations赋值实现

* 过度动画
* 针对的是布局变化，用属性动画来表示比较困难的情况，比如交换文本或图片，属性动画只能用于图层动画的属性，如果要改变不可动画的属性或增加或删掉layer，属性动画就不能用了
* 过渡动画不是为了平滑变化，而是一种用来分心的战术。过渡动画影响整个layer而不是某个属性。过渡动画先对就的layer做一个快照，然后动画到新的外观
* 过渡动画CATransition有type和subtype用来表示过渡效果

* 瘾式过渡动画，由于过渡动画可以平湖的动画在layer上，apple将layer的contents改变设置为过渡动画，但是跟view上的layer一样，不起作用，只有在子layer上才能有效果

* 自定义过渡动画

```
- (IBAction)performTransition {
	//preserve the current view snapshot
	UIGraphicsBeginImageContextWithOptions(self.view.bounds.size, YES, 0.0); 
	[self.view.layer renderInContext:UIGraphicsGetCurrentContext()]; 
	UIImage *coverImage = UIGraphicsGetImageFromCurrentImageContext();
	//insert snapshot view in front of this one
	UIView *coverView = [[UIImageView alloc] initWithImage:coverImage]; 
	coverView.frame = self.view.bounds;
	[self.view addSubview:coverView];
	//update the view (we'll simply randomize the layer background color)
	CGFloat red = arc4random() / (CGFloat)INT_MAX; 
	CGFloat green = arc4random() / (CGFloat)INT_MAX; 
	CGFloat blue = arc4random() / (CGFloat)INT_MAX; 
	self.view.backgroundColor = [UIColor colorWithRed:red green:green blue:blue alpha:1.0];
	//perform animation (anything you like)
	[UIView animateWithDuration:1.0 animations:^{
		//scale, rotate and fade the view
		CGAffineTransform transform = CGAffineTransformMakeScale(0.01, 0.01);
		￼
		transform = CGAffineTransformRotate(transform, M_PI_2); 
		coverView.transform = transform;
		coverView.alpha = 0.0;
	} completion:^(BOOL finished){
		//remove the cover view now we're finished with it
		[coverView removeFromSuperview]; 
	}];
}
```

在动画过程中取消动画，只要动画被移除，图层的显示马上会更新为当前model的值。当动画被做完后，如果**removedOnCompletion**没有被设为NO，动画会自动被移除

### 图层时间
* CAMediaTiming定义了一组属性用来控制动画中国年流过的时间，CALayer和CAAnimation都实现了这个协议，因此时间可以在每层和每个动画上被控制
* duration用来描述动画中一遍所有的时间，repeatCount用来描述动画执行多少遍，repeatDuration重复多长时间，autoreverses是否反转
* beginTime描述在动画开始前延迟多长时间，speed时间加倍因子，默认是1，对于speed为2，duration时间为1的动画，实际执行的时间为0.5s
* timeOffset用来快速定位到动画的偏移位置
* fillMode用来表示开始或执行完成动画后的填充模式
* 动画的时间层次于CAAnimationGroup关联在一起
* layer或CAGroupAnimation的duration，repeatCount/repeatDuration不会影响到子动画，beginTime，timeOffset，speed会影响到子动画
* global time 
* CFTimeInterval time = CACurrentMediaTime();

* 用来在不同的layer之间同步动画，特别是speed，tiemoffset和beginTime不同时

```
- (CFTimeInterval)convertTime:(CFTimeInterval)t fromLayer:(CALayer *)l; 
- (CFTimeInterval)convertTime:(CFTimeInterval)t toLayer:(CALayer *)l;
```

* 通过给layer的speed属性设置为0，调节timeOffset可以回放任意动画序列。

### 缓冲
* 系统用缓冲来平滑和自然动画
* 动画都是以一定的速率来进行的
* velocity ＝ change ／ time
* 对于速度是常量的动画通常叫做线性步调
* CAAnimation有一个属性timingFunction，用于设置缓冲函数，即速度
* 在关键帧动画中可以通过animation.timingFunctions给不同的关键帧之间赋予不同的速度
* CAMediaTimingFunction将输入的时间转成丛开始到结束的成比例变化。

### 基于时间的动画
* 帧时间
* 动画看上去是在连续移动，但是当显示的像素位置固定时时不可能的。通常的显示也不能显示连续的移动。只能显示一系列连续的静态图像，只是足够块，让你感觉它在动
* iOS每秒刷新60次，CAAnimation要做的就是计算一个新的显示帧，然后在屏幕刷新时同步绘制出来。CAAnimation做的巧妙的地方在于它加入了插值和通过缓冲计算如何显示
* 我们也可以通过自己计算显示帧的顺序列。

* CADisplayLink也是一个类似NSTimer的类，它总是在屏幕被刷新前被触发。如果漏过了调度某帧，将会忽略它，并进入下一个调度时间

### 性能调优
* 让核心动画获得最佳性能是一门艺术，其关注的是让GPU和CPU能达到最佳的负载
* 在绘制和动画中需要用到两种处理器：CPU和GPU
* 通常，我们可以在CPU中做任何事情，但是对于GPU，由于它对图像处理中用到得并行浮点运算做了优化，使得它在图形图像方面更快。因此我们希望能将更多的屏幕渲染工作交给GPU来完成。
* 核心动画不仅仅只是在应用内部的事情，动画和屏幕上显示的layer实际上是由一个独立的进程来处理，而不是我们的应用程序，是render service，在iOS5之前是由SpringBoard进程负责，在iOS6之后由一个新的进程叫BackBoard进程负责
* 在执行一个动画时，可以分为4个独立的阶段：
	* **layout**，该阶段准备你的*view／layer*的层次，设置*layer*的属性（如*frame*，*backgroundcolor*，*border*等等）
	* **display**，layer上的图片会被绘制出来，可能会包括调用**-drawRect:**或**-drawlayer:inContext:**方法
	* **prepare**, 该阶段核心动画会给**render server**发送动画数据。同时核心动画会执行一些任务，如解压图片，这些会在后面的动画中用到
	* **commit**，这是最后的阶段，核心动画将*layer*和动画属性打包发给IPC来渲染显示
* 上面的4步是发生在应用程序内部，要显示在屏幕上，还需要一些额外的工作要做。一旦这些被打包好的*layer*和动画到了**render server**进程后，他们会被解序列化成一个渲染树，**render server**就是使用这个树来做动画的每一帧：
	* 计算的*layer*属性的中间值，建立*OpenGL*纹理切片来执行渲染
	* 渲染屏幕上可视的三角形
* 前5步由CPU执行，最后一步由GPU执行，我们能控制的只有前两步，剩下的由核心动画的框架接管，我们无法控制
* GPU被优化来做下面的事情，获取图片喝物理尺寸，执行图形变换，应用纹理和混合，最后将他们显示到屏幕上；如果我们没有绕过核心动画层，在OpenGL渲染，我们只能采用一组固定的硬件加速操作集合，剩下的事情都得由CPU来完成
* 有些事情会影响GPU的*layer*绘制：
	* 太多的几何角，图形芯片可以处理上百万的三角形，GPU的处理能力不是瓶颈，但是在显示之前*layer*必须被预处理并发送给*render server*，太多的*layer*将会导致CPU成为瓶颈。这将会限制一次显示不了过多的*layer*
	* 太多的重绘，这个主要是由太多的半透明图层重叠造成。GPU只有有限的填充率（指GPU用颜色填充像素的速率），因此重绘（在每帧上填充同一像素多次）是必须避免的
	* 离屏绘制，发生在特殊的效果不能通过直接在屏幕上绘制达到，而必须首先在离屏的图形环境中绘制。离屏绘制要么采用CPU要么采用GPU绘制，但它们都需要申请额外的用于离屏图形的内存空间和切换绘制上下文，会导致GPU的性能下降。对lyer使用某些特效，如圆角，遮罩，阴影或像素化都会使得核心动画去做离屏渲染。需要注意这些地方可能会造成性能问题
＊ 大部分的CPU工作都发生在动画发生前，因此这些工作不会影响到帧率，但它会延迟动画的开始，让你的界面感觉不能及时响应，这些行为由：
	* layout计算，在iOS6的自动布局机制中用到复杂的视图层次
	* lazy视图加载，如果在视图加载之前需要做大量的工作，会导致界面没有响应
	* 核心图形绘制，实现**－drawRect:**或**－drawLayer:inContext:**会引入重要的性能问题
	* 图形解压，*PNG*和*JPEG*都是经过压缩的位图，在这张图片被绘制前都需要解压，（长＊宽＊高＊4），为了节约内存通常会直到绘制前才解压图片
* 访问硬件如闪存或网络接口，某些数据可能会需要加载到*flash*中，如从*nib*文件中*load*它的内容。*flash*比内存慢的多
* 对于影响动画的因素，我们该如何避免它们？首先是先通过工具测量而不是猜测
* 在真机上测试而不是模拟器上	
* 使用**release**配置，而不是*debug*配置
* 在能支持的低端设备上测试
* 为了保持平滑的动画，需要运行在60FPS。对于采用**NSTimer**或**CADisplayLink**的动画，可以保持在30FPS上。对于掉帧的情况，会给用户体验差的感觉

* Instrument
	* **Time Profiler**，用于测量每个函数占用CPU时间
	* 核心动画，用于测量所有种类的核心动画性能问题
		* **Color blended Layers**，高亮屏幕中使用蒙层的地方（使用半透明layer），阴影的地方，蒙层会导致GPU性能和滚动的动画
		* **Color Hits Green and Misses Red**，当使用shouldRasterize，耗时的layer绘制会被cached并被渲染成一幅单独的图片。该选项在这些cache被重现生成时会标成红色，如果会持续变红，说明可能会由性能问题
		* **Color Copied Images**，有时核心动画会强迫copy主干图给render server，而不是将原图的指针发送过去，这种情况下会将这些图标成蓝色，拷贝图会耗费大量内存和CPU
		* **Color Immediately**，通常Instrument更新layer是每10微妙一次，该选项让每帧更新一次
		* **Color Misaligned Images**，高亮缩放显示的图片，或没有正确的边界对齐图片
		* **Color Offscreen-Rendered Yellow**
		* **Color OpenGL Fast Path Blue**，对于直接采用OpenGL的用蓝色高亮
		* **Flash Updated Regions**，用黄色标志重绘的区域
	* **OpenGL ES Driver**，用来测量GPU性能问题，通常用来测量OpenGL代码，但也可以用于核心动画
		* **Renderer Utilization**，如果这个值高于50%，可能动画在填充率上受限了，可能上离屏渲染或overdraw造成
		* **Tiler Utilization**，如果这个值高于50%，表明动画的几何受限，可能是由太多的图层在屏幕上

### 高效绘制
* drawing在核心动画中指利用CPU绘制，主要由Core Graphics框架完成，相对于核心动画和OpenGL要慢很多，另外也需要跟多的内存
* 对于使用向量图形，我们可以用C**AShapeLayer**来绘制，而不是用cpu的**－drawRect**绘制
* 对于需要采用cpu绘制的，可以限制重绘区域来减少额外的绘制，**-setNeedsDisplayInRect:**
* CATiledLayer支持针对每个独立的tile自动更新，并且对**-drawLayer:inContext:** 可以做到并发到多个线程
* CALayer上的**drawsAsynchronously**可以在让当前的任务行，绘制的任务会排队

### 图像IO
* 一个图片文件被加载的速度不仅受限于CPU，也受限于IO延迟。flash山村比普通的硬盘要快，但是比起RAM，还是慢了200多倍
* 对于按下按钮到看到屏幕上有反应，用户能够忍耐的最佳延迟是**200ms**，渲染一帧的时间是**16ms**
* 看门狗会在你的app**20s**还没有启动完成的情况下将你的app杀掉
* 如果图片很大，对于需要滑动的界面，可能在主线程上读取图片会卡住主线程，改进的方法是将它放在其它线程
* 一旦一个图片文件被加载，它需要被解压，这个过程也是非常耗费时间的，被解压后的文件也可能会耗费更多的内存，对于**PNG**格式，加载时间要长于**JPEG**格式，但是解压要比**JPEG**快，Xcode也推荐在工程中用PNG格式
* 解决延迟解压的方案
	* 最简单的避免延迟解压的方式是使用**UIImage +imageNamed:**方法。这个方法在加载后直接解压，不过它只能工作在图片放在资源bundle中的情况
	* 另一个方法是给layer的**content**赋值，或者给UIImageView的**image**属性赋值。不过这些方法都需要在主线程上完成
	* 第三个方法是使用ImageIO框架

```
	NSInteger index = indexPath.row;
	NSURL *imageURL = [NSURL fileURLWithPath:self.imagePaths[index]]; 
	NSDictionary *options = @{(__bridge id)kCGImageSourceShouldCache: @YES}; 
	CGImageSourceRef source = CGImageSourceCreateWithURL((__bridge CFURLRef)imageURL, NULL);
	CGImageRef imageRef = CGImageSourceCreateImageAtIndex(source, 0, (__bridge CFDictionaryRef)options);
	UIImage *image = [UIImage imageWithCGImage:imageRef]; CGImageRelease(imageRef);
	CFRelease(source);
```

它可以让你通过kCGImageSourceShouldCache，它会立即解压
	* 第四种方法，使用UIKit，然后将它立即绘制在CGContext中，采用这种方式的好处是可以将它放在其它线程上去，不会阻塞主线程
* 如果不能用**imageNamed：**次优方法应该是**CGContext**

```
	dispatch_async( dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_LOW, 0), ^{
		//load image
		NSInteger index = indexPath.row;
		NSString *imagePath = self.imagePaths[index];
		UIImage *image = [UIImage imageWithContentsOfFile:imagePath];
		//redraw image using device context
		UIGraphicsBeginImageContextWithOptions(imageView.bounds.size, YES, 0); [image drawInRect:imageView.bounds];
		image = UIGraphicsGetImageFromCurrentImageContext(); UIGraphicsEndImageContext();
		//set image on main thread, but only if index still matches up
		dispatch_async(dispatch_get_main_queue(), ^{
			if (index == imageView.tag) {
				imageView.image = image; 
			}
		}); 
	});
```

* cache
* **imageNamed:**可以用于缓存后以后在用
* NSCache

```
- (UIImage *)loadImageAtIndex:(NSUInteger)index {
	//set up cache
	static NSCache *cache = nil; 
	if (!cache) {
		cache = [[NSCache alloc] init]; 
	}
	//if already cached, return immediately
	UIImage *image = [cache objectForKey:@(index)]; 
	if (image) {
		return [image isKindOfClass:[NSNull class]]? nil: image; 
	}
	//set placeholder to avoid reloading image multiple times
	[cache setObject:[NSNull null] forKey:@(index)];
	//switch to background thread
	dispatch_async( dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_LOW, 0), ^{
		//load image
		NSString *imagePath = self.imagePaths[index];
		UIImage *image = [UIImage imageWithContentsOfFile:imagePath];
		//redraw image using device context
		UIGraphicsBeginImageContextWithOptions(image.size, YES, 0); 
		[image drawAtPoint:CGPointZero];
		image = UIGraphicsGetImageFromCurrentImageContext(); 
		UIGraphicsEndImageContext();
		//set image for correct image view
		dispatch_async(dispatch_get_main_queue(), ^{ //cache the image
			[cache setObject:image forKey:@(index)];
			//display the image
			NSIndexPath *indexPath = [NSIndexPath indexPathForItem:
			index inSection:0]; 
			UICollectionViewCell *cell = [self.collectionView cellForItemAtIndexPath:indexPath]; 
			UIImageView *imageView = [cell.contentView.subviews lastObject]; 
			imageView.image = image;
		}); 
	});
	//not loaded yet
	return nil; 
}
```

### layer性能
* 非显示重绘
	* CATextLayer和UILabel都是将文本绘制在layer的backing image上，它们的绘制都是CPU重绘，建议不要经常改变他们的尺寸
	* shouldRasterize属性是为了避免在非常复杂的子layer上重绘耗费大量的时间，而将这些子layer和自己做成一幅离屏图片，如果这些内容会经常变化，就失去了它的意义
	* 圆角layer，mask，shadow都会造成离屏渲染


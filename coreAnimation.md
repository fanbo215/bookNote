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
	> [anchorPoint, frame, positon](http://wonderffee.github.io/blog/2013/10/13/understand-anchorpoint-and-position/)

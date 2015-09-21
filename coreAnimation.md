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
` layer.content = (__bridge id)image.CGImage `
* CALayer的contentsScale属性是支持高清屏机制的一部分。它决定了由layer自动创建图片的尺寸和显示图片内容的缩放。该值默认是1.0，表示采用每个像素一个点。如果是2.0表示每个像素两个点（高清屏），像素时物理单位，点是虚拟单位。对于CGImage，其内部没有缩放的概念。UIImage是有这个概念的。当我们采用kCAGravityResizeAspect时，图片会缩放适配layer，因此这个概念不起作用。
` layer.contentScale = [UIScreen mainScreen].scale `
* *masksToBounds*用来去掉超出layer的内容部分
* *contentsRect*属性可以用来描述图片在layer区域内显示的子矩形区域。即可以将图层的内容裁剪显示。
* contentsCenter属性是一个用来定义在layer内缩放区域以及边界不变区域的矩形。默认是{0, 0, 1, 1}，即随着layer做拉伸。**通常用于小图拉大图**。
* UIView没有提供默认的**-drawRect:**实现，如果检测到有提供**-drawRect:**，会申请一个视图大小的区域乘以contentsScale空间。对于通过contents设置图片或给背景设置一个固定的颜色是不需要一个可定制的寄宿图的，因此也就不需要实现**-drawRect:**。如果没有必要，只会浪费内存和CPU时间。
* 在视图第一次显示在屏幕上时，drawRect会被调用，在这个函数中，可以使用Core Graphics在寄宿图上绘制，结果会被缓存起来直到需要被更新（如调用setNeedDisplay）
* CALayer有一个delegate属性，用来向layer提供内容信息。
* - (void)displayLayer:(CALayerCALayer *)layer;当需要重绘时，可以通过这个接口来设置它的contents，然后就不会调别的接口了。如果没有实现这个接口会调用- (void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx;在调用这个接口前，CALayer会创建一个空的合适大小的图用于在上面绘制
* 对于使用了代理的layer，需要用户手动调用display来更新；另外，masksToBounds属性默认会enable，因为绘制的上下文限定了绘制区域
* 当使用视图的layer时，没必要实现-displayLayer:或-drawLayer:inContext:在图层的寄宿图上绘制；只需实现-drawRect: ，UIView会处理好这一切，包括在需要的时候调用-display

# iOS Core Animation Advanced Techniques
* CALayer的contentsScale属性是支持高清屏机制的一部分。它决定了由layer自动创建图片的尺寸和显示图片内容的缩放。该值默认是1.0，表示采用每个像素一个点。如果是2.0表示每个像素两个点（高清屏），像素时物理单位，点是虚拟单位。对于CGImage，其内部没有缩放的概念。UIImage是有这个概念的。当我们采用kCAGravityResizeAspect时，图片会缩放适配layer，因此这个概念不起作用。
* contentsRect属性可以用来描述图片在layer区域内显示的子矩形区域。即可以将图层的内容裁剪显示。
* contentsCenter属性是一个用来定义在layer内缩放区域以及边界不变区域的矩形。默认是{0, 0, 1, 1}，即随着layer做拉伸。**通常用于小图拉大图**。
* UIView没有提供默认的**-drawRect:**实现，如果检测到有提供**-drawRect:**，会申请一个视图大小的区域乘以contentsScale空间。如果不需要

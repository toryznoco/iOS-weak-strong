# iOS-weak-strong
为什么 iOS 开发中，控件一般为 weak 而不是 strong？

首先有一点，在OC中，如果对象没有强引用，就会被自动释放，那么为什么控件还可以设为weak?

1.从storyboard或者xib上创建控件，在控件放在view上的时候，已经形成了如下的引用关系,以UIButton为例：

UIViewController->UIView->subView->UIButton然后你为这个UIButton声明一个weak属性

```Objective-C
@property(nonatomic, weak) IBOutlet UIButton *btn;
```
相当于xib/sb对这个Button是强引用，你生命的属性对它是弱引用。

2.手动创建控件

a). 将控件声明成strong

```Objective-C
@property(nonatomic,strong) UIButton *btn;
```

那么你在实现这个控件时只需这样：

```Objective-C
_btn = [[UIButton alloc]init];
[self.view addSubview:_btn]
```

b). 将控件声明成weak

```Objective-C
@property(nonatomic,weak) UIButton *btn;
```

那么你在实现这个控件时需要这样：

```Objective-C
UIButton *button = [[UIButton alloc]init];
_btn = button;
[self.view addSubview:_btn];
```

============================
最近看的黑马iOS视频上给的建议的是：

1.如果用Stroyboard拖线，用weak

2.如果自定对象，用strong(但我还是习惯用weak暂时=_=)

 ![image](https://github.com/toryznoco/iOS-weak-strong/blob/master/images-folder/relationship.png)

其实不管声明的属性是强引用还是弱引用，在控制器消失的时候，这个属性消失，View消失，subViews消失，控件也就消失了。

=============================
之前专门搜过相关的问题，贴上来：

IBOutlet的属性一般可以设为weak是因为它已经被view引用了，除非view被释放，否则IBOutlet的属性也不会被释放，另外IBOutlet属性的生命周期和view应该是一致的，所以IBOutlet属性一般设为weak。

可参考如下:

From a practical perspective, in iOS and OS X outlets should be defined as declared properties. Outlets should generally be weak, except for those from File’s Owner to top-level objects in a nib file (or, in iOS, a storyboard scene) which should be strong. 

·Outlets that you create will therefore typically be weak by default, because:Outlets that you create to, for example, subviews of a view controller’s view or a window controller’s window, are arbitrary references between objects that do not imply ownership.

·The strong outlets are frequently specified by framework classes (for example, UIViewController’s view outlet, or NSWindowController’s window outlet).

简单的说，如果IBOutlet对象是nib/sb scene的拥有者（File'sowner）所持有的对象，那么很显然拥有者必须“拥有”对象的指针，因此属性应设置为strong。而其他的IBOutlet对象的属性需要设置为weak，因为拥有者并不需要“拥有”他们的指针。举例来说，UIViewController的view属性是strong，因为controller要直接拥有view。而添加到view上的subviews，作为IBOutlet只需要设置为weak就可以了，因为他们不是controller直接拥有的。直接拥有subviews的是controller的view，ARC会帮助管理内存。

紧接着，[文档里](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/LoadingResources/CocoaNibs/CocoaNibs.html#//apple_ref/doc/uid/10000051i-CH4-SW8)又提到：

Outlets should be changed to strong when the outlet should be considered to own the referenced object:

·As indicated previously, this is often the case with File’s Owner—top level objects in a nib file are frequently considered to be owned by the File’s Owner.

·You may in some situations need an object from a nib file to exist outside of its original container. For example, you might have an outlet for a view that can be temporarily removed from its initial view hierarchy and must therefore be maintained independently.

第一种情形前面已经解释过了，对于第二种，通俗点将，就是controller需要直接控制某一个subview并且将subview添加到其他的view tree上去。单纯从ARC的角度思考，用weak也是很显然的：因为subview添加到view上时，view
会“拥有”subview。当然，给IBOutlet属性设置为strong也没有错，“纠结谁对谁错“的问题可能需要上升到模式或者编码习惯的问题，已经超出本文的范围。

著作权归作者所有。

商业转载请联系作者获得授权，非商业转载请注明出处。

作者：gao6708

链接：https://www.zhihu.com/question/29927614/answer/46262733

来源：知乎

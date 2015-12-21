以给UIImageView的xib增加view.layer.cornerRadius属性为例。
新建UIImageView子类CRImageView,代码如下：

```
//
//  CRImageView.h
//

#import <UIKit/UIKit.h>
IB_DESIGNABLE
@interface CRImageView : UIImageView
@property (nonatomic, assign)IBInspectable CGFloat radius;
@end

```

```
//
//  CRImageView.m
//
#import "CRImageView.h"
@implementation CRImageView
- (void)setRadius:(CGFloat)radius{
    self.layer.cornerRadius = radius;
}
- (CGFloat)radius{
    return self.radius;
}
@end
```

然后在xib中拖入一个imageView,在特性检查器（Show the Identity Inspector）中的Custom Class中选择刚才写的CRImageView，再打开属性检查器，就会发现最上方多出来一栏Radius属性了。

# UIView 转为 UIImage 

## 

```objective-c
#import "UIView+Utils.h"

@implementation UIView (Utils)

- (UIImage *)viewToImage {
    UIGraphicsBeginImageContextWithOptions(self.bounds.size, NO, 0);
    CGContextRef context = UIGraphicsGetCurrentContext();
    [self.layer renderInContext:context];
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    
    return image;
}

@end
```

# 图片保存到本地相册

```objective-c
- (void)_saveCurrentImageToAlbum {
    UIImage *image = [self.view viewToImage];
    UIImageWriteToSavedPhotosAlbum(image, self, @selector(image:didFinishSavingWithError:contextInfo:), nil);
}

// 保存图片到相册的回调函数
- (void)image:(UIImage *)image didFinishSavingWithError:(NSError *)error contextInfo:(void *)contextInfo {
    if (error) {
        NSLog(@"保存失败");
    } else {
        NSLog(@"保存成功");
    }
}
```


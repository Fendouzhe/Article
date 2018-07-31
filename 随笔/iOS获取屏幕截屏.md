```
//获取屏幕截屏  
- (UIImage*)getScreenShotsOfView:(UIView *)view{  
    [view.superview layoutIfNeeded];
    CGSize size = view.frame.size;  
    UIGraphicsBeginImageContextWithOptions(size, NO, [UIScreen mainScreen].scale);  
    CGContextRef context = UIGraphicsGetCurrentContext();  
    [view.layer renderInContext:context];  
    UIImage *img = UIGraphicsGetImageFromCurrentImageContext();  
    UIGraphicsEndImageContext();  
    return img;  
}
```

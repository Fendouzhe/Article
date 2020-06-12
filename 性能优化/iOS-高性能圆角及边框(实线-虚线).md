```
/**
 设置任意圆角及边框(实线/虚线)
 
 @param corners 要设置的圆角位置集合
 @param radius 圆角半径
 @param lineWidth 边框宽度
 @param lineColor 边框颜色
 @param lineDashPattern 虚线集合
 */
- (void)rounderWithCorners:(UIRectCorner)corners radius:(CGFloat)radius lineWidth:(CGFloat)lineWidth lineColor:(UIColor *_Nullable )lineColor dash:(NSArray<NSNumber *>*_Nullable )lineDashPattern{
    
    ///解决masonry布局获取不了正确的frame
    [self.superview layoutIfNeeded];
    
    //绘制圆/圆弧---适用于全圆
    //UIBezierPath *maskPath = [UIBezierPath bezierPathWithArcCenter:self.bounds radius:iconWidth*0.5 startAngle:0 endAngle:2*M_PI clockwise:YES];
    //矩形---圆角
    //UIBezierPath *maskPath = [UIBezierPath bezierPathWithRoundedRect:self.bounds cornerRadius:self.iconImage.width*0.5];
    //宽高相等为全圆，不等为椭圆
    //UIBezierPath *maskPath = [UIBezierPath bezierPathWithOvalInRect:self.bounds];
    
    UIBezierPath *maskPath = [UIBezierPath bezierPathWithRoundedRect:self.bounds byRoundingCorners:corners cornerRadii:CGSizeMake(radius,radius)];
    CAShapeLayer *maskLayer = [CAShapeLayer layer];
    maskLayer.frame = self.bounds;
    maskLayer.path = maskPath.CGPath;
    
    CAShapeLayer *borderLayer = [CAShapeLayer layer];
    borderLayer.lineWidth = lineWidth;
    borderLayer.strokeColor = lineColor.CGColor;
    borderLayer.fillColor = [UIColor clearColor].CGColor;
    if (lineDashPattern) {
        borderLayer.lineDashPattern = lineDashPattern;
        //borderLayer.lineCap = @"butt";
    }
    borderLayer.path = maskPath.CGPath;
    [self.layer insertSublayer:borderLayer atIndex:0];
    self.layer.mask = maskLayer;
}


/**
 设置任意圆角 --- 性能优化
 
 @param corners 要设置的圆角位置集合
 @param radius 圆角半径
 */
- (void)rounderWithCorners:(UIRectCorner)corners radius:(CGFloat)radius{
    
    [self.superview layoutIfNeeded];
    UIBezierPath *maskPath = [UIBezierPath bezierPathWithRoundedRect:self.bounds byRoundingCorners:corners cornerRadii:CGSizeMake(radius,radius)];
    CAShapeLayer *maskLayer = [CAShapeLayer layer];
    //maskLayer.frame = self.bounds;
    maskLayer.path = maskPath.CGPath;
    self.layer.mask = maskLayer;
    
}

/**
 设置圆角 --- 性能优化
 
 @param radius 圆角半径
 */
- (void)roundCornerWithRadius:(CGFloat)radius{
    
    [self rounderWithCorners:UIRectCornerAllCorners radius:radius];
    
}
```

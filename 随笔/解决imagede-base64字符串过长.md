```
    //1. 这里需要进行压缩，否则生成的base64字符串过长导致内存奔溃
    NSData *imageData = UIImageJPEGRepresentation(image, 0.68f);
    //2. Options写0不要写成 NSDataBase64Encoding64CharacterLineLength
    NSString *base64String = [imageData base64EncodedStringWithOptions:0];
```

```
    //图片base64码还原成图片
    NSData *decodedImageData  = [[NSData alloc] initWithBase64Encoding: base64String];
    UIImage *decodedImage  = [UIImage imageWithData:decodedImageData]
```

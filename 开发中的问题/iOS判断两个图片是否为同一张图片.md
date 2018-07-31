方法1：将图片转成data再进行md5后进行比较
```
#import <CommonCrypto/CommonDigest.h>
然后再加上下面这四句话
/*
unsigned char result[16];
NSData *imageData = [NSData dataWithData:UIImagePNGRepresentation(image)];
CC_MD5((__bridge const void *)(imageData), (uint32_t)[imageData length], result);

NSString *imageHash = [NSString stringWithFormat:@"%02X%02X%02X%02X%02X%02X%02X%02X%02X%02X%02X%02X%02X%02X%02X%02X",
                       result[0], result[1], result[2], result[3],
                       result[4], result[5], result[6], result[7],
                       result[8], result[9], result[10], result[11],
                       result[12], result[13], result[14], result[15]
                       ];
*/
    NSData *data = UIImageJPEGRepresentation(image, 1.0f);
    CC_MD5_CTX md5;
    CC_MD5_Init(&md5);
    CC_MD5_Update(&md5, [data bytes], (CC_LONG)data.length);
    unsigned char digest[CC_MD5_DIGEST_LENGTH];
    CC_MD5_Final(digest, &md5);
    NSString * s = [NSString stringWithFormat: @"%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x",
                    digest[0], digest[1],
                    digest[2], digest[3],
                    digest[4], digest[5],
                    digest[6], digest[7],
                    digest[8], digest[9],
                    digest[10], digest[11],
                    digest[12], digest[13],
                    digest[14], digest[15]];
    NSLog(@"md5 -- s = %@",s);
```
方法2：将图片转成data直接进行比较如下：
```
    NSData *data1 = UIImagePNGRepresentation(image1);
    NSData *data = UIImagePNGRepresentation(image);
    if ([data isEqual:data1]) {
        NSLog(@"is equal");
    }
```

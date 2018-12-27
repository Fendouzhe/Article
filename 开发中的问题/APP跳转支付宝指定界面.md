```
    NSString *urlString = @"";
    ///跳转支付宝指定界面前面需要拼接 alipays://platformapi/startapp?appId=20000067&url=%@
    NSString *alipayUrl = [NSString stringWithFormat:@"alipays://platformapi/startapp?appId=20000067&url=%@", urlString.URLEncodedString];
    SMLog(@"alipayUrl = %@",alipayUrl);
    ///是否可以打开支付宝,打不开说明未安装
    if ([[UIApplication sharedApplication] canOpenURL:[NSURL URLWithString:@"alipays://"]]) {
        [[UIApplication sharedApplication] openURL:[NSURL URLWithString:alipayUrl] options:@{} completionHandler:nil];
    } else {
        UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"提示" message:@"检测到您尚未安装支付宝，是否下载并安装支付宝完成认证?" preferredStyle:UIAlertControllerStyleAlert];
        [alert addAction:[UIAlertAction actionWithTitle:@"确定" style:0 handler:^(UIAlertAction * _Nonnull action) {
            NSString *appstoreUrl = @"itms-apps://itunes.apple.com/app/id333206289";
            [[UIApplication sharedApplication] openURL:[NSURL URLWithString:appstoreUrl] options:@{} completionHandler:nil];
        }]];
        [alert addAction:[UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleCancel handler:nil]];
        [weakSelf presentViewController:alert animated:YES completion:nil];
    }
```
```
/**
 *  URLEncode
 */
- (NSString *)URLEncodedString
{
    
    NSString *unencodedString = self;
    NSString *encodedString = (NSString *)
    
    CFBridgingRelease(CFURLCreateStringByAddingPercentEscapes(kCFAllocatorDefault,
                                                              (CFStringRef)unencodedString,
                                                              NULL,
                                                              (CFStringRef)@"!*'();:@&=+$,/?%#[]",
                                                              kCFStringEncodingUTF8));
    /*
     CFBridgingRelease(CFURLCreateStringByAddingPercentEscapes(kCFAllocatorDefault,
                                                              (CFStringRef)unencodedString,
                                                              NULL,
                                                              (CFStringRef)@"?!@#$^&%*+,:;='\"`<>()[]{}/\\| ",
                                                              kCFStringEncodingUTF8));
     */
    return encodedString;
}
```
